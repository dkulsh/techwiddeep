---
title: "ToT: Techies Overvalue Technology"
meta_title: "Techies Overvalue Technology: The Hidden Costs of New Tech Adoption | Deep Kulshreshtha"
description: "Explore why techies overvalue new technologies and how this bias leads to poor technical decisions. Learn about the hidden costs, blindspots, and the undervalued resources in software development."
date: 2021-08-12
image: "/images/tot-techies-technology/swoon1.jpeg"
categories: ["Opinions", "Technology"]
author: "Deep Kulshreshtha"
tags: ["technology bias", "software development", "engineering decisions"]
draft: false
toc: false
---

Techies have a love affair with shiny new tech, but sometimes it bites back—hard! **We overvalue the latest gadgets and gizmos, and it can tank projects and careers.**

#### The Shiny New Toy Syndrome

John and his team received a freshly minted contract to build an application. The required app was of medium complexity, and the contract was the first major one the organization had received.

Not only was the contract fresh, but the team was as well. Eager to impress the client, the team chose a new programming language—Scala. This choice went against tried-and-tested languages like Java, .NET, Python, C, or C++.

![New Technology](/images/tot-techies-technology/image1.jpg)

For non-techies: *Scala is a JVM (Java Virtual Machine) based programming language. It is known to be blazing fast and requires fewer lines of code. Additionally, Scala's style is a mix of object-oriented and functional programming*.

This sets the stage for understanding the challenges ahead.


##### The Hidden Cost of Cool

*Functional programming languages are cool to "look at."* They use 2 lines of code where a regular one takes 20.

Yet most people don't realize that while **the complexity of writing 10 lines gets hidden, the ease of debugging the same lines also gets hidden**. Essentially, the trade is:

- **Giving away the complexity of writing more lines of code**.
- **Accepting the complexity of debugging the same code**.

This means that while it might be easy to code in a functional programming language, it is **damn hard** to debug it.

**Complexity, like energy, is never destroyed**. While it can change its shape, form, or location, it will never be destroyed.

Building on this technical insight, let's return to the story.

#### Continued

The team went to work with their chosen tech stack. Of course, development went smoothly in the beginning—until the day bugs started pouring in.

Recall that debugging functional code is difficult. Suddenly, developers' lives became hard. They worked endlessly day and night, weekends and holidays. After a while, as most do in such scenarios, developers started quitting en masse. Compounding the attrition, **the company could not hire new developers**.

![Developer Attrition](/images/tot-techies-technology/image7-237w316h-225x300.jpg)

Any new programming language has a smaller ecosystem around it. That is, there are few developers, still-maturing tools, and smaller support communities. Using the supply-demand formula, we know that when something is scarce, it costs more. This means that when a programming language has few developers, they command high salaries.

Back to our story: The marketplace did not have many Scala developers in the first place. Unfortunately, the few who were found and did qualify could not be hired. Their asking salaries were way above the company's budget. So much so that the **work ground to a halt.**.

These challenges led to a pivotal decision.


#### The Bitter End

Increasing complexity and hiring challenges forced the company to abandon Scala.

As a result, the whole app was redone in a more "regular" programming language—Python. That is, the team restarted from scratch.

Of course, the rewrite meant:

- **Losing money**
- **Making the customer unhappy**
- **Missed timelines**
- **Blown opportunities**

Essentially, techies were eager to ride the "new technology bandwagon." The same wagon became the biggest bottleneck in the software development machine.

Moral of the story: **Techies overvalue technology, and sometimes it hurts them badly!**

Now, let's explore why such issues persist.

#### Why

This IT fiasco is all too common. **Why do smart pros make dumb tech choices?**

![Decision Making](/images/tot-techies-technology/image3-252w252h.jpg)

Undoubtedly, the decision-makers at John's organization were experienced professionals. For the most part, they knew what they were doing (or thought so). So, what explains such blunders by seemingly qualified professionals?

##### POBO (Plain Old Bias Object)

**A new technology is like a shiny new toy that every techie wants to get.** Until 10 years ago, it was Android, Mongo, Cassandra, Go, Scala. Now we have Artificial Intelligence, Blockchain, Virtual Reality. In the next 10 years, newer techs will come up.

Learning and adopting new technologies is great. But the problem occurs when techies only see the green grass on the other side. That is to say, **techies overestimate new technologies and underestimate existing ones**.

We become biased toward anything we stay in proximity to, such bias shouldn't be a surprise (it also explains the poor decisions). Instead, the surprise is that organizations do not take proactive steps to avoid the biases.

_PS: POBO sounds like a Java phrase—POJO (Plain Old Java Object), hence funny to techies._


##### Pluralistic Ignorance

During the same time, a competitor was pushing its developers towards Scala too. The idea was: If John's team is using it, then it must be good. In this case, let's not be "left behind" in tech fashion world. **Keeping up with the Joneses, org style!**

![Monkey See Monkey Do](/images/tot-techies-technology/image4-245w320h-230x300.jpg)

*Monkey see, monkey do.*

The competitor wanted to chase Scala because John's team was using it, not because they needed it. Interestingly, John's team might have used Scala copying another team. And that other team might have been doing it because of yet another. Together, all teams might have been making a poor choice, but with the "social proof" of their collective choices.

When my decision to choose technology is based on the other guy's choice, then I am being ignorant. **Such ignorance spread across the industry is called "pluralistic ignorance."**

##### The Resume Objective Fiasco

Look at resumes: that pointless "Objective" section? "To excel in a dynamic environment..." Blah blah. **Adds zero value, yet everyone's got one—pluralistic ignorance in action!** Ever copied a trend without questioning why?

##### Follow the leader

Techies foolishly think: "Google uses Scala, so let's use it too." Never mind the fact that:

- **Google processes petabytes of data every day**
- **Our complete application is less than a few gigabytes**

Pluralistic ignorance is another reason for poor tech decisions. That is, **it ignores real proof in favor of social proof, only to pay later.**


##### Blindspots


Every new technology has blindspots and takes time to mature through them. Part of our job as techies is to recognize these blindspots.

![Blindspots](/images/tot-techies-technology/image8-264w257h.jpg)


- New tech dazzles, but **blindspots lurk.** Kafka? Great MQ, but as a DB or scheduler? Not really.
- JavaScript mobile apps? Slick, but laggy compared to native. **Mitigate with customs, but issues sneak up post-commitment.**

While each these can be mitigated by custom solutions, the need for them isn't obvious during the trial run. In other words, new technologies have blindspots, and they don't show until the technology has charmed us into marriage.

**Due to our technology bias, we select the best horse in a world of cars ! The best, but not relevant.**


##### Step Children

Having understood the "overrated" aspects of technology, let's talk about some "underrated" ones. In the world of the new, all love goes to the newborn techs. During such times, what are treated like stepchildren?

<!-- ![Step Children](/images/tot-techies-technology/image2.jpg) -->

###### Oldies

.NET has flying reviews. Here is one from Stack Overflow. As the highest-ranked framework, it is loved by a third of developers worldwide. ([review link](https://insights.stackoverflow.com/survey/2021#section-most-popular-technologies-other-frameworks-and-libraries))

![Stack Overflow Survey](/images/tot-techies-technology/Screenshot-2021-08-12-at-9.55.59-AM-300x141.png)

The liking rose more when the survey considered professional developers. ([link](https://insights.stackoverflow.com/survey/2021#most-popular-technologies-misc-tech-prof))

![Professional Developers Survey](/images/tot-techies-technology/image10-624w285h-300x137.png)

In for a long time, .NET has covered most of its blindspots. For example, there aren't many threading surprises, memory leaks, or garbage collection issues. That is, once something is written, it works. No ifs, no butts.

For these reasons, .NET apps work like a charm. This makes leaders happy and keeps developers' lives peaceful, also explaining the love for such old technology.

**Battle-tested, few surprises—apps run like clockwork.** Yet, techies ditch it for "new." I know orgs strong in .NET who switched anyway. **Why? It's "old."**


Similarly, relational databases are starting to get treated as stepchildren, in spite of the fact that they play a central role in many great apps and are a more polished product than most realize. In fact, if the DB structure is mostly fixed, a relational DB is a better choice by a big margin.

A successful telecom support organization told me that they use MySQL. Although treated as a stepchild by others, it works great for the org with millions of records each day. **Unlike many other organizations, they did not want to touch NoSQL DBs**.

Though relational DBs are stepchildren at many places, they are gold at this organization.

###### The Most Unloved Stepchild: Engineers

**We tend to forget that the human component is always the trump card.**

![Engineers](/images/tot-techies-technology/image5-378w213h-300x169.jpg)

**Humans outshine any tech.** Spring Boot scheduler? Great, but thread pools, ad-hoc calls?

- What thread is picked by the scheduler? Does it pick one from the application's pool, or does it fetch a new one from the JVM? Too many threads might cause the app to slow down.
- In addition to the scheduled processing, what if we need ad hoc calls? Spring does not provide any such provision, so one will have to be designed and custom-built.

Solving such problems is an engineer's job. This goes to prove that, despite the best libraries, **we depend on humans for the best solutions.**

Here are some other examples:

- **Small data? Skip normalization.**
- **No perf issues? Single-thread fine.**
- **HTTP suffices? Ditch events.**

Each of the above points has a common decision-maker: engineers. This translates to, **while each technology has one strength area, engineers have many.**

Finally, let me ask: Are engineers valued as the best solution providers?

No. They are treated as a stepchild.

Exactly my point! Let's tie these ideas together.

#### Wrap Up

We saw John's tech bias torpedo his project, unpacked the causes, and spotlighted the underrated. **Techies overvalue tech—and it bites back.** Acknowledge biases first. Ask: **What's this tech's drawback?** Balance hype with reality to avoid egg on your face.


![Conclusion](/images/tot-techies-technology/image6-190w246h.jpg)

We started by discussing how John's organization hurt itself by its tech bias. Then we went on to understand the causes of such poor decisions. Finally, we looked at the other side of the bias—the undervalued resources.

Wrapping up, I mention the point again: **We techies overvalue technology, and the bias comes back to bite us in the ass.** The number one thing we can do is acknowledge our biases. Once we do, steps toward mitigating them become easier.

I'm all for innovation, but with eyes wide open.
