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

Helped reduce Ops team from 11 to 3

- Direct savings: ~30 lakhs / year
- Direct Processing savings: ~1 crore / year
- Indirect savings: ~50 lakhs / year of dev effort was saved

This redesign addressed key challenges in the incoming payments system, leading to significant improvements.

### Context

Crypto is an upcoming field and does not have sufficient support from the Government; therefore the banks.

Incoming payments (on-ramp) integration is therefore done by accepting statement files from various banks.

### Problems

The previous design had been done with many design flaws. For example:

{{< figure src="/images/nerdy-stuff/trading-org-contributions/design-flaws-diagram.png" alt="Design Flaws Diagram" caption="Visual representation of the problematic design patterns" >}}

- **Lock was taken on prefix + user.**

This meant that whenever two payments were processed for the same user (different flows - UPI / bank deposit), sometimes the lock wasn't taken.

This meant that the user was either double credited or double debited.

On one hand, this resulted in direct losses to the organization. On the other hand, it created a *heavy* operation cost.

- Extra credits had to be recovered manually via legal notices.
- Extra debits had to be refunded with penalties

Solved by changing the lock on a single mutex - userId.

- **Code bugs - locking/unlocking**

Among hundreds of locks, a few missed unlocking a user. This made all other requests for the user wait for the expiration time of 1 min, increasing the turnaround time.

Solved by creating a lambda function for locking + unlocking. A method could be passed as an argument in the method.

Ensured guaranteed locking/unlocking, solving the issue across the codebase.

- **If-else based code**

The previous version of the code was written with if/else conditions for each bank integration. Over time, the number of conditions became so many that managing the code became too much.

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

  - Logs were difficult to parse through. Were filtered down for cost reasons—only 30% of the logs were brought to Datadog (similar to Kibana).
  - Logs persisted only for 15 days due to cost reasons.
  - Parsing through logs was time-consuming at peak times of production issues.

Solved by creating webhook callback tables. The tables were purged after 30 days to not take up lots of space.

All logs were available with success/failure messages.

These fixes streamlined the system, paving the way for efficient operations.

### Time Spent

- Me and two other engineers (6 years and 3 years experience) respectively, worked for ~1 month.
- Change included:
  - Code rewrite
  - DB redesign
  - New modules - purge, flows, etc.

### Result

- Losses reduced to ~0 from ~75 lakhs, in the previous quarter. Any losses were due to Ops team errors.
- New integration time was reduced from 14 days to ~3 days.
- Ops team size reduction (11 to 3) saved further costs to the organization.

---

## Reliability / Cost Reduction / Performance Improvement

### Impact

Reduced cloud costs by 30% (still going down).

Below are the projected numbers.

- Direct savings: ~30-60 lakhs / year
- Direct Processing savings: ~2 crore / year
- Indirect savings: ~1 crore / year of dev effort was saved

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

| Performance | Before Index | After Index | Increase in Performance (old - new) / new * 100 |
| --- | --- | --- | --- |
| Max | 1799 ms | 2.5 ms | 71860% |
| P99 | 1747 ms | 1.5 ms | 116300% |
| P95 | 628 ms | 1 ms | 62700% |
| P90 | 609 ms | 0.933 ms | 65100% |

- GIN Index (Generalized Inverted Index)

GIN index helps with full text search.

Certain fields needed full and partial search (some banks gave partial IDs).

| Performance | Before Index | After Index | Increase in Performance (old - new) / new * 100 |
| --- | --- | --- | --- |
| Max | 5.3 sec | 471 ms | 10200% |
| P99 | 4.94 sec | 193 ms | 24600% |
| P95 | 4.94 sec | 97.7 ms | 49900% |
| P90 | 4.94 sec | 53.4 ms | 91500% |

- B Tree Indexes

  - Composite

The columns in the Index were verified to be in the same sequence as that in the query. This is important to ensure the index performs optimally.

  - Expression

JSON columns with particular key searches are created as Expression indexes.

| Performance | Before Index | After Index | Increase in Performance (old - new) / new * 100 |
| --- | --- | --- | --- |
| Max | 8.91 sec | 3.72 ms | 239400% |
| P99 | 8.76 sec | 3.38 ms | 259000% |
| P95 | 8.36 sec | 2.85 ms | 835700% |
| P90 | 8.11 sec | 1.96 ms | 413600% |

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

Reduced cloud utilization by 30% (hence 30% costs saved).

Below are projected numbers.

- Direct savings: ~30-60 lakhs / year
- Direct Processing savings: ~2 crore / year
- Indirect savings: ~1 crore / year of dev effort was saved

---

## Checklist Creation

Team manager gave feedback that the same issues repeated in the team, e.g., missed deployment steps, partial testing deployments etc.

So I created this [checklist](https://docs.google.com/spreadsheets/d/15ANlO2peeZr6AUV8pu4-Kmetr8GrCDNR2j4OeHgJT6s/edit?usp=sharing). This helped reduce 95% issues.

{{< figure src="/images/nerdy-stuff/trading-org-contributions/deployment-checklist.png" alt="Deployment Checklist" caption="Comprehensive checklist to reduce deployment issues" >}}
