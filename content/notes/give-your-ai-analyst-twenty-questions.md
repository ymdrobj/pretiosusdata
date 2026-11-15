---
title: "Give your AI analyst twenty questions"
date: 2026-11-12
tags: ["ai", "analytics", "data-platform"]
draft: false
---

Suppose your account managers keep asking the data team for customer numbers before renewal meetings. You want to try an AI analyst: a chat interface that takes a question in plain English, generates SQL against approved data, and explains the result.

The appeal is straightforward. An account manager could ask which customers reduced their usage last month, follow up by region, and get an answer without another request in the data team's queue.

I'd start that pilot with about twenty questions from those requests. That's small enough to review properly and varied enough to expose some problems before people use the answers in customer conversations.

Take “Which active customers reduced their usage in October?” In this hypothetical business, finance treats a paid subscription as active, while account managers include customers still in a free onboarding period. Using the finance definition would leave some accounts out of the follow-up list.

The team needs to choose the definition for this task and make it available to the assistant. It also needs to specify how usage is compared: total activity can fall simply because the comparison periods have different numbers of days.

For each question, record those choices and have an analyst produce a reference answer against a fixed data snapshot. Include a few cases where the assistant should ask for clarification. “Show me customers at risk” is too open-ended unless the business has agreed on what counts as risk.

You can feed reviewed examples into existing products. Snowflake's [Verified Query Repository](https://docs.snowflake.com/en/user-guide/views-semantic/verified-query-repository) lets Cortex Analyst use question-and-SQL pairs when answering similar questions. Bedrock's [structured-data knowledge bases](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-structured-create.html) accept descriptions and curated query examples. AWS describes that context as best effort and advises checking the generated SQL.

Keep separate questions for evaluation. If every test question has already been supplied as an example, you've learned little about how the assistant handles an unexpected follow-up. Change the reporting period, ask about a customer group absent from the examples, and try a request the available data cannot answer.

Check the SQL alongside the numbers. A missing filter can go unnoticed when the test data happens to contain only the customers you wanted. Include cases that make those mistakes visible, and repeat important questions to check whether the behavior is consistent.

I'd limit the pilot to read-only access to the views needed for this task, with permissions enforced in the database. An analyst should review the answers before account managers use them externally.

That review effort belongs in the evaluation. If someone spends ten minutes checking every answer, account managers may be getting faster responses while the data team takes on more work. Compare the total effort with the existing request process. A dashboard may also cover most of the need if follow-up questions rarely vary.

Run the pilot through a week of renewal preparation. Record corrections and review time, then ask the account managers whether the answers helped them prepare. Continue if the results are dependable enough for the agreed use and the combined workload falls. Otherwise, use the failures to decide whether to narrow the questions or improve the data definitions before another trial.
