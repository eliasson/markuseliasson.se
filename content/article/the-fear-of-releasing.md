+++
date = 2021-09-16T13:00:00+02:00
title = "The fear of releasing"
description = "The fear of releasing is real, it is the fear of failure. Post #3 in a series of post about obstacles around software development."
tags = ["Software Development", "Obstacles"]
draft = true
+++
The fear of releasing is real, it is the fear of failure. Will this feature work as intended? Will the on-call team be woken up in the middle of the night? Maybe we should have added this other feature before releasing as well?

Fear of breaking is easiest to remedy. Write good tests, use practices as Test-Driven Development, pair-programming and close collaboration with domain experts to increase the quality of software. These alone will not produce bug free software, but I still see too many organizations that simply do not write tests and then struggle hard to change the system, and it is riddled with bugs.

I think the other fear, the fear of insufficient features is much harder to overcome.

Not too long ago, I was part of a project where we were tasked to build a HTTP API. Initially, only a handful of API:s were needed to solve the problem at hand. After a lengthy discussion, mainly about level of ambition, one of the developers shouts out

> “if it is worth doing, it is worth doing well!”.

That is true about many things, e.g. laying the foundation to a new house, or building a bridge. Software on the other hand has this incredible attribute that it can always be changed and adapted along the way, when more is known. So, instead of covering all possible needs too early, embrace the nature of software and handle it first when needed. I would rather say

> “If it turns out worth doing, it is worth doing well”

Sometimes the benefit of frequent releases is pitched as a risk mitigation strategy. This is true, but might not be valued as high by product managers or business people as by the development team. Instead, try shifting views and express what needs to be done with the suffix “...before we can monetize”, that will help put things in perspective.

Many teams still consider bi-weekly releases frequent. Given a team of five people, that is still 400 hours worth of investments that are waiting to be monetized.

- Automatic tests are a must to release often
- Do not be afraid to release, be afraid to not release
- Release early and often

{{% post-scriptum %}}
This was the third part of a series of articles, you can find the other ones here:
- [Your obstacles are YOUR obstacles](/article/your-obstacles-are-your-obstacles/)
- [Technical debt](/article/technical-debt/)
{{% /post-scriptum %}}
