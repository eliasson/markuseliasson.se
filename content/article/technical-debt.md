+++
date = 2021-09-23T10:00:00+02:00
title = "Technical debt"
description = "What is technical debt, who owns it and when is it due? Post #2 in a series of post about obstacles around software development."
tags = ["Software Development", "Obstacles"]
+++
> This is the second in a series of articles that claim that the obstacles you have in your organisation are put there by the very same organisation, yours. And it is up to your organisation to remove these obstacles, no one else will do it for you. [Part one.](/article/your-obstacles-are-your-obstacles/)

Have you ever experienced a full stop on new features due to the team being busy working on technical debt?

What is technical debt, who owns it and when is it due? Ask that question to a development team and you will likely get vague answers, something about architecture, slow development time, lack of unit-tests or that the code is too old.

The truth is that **there is no such thing as technical debt!**

You either have a problem or you do not. Everything else is speculation.

## Veto

If I were to be cynical, I would say that some developers use the term technical debt as a veto in discussions with project managers or product managers. If you are not technically oriented in the system it is hard, near impossible, to separate what is actually a problem from what is a desire to use the latest technology or to polish something into perfection.

No, I am not saying that developers are playing the technical debt card out of malice. They are most likely blinded by their relation to the code, they want to improve it in some way. If a piece of code is terrible to work with, but you only do it very rarely, is it worth fixing?

Remember the [basic premise](/article/your-obstacles-are-your-obstacles/), that we alone choose where to spend our time and energy? If that piece of code really is blocking progress, fix it! But, if there are other more important areas that would benefit more from the same effort I would choose the one with the greatest customer value.

## Stuck with bad code?

Sure, you might be stuck with a codebase that is terrible and that stops you from making changes in a steady and safe manner. Calling it technical debt in such a situation is in my experience not productive. Technical debt tends to be rooted in emotions and is overly focused on code - not why that code came to be that way.

There are many reasons for why a codebase is in a bad shape, here are a few reasons I have identified during my years as a developer.

**Overcomplicating things** - there are many reasons for why developers overcomplicate things. I would say that most often it is about trying to predict the future. Adding variations, features or options that will enable future use-cases.

Predicting the future however is super-hard and you are better off making your code robust to future changes (unit-tests, boundaries, etc) rather than adapting the code to potential changes too soon.

**Bad test-coverage** - some of the systems I have worked with did not have any unit-tests, some had a few unit-tests and a few of them had many unit-tests.

Needless to say, the systems with more unit-tests were _a lot_ simpler to change and grow with new features. Not having good test coverage is either _extremely risky_ or _extremely slow_ - either you hope your change is OK and then you ship it, or you need extensive manual testing.

If your system is lacking good test-coverage, Test-Driven Development is an established technique that not only gives excellent test coverage but also improves the code in general.

**Lack of understanding** - building the wrong thing is not only expensive in time and resources. The cost of having the wrong solution or abstractions in the code base should not be ignored.

A developer obviously needs to understand the existing design and code. This is most likely the easy part, most developers are used to working in different codebases and have their colleagues close by when in doubt. The other part of understanding is to understand what to build and why. If this is not clear there is a great risk that what is built is not solving the right problem. The developers need to collaborate with key stakeholders for a feature, this is crucial. The further away you keep your developers, the further away you put your chance of success!

**Stress** - is an obvious factor, people perform badly under too much stress, regardless of profession. Stress leads to mistakes, ignoring code-hygiene such as unit-tests or not taking the time to understand the task at hand.

**Lack of knowledge** - sometimes the person(s) who wrote a piece of code might just not have known better. This can be due to the person not having the required technical knowledge or domain knowledge. That kind of problem can often be mitigated with training, coaching and pair-programming. Other times it is a lack of interest or motivation which require different actions.

Some people have pointed out the case where the team takes a shortcut to accomplish some goal. Later it turns out that the shortcut is causing problems, making it harder to continue developing features. I still do not think that is technical debt. The shortcut should have been made by weighing its pros and cons and the decision to take the shortcut was probably the correct decision at that time. Maybe the decision was needed due to a closing market window and implementing the feature right would have missed that window? You now have the possibility to correct the shortcut, it should be easy to describe the problem more specifically and it should be easier to indicate the value of such a fix.

## Cause and effect

As you can see far from every reason originates from code, many of them span across the organisation. What is common for all these reasons however is that they manifest themselves in the code. Hence it is easy to label these problems as _technical debt_. The developers have the responsibility of the code, they should make sure it is correct and in overall good shape, but it is naive to think that it solely lies within the developers domain. Hence, to find and correct these problems you sometimes must broaden your view to include a larger part of organisation to find the root cause.

What you need to do is to find your bottlenecks to increase customer value. Find out ways to improve that, label it and prioritize it as any other work. If you do not name and describe work correctly, how can you know that you are working on the correct task?

To summarize:

- Technical debt is a vague term, define concrete tasks and prioritize accordingly
- Use techniques as Test-Driven Development and pair programming to improve the code quality in the first place
- Collaborate and make sure features are well understood
