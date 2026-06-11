---
title: "S3 durability is not disaster recovery"
date: 2026-06-11
tags: ["aws", "s3", "disaster-recovery", "data-platform"]
draft: false
---

A client's CEO asked for a DR assessment of the data platform. The storage layer looked like a freebie. It's S3. Eleven nines of durability. Next.

That reflex was the actual finding.

Eleven nines answers exactly one question: will AWS lose your bytes? No. Store ten million objects, lose one every ten thousand years. It's an engineering marvel, and it's beside the point.

Because nobody loses data to AWS. You lose data to yourself.

A pipeline overwrites good partitions with garbage — the write succeeds, and the garbage is now eleven-nines durable. Someone deletes a prefix that "looked stale." A leaked key runs `DeleteObject` all night. The durability SLA sleeps through all of it. It insures the one failure nobody was worried about.

DR is the opposite question: when you break it — and it will be you — how fast do you recover, and to what point in time?

So: versioning, because deletes should be undoable. Replication to a *separate account* — same-account replication just copies your compromise at high fidelity. A backup vault with a lock, so an admin key can't torch it. And a restore you've actually run. An untested restore isn't a restore. It's a prayer.

The storage-layer assessment didn't ask "is the data safe?" It is. It asked for an RTO and an RPO per failure mode. That's when there was something real to write.

Durability is AWS's promise about their mistakes. DR is your answer for yours.

## Notes

The same category error is everywhere: RAID isn't backup. Replication isn't backup. Multi-AZ isn't DR. Infrastructure reliability is not organizational recovery.

## References

- [Amazon S3 data durability](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html)
- [AWS Backup Vault Lock](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html)
- [Replicating objects within and across Regions](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
