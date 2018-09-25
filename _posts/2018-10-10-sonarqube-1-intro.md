---
layout: post
author: wachulski
title: "Code of mine, better than yours, of course..."
subtitle: "SonarQube Series #1. Automating code review process to filter out boilerplate and enable human-oriented reviews."
categories: [software]
tags: [software, SonarQube, "clean code", "technical debt"]
---

Just committed, with love. And what code it is&mdash;well thought, tested, self-documented, clean. Our code, the code. Its review must be a piece of cake, certainly. Then a reviewer comes in&mdash;a mate we always have geeky discussions with. And it begins. First frown, next one, puzzlement. A quarrel is approaching and what if we're not used to leveraging emotional intelligence? What if potential arguments devour review benefits like correctness check, rational design, backward compatibility etc.

Company coding guidelines or `CONTRIBUTING.md#CodingStyle` to the rescue! We create such as we generally believe they should guard code quality, tame wild emotions and alleviate the effect of rush of blood to developer heads. Let's look at some properties of development that are common in most of companies:

1. There is always **a difference in subjective opinions**: what is better, cleaner etc. We need to agree on some common mindset in our team or company (providing we don't work alone). This is particularly important in complex scenarios.
2. It is generally easy to detect some **obvious, repeatable errors**.
3. While reviewing work, we also look at **quantifiable metrics** like: code coverage, code structural complexity, number of dependencies etc.

The problem remains: how to do it effectively? In other words, how to make the process automatic or how to automatically feed developers with hard data. The data that would help them break ties, make tradeoffs and find correct decisions in less amount of time.

# Are your codes neat and clean?

Pun intended. That was in fact a real question in a job offer sent by some recruitment agency. Its author probably consulted a programmer who shared some fancy phrases to lure a good candidate. It is also quite possible that they were familiar with the book [*Clean Code* by Uncle Bob (Robert C. Martin)][book-cleancode]. It worked and it attracted attention. And caused amusement. I remember my colleagues discussing the offer (they also had got it) at morning coffee. Together we made three observations:

1. It is very likely that there is at least one programmer in that company who reads professional software books. Good, valuable literature. And this is a great attitude. Unfortunately, [the vast majority of us don’t read][we-dont-read].
2. *Neat* and *clean* are adjectives expressing a subjective opinion. They usually denote various things for developers. And there are many more related to personal taste. Agreeing upon some consistent convention in our team/company is vital.
3.	*Neat* and *clean* cannot be objectively measured nor evaluated. As far as code length is concerned it can be measured by the number of lines and decided whether it is short or long. However, there is no particular method that helps you analyze the cleanness of code. Thus, we need to find a quantifiable approach to measure and control properties of code quality.

To each his own? Or rather a trustworthy and objective approach to that problem exists?

# Agree, measure, control - for the win!

There are [standard quality measures which have been established over the years][book-measurement]. So are techniques of reaching the consensus within a team. Majority of us have already heard about them and some can even name and describe a few. In *SonarQube Series* I would like to pay special attention to the measurement process which is often undervalued. Let's cheer this endeavour with the following quotes:

> "You cannot control something that you cannot measure."
>
> [T. DeMarco and T. Lister, Peopleware][book-peopleware]

> For every attribute of a project there is a way of measuring it which is better than not measuring it at all.
>
> [T. DeMarco, Controlling Software Projects][book-controlling]

One needs to be careful here though, as the following is also true:

> What is measured, it is done.
> 
> [S. McConnell, Code Complete][book-codecomplete].

Till now it should be rather obvious that measuring properly is crucial for the control to happen. Every day we have an insight into a patient’s medical history. Therefore we can evaluate their state correctly and make adequate actions to either remedy issues (if they suffer from disease) or improve weak areas (if they pursue sport excellence).

This is the place where SonarQube (aka Sonar) enters the stage&mdash;open source software which is actively developed and supported by a Swiss company [SonarSource SA][sonarsource]. It aims at detecting quality rules infringements that have been agreed upon. But not only, as it calculates various code quality metrics, records their history and visualizes in a number of charts. It benefits from client-server&mdash;consistent management at the team level or the whole organization. Moreover, its capabilities can be easily extended with plethora of built-in plugins (3rd party or developed in-house). Well, having all that in mind, let's see what SonarCube can do for us.

# Next

This article is meant for a short introduction only. Now, let's dive into practical use and its benefits.

[The next post][next] of this series explains in details how to setup SonarQube analysis in Windows/.NET stack.

If you want to examine the analysis results right away, then jump directly to [SonarQube Series #3: Crushing Jon Skeet's code][next-results].

[next]:             /software/2018-10-10-sonarqube-2-setup-environment "SonarQube #2 Installation of the SonarQube server and project configuration"
[next-results]:     /software/2018-10-10-sonarqube-3-jon-skeets-libs-analysis-results "SonarQube #3 - Analysis results of Jon Skeet's libraries"

[book-cleancode]:   https://amzn.to/2oVLQmD "Martin, R.C., Clean Code"
[we-dont-read]:     https://blog.codinghorror.com/programmers-dont-read-books-but-you-should/ "Jeff Atwood on why we don't read professional literature"
[book-measurement]: https://amzn.to/2O4tr1N "C. Jones, Applied Software Measurement: Assuring Productivity and Quality, New York: McGraw-Hill, 1997"
[book-peopleware]:  https://amzn.to/2PbAaYc "T. DeMarco and T. Lister, Peopleware: Productive Projects and Teams, Addison-Wesley Professional, 2013."
[book-controlling]: https://amzn.to/2ydTPj2 "T. DeMarco, Controlling Software Projects: Management, Measurement, and Estimates, Prentice Hall, 1986."
[book-codecomplete]:https://amzn.to/2CseeVV "S. McConnell, Code Complete: A Practical Handbook of Software Construction, Microsoft Press, 2004."
[sonarsource]:      https://www.sonarsource.com/ "SonarSource - the company that is responsible for SonarQube development."