---
title: "Redshift trusts the keys you declare"
date: 2026-11-24
slug: "redshift-trusts-the-keys-you-declare"
description: "Make key validation a condition of publishing a Redshift load, because the optimizer trusts constraints the database does not enforce."
tags: ["redshift", "data-engineering", "data-quality", "sql"]
draft: false
---

Suppose a data engineer refreshes a customer dimension each morning so finance can report revenue by customer segment. In this hypothetical pipeline, the source export starts including historical customer versions. The load succeeds, but the dimension now contains two rows for some customer IDs.

The table declares `customer_id` as its primary key. That declaration will not reject those duplicates in Amazon Redshift. I would make validation of the completed candidate dataset a condition of publishing the refresh.

Redshift's [constraint documentation](https://docs.aws.amazon.com/redshift/latest/dg/t_Defining_constraints.html) says uniqueness, primary-key, and foreign-key constraints are informational. The query planner uses them and assumes they are valid. AWS explicitly warns that invalid keys can cause incorrect query results. These declarations carry an operational obligation for whoever maintains the data.

A duplicate customer row can multiply matching sales rows in an ordinary join and inflate totals. Declaring an invalid key adds another concern: the optimizer may plan queries using a uniqueness assumption the data no longer satisfies. Removing that declaration alone would still leave the duplicated dimension to repair.

Start with the intended grain. This dimension promises one current row per customer. A source that includes history needs an explicit rule for selecting the current version. Choosing an arbitrary row to make a duplicate check pass could assign revenue to the wrong segment.

For the full refresh, I would load an isolated staging table without informational key constraints, apply the current-version rule, then check the finished candidate. This illustrative SQL is untested:

```sql
SELECT customer_id, COUNT(*) AS row_count
FROM stage_customer_current
GROUP BY customer_id
HAVING COUNT(*) > 1
    OR customer_id IS NULL;
```

Any returned row should block publication. Preserve the rejected data and the validation result so the engineer can inspect which source versions collided. Keep the previous validated dimension available, with its refresh timestamp visible to report consumers.

The check must cover the rows that will actually be published. Freeze that candidate after validation, and prevent another writer from changing it before publication. A successful check against yesterday's staging table says nothing about today's replacement.

An incremental pipeline needs a different scope. A batch can be unique within itself and still conflict with existing target rows. It must preserve uniqueness across the resulting table, including overlapping loads and repair scripts.

Using `MERGE` does not remove that responsibility. Redshift's [MERGE documentation](https://docs.aws.amazon.com/redshift/latest/dg/r_MERGE.html) describes failures when multiple source rows match one target row. Its `REMOVE DUPLICATES` mode removes matching target duplicates, while leaving unmatched target duplicates unchanged. Neither behavior establishes that the entire dimension satisfies its declared key.

There is a cost to blocking a refresh: finance may have to work with older customer segments while the engineer investigates. For this pipeline, I would prefer that visible delay to publishing ambiguous customer mappings. The acceptable delay needs agreement with finance, and the failed validation needs an owner who can resolve it.

Before releasing the change, feed the pipeline two competing current versions of one customer and verify that publication stops. Then supply an unambiguous current version and confirm that the refresh succeeds and a known sales total remains correct after the join. Keep the primary-key declaration when every supported write path preserves that promise; until then, leave it undeclared and repair the enforcement gap.

