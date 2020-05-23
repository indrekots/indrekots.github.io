---
layout: post
title: "The Three Ways of DevOps"
excerpt: ""
modified: 2020-03-31 18:56:18 +0300
categories: articles
tags: [devops]
image:
  path: /images/2020-04-01-3-ways-of-devops/cover.jpg
  thumbnail: /images/2020-04-01-3-ways-of-devops/cover_thumb.jpg
  caption: "Photo by [Javier Allegue Barros](https://unsplash.com/photos/C7B-ExXpOIE)"
comments: true
share: true
published: true
aging: false
amazon_links: true
---

If you ask three persons to describe DevOps, you'll get four different answers.
Sometimes, developers doing operations work is branded as DevOps.
Others say it's about automation of infrastructure and deployments.
Occasionally, you can see that DevOps is a modern title for sysadmins.
So what's the truth then?
What is DevOps?
We can clearly see that the term has a *buzzword* status.
It has a different meaning in different contexts, kind of like *agile* did years ago.

I don't consider myself an expert or an authority in the field to give a definitive description of DevOps.
The fact is, [words change meaning over time](https://ideas.ted.com/20-words-that-once-meant-something-very-different/).
Instead, in this post, I'm going to summarize the three core principles of DevOps—*the three ways*—according to the book [*The DevOps Handbook: How to Create World-Class Agility, Reliability, and Security in Technology Organizations*](https://amzn.to/3bAWtC0).

## 1. Principles of Flow

// define value stream

The first principle is about creating a smooth flow of work through the different functional areas of an organization.
Emphasis should be put on the global goals of the entire system, not on the local goals of individual departments.
In technology organizations however, a lot of work is invisible.
To prioritize work in the context of global goals, work should be made visible.
For starters, this can be achieved with Kanban boards.

<figure class="align-center">
  <img src="{{ '/images/2020-04-01-3-ways-of-devops/kanban.png' | absolute_url }}" alt="">
  <figcaption>Example of a Kanban board from <a href="https://trello.com/">Trello</a>. Work is visualized and moves from left to right.</figcaption>
</figure>

Kanban boards help visualize *work-in-progress* (WIP).
This is the amount of work that has been started but is not yet completed.
A high amount of WIP could be sign of multitasking.
Context switching (citation needed) is expensive and dramatically hinders the flow of work.

// Stop starting, start finishing.

To constrain WIP, we should reduce batch sizes.
The idea originates from [Lean Manufacturing](https://en.wikipedia.org/wiki/Lean_manufacturing "Lean manufacturing").
It was common in manufacturing to produce components in large batches.
Setting up new machinery and switching between jobs was costly and time consuming.
Therefore, it was considered practical to produce as many parts as possible once machinery had been set up.

For example, a car production plant would produce a lot of body panels at a time to reduce the number of changeovers.
This, however, creates a large amount of WIP.
The variability in the flow of work cascades through the entire manufacturing plant, resulting in long lead times.
It doesn't improve quality as well.
Imagine, what would happen if a flaw was found in the produced body panels when they're being assembled?
Most likely, the entire batch has to be discarded and redone.
Producing large batches delays feedback.

The same ideas apply to software development.
However, instead of machinery and body panels, we're dealing with code.
Every commit into version control increases the batch size, creates WIP and variability in flow in the software development value stream.

> The batch size is the unit at which work-products move between stages in a development process.
> <footer><strong>Eric Ries</strong> &mdash; <a href="http://www.startuplessonslearned.com/2009/02/work-in-small-batches.html">StartUp Lessons Learned</a></footer>

A classic example is an annual production deployment schedule.
If a deployment is done once a year, the batch size is huge; a year's worth of work being deployed in a single step.
Similarly to the car plant, if anything goes wrong, the entire batch has to be rolled back.
We must find and fix issues in it, test it and redeploy it.
These kinds of events are disruptive to the flow of work.
Not only do we have to deal with unplanned work, but we have to deal with work that moves from *right to left* in the value stream.
Because of delayed feedback, it's more time consuming to resolve the issues that caused the failure.
The time between when the error was made and when it was discovered is long and naturally, knowledge fades over time, making it more difficult to fix the issues.

<figure class="align-center">
  <img src="{{ '/images/2020-04-01-3-ways-of-devops/large-batch.png' | absolute_url }}" alt="">
  <figcaption>Large batch size, lots of changes over time. <a href="https://www.slideshare.net/jallspaw/ops-metametrics-the-currency-you-pay-for-change-4608108">Ops Meta-Metrics by John Allspaw</a></figcaption>
</figure>

Large batches create issues not only in deployments.
For example, let's look at long-lived feature branches.
The longer a branch stays isolated and the more changes it sees, the larger the batch size.
Over time, it becomes increasingly difficult to integrate it back to mainline.
The potential for merge conflicts high.
Since feedback is delayed, the potential for rework is high.
These factors are disruptive to the flow of work and increase deployment lead times.

<figure class="align-center">
  <img src="{{ '/images/2020-04-01-3-ways-of-devops/small-batch.png' | absolute_url }}" alt="">
  <figcaption>Smaller batch size, less changes over time. <a href="https://www.slideshare.net/jallspaw/ops-metametrics-the-currency-you-pay-for-change-4608108">Ops Meta-Metrics by John Allspaw</a></figcaption>
</figure>

To improve deployment lead times, batch sizes need to be made smaller.
Long-lived feature branches are discouraged.
In order to get faster feedback, it's better to integrate early and often.
If we continue to reduce the batch size, we eventually arrive at a _single piece flow_, where every commit to version control flows through the entire software development value stream.
Once all automated checks have passed, changes end up in production.

Teams that are able to pull that off make use of practices such as [trunk-based development](https://trunkbaseddevelopment.com/), [continuous integration](https://martinfowler.com/articles/continuousIntegration.html), [continuous delivery](https://martinfowler.com/bliki/ContinuousDelivery.html) and [continuous deployment](https://en.wikipedia.org/wiki/Continuous_deployment).
They've invested in test automation and have designed their software for low-risk releases.
They have also organised themselves so that the number of required hand-offs is reduced.

Every time work needs to be handed off to another team, we increase deployment lead times.
A hand-off requires communication and coordination.
Unfortunately, even under the best circumstances, some knowledge gets lost.
This is a potential spot where errors can creep in and work can pile up, disrupting the flow and increasing deployment lead times.

Continually identifying and eliminating constraints in our work is key to improve throughput and reduce lead times.
In [Beyond the Goal: Theory of Constraints](https://amzn.to/2yDMvRD), the author Dr. Goldratt states

> In any value stream, there is always a direction of flow, and there is always one and only one constraint; any improvement not made at that constraint is an illusion.

An example from a technology value stream is _environment creation_.
If it takes hours to set up a testing environment, any improvements we do to work that happens before it in a value stream is an illusion.
For instance, if we reduce our build time from 10 to 3 minutes, we get faster builds.
But nothing gets done faster overall.
Environment creation is still a blocker.
What's worse, WIP is increased.
New builds pile up even faster now, waiting to be deployed to the test environment.
Since environment creation is blocking new work form passing through, steps that should happen after are starved of work.
We should find the _single_ constraint in our value stream and deal with it.
When it comes to environment creation, it should be automated and on-demand.

We should keep our eyes open for anything that doesn't add value to the customer and eliminate it from the _flow_.
_The first way_ was about work that moves from left to right, whereas the next principle we're going to look at is about feedback that moves from right to left in the value stream.

## 2. Principles of Feedback

*The second way* of DevOps is about creating fast feedback loops that allow us to build safer systems.
Whether you like it or not, software is complicated.
Even the smallest of changes can have catastrophic consequences.
Without fast feedback, we distance the cause and effect.
Errors can slip in undetected, only to be discovered later when the cost and effort to fix them is higher.

Ideally, we'd like to discover problems as they occur.
If you've written any code in an IDE or text editor that points out issues as you type them, then you know what I'm talking about.
Ah, the *red squiggly line*.
This type of real-time feedback allows us to fix issues when it is cheap and easy to do so.
Additionally, we get to learn from our failures quickly.

> "It is impossible for a developer to learn anything when someone yells at them for something they broke six months ago - that is why we need to provide feedback to everyone as quickly as possible, in minutes, not months." - Gary Gruver

Unfortunately, we don't have a *red squiggly line* in our editors for future production issues.
Nor do we have them for features customers don't like.
The next best thing is to reduce the time between when the issue is introduced and when it is detected as much as possible.

In environments, where quality is [*somebody else's problem*](https://en.wikipedia.org/wiki/Somebody_else%27s_problem), we involuntarily delay feedback.
For instance, if QA is in charge for the correctness of the work produced by developers or operations is solely responsible for the software in production, quality is moved further from the source.
Feedback is received by handing over work to the next work center in the value stream.
As was discussed before, handovers potentially create delays.
Work can pile up because, say, QA is busy testing something else or operations doesn't have time to provision a testing environment.
Feedback is delayed.

As we saw when discussing *the principles of flow*, if we don't get immediate feedback, there's a risk of passing on defects to downstream work centers.
This creates disruptions in the flow of work.
Not only are issues more difficult to fix, we also increase *work-in-progress* and batch size.
While developers are waiting for work to be tested, most likely new work is being introduced.
This can lead to a downward spiral of ever increasing lead times.
Fast feedback, on the other hand, prevents the start of new work which is more likely to introduce new errors since it's built on top of the previous error.

It seems counterintuitive, but adding more inspection steps and approval processes increases the likelihood of failures.
Surely, having more eyes on the problem should produce better results, right?
That might be true if everybody is fully informed of the work at hand.
But as we start to push decision making further from where the work is performed, the effectiveness of approval processes decreases.
Getting approvals from people who are distant from the work has several issues.
First of all, they don't have the most up to date information.
It could lead to mistakes.
Secondly, they might be busy with other work.
Approvals can start to pile up, creating delays.
Under pressure, they could rubber stamp decisions.
These kind of quality control measures can give you a false sense of security.
To receive faster and more effective feedback, we should instead push quality closer to the source.

In tech value streams, fast feedback can be achieved with build automation, automated testing, continuous integration, automated deploys, production telemetry and alerting.
Fast feedback loops enable quick detection and recovery of problems.
It also informs us on how to prevent these problems in the future.
It prevents the work center to start new work which is likely to produce new errors
Preventing the introduction of new work enables CI and CD, a single-piece flow.

Quality should be everyone's responsibility.
We should build software for customers and also for downstream work centers.
For instance, dev should build great software for end users but they should also optimize it for operations.
monitoring, logging, deployments
In a way, operations are also customers for developers.

## 3. Principles of Continual Learning

The third way is about creating a culture of continual learning and experimentation.
We saw that fast feedback enables us to learn from our mistakes.
But that's only part of the solution.
We want to constantly learn, improve, experiment and create new knowledge in the entire organization.
After all, organizations that are able to learn quicker than others have a competitive advantage.

Work environments where incidents are followed by the *name, shame, blame* pattern most likely have systematic quality issues.
If you're punished for your mistakes, you're incentivized to stay in your comfort zone; that happy little place where you're less likely to make mistakes.
This, however, prevents organizational learning.
Fear of mistakes creates a culture where instead of admitting failure, it's perhaps more convenient to hide it under the covers.
People are less likely to speak up when problems are found.
They're also less likely to propose novel solutions to problems because the last time it resulted in a failure.

When small failure signals are withheld, they accumulate until a major catastrophe happens.
This usually results in more processes and approvals.
As discussed previously, this can lead to lower throughput and longer lead times.

Ron Westrum, [in his 2004 paper titled "A Typology of Organisational Cultures"](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1765804/), describes this as having a pathological organization culture.

> When things go wrong, pathological climates encourage finding a scapegoat [...] Pathological environments will discourage taking responsibility, and can be expected to conceal their problems.

Westrum defines three dominant culture types - pathological, bureaucratic, and generative.
On the opposite side to pathological, we have *generative organizations*.

> Generative organisations require empowerment for maximum performance. Individuals’ minds are harnessed to fulfill the organisation’s goals through a culture of conscious inquiry. They are encouraged to speak up, think outside the box, and to act as fully conscious participants in a great cooperative enterprise.

[The 2014 State of Devops Report](https://www.researchgate.net/publication/263198947_2014_State_of_DevOps_Report) goes on to say that in generative environments, it is understood that continuous improvement leads to ever-higher levels of throughput and stability.
Learning is actively promoted.
Everybody is encouraged to run experiments to learn how to improve both processes and the products and services they build.
Instead of blaming humans for errors, they look into ways to redesign the system to prevent another one.
And when failures do happen, they're treated as learning opportunities.
[Blameless post-mortems](https://www.atlassian.com/incident-management/postmortem/blameless "How to run a blameless postmortem") are held to create organizational learning from incidents.

> By removing blame, you remove fear; by removing fear, you enable honesty; and honesty enables prevention
> <footer><strong>Bethany Marci</strong></footer>

 Institutionalize the improvement of daily work
  * reserve time for improvement and learning
  * Toyota kata
  * Teams that are not able to improve their daily work, not only will they continue to suffer, the suffering increases due to chaos and entropy, processes degrade over time.
  * In Tech: relying on daily workarounds, tech debt increases, increases cycle time and productivity
  * Lean IT: Even more important than daily work is the improvement of daily work
  * Reserve time in every iteration to pay down technical debt.
* Transform local discoveries into global improvements
  * When new learnings are discovered locally, the entire org should know about it
  * E.g. blameless post-mortems public and searchable
* Inject resilience patterns into our daily work to force improvement
  * Continually introduce tension to the system to improve performance and resilience - chaos engineering, antifragile systems
  * Tech value stream, find ways to reduce deployment lead time, increase test coverage, decrease test execution times, re-architect systems

Paul O'Neill, CEO, Alcoa, started to be notified about accident near-misses in addition to actual accidents. Find and detect ever weaker failure signals and act upon them, before they result in a catastrophic failure.

space shuttle Columbia disaster, foam dislodgement, weak failure signal that was not taken seriously

improvement blitz, kaizen blitz, improvement of daily work over daily work itself

!!the only way to master something is to practice a lot, frequent deployments, if you do something a lot, you get really good at it

## Summary

Set of principles and practices, org culture
The principles behind DevOps work patterns are the same that transformed manufacturing (lean)
busy/wait time
four key metrics
