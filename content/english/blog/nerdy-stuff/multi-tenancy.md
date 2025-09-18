---
title: "Multi Tenancy"
meta_title: "Multi Tenancy - Architecture Design Patterns and Implementation"
description: "Design considerations are abstract ideas. Learn about multi-tenancy architecture, database design options, and practical implementation strategies."
date: 2021-12-29T00:00:00Z
image: "/images/nerdy-stuff/multi-tenancy/multitenant-architecture.png"
categories: ["Technology", "Architecture"]
author: "Deep Kulshreshtha"
tags: ["architecture", "design", "multitenant", "database", "scalability"]
draft: false
toc: true
---

## Prologue

Design considerations are abstract ideas.

What the hell does *'polymorphic'* mean anyway? Add words as *stateless*, *managed* and we have a recipe for unknown.

"Really" understanding these concepts is difficult, without working at the code level. At least for me.

One such word is multitenancy. Though I had heard of this word many times, I had never really understood the concept.

To delve deeper, let's explore its definition and importance.

### What Is It? and Why Is Multitenancy Important?

Let's say we have telecom-based software. Now, consider one server caters Vodafone, Idea, Airtel, Jio, TMobile, Verizon, and others.

*One instance of an application catering to multiple clients is called multitenant architecture*. This would be as opposed to a different deployment for each client. *Also, the word 'client' intends to highlight a different client, as compared to another user.*

Think of almost all game-changing software around us.

- This is the reason - The same SAP server caters to 100 people from 2 different companies.
- 1 Salesforce cluster caters to 100s of organizations. It uses this concept.

Fancy enough?

With this foundation, let's examine the challenges I faced.

## The Obstacle

I had worked on single-tenant applications. They are built for a single consumer. An example would be a banking application, where all users will be from the same department.

I had also worked on polymorphic applications. Ones that have separate deployment for each client. And change behavior with each client. The changes are based on configurations or customizations.

But I hadn't worked on multi-tenant applications. (Truth be told, I was wrong to think I even understood the concept.)

- How should one design the DB?
- What should be the config considerations? Should they be per client or per functionality?

This is where my struggle started.

Building on these questions, here's how the challenge unfolded.

## The Struggle

Luckily enough, I got an invite from a European organization for an interview. The second round required implementing a high-performance, multitenant system (unluckily enough). The system was to cater as an aggregation platform for telecom operators.

Aggregation, performant, multitenant... Didn't I say that it gets confusing?

What to do, where to start, how to do it? This is where my struggle started.

To overcome this, I turned to research for guidance.

## The Way

Our mutual best friend Google helped me out. It told me many things about multitenancy.

First, let's look at database design options.

### DB Design

A multitenant application can have DB designed in the following ways:

- **A different DB for each tenant**

  While this provides data isolation, adding a tenant (and a new DB) becomes more complex.

- **Single DB, but a different schema for each tenant**

  Data is logically divided, but physically on the same disk. Since one CPU serves all schemas, resource allocation becomes a bottleneck, depending on the DB server's resources.

  And again, adding a tenant is complex.

- **Single DB, and a single schema for all tenants**

  All tables contain data for all tenants (separated by a tenant Id column).

  This is a fast approach but can result in scalability bottlenecks. For example, many requests for one tenant can 'starve' the requests of CPU time for another tenant.

Next, consider how access is managed.

### Access Design

Access can be designed in the following ways:

1. **URL based**

   For example - **infosys**.atlassian.net, **tcs**.atlassian.net, and **wipro**.atlassian.net. The load balancer can redirect traffic to the appropriate server/endpoint. (Though, such a design wouldn't be 100% multitenant.)

2. **Information based**

   Providing additional information to access the right resources.

   An *access token, header, or a param*. This can later be used to redirect the traffic to the right resources.

Finally, the application itself needs careful design.

### App Design

Since the application ought to be multitenant, the app should have a common code. Meaning, the application should not have tenant-specific features.

What the hell does that mean?

If Atlassian were to bring in a new feature - let us say "Exalate". Then the new feature should be a configuration that could be set or not set for a particular client. E.g.

*Infosys - ON*

*TCS - OFF*

*Wipro - ON*

*Satyam - OFF*

The feature exists within the application. But only the clients that have paid for it see the feature. The same code works differently for the two clients.

PS: More complexities can be brought in by the use of virtualized app environments per tenant. But they are beyond the scope of most of our discussion.

With these insights, I applied what I learned.

## The Result

Guru Google taught me well. And I created my first multi-tenant application.

A side effect was, I understood that...

Problems seem bigger from a distance.

The company went through restructuring and only kept the core tech team. Others were let go. The HR from the company replied saying that she needed a job.

Regardless, I was happy about having learned something new. More so about concepts I was so close to and did not know well (despite believing otherwise).

While I am sure the happiness will be short-lived. That is, I will fail at something else very soon! But here I am enjoying the fleeting moment of happiness. Enjoying my journey from failure to a step of success!

A small failure + some effort = becoming better

Better self -> Lead to -> Better results
