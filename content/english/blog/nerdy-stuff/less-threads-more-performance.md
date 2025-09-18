---
title: "Reduce Threads to Increase Performance"
meta_title: "Reduce Threads to Increase Performance - Thread Pool Optimization"
description: "How a reduction in the number of threads led to a performance improvement at Fudr. Learn about thread pool optimization and resource management."
date: 2022-06-05T00:00:00Z
image: "/images/nerdy-stuff/less-threads-more-performance/thread-pool-optimization.png"
image_max_width: 400  # Set any pixel value you want
categories: ["Technology", "Performance"]
author: "Deep Kulshreshtha"
tags: ["code", "design", "java", "threads", "performance", "optimization"]
draft: false
toc: true
---

How a reduction in the number of threads led to a performance improvement at [Fudr](https://fudr.in/scan)!

## Introduction

In the technology world, we techies are always crunched for performance. From seconds to sub-seconds, less than 500 milliseconds, or more commonly these days, less than 100 milliseconds.

Too often the answer is creating a new thread. It works, but only up to a certain limit.

---

For the non-techies among us: A thread is like a servant who does the work on our behalf. Lots of work can get delegated, and so the work can finish quickly.

Having lots of servants seems like a good idea! Doesn't it?

---

Sanskrit has a rule that says: *Ati Sarvatra Varjayet*. Meaning, that the excess of anything is bad. This applies to threads too!

Let's explore how reducing threads actually helped improve performance.

## Threads Breeding Like Rabbits

A piece of code was doing something like this:

```java
private void doMyWork(final GuestDto guestDto) {
    CompleteableFuture.runAsync(() -> doWorkInBackground(guestDto));
}
```

While the work gets done in the background, the code creates a **new** thread for each request. Meaning, for 500 requests, 500 threads get created.

On a 2-core CPU, it can only handle 4-8 threads efficiently. Why?

- When a new thread is born, it consumes certain CPU and memory.
- Context is copied from the main thread to the worker thread, which takes some CPU time.
- When an existing thread ends, the garbage collector has work to do.

---

Think of the overhead as a servant's salary.

Too many servants will take too much salary. After a while, the total expense might become too big to be justifiable.

The same goes for threads.

---

In such cases, 500 threads is surely an overload, especially during peak volume times. Here is a JMeter performance report for an API with such load.

{{< figure src="/images/nerdy-stuff/less-threads-more-performance/performance-before-optimization.png" alt="Performance Before Optimization" caption="JMeter performance report showing poor performance with excessive threads" >}}

This overload highlighted the need for a better approach.

## Simple Solution

Though recognizing the problem might be a bit tricky, the solution is pretty simple.

### Create a Thread Pool

Executor service provides an easy way to create and use the pooling functionality.

```java
ExecutorService pool = Executors.newFixedThreadPool(numberOfOptimalThreads);
```

### Use It!

The pool can then be simply used to trigger all code!

We used a pool with a min size of 10 and max of 20.

```java
private void doMyWork(final GuestDto guestDto) {
    CompleteableFuture.runAsync(() -> doWorkInBackground(guestDto), pool);
}
```

With this implementation in place, we observed significant improvements.

## Results

We saw immediate results.

{{< figure src="/images/nerdy-stuff/less-threads-more-performance/performance-after-optimization.png" alt="Performance After Optimization" caption="Performance improvement after implementing thread pool" >}}

Here is the math:

Old performance: 185 ms

New performance: 165 ms

Improvement: 20 ms

% Improvement: (20/185) * 100 = ~10%

A few minutes of changes led to a 10% performance improvement. The benefit-to-effort ratio is huge.

## Wrap Up

Servants, like threads, are good. But too many of them and the salaries to be paid become an overhead.

Too many servants and the master might go bankrupt!
