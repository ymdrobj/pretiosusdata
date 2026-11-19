---
title: "Budget the whole AI investigation"
date: 2026-11-19
slug: "budget-the-whole-ai-investigation"
description: "Bound an AI data assistant's total query work with an application-enforced execution allowance and Athena workgroup controls."
tags: ["ai", "data-engineering", "athena", "agents", "cost"]
draft: false
---

An AI assistant investigating a late data load can run several useful queries before it explains anything. It checks source counts, compares partitions, then changes the date range to test another explanation. For the engineer waiting for an answer, that exploration saves manual work. It also gives one question the ability to launch an uncertain amount of database work.

Consider a hypothetical assistant using Amazon Athena to investigate missing transactions in an S3 data lake. It has read-only data access and submits SQL through an application tool. Each query looks reasonable. After several inconclusive results, it keeps exploring.

I would give that investigation an execution allowance enforced by the application. A prompt asking the model to be economical helps express intent, but the tool must decide whether another query is allowed.

Athena provides part of the mechanism. A workgroup's [per-query data usage control](https://docs.aws.amazon.com/athena/latest/ug/workgroups-setting-control-limits-cloudwatch.html) cancels a query that exceeds its scan limit. The aggregate workgroup thresholds behave differently: they can trigger notifications, but do not automatically cancel queries when reached. Canceled queries can still incur charges.

The assistant can stay below the scan limit on every query and still run up a large bill. An alert may tell an engineer that usage has grown while the assistant continues submitting work. Any automated response to that alert needs its own implementation.

For a first version, I would allow a small number of executions per investigation and run them sequentially. Six submissions could be a starting policy for testing, with failed and canceled executions consuming the allowance too. The number is a product choice, not an Athena recommendation. It needs adjustment against real diagnostic tasks.

The application should allocate each slot before submission and associate it with the investigation. A second tool call must not receive a fresh allowance simply because the model rephrased the question. Concurrent calls need an atomic allocation check; otherwise two requests can both claim the final slot. Keep that decision in application state.

I would also put this assistant in a dedicated workgroup. Athena supports [IAM permissions scoped to workgroups](https://docs.aws.amazon.com/athena/latest/ug/workgroups-iam-policy.html). Restrict the tool's execution role to the intended workgroup and keep permission to change its controls out of that role. The model should have no route to select a more permissive workgroup after a cancellation.

Record the outcome of every execution. Athena's [GetQueryExecution API](https://docs.aws.amazon.com/athena/latest/APIReference/API_GetQueryExecution.html) exposes status and statistics including bytes scanned. Those records let the team compare successful investigations with expensive dead ends. A submission count plus a scan threshold gives useful operational bounds, but I would not present it as an exact billing ceiling.

The tradeoff is that the assistant may stop one query short of a useful answer. Its response should preserve what it established, state what remains unknown, and describe the next query it wanted to run. An engineer can then decide whether to authorize more work.

For the missing-transactions investigation, test both an ordinary diagnosis and a deliberately inconclusive case. Check that the ordinary case reaches a useful answer within the allowance, and that the second case stops without launching extra queries. Review scanned bytes and the usefulness of the handoff. Increase the allowance when the additional work consistently helps the engineer finish the diagnosis; repeated variations of the same query call for a change to the investigation strategy.
