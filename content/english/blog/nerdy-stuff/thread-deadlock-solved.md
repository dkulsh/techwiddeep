---
title: "Thread Deadlock: Solved"
meta_title: "Thread Deadlock: Solved - Database Deadlock Solution Explained"
description: "This blog is the solution to the earlier stated problem: Solve the Deadlock. Learn how to identify and fix database deadlock issues through real-world examples."
date: 2021-09-22T00:00:00Z
image: "/images/nerdy-stuff/thread-deadlock-series/deadlock-solution-screenshot.png"
categories: ["Technology", "Programming"]
author: "Deep Kulshreshtha"
tags: ["solved", "tech", "trivia", "deadlock", "solution", "python", "database"]
draft: false
weight: 2
toc: true
---

This blog is the solution to the earlier stated problem: [Solve the Deadlock](/blog/nerdy-stuff/tech-trivia-solve-the-deadlock/)

{{< figure src="/images/nerdy-stuff/thread-deadlock-series/tip-icon.png" alt="Tip" caption="Read the error message carefully. It is more helpful than we tend to think." >}}

The error tells us that the problem happened during the update of the "person" object.

With this insight, let's examine the relevant code.

## Identifying the Culprit Lines

A bit of digging reveals lines #28 and #37, which are:

```python
person_queryset.update(**kwargs)
```

and

```python
Person.objects.update(session=Subquery(mapping_queryset))
```

## Analyzing the Updates

Since these are the only lines that update the "person" object, we can be reasonably sure they are the cause of the problem.

If unfamiliar with Python or the framework, one might assume line #28 updates a subset of "person" objects, while line #37 updates another subset.

It might be possible for each database update to go via a different worker thread, with threads locking on different tables or rows, causing the deadlock.

This leads us to the root cause.

## The Root Cause

It turns out this was indeed the problem!

However, the method annotation caused some confusion. Methods marked *transactional* usually use a single database transaction for updates.

Yet, each library or framework might implement *transactional* behavior differently—a fact worth remembering.

{{< figure src="/images/nerdy-stuff/thread-deadlock-series/lesson-learned.png" alt="Lesson" caption="Each library and framework implements functionality differently." >}}

The method updated a fixed number of "persons" and did *not* need to update different sets.

## Implementing the Solution

Knowing this, line #37 was rewritten from:

```python
Person.objects.update(session=Subquery(mapping_queryset))
```

to:

```python
person_queryset.update(session=Subquery(mapping_queryset))
```

Voila! Problem solved.

Shoutout to the guy who figured this out. You rock!

## Navigation

**← Previous**: [Tech Trivia : Solve the Deadlock !](/blog/nerdy-stuff/tech-trivia-solve-the-deadlock/)
