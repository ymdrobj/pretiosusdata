---
title: "Keep Iceberg history long enough to find the mistake"
date: 2026-11-17
slug: "keep-iceberg-history-long-enough-to-find-the-mistake"
description: "Choose Iceberg snapshot retention around error detection and investigation time, then test what survives maintenance."
tags: ["iceberg", "data-engineering", "reliability", "data-lake"]
draft: false
---

Suppose a finance analyst discovers on Tuesday that last Thursday's revenue load assigned some transactions to the wrong region. The data engineer wants to compare the table before and after that load. In this hypothetical pipeline, daily maintenance has already expired snapshots older than three days. The useful comparison point is gone.

Apache Iceberg tracks versions of a data lake table as snapshots. Engineers can query retained snapshots to investigate changes without first restoring a separate copy. That makes retention part of the investigation workflow. I would choose its duration from how long errors take to surface and investigate, before tuning it for storage savings.

Iceberg's [maintenance documentation](https://iceberg.apache.org/docs/latest/maintenance/#expire-snapshots) explains that expiration removes snapshots from table metadata, making them unavailable for time travel. It can also delete files no remaining snapshot needs. A successful cleanup job can therefore leave Tuesday's engineer with a healthy current table and insufficient history to explain Thursday's error.

The policy needs to cover the delay before someone notices a problem, plus enough time to investigate and preserve the relevant state. For this revenue table, I would start by checking the reconciliation calendar. If a meaningful check runs weekly, three days of history cannot reliably support it. Daily row-count checks help, but a regional classification error may leave the total count unchanged.

Snapshot count needs separate attention. The Spark `expire_snapshots` procedure has an age cutoff, `older_than`, and a count floor, `retain_last`. The [procedure documentation](https://iceberg.apache.org/docs/latest/spark-procedures/#expire_snapshots) says the latter preserves a number of ancestor snapshots regardless of age. Keeping twenty snapshots gives no fixed duration: a busy table can consume that allowance much faster than a daily batch table. Review the actual maintenance invocation as well as table properties; an explicitly supplied cutoff matters.

There is a reasonable cost objection. Longer retention can keep replaced files around after updates or compaction. The cost depends on which files retained snapshots still reference, so counting snapshots alone gives a poor estimate of storage overhead. I would measure retained storage over a representative load cycle before choosing a shorter investigation window.

For selected checkpoints, Iceberg supports [tags with their own retention policies](https://iceberg.apache.org/docs/latest/branching/). A tag can preserve the snapshot used for an approved month-end report while routine intermediate history expires sooner. It retains a reference to table state; it does not create an independent backup. Check support in the engine and catalog you actually run before building the process around it.

A month-end tag would not necessarily rescue Thursday's investigation. Comparing immediately before and after the faulty load may require intermediate snapshots. I would keep a rolling investigation window and add longer-lived checkpoints only where a specific reporting obligation needs them.

Retained history also leaves a repair decision. Rolling the revenue table back to Thursday would remove later valid changes from its current view. Tuesday's engineer may instead use the historical comparison to identify affected transactions and apply a targeted correction.

Before shortening retention, rehearse this incident on a test table. Apply the bad load, add subsequent valid loads, and run maintenance with the proposed cutoff. Verify that the engineer can still inspect the required snapshots and derive the correction. Measure the storage retained alongside that result. If the history disappears before the scheduled reconciliation would find the error, extend the window or move that check earlier.
