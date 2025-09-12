---
title: "Got Hacked. Yay!"
meta_title: "Got Hacked. Yay! - Security Vulnerability & Fix"
description: "How Fudr was hack-attempted and the technical patch implemented to fix the Spring Boot vulnerability. A learning experience through pain."
date: 2022-06-12T00:00:00Z
image: "/images/nerdy-stuff/got-hacked/hacked-cover.jpg"
categories: ["Technology", "Security"]
author: "Deep Kulshreshtha"
tags: ["security", "spring-boot", "vulnerability"]
draft: false
toc: true
---

How Fudr was hack-attempted. And the tech-patch done to fix the vulnerability.

Read till the end to know why this is good news.

## Introduction

"Pathemata Mathemata" is a Greek proverb meaning, "Learning through pain and suffering." Like

- Proposing to a girl face to face.

The risk/humiliation filters out most wavering candidates.

- Learning sales by doing sales.

The pain of rejection is enough to educate. And fast.

The tech world offers its fair share of *learning through pain*.

Solving production issues in one's own code is enough to test the code thoroughly (not to mention fixing the corrupted data and patch fixes).

One such instance occurred when a restaurant informed Fudr that a patron had reduced the price of an item (from 250 to 25) and had successfully placed an order in the system.

Alarm bells rang and the team got to work.

To understand the issue, let's examine how the vulnerability was identified.

## Find the Leak

Modern-day browsers provide enough tools to figure out the anatomy of any network call. Considering this, monitored traffic would surprise no institution.

The catch comes when the traffic can be messed with!

{{< figure src="/images/nerdy-stuff/got-hacked/network-traffic-monitoring.png" alt="Network traffic monitoring" caption="Modern-day browsers provide enough tools to figure out the anatomy of any network call." >}}

Now, let's dive into the technical details behind this.

### Nuts and Bolts

Next: How was the traffic messed with?

The backend is designed with Spring Boot. And Spring Boot, by default, exposes all repository interfaces.

This means someone with a reasonable tech mind can figure out the ins and outs of a data resource. As an example, a resource named "Advisor" would be exposed at:

| Command | URL |
| --- | --- |
| curl --request GET --url http://localhost:5000/restaurant-service/advisors/3e17574b | (Read command) |

Where...

- advisors - would be the resource name. And
- 3e17574b - would be the resource id.

The above is a "read" command, but we can easily create a "create" or "edit" command too. In simple words: someone technical enough is able to access and edit all data.

This is how the hack was done. Problem figured out!

For those less familiar with tech, here's a simpler explanation.

### Non-Tech Lingo

For the non-techies among us: Imagine getting a custom-tailored suit.

- Finding a good shop.
- Buying a piece of cloth. Right color, right look and feel, in fashion.
- Finding a good tailor.
- Giving requirements: 2 or 3 pieces, tight or comfort fitting, 2 or 3 buttons, lapel size, etc.
- Going for fittings.
- Then the final suit comes in!

Do you see the hassle?

Spring Boot is like a ready-made suit that works in most ways for most people.

One-stop purchase and we are done.

However, the framework, while great, has faults. We encountered one such fault.

With the problem understood, let's look at how we fixed it.

### The Plug

{{< figure src="/images/nerdy-stuff/got-hacked/spring-boot-fix.png" alt="Spring Boot fix implementation" caption="Like all things Spring Boot, the fix had a few approaches." >}}

Like all things Spring Boot, the fix had a few approaches.

First, through configuration.

#### Configuration

Adding the following config, and we would be done.

| Property | Value |
| --- | --- |
| spring.data.rest.detection-strategy | annotated |

Alternatively, we could implement it via code.

#### Code

Exposing the same config via a config Spring bean.

```java
@Configuration
public class CustomSpringDataRestConfig extends RepositoryRestConfigurerAdapter {
    @Override
    public void configureRepositoryRestConfiguration(final RepositoryRestConfiguration config) {
        super.configureRepositoryRestConfiguration(config);
        config.setRepositoryDetectionStrategies(RepositoryDetectionStrategies.ANNOTATED);
    }
}
```

We used one of the above approaches. No prizes for guessing.

In summary, this experience highlighted some positive aspects.

## Wrap Up

No one attempts to hack John Nobody. In other words, an organization needs to be big enough to be challenged.

The attempt, which was a sales pitch (and I might add a fair one—not too good, and not too bad), only proved that the team is doing something well.

This is what calls for celebration. 🍻 🍻

As to Pathemata Mathemata, I'll borrow from Nassim Taleb:

*Let us return to pathemata mathemata (learning through pain) and consider its reverse: learning through thrills and pleasure.*

*Skin in the game can make boring things less boring. When you have skin in the game,*

- *Dull things like checking the safety of the aircraft because you may be forced to be a passenger in it cease to be boring.*
- *If you are an investor in a company, doing ultra-boring things like reading the footnotes of a financial statement (where the real information is to be found) becomes, well, almost not boring.*

Reading the most boring Spring Boot documentation suddenly became very "non-boring."

! Learning through pain in action !



