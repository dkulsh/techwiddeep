---
title: "DB Design Inefficiency: Enum"
meta_title: "DB Design Inefficiency: Enum - Database Design Patterns"
description: "We discuss a DB design inefficiency today. Picked up from a real-world use case, I hope this helps clarify both how to and how not to design enumeration tables."
date: 2021-12-10T00:00:00Z
image: "/images/nerdy-stuff/db-design-inefficiency-enum/design-e1639132250916.jpg"
categories: ["Technology", "Database"]
author: "Deep Kulshreshtha"
tags: ["database", "design", "inefficiency", "enum", "optimization"]
draft: false
toc: true
---

We discuss a DB design inefficiency today. Picked up from a real-world use case, I hope this helps clarify both how to and how not to design enumeration tables.

## Introduction

Enums are objects of a class. Statically defined, they help reduce the time to create and use objects (among others).

Enums from the programming world are sometimes mapped into the DB also. As an example, we might have tables called Cities, Countries, or Types. These tables will have specific values, e.g.

Cities enum:

| Name     | Code |
|----------|------|
| New Delhi| NDLS |
| New York | NY   |

Similarly, other enumerations have their specific names and codes.

To better understand the challenges, let's explore how enums can be stored in databases.

## Lights, Camera...

Enums can be kept in the DB in multiple ways.

One can be to keep all city names in a table. Other tables or programs can read from this table and keep the name.

But this method has a drawback. The name of the city needs to be duplicated in every single row. Meaning, the string "New York" is stored across many tables/rows. Storage across multiple rows consumes huge redundant space on the DB server.

From [Stackoverflow](https://stackoverflow.com/questions/16525003/is-varcharn-saving-memory-space-in-postgresql):

*The storage requirement for a short string (up to 126 bytes) is 1 byte plus the actual string, which includes the space padding in the case of character. Longer strings have 4 bytes overhead instead of 1.*

This means, for an average of 10,000 rows we end up giving away 1 MB of space (~100 bytes * 10000). And we haven't considered...

- Replications in the same cluster (or across clusters/availability zones)
- Indexes (they store the value separately)
- Bigger tables
- Longer strings, or
- Inefficient DB engines

PS: Memory is the only consideration till now. But it will not remain so. Let's see.

Building on these storage concerns, let's examine ways to mitigate them.

## Action

One way of mitigating this is using a representation of the City. This can be done in various ways:

a. Use a unique identifier (GUID, UUID), etc. representing each enum value.  
b. Use a unique code for each enum.  
c. Use an integer representing each enum.

### GUID/UUIDs

| Name     | Code                                 |
|----------|--------------------------------------|
| New Delhi| 4eecc9f5-7535-4ff2-ae93-ee6d0a9a281c |
| New York | e728695b-b9d7-4724-8942-6dc472fcecb6 |

The string "New York" takes ~100 bytes, and a UUID takes 16 bytes. Therefore, GUID, UUID identifiers representing strings reduce memory usage. Great!

But wait a sec... did we introduce new problems? 😕 😕

- When we have to map a data table to an enum table, a comparison will be needed.

[Yves Trudeau explains](https://www.percona.com/blog/2019/11/22/uuids-are-popular-but-bad-for-performance-lets-discuss/): *UUID comparisons are about 28 times slower. Even if the difference appears rapidly in the char values, it is still about 2.5 times slower.*

We did save space but introduced a performance issue. 😹 😹

- The second problem is less quantifiable.

For developers, 'NY' is easier to understand compared to understanding 'b9def35e-2d79-4c2f-b0a8-aae6e8b000ef'.

GUIDs or UUIDs are random numbers and therefore cannot be interpreted easily. Even long-working developers need an additional query to understand their mapping.

But, this is an invisible impact. How do we measure it?

Assuming this additional translation takes 1 minute each time (I am being conservative):

- Writing the additional join condition.
- Coding for the new table conditions.

An approximation can be: 15 mins of additional effort understanding data, per developer. For a 100-member team. This becomes 100 * 15 = 25 hours!

Meaning, 1 day of productivity is lost among a 100-member team, each day!

Insignificant? I think not!

PS: Obscurity by UUIDs might be a good thing, in case the use case needs it.

Next, we move on to codes.

### Codes

What happens when we refer to our enumerations using codes?

| Name     | Code |
|----------|------|
| New Delhi| ND   |
| New York | NY   |

While the string "New York" took ~100 bytes, "NY" takes less (let's say ~20 bytes). So, a smaller memory footprint is the good news.

The second benefit is processing time related. Comparing 2-alphabet codes is faster than comparing 36-alphabet ones (UUID). (See Trudeau's article for details.)

Yet another benefit is that the code is human-understandable. We discussed the cost of human readability at 25 hours for a 100-member team. In this case, we might have reduced the cost by 80%. Saving 22 hours each day for an organization is a significant achievement.

Satisfied? Or do we still have a *need for speed*? (Yes, the reference is to the popular video game!)

Let's check out the final option.

### Integers

What if we represent each city/country with an integer?

| Name     | Code |
|----------|------|
| New Delhi| 1    |
| New York | 2    |

Since an integer takes up 4 bytes of space, we have further improved memory usage. At the same time, we know that integers process faster. Cherry on the top!

But we've done so at a cost... can you guess what?

We gave away the 'human readability' factor, adding ~25 hours for a 100-member team. Not an insignificant number.

Now that we've explored the options, let's draw some conclusions.

## Moral of the Story

My personal favorite among the choices is the use of codes. While it does give away memory and CPU clocks, it provides human readability.

If not codes, I would go with integers. Using identifiers like GUID/UUID would be my last preference and would be based on the use case.

But do *not* take my word. Please think for yourself.

Each organization needs to weigh between memory and human legibility costs. As examples:

- Is the data set of the order of a few thousand records?

Modern-day databases are designed to handle much bigger loads. In this case, the redundancy is less. Therefore, human readability can be given greater weightage. Meaning, we could go with NY-type codes.

- Is the data set of the order of hundreds of millions?

~20 bytes taken up by the code NY copied millions of times is a bigger inefficiency. Using integers will be a better approach in this case.

- Maybe your developers are mature and therefore the legibility costs are minimal.

So, *to each his own*.

With this, we wrap up a DB design inefficiency (for enumerations). More coming up!
