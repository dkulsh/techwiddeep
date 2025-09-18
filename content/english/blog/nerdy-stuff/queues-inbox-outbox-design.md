---
title: "Queues : Inbox Outbox design"
meta_title: "Queues : Inbox Outbox design - Message Queue Architecture Pattern"
description: "Software loses data when working with message queue (MQ) systems. Learn how the Inbox/Outbox pattern solves data loss and ensures reliable message processing."
date: 2021-10-07T00:00:00Z
image: "/images/nerdy-stuff/queues-inbox-outbox-design/inbox-outbox-diagram.png"
categories: ["Technology", "Architecture"]
author: "Deep Kulshreshtha"
tags: ["architecture", "design", "queues", "messaging", "reliability"]
draft: false
toc: true
---

## The Problem in Simple Words

Software loses data when working with message queue (MQ) systems.

High-volume processing requires data to be managed within memory. MQ-based systems like Kafka, RabbitMQ, and ActiveMQ are some examples.

While these systems manage millions of messages, they are transient in nature. This means that once a message is delivered, it's done! There is no redelivery, no success or failure feedback, and no querying the data back.

This requires the software to work with 100% reliability. However, anyone in the real world knows that simply isn't true.

In the real world, software breaks. Code runs into unhandled conditions and error scenarios.

To delve deeper, let's examine the issue from a technical perspective.

## The Problem in Nerdy Words

From: [https://event-driven.io/en/outbox_inbox_patterns_and_delivery_guarantees_explained/](https://event-driven.io/en/outbox_inbox_patterns_and_delivery_guarantees_explained/)

The problems of "at least once" and "exactly once" are *not* managed by MQ systems.

### At Least Once

A guarantee that a message will be delivered at least once, maybe more.

If we receive a message multiple times, then our system should be able to handle it (be *idempotent*).

### Idempotent

If I send a message once or multiple times, the system will remain in the same state.

In other words, sending the message multiple times will *not* mess up the system.

### Exactly Once

The guarantee that a message will be delivered once and only once.

Kafka provides this with the groupId and offset counts for each topic.

### At Most Once

The message will be delivered a maximum of once.

And it is possible for the message to *not* be delivered even once. Of course, we don't want this to happen.

We want a message to be delivered and processed only once.

Since a memory-only system is transient in nature, persisting message states and guaranteeing "exactly once" delivery is *not* possible within MQ systems.

Now that we've outlined the challenges, let's explore a practical solution.

## The Solution in Simple Words

Persist incoming and outgoing data into local storage using the Inbox Outbox pattern.

A mailing system works as follows:

1. Fetch incoming mails from a server to a local folder - Inbox
2. Send outgoing mails from a local folder to the server - Outbox

This design decouples the data exchange (message delivery) from data processing (message processing).

How does it help?

- In case of a read-time failure, we can re-read the data from Inbox.
- In case of a write-time failure, we can re-send the data from Outbox.

{{< figure src="/images/nerdy-stuff/queues-inbox-outbox-design/inbox-outbox-diagram.png" alt="Inbox Outbox Diagram" caption="Visual representation of the Inbox/Outbox pattern for reliable message processing" >}}

Building on this concept, let's look at the technical implementation.

## The Solution in Nerdy Words

Use the Inbox/Outbox design pattern.

### Inbox

1. Create a table with the relevant fields.
2. All incoming messages should come into this table.
3. A daemon reads data from the table and sends them for processing.

### Outbox

1. Create a table with the relevant fields.
2. All outgoing messages are first written into the table.
3. A daemon reads data from the table and sends them to MQ.

The tables can also be used for audit and reporting purposes.

{{< figure src="/images/nerdy-stuff/queues-inbox-outbox-design/inbox-outbox-technical-diagram.png" alt="Inbox Outbox Technical Diagram" caption="Technical implementation showing database tables and daemon processes" >}}

Problem solved!
