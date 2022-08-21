+++
date = 2022-08-19T13:00:00+02:00
title = "The fear of releasing"
description = "The fear of releasing is real, it is the fear of failure."
tags = ["Software Development", "Obstacles"]
+++
> This is the third in a series of articles that claim that the obstacles you have in your organisation are put there by the very same organisation, yours. And it is up to your organisation to remove these obstacles, no one else will do it for you. [Read part one here](/article/your-obstacles-are-your-obstacles/).

The fear of releasing is real, it is the fear of failure. Will this feature work as intended? Will the on-call team be woken up in the middle of the night? Maybe we should have added this other feature before releasing as well?

I think fear of breaking is easiest to remedy. Write good tests, use practices as Test-Driven Development, pair-programming and close collaboration with domain experts to increase the quality of software. These alone will not produce bug free software, but I still see too many organizations that simply do not write tests and then struggle hard to change the system, and it is riddled with bugs.

If releasing a feature causes instability or service disruptions I would argue that the best way to improve is to release more often, making small improvements each time. Preferably you should automate as much as reasonably possible in order to make future releases cheaper and less error prone.

Your development process will also benefit from incremental improvements. If your software is lacking automated tests, start by adding tests when bugs appear. Then at least each bug will only appear once. If you implement a new feature, try writing the test first and see if that makes things better.

Fearing wrong or insufficient features is not something that you can solve on your own. You need to communicate with your end users and gather their feedback on what changes are needed in order to solve their needs as efficiently as possible.

There is another perspective on insufficient features as well, that is not a fear - but should be one. The _“all or nothing”_ behavior. Developers and product owners alike often assume that a feature can under no circumstances be released until it is 100% complete. This is not limited to polish or gold-plating but rather a misconception that a feature is not useful until it is complete.

Not too long ago, I was part of a project where we were tasked to build a HTTP API. Initially, only a handful of API:s were needed to solve the problem at hand. After a lengthy discussion, mainly about level of ambition, one of the developers shouts out:

> “if it is worth doing, it is worth doing well!”.

The person argued that the HTTP API must support all operations (querying, filtering, etc), else it was of no use.

That is true about many things, e.g. laying the foundation to a new house, or building a bridge. Software on the other hand has this incredible attribute that it can always be changed and adapted along the way, when more facts are known. So, instead of covering all possible needs too early, embrace the nature of software and handle each need first when required. I would rather say:

> “If it turns out worth doing, it is worth doing well”

Sometimes the benefit of frequent releases is pitched as a risk mitigation strategy. While this is true, it might not be valued as high by product managers or business people as by the development team. I can think of at least two more important attributes of releasing frequently.

**Ensure you are on the right path** - what if the feature you’re building is something your end-users would prefer functioned in a different way? Or even worse, something they don’t even need? Releasing often will shorten your feedback loop helping you to minimize the risk of waste and to fulfill the customer’s needs sooner.

**Monetize as soon as possible** - instead of releasing a feature once it is complete, try shifting views and express what needs to be done with the suffix _“…before we can monetize”_. Many features bring value to users long before they are fully complete, and who knows, maybe it turns out that the last set of functions was not all that important after all.

Many teams still consider bi-weekly releases frequent. Given a team of five people, this is in the best of worlds 400 hours worth of investments that are waiting to be monetized. And in the worst case 400 hours worth of waste. Do not be afraid to release, be afraid to not release.

To summarize:

- If you fear stability issues or bugs, focus on your development practice
- Consider a feature the smallest thing that can possibly work and release that
- Use early user feedback to adjust and prioritize your backlog

_These ideas are not new, most of this is covered by Extreme Programming (XP). I highly recommend you to go read up on XP, although it’s more than 20 years old it is a breath of fresh air in today's software development._

{{% post-scriptum %}}
This was the third part of a series of articles, you can find the other ones here:
- [Your obstacles are YOUR obstacles](/article/your-obstacles-are-your-obstacles/)
- [Technical debt](/article/technical-debt/)
{{% /post-scriptum %}}
