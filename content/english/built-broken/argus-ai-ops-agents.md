---
title: "Argus: Taming a Lakh Alerts a Day with a Fleet of AI Agents"
meta_title: "Argus — an AI agent fleet that cut production alerts by ~99%"
description: "How I designed and shipped an AI agent fleet that turned ~1 lakh production alerts a day into a few hundred — reclaiming a senior engineer's time daily, and giving the team something worth far more: confidence."
date: 2026-08-23T00:00:00Z
image: "/images/nerdy-stuff/multi-tenancy/multitenant-architecture.png"
categories: ["Technology", "AI", "Observability"]
author: "Deep Kulshreshtha"
tags: ["ai-agents", "llm", "observability", "devops", "automation", "platform"]
draft: false
toc: false
weight: 1
---

I inherited a monitoring setup that was, in effect, screaming non-stop.

Around **20 microservices** all logged to a central log platform. A blunt filter counted anything matching `Error | ERROR | Exception` as an "alert." The result: **~1 lakh (100,000) alerts a day**. Some were real. Many were noise. But the sheer volume meant nobody could tell which was which — so nobody could act on any of it.

This is the story of **Argus**: a fleet of AI agents I designed and built to fix that. Not by writing a smarter regex, but by giving the noise a brain.

{{< notice "success" >}}
**Reclaimed roughly one senior engineer's time — every single day.**

- **Direct value:** ~**₹40–75 lakh / year** in reclaimed senior-engineering capacity
- **Alert volume:** **~1 lakh/day → a few hundred/day** (**~99% reduction**)
- **Effort:** ~**1 engineer-day/day** of manual triage → ~**30 minutes/day**

And yet the rupees were the *smaller* prize. The bigger one was **confidence** — I'll come back to that.
{{< /notice >}}

## The Problem

The math was brutal. At ~1 lakh alerts a day, triage was a full-time job for **one to two senior developers**: read an alert, find the cause, decide false vs. real, fix the real ones, ship. Every day.

I did dozens of those fixes myself. Then it hit me — this wasn't engineering, it was **manual labour dressed up as engineering**. The volume wasn't just expensive; it was *demoralising*. When everything is an alert, nothing is. Real incidents hid in plain sight, and the team slowly learned to ignore the firehose entirely.

You cannot fix a 1-lakh-a-day problem one ticket at a time. So I stopped trying to.

## The Turn: Build a System, Not a Backlog

Over roughly **six weeks — solo, alongside my existing responsibilities** — I built Argus: an autonomous pipeline of specialised AI agents that watches every service, does the triage a human used to do, and only ever taps a human on the shoulder when something genuinely needs one.

The design goals mattered more than any single line of code:

- **It had to run itself.** No babysitting. Once a day, at **9:30 a.m.**, it wakes up and works through its pipeline one crash-safe, idempotent step at a time — so a failure never doubles up or drops a report. (It's built to run as often as **every 15 minutes**; that higher cadence is ready but switched off for now — daily is all the estate needs today.)
- **It had to be cheap.** A **lightweight, inexpensive model** does the high-volume routine work (counting, summarising, first-pass triage). A **stronger reasoning model** is spent only on the rare deep investigation — reading source code, logs and database state to explain *why* something broke.
- **It had to be extensible.** Everything is data-driven: a three-level model of **Teams → Infrastructure → Configuration**, all in a database. Onboarding a brand-new team or service means **inserting a few rows — zero code, zero redeploy.** The system discovers the new work on its own.
- **It had to get smarter over time.** Argus learns its own noise. Once a pattern is confirmed harmless, it's suppressed permanently — so the same false alarm never wastes a human's attention twice.

Under the hood it's a relay of agents, each with one job:

1. **Monitors** collect the raw truth — error counts per service, plus infrastructure and data-quality signals.
2. **Reporter** turns those raw numbers into a readable daily digest.
3. **Triage** decides what actually deserves a human's attention.
4. **Investigator** deep-dives the few that do, and recommends a fix.
5. **Notifier** posts the result to the team — clean, ranked, and attributable.

## What It Actually Produces

No dashboards to open, no queries to run — the signal comes to you. Argus posts clean, ranked digests straight into the team's channels. The formats below are the real thing; the names and numbers are illustrative.

**The daily error digest** — one glance tells you exactly where to look, sorted by severity:

```
🔴 Error Alerts — Last 24h
┌──────────────────────────────┬────────┬──────────┐
│ Service                      │ Errors │ Severity │
├──────────────────────────────┼────────┼──────────┤
│ Import Service               │  3,909 │ CRITICAL │
│ Audit Service                │    724 │ WARNING  │
│ Processor                    │    568 │ WARNING  │
│ Distribution Service         │     24 │ NORMAL   │
│ Mapper                       │      2 │ NORMAL   │
│ Receiver                     │      1 │ NORMAL   │
├──────────────────────────────┼────────┼──────────┤
│ TOTAL                        │  5,228 │ 6 svcs   │
└──────────────────────────────┴────────┴──────────┘
```

**The infrastructure digest** — databases, queues and functions, watched the same way. CPU headroom, queue backlogs and their oldest waiting message, slow functions:

```
🟡 Infra Alerts — Last 24h

RDS
┌──────────────────────────────┬──────────┬──────────┐
│ Instance                     │ CPU Peak │ Severity │
├──────────────────────────────┼──────────┼──────────┤
│ Prod DB - Writer             │   98.33% │ CRITICAL │
│ Prod DB - Reader             │   48.12% │ NORMAL   │
└──────────────────────────────┴──────────┴──────────┘

SQS
┌──────────────────────────────┬──────────┬──────────┐
│ Queue                        │ Backlog  │ Peak Age │
├──────────────────────────────┼──────────┼──────────┤
│ Audit Queue                  │    2,002 │   2,068s │
│ Push Queue                   │    2,129 │      12s │
│ Converter Queue              │      126 │   2,116s │
└──────────────────────────────┴──────────┴──────────┘

Lambda
┌──────────────────────────────┬──────────┬──────────┐
│ Function                     │ Avg Dur  │ Severity │
├──────────────────────────────┼──────────┼──────────┤
│ Converter Function           │ 59,834ms │ WARNING  │
└──────────────────────────────┴──────────┴──────────┘
```

And when something genuinely needs a human, the investigator agent doesn't just flag it — it **explains** it:

```
🔎 Investigation — Processor error spike

• 94% of the day's errors trace to ONE failure: a duplicate-key
  violation during a bulk import → the whole save then rolls back.
• Trigger: the same file was uploaded 3× in ~75 minutes. Run 1
  applied cleanly; runs 2 & 3 re-inserted rows that already existed.
• Root cause: the import isn't idempotent — a re-run re-inserts
  instead of skipping.
• Impact: noisy, but NON-corrupting — the DB constraint held;
  exactly one row per record.
• Recommended fix: skip-if-exists on re-import, and log duplicates
  as "already applied" instead of hard-failing the whole batch.
```

That last block is the difference between a dashboard and an agent. A dashboard shows you *"568 errors."* Argus tells you **what broke, why, whether it actually matters, and how to fix it** — the exact reasoning a senior engineer used to do by hand, every morning.

## The Results

Argus has now run **unattended for ~4.5 months**. The numbers below are pulled straight from its own production database.

**Alert volume — the trajectory.** This was never a clean downward line, and that's the point. Each spike is a *real* problem the system made **visible and traceable** — after which it got fixed or suppressed, then fell away.

| Timeframe | Avg alerts / day |
| --- | --- |
| First full week live | **~57,000** |
| Worst spike week | **~138,000** |
| A typical recent week | **~557** |

(That ~1 lakh/day I started with was the *raw*, unfiltered firehose; by the time Argus was measuring, its first crude exclusions had already pulled the baseline down to ~57,000/day — and it kept falling from there.)

The jagged path from tens of thousands to a few hundred a day is the whole story: for the first time, every burst of noise had a name and a cause. That is a **~99% reduction to steady state** — but more importantly, the remaining few hundred are *signal*, not static.

**Engineering effort — what it gave back.**

| Metric | Before | After |
| --- | --- | --- |
| Senior-dev triage time | ~**1 engineer-day / day** | ~**30 minutes / day** |
| Real incidents | buried in the firehose | **surfaced, ranked, attributed** |
| False alarms | re-triaged endlessly | **learned once, suppressed forever** |

**Operating scale — what "runs itself" actually means.**

| Metric | Value |
| --- | --- |
| Reports generated | **22,282** |
| Investigations opened | **226** (**197** resolved) |
| Uptime | ~**4.5 months**, unattended |
| Cadence | **daily at 9:30 a.m.** (15-min mode built, disabled) |
| Build effort | ~**6 weeks**, solo, alongside a full workload |
| Running cost | negligible — cheap model for volume, strong model only when it counts |

Built in six weeks. It pays back its own build cost in a matter of weeks, then compounds — for the price of a rounding error in the cloud bill.

## The Hard Parts

None of this was a weekend script. Three problems took real engineering judgement:

- **Learning noise without going deaf.** The whole point was to suppress false alarms — but over-suppress and you bury the one error that matters. Argus silences only *confirmed* noise, pattern by pattern and service by service, and keeps everything else loud. Drawing that line — and redrawing it as services change — was the core challenge.
- **Trustworthy while unattended.** A system that runs on its own schedule with nobody watching has to be boringly reliable. Every phase is idempotent and crash-safe: a failure mid-run never double-posts a report or silently drops one. "Runs itself" only earns trust if it never surprises you.
- **Intelligence on a budget.** Pointing a strong reasoning model at a lakh log lines a day would cost more than the engineers it freed. So the work is tiered — a cheap, fast model handles the high-volume routine, and the expensive reasoning model is spent only on the handful of issues that truly need it.

## The Real Prize: Data → Confidence

Here's the part that matters more than any rupee figure.

Before Argus, the honest answer to *"how healthy is production?"* was a shrug. We had a number — a lakh — and it meant nothing. There was no trend, no attribution, no way to tell a good week from a bad one.

Now there is. Every service, every day, has a measured, trending, explained baseline. When something moves, we know **which service, how much, and why** — often before a customer ever notices. When we ship a fix, we can *watch* the line come down and **prove** it worked.

For a team that was flying blind, that shift — **from anecdote to data** — changed how we operate. We ship faster because we can see the blast radius. We onboard new services calmly because the system just picks them up. We sleep better because silence now means *healthy*, not *ignored*.

For a **scaling organisation, that confidence is worth far more than the ~₹40–75 lakh/year of reclaimed engineering time.** Money saved is a one-time win you can bank. A team that can *trust its own production signal* is a capability that compounds with every service, every hire, every release. The rupees get the room's attention; the confidence is what actually moves the company.

That confidence is also what turned a side-project into a mandate. Argus began as something I built on my own initiative; once the data was undeniable, leadership backed it — and it grew from watching one team's services into a multi-tenant platform other teams can onboard onto by adding a few rows. Crucially, the senior-engineering time it freed didn't vanish into a line-item called *saved cost* — it was redeployed onto revenue-generating work. That is the quiet compounding: one engineer's initiative became a capability the whole organisation leans on.

## What's Next — Built, and Deliberately Switched Off

Some of the most valuable parts of Argus already exist in the codebase but are **intentionally turned off** — because good judgement is knowing when *not* to flip a switch:

- **Autonomous investigation & fix suggestions.** The agents can already read code, logs and database state to diagnose an issue end-to-end. It's gated off until the AI has enough surrounding context that I'd trust its conclusions unsupervised. Restraint here is a feature, not a limitation.
- **Executive summaries for leadership.** Weekly and monthly digests — the health of the whole estate, distilled for a busy leader — are designed and wired to their own channel, ready to switch on when there's an audience for them.
- **From errors to opportunities (in progress).** The same pipeline that surfaces what's *broken* is being pointed at what's *possible* — turning raw operational data into business-value opportunities, in custom reports tailored per audience.

And I'll be honest about the ceiling. Getting from the firehose to a 99% cut was the *achievable* win. The final 1% — the long tail of rare, ambiguous errors that fit no pattern — is genuinely hard, and it's still ongoing. Ninety-nine percent isn't "done"; it's the point where the problems that remain are finally few enough to solve one at a time.

## The Point

Anyone can write the code. The value wasn't in the Python.

It was in seeing that a 1-lakh-a-day problem couldn't be solved by working harder — only by working *differently*. In designing a system that runs itself, extends itself, and earns trust with data. In building it solo, shipping it, iterating on it in production for months, and showing leadership — with evidence — what it was worth.

The alerts went down by 99%. But the thing I'm proudest of is what went **up**: the team's confidence that, whatever production is doing right now, we can *see* it.
