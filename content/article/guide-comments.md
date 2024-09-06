---
title: "Guide Comments"
date: 2024-09-06T16:56:00+02:00
description: Code comments is a subjective topic in a world of collaboration. What is superfluous for some might be
  helpful for others. Personally, I come to like something referred to as guide comments.
---

A few months ago I stumbled upon an [article](http://antirez.com/news/124) about code comments and documentation. The
article was written by none other than _antirez_, the creator of Redis, and a well known{{< note 1 >}} C programmer.

There is one recommendation in his article that surprised me, it was the use of what he calls **“Guide comments”**.
These  are comments that do not add any new information, they just state what the code is doing.
{{< sidenote >}}
1. Antirez has a reputation for writing well written and easy to understand code. I remember that I was
   impressed by his toy text editor [Kilo](https://github.com/antirez/kilo) in just 1 000 lines of C.
{{< /sidenote >}}

Below is an excerpt from a time-reporting tool that deals with merging adjacent and overlapping slots of registered
time. The comments do not add any details that are not part of the if-conditions, yet they assist the reader in
interpreting the code.

```csharp
if (current.Offset >= slot.Offset && current.EndsAt <= slot.EndsAt)
{
    // Completely overlaps (i.e. replaces)
    drop = true;
}
else if (current.EndsAt == slot.Offset && isSameActivity)
{
    // Just before (i.e. merge)
    slot.Offset = current.Offset;
    slot.Duration += current.Duration;
    drop = true;
}
else if (current.Offset == slot.EndsAt && isSameActivity)
{
    // Just after (i.e. merge)
    slot.Duration += current.Duration;
    drop = true;
}
else if (current.EndsAt > slot.Offset && current.EndsAt <= slot.EndsAt)
{
    // Slot overlaps tail part of current
    if (isSameActivity)
    {
        // Merge into one slot
        var delta = Math.Abs(slot.Offset - current.EndsAt);
        slot.Duration = slot.Duration + (current.Duration - delta);
        slot.Offset = current.Offset;
        drop = true;
    }
    else
    {
        // Shrink the current (old) slot
        var delta = Math.Abs(slot.Offset - current.EndsAt);
        current.Duration -= delta;
    }
}
```

Younger me would have dismissed this recommendation as too trivial, code should be self documenting and all that 🤦.
But I like them, and find them valuable. I just hadn’t thought of them as a guide.

The last few months one of the teams I work with have started using these types of comments more frequently (when it
has a name it is easier to discuss and adopt). And I think they are great and that more people should use them!

When I write code, they help me communicate my intention, sometimes I even write them before I write the code. They are
also a good way to highlight sections{{< note 2 >}} of a function, especially test setups.
{{< sidenote >}}
2. There was a misconception during the Clean Code era that all such sections should be extracted to separate
functions. Luckily we are not as dogmatic today.
{{< /sidenote >}}

The greatest benefit however is when I read code. It really is like having a guide, it gives me confirmation that I
have read and interpreted the code correctly. Or allows me to skim some sections that are less relevant.

> Code is written once but read many times.

Given how often code is read, I think it deserves a narrator. Do not be afraid to state the obvious, it might help
future you and your teammates to better understand and navigate your system.
