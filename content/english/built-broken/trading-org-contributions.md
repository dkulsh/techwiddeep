---
title: "Trading Org - Contributions"
meta_title: "Trading Org - Contributions - Crypto Exchange System Optimization"
description: "Worked at a Crypto exchange. Following are the major tasks accomplished while there, including payment system redesign and performance improvements."
date: 2023-08-23T00:00:00Z
image: "/images/nerdy-stuff/trading-org-contributions/trading-contributions.png"
categories: ["Technology", "Performance"]
author: "Deep Kulshreshtha"
tags: ["contributions", "project", "tech", "crypto", "optimization", "payments"]
draft: false
toc: true
weight: 3
---

Worked at a Crypto exchange. Following are the major tasks accomplished while there.

## Incoming Payments Redesign

### Impact

{{< figure src="/images/nerdy-stuff/trading-org-contributions/payment-system-impact.png" alt="Payment System Impact" caption="Significant improvements in payment processing efficiency" >}}

{{< notice "success" >}}
**Reduced the Ops team from 11 → 3.**

- **Direct savings:** ~₹30 lakh / year
- **Processing savings:** ~₹1 crore / year
- **Indirect savings:** ~₹50 lakh / year in dev effort
{{< /notice >}}

This redesign addressed key challenges in the incoming payments system, leading to significant improvements.

### Context

Crypto is an upcoming field and does not have sufficient support from the Government; therefore the banks.

So on-ramp integration works by ingesting statement files from various banks.

### Problems

The previous design had been done with many design flaws. For example:

{{< figure src="/images/nerdy-stuff/trading-org-contributions/design-flaws-diagram.png" alt="Design Flaws Diagram" caption="Visual representation of the problematic design patterns" >}}

- **Lock was taken on prefix + user.**

So when two payments for the same user ran on different flows (UPI / bank deposit), the lock sometimes wasn't taken — double-crediting or double-debiting the user.

On one hand, this resulted in direct losses to the organization. On the other hand, it created a *heavy* operation cost.

- Extra credits had to be recovered manually via legal notices.
- Extra debits had to be refunded with penalties

Solved by changing the lock on a single mutex - userId.

- **Code bugs - locking/unlocking**

Among hundreds of locks, a few missed unlocking a user. This made all other requests for the user wait for the expiration time of 1 min, increasing the turnaround time.

Solved with a lambda that wrapped lock/unlock around any method passed into it.

Ensured guaranteed locking/unlocking, solving the issue across the codebase.

- **If-else based code**

The previous version of the code was written with if/else conditions for each bank integration. Over time the conditions multiplied until the code became unmanageable.

The code was modularized into classes. The classes had specific logic.

Each logic was called for a specific banking partner.

Modular code reduced the management cost.

- **Each flow had its own logic**

Logic was written once and triggered from multiple places.

- **Design flaws**

{{< figure src="/images/nerdy-stuff/trading-org-contributions/table-design-issues.png" alt="Table Design Issues" caption="Problems with horizontal table expansion approach" >}}

- Tables were not designed to allow logic expansion.

Each new change needed an ad hoc column. This created problems later.

For example, the same column updated twice lost its history. History was needed for business logic.

Changed logic to vertical expansion from horizontal table expansion.

For example, instead of adding a new column called vipStatus, design the table to have two columns called PropertyName and PropertyValue.

Now the table can store any number of properties uniquely without another property of the same user.

- Related ad hoc properties were stored in a different data structure (tables).

This meant:

  - Additional query joins, therefore higher DB costs, and slow queries.

These properties were redesigned to be stored in a JSON column in the same table. This reduced both storage costs + the need for additional joins.

- Webhook callbacks were not persisted.

This created issues while debugging.

  - Logs were hard to parse, and for cost reasons only ~30% were sent to Datadog (similar to Kibana).
  - Logs persisted only for 15 days due to cost reasons.
  - Parsing through logs was time-consuming at peak times of production issues.

Solved by creating webhook callback tables. The tables were purged after 30 days to not take up lots of space.

All logs were available with success/failure messages.

These fixes streamlined the system, paving the way for efficient operations.

### Time Spent

- Me and two other engineers (6 and 3 years' experience) worked for **~1 month**.
- Change included:
  - Code rewrite
  - DB redesign
  - New modules - purge, flows, etc.

### Result

- **Losses reduced from ~₹75 lakh to ~₹0** in the previous quarter. Any residual losses were due to Ops-team errors.
- **New integration time: 14 days → ~3 days.**
- **Ops team: 11 → 3**, saving further costs to the organization.

---

## Reliability & Cost Optimization

### Impact

{{< notice "success" >}}
**Reduced cloud costs by 30%** (still going down). Projected annual figures:

- **Direct savings:** ~₹30–60 lakh / year
- **Processing savings:** ~₹2 crore / year
- **Indirect savings:** ~₹1 crore / year in dev effort
{{< /notice >}}

These enhancements built on the previous redesign to further optimize the system's performance.

### Context

Organization was one of the highest $$ payers to AWS for cloud costs.

At the same time, the application faced issues during the months - Feb, March, and April 2023.

### Improvements

Following were done to improve the application:

- **DB Index creations.**

Indexes were either missing or not being used in the tables.

Logs were analyzed to find the slowest APIs. API queries were checked.

Each query was analyzed to find whether it used an appropriate index.

The following types of indexing were done.

- Hash Indexes - Simple equality comparisons.

Queries that referenced a direct = or in comparison. Hash indexes were created.

| Percentile | Before | After | Speedup |
| --- | --- | --- | --- |
| Max | 1799 ms | 2.5 ms | **~720× faster** |
| P99 | 1747 ms | 1.5 ms | **~1,165× faster** |
| P95 | 628 ms | 1 ms | **~630× faster** |
| P90 | 609 ms | 0.933 ms | **~650× faster** |

- GIN Index (Generalized Inverted Index)

GIN index helps with full text search.

Certain fields needed full and partial search (some banks gave partial IDs).

| Percentile | Before | After | Speedup |
| --- | --- | --- | --- |
| Max | 5.3 s | 471 ms | **~11× faster** |
| P99 | 4.94 s | 193 ms | **~26× faster** |
| P95 | 4.94 s | 97.7 ms | **~51× faster** |
| P90 | 4.94 s | 53.4 ms | **~93× faster** |

- B Tree Indexes

  - Composite

The columns in the Index were verified to be in the same sequence as that in the query. This is important to ensure the index performs optimally.

  - Expression

JSON columns with particular key searches are created as Expression indexes.

| Percentile | Before | After | Speedup |
| --- | --- | --- | --- |
| Max | 8.91 s | 3.72 ms | **~2,395× faster** |
| P99 | 8.76 s | 3.38 ms | **~2,592× faster** |
| P95 | 8.36 s | 2.85 ms | **~2,933× faster** |
| P90 | 8.11 s | 1.96 ms | **~4,138× faster** |

- **N + 1 Query problem**

A parent query fetches many child objects. Each child object fetches details via a query.

Such queries were batched to fetch details in one go instead of multiple ones.

- **Reads in a loop - SQL / Redis / Dynamo DB**

Such queries were batched to fetch details in one go instead of multiple ones.

- **Poorly designed code**

  - High complexity Redis commands like - HGETALL (gets all keys with a prefix) were used.

These were fixed to use the right data structures.

- **High response time from internal / external services**

  - Cache responses where possible, e.g., Master data - currencies etc.
  - Optimized APIs

- **Dynamodb - insufficient read/write capacity**

  - Right size was provisioned

- **Archival of data**

  - Created a Util file - taking DB name, table name, where condition, batch size, deleted by.
  - Trigger to the util did the following:
    - Read data in batches
    - Write it in a file
    - Encrypted file using a random key
    - Calculated Hash of the file
    - Create an entry into the table with details of the DB, table, where condition, batch size, deleted by etc.
    - Move the file to Amazon S3
    - Delete the data.

- **Create Partitions in the table.**

These were date range based partitions. Such tables were used in the range of last 12 months.

Data before that wasn't needed.

These improvements collectively enhanced the system's efficiency and reduced operational costs.

### Result

**Reduced cloud utilization by 30%** (hence ~30% cost saving). Projected annual figures:

- **Direct savings:** ~₹30–60 lakh / year
- **Processing savings:** ~₹2 crore / year
- **Indirect savings:** ~₹1 crore / year in dev effort

---

## Checklist Creation

Team manager gave feedback that the same issues repeated in the team, e.g., missed deployment steps, partial testing deployments etc.

So I created this [checklist](https://docs.google.com/spreadsheets/d/15ANlO2peeZr6AUV8pu4-Kmetr8GrCDNR2j4OeHgJT6s/edit?usp=sharing). This **reduced ~95% of such issues**.

{{< figure src="/images/nerdy-stuff/trading-org-contributions/deployment-checklist.png" alt="Deployment Checklist" caption="Comprehensive checklist to reduce deployment issues" >}}
