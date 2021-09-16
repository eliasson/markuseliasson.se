+++
date = 2021-09-16T13:00:00+02:00
title = "Technical debt"
description = "What is technical debt, who owns it and when is it due? Post #2 in a series of post about obstacles around software development."
tags = ["Software Development", "Obstacles"]
draft = true
+++
Have you ever experienced a full stop on new features due to the team working on technical debt?

What is technical debt, who owns it and when is it due? Ask that question to a development team and you will likely get vague answers, something about architecture, slow development time and guaranteed - lack of unit-tests.

The truth is that **there is no such thing as technical debt!**

You either have a problem or you do not. Everything else is speculation.

If I were to be cynical, I would say that some developers use the term technical debt as a veto in discussions with project managers or product managers. If you are not technically oriented in the system it is hard, near impossible, to separate what is actually a problem from what is a desire to use the latest technology.

Now, I am not saying that developers are playing the technical debt card out of evil. They are most likely blinded by their relation to the code, they want to improve it in some way. If a piece of code is terrible to work with, but you only do it very rarely, is it worth fixing? Remember the ingress, that we alone choose where to spend our time and energy? If that piece of code really is blocking revenue, fix it! But, if there are other more important areas that would benefit more from the same effort I would choose the one with the greatest customer value.

- Technical debt is a vague term, define concrete problems if any
- It is not what you choose to do, it is what you choose not to do
- Use techniques as Test-Driven Development and pair programming to avoid the situation in first place

{{% post-scriptum %}}
This was the second part of a series of articles, you can find the first one here:
- [Your obstacles are YOUR obstacles](/article/your-obstacles-are-your-obstacles/)
{{% /post-scriptum %}}
