+++
date = 2021-09-16T15:00:00+02:00
title = "Your obstacles are YOUR obstacles"
description = "Why do we have obstacles in software development when software is almost free from constraints?"
tags = ["Software Development", "Obstacles"]
draft = false
+++

Software is magic, there are few physical limitations on what can be done. A change can be made quickly and its benefits are immediate. The same is true for how we work with developing software, there are very few rules or obstacles that stop us from doing what is required as swift and efficiently as possible.

Looking at the pre-conditions, we have the **speed of light** and we have **24 hours per day**, if we are strict those are the only ones. Surprisingly enough, we still keep hitting obstacles on our way forward, slowing us down or even stopping us from doing what we need done. 

Let us assume another pre-condition, you and your organization are free to choose how you work and how your software is built? **Then it is safe to assume that your organization is the one having created those obstacles you keep hitting.**

Each and every one of us share these pre-conditions, how you choose to spend your time is likely to determine the outcome of your next project.

During my years as consultant I have been fortunate enough to work with a lot of great customers, many of whom call them self agile and modern thinking. Still many of them have struggled with obstacles they easily could have removed. 

This will be a series of small articles detailing some of the types of obstacles that I have encountered and how I think you should handle them.

# The imaginary scale

In the day and age where the large IT companies are the ones that establish trends, frameworks, and talk at conferences about their challenges with massive scale - it is easy to get carried away.

Twitter deals with c:a 500 million tweets per day and has about 100 million daily users. Facebook has 2.3 billion users. Do you share the same situation as they do? Probably not.

Are you using microservices? An architecture tailored to scale the organization as well as individual services based on individual load. It has been a popular architecture for the last 5-10 years much thanks to Netflix, Google, Amazon, etc that has built tools and platforms to deal with these complex systems.

Distributed systems are an order of magnitude more complex than monolithic systems. Still many organizations are using them as their goto solution. Why? 

A few years ago I was hired as a technical lead for a team of developers. Our task, build a “microservice platform”. This was a requirement, straight from the business people I tell you! Some people referred to it as a “Marketecture”. To be fair, there were other requirements too, but the need for microservices was real.

Unfortunately that project is far from being alone. Large and complex systems lure developers like sirens. Totally blindsighted from the additional cost, too many organizations start with an overly complicated system when their needs are fairly modest.

A small system has many benefits over a larger one:
- Fewer people involved (on all levels)
- Fewer lines of code
- Fewer moving parts

These attributes all result in fewer bugs and a higher velocity. So not only will you deliver better quality software, you get a chance to do that more often. This allows you to get insight from the actual customer value and to adjust and reiterate faster. That is a pretty attractive situation for most organizations!

To scale up a system typically gives the exact opposite characteristics. A situation that quickly becomes an obstacle.

Most people know this, I would say it is pretty rational after all. Decisions, however, are often not rational but emotional. I believe that in many organizations there is a clear relationship between status and the size of departments or teams. The more people you are in charge of, the higher official or unofficial status will be granted to you. This is equally true for architects and engineers - the bigger the system the higher status.

Do not base your architecture on cult or office-politics, base it on its effect. If you are building an unnecessarily complex system, you are wasting time and effort that could have been better invested elsewhere. 

Most likely there are only a few parts of your architecture that needs to scale or perform differently from the rest. Identify these components and how to scale them without splitting your entire system up into pieces.

To summarize:

- Find your bottleneck and resolve it, repeat
- Optimize where the effect is the greatest
- Do not distribute more than necessary

{{% post-scriptum %}}
This was the first post in a series of posts. Next week, I will tackle [technical debt.](/article/technical-debt/)
{{% /post-scriptum %}}