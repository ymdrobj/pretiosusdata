---
title: "Give your data agent a request ID"
date: 2026-11-15
tags: ["ai", "aws", "redshift", "data-platform"]
draft: false
---

Suppose a data engineer uses an AI agent to investigate a late warehouse load. The agent reads pipeline status, runs diagnostic SQL, and can request an approved repair through a restricted tool. In this hypothetical setup, the engineer authorizes an insert of missing records into a staging table. The tool submits it, but the response never reaches the agent.

Now the agent has an error message and a plausible next step: try again. If the first insert committed, a second submission could duplicate the records.

I would settle this recovery behavior before giving the agent write access. The tool needs to remember each authorized operation across retries, including retries that happen after the agent process restarts.

Amazon Redshift provides useful building blocks. Its [ExecuteStatement API](https://docs.aws.amazon.com/redshift-data/latest/APIReference/API_ExecuteStatement.html) accepts a `ClientToken` for idempotency and returns a statement ID. The [Data API guide](https://docs.aws.amazon.com/redshift/latest/mgmt/data-api.html#data-api-idempotency) documents an eight-hour token lifetime. AWS SDKs generate tokens for their own retries. That protection should not be assumed to cover a fresh tool invocation after a restart: the application must preserve the operation's identity.

For this repair tool, I would create a durable operation record before submission. It would bind a server-generated request ID to the approved SQL, parameters, target database, authorization context, and client token. Repeating the same operation would reuse that record. Changed SQL or parameters would require a new operation and approval.

This is application design advice. The API supplies idempotency; it does not decide whether two agent requests represent the same authorized repair.

Once submission returns, save the statement ID. Subsequent calls can use [DescribeStatement](https://docs.aws.amazon.com/redshift-data/latest/APIReference/API_DescribeStatement.html) to inspect that execution. Its statuses distinguish work still in progress from `FINISHED`, `FAILED`, and `ABORTED`. A tool response should preserve those distinctions so the agent knows whether to wait, inspect a failure, or retrieve a result.

The awkward case is a crash between submission and saving the statement ID. Within the token lifetime, the recovery path can repeat the unchanged submission with the persisted token. After that window, I would block automatic resubmission until an operator reconciles the original execution. Generating a new token would permit another execution while the first outcome remains uncertain.

There is a cost here: durable state, concurrency control, and a recovery path someone has to maintain. For a read-only diagnostic assistant, duplicate queries may be tolerable, though they still consume capacity. A tool that changes warehouse data needs a stricter decision. I would start with a small set of parameterized repairs and require explicit approval for each operation.

Idempotent submission also leaves the SQL's correctness unresolved. The repair must still handle overlapping runs and records arriving while the investigation is underway. The request record prevents one class of duplicate submission; the data model and transaction design must handle the others.

Before enabling this tool, test the opening failure deliberately: let Redshift accept the request, drop the response, then restart the agent. Check that recovery follows the original operation and that the staging table contains the expected records exactly once. Repeat with an expired token and verify that the tool stops for reconciliation. Those results would give me a concrete basis for allowing the first supervised repair.
