---
title: "Ecommerce Org - Contributions"
meta_title: "Ecommerce Org - Contributions - Cache Redesign and Performance Optimization"
description: "I worked at a major e-commerce organization. Following are the major tasks I accomplished while there, including cache redesign and rewards reporting improvements."
date: 2023-08-20T00:00:00Z
image: "/images/nerdy-stuff/ecommerce-org-contributions/ecommerce-contributions.png"
categories: ["Technology", "Performance"]
author: "Deep Kulshreshtha"
tags: ["contributions", "projects", "tech", "ecommerce", "cache", "optimization"]
draft: false
toc: true
weight: 4
---

I worked at a major e-commerce organization. Following are the major tasks I accomplished while there.

## Cache Redesign

### Impact

- Direct savings in infra costs: ~30 lakhs / year
- Projected savings in infra costs: 1 cr in the next 2 years

{{< figure src="/images/nerdy-stuff/ecommerce-org-contributions/cache-redesign-diagram.png" alt="Cache Redesign Diagram" caption="Visual representation of the cache system architecture" >}}

### Infra

- Normal time site load: 30,000 requests/ second
- Sale time load: 60,000 requests/ second

- Normal Pod throughput: ~2000 requests/ second
- Sale time throughput: ~3000 requests/ second

- Pod RAM: 16 gb
- Pod CPU: 8 cores

- Redis: 3 master 3 slave clusters

### Context

Client is an ecommerce website. One of the existing services - Aggregator fetched data from ~7 different services.

a. Price service (fluctuated frequently)

b. Inventory service (fluctuated frequently but less than Price)

c. Promo service (What promotions are available with the product. Varied every sale season)

d. Size chart service (Mostly static data)

e. Cohort service or Classification service. (Static but varied based on business needs)

f. [Additional details]

Data was saved on Redis collectively for each product e.g.

Key = Price service key + Inventory service key + Promo service key + ....

Value = Price of product + Inventory of product + Promos on product ... etc.

This design was copied from a legacy system bought by the organization. (forgetting the name)

#### Cons of Design

- **TTL was set to the minimum of all the ~7 services.**

Prices fluctuated most frequently. So, "price" had the smallest TTL of ~15 seconds.

After 15 seconds, however, all other data along with price expired too. And had to be fetched again.

Cohort / Size chart data was also deleted. These had to be fetched again. This premature expiry created an unwanted load on Cohort and Size service.

Data was fetched repeatedly despite it *not* changing.

- **Key creation was slow**

Each key was heavy (a big string) and had to be created slowly.

It had to be created by 7 different adapters, working sequentially. This was because *not* all services used the same product property.

e.g. Size Chart will *not* always use a product code instead it could also use a standard size metric - Indian, UK, US etc.

- **Total Redis IO was high ~ 50 ms.**

Each key - value pair carried data from 7 different services. Therefore each pair was heavy.

Each redis IO therefore took ~50 ms - ~100 ms.

#### Pros of Design

- Only 1 call was made to redis for each read.

However, the same data was read sequentially from 6 upstream systems. And posted to redis every "n" seconds.

### Redesign

As part of the redesign, following changes were done:

{{< figure src="/images/nerdy-stuff/ecommerce-org-contributions/new-cache-architecture.png" alt="New Cache Architecture" caption="Improved cache design with separate keys and TTLs" >}}

- **Created a separate Redis key + TTL for each service.**

When data from one service fluctuated, then it did *not* impact data from the other services. e.g. Price fluctuations did *not* impact size chart data.

*We ended up not caching Price data. It was always fetched live.*

Some services had TTL set to 6 hours while others had TTL in days.

Size chart - 24 hours

Promo - 3 hours

Enabling Caching control - Given to the business team.

TTL time for each service - Given to the business team.

#### Con

1. The number of calls required to read data increased. From 1 to 7.

2. After the 7 calls, all data has to be merged to produce a single output for the frontend server.

3. This meant that the slowest response added to the overall response. (weakest link in the chain)

#### Pros

1. Traffic on static services decreased.

Only Price service was called frequently. For others, traffic dropped.

2. Upstream service failure impact reduced.

Data was served via cache. (In past ~2-5% requests failed due to upstream errors)

3. Reduced IO duration for each Redis request to ~5 ms

This was because the size of each read/write became smaller.

### Time Spent

1. Me and 1 other mid level engineer (7 years experience) worked on the redesign for ~3 weeks.

2. Changes included

   a. Code rewrite

   b. Prometheus integration for the new calls.

   c. Creating a dashboard on Grafana for the Prometheus stats

### Results

{{< figure src="/images/nerdy-stuff/ecommerce-org-contributions/performance-improvement-graph.png" alt="Performance Improvement Graph" caption="Significant performance improvements after cache redesign" >}}

1. Throughput increased from 1200 requests/ sec to ~2100 requests/ sec.

Company was moving away from 24 core machines to 8 core machines.

On the old machine performance was ~1200 requests/ second. So, the requirement on the new machine was ~1200 requests/ second.

But the new design served ~2100 requests/ second.

Meaning, despite the CPU being reduced 66%, the throughput increased ~80%.

2. CPU utilization fell from ~20% to ~4% during most times.

3. Response time remained around 50 - 100 ms/ for each request.

4. Service was tunable at each upstream level.

Business team could choose to tune TTL for each upstream system.

In addition to optimizing the cache system, I also addressed issues in the rewards reporting process to improve efficiency and reduce resolution times.

## Failed Rewards Reporting

### Impact

- Turn around reduction: 3 weeks to 3 mins

{{< figure src="/images/nerdy-stuff/ecommerce-org-contributions/rewards-reporting-process.png" alt="Rewards Reporting Process" caption="Streamlined rewards reporting workflow" >}}

### Context

When users bought from the website, they receive rewards e.g. 50OFFDOMINOS, 75OFFOVENSTORY etc.

Rewards are of 2 types...

1. Immediate

2. Delayed (*given after the order cancellation duration is over*)

#### Players

- Business team: configured sales times.

- Business team: estimated coupons required

- Promo team: prepared coupons in advance

- Promo engine: validated, calculated and awarded coupons. *This was our service*

- Email service: notified customers.

#### Problem

Players malfunctioned. e.g.

- Business team forgot to enable rewards at midnight.

So, the orders placed between midnight till now, would *not* get rewarded

- Business team miscalculated sale volumes.

So many orders would *not* get the coupons. This was because the Coupon engine would *not* have any more coupons left.

By the time the coupon engine would generate more coupons (from a third party service). Sale time would be over, and our service would be unable to disburse them.

- Bugs in Promo engine

Some customers would *not* get coupons.

- Bugs in Notification engine (mails or app notifications would *not* get delivered)

### Solution Chain

After *not* having received the coupons, following would typically happen.

- Customers would call Ajio customer care.

- Customer care would redirect to the business team.

- Business team would come to us (the Dev team)

- We did *not* have access to production env, so we contacted TechOps team.

- After a few days, some issues were figured out.

- To solve the error, some action would need to be taken.

- We would ask for data from the PII122 team. TechOps did not have access to sensitive data like phone numbers or emails.

- After getting exceptions etc, the PII team would give data.

- Dev team would create a script -> give it to TechOps.

- TechOps would trigger the script (from Dev team) + Data (from PII team)

All of the above typically took ~3 weeks. And was very tiring for the tech team.

### Redesign

I recommended, got approved and started on a new business process.

- Generate a Rewards report, for each order.

The report would be used by the Customer Care and Business team.

- They would be able to see a 360 degree view of each order + rewards.

- To do the above...

  - For each Reward

    - Given - Generate an event with ALL information on the reward given.

    - NOT Given - Generate an event will ALL information for the failure reason.

  - For each email / notification

    - Success - Generate an event

    - Fail - Generate an event.

  - At PII layer... read and consolidate all events.

  - Create a report in Tableu (used for reporting) which allows for users to see

    - User data

    - Order data

    - Reward data

    - Notification/ email data.

### Result

When an Ajio customer calls. Customer care would have a complete view of what happened.

80% of the requests could be handled at level 1 itself. They could redirect the solution without the intervention of the Tech team.

**The time of 3 weeks would reduce to 3 mins.**

I coded the highlighted part of the design. Also initiated the Tableau report creation. And then moved to my next organization.
