---
title: "Don’t round twice"
date: 2025-02-22T09:00:00+02:00
description:
  When debugging what seemed to be a rounding error, I learned a surprising thing about the way .NET
  round decimal values.
---

The issue I was facing was that I had a list of _things_ where each of these things had a number of properties
that I wanted to group on. Let’s say that these things were liquorice bars with fillings{{< note 1 >}}.
{{< sidenote >}}
1. These are commonly found at the cash register in Swedish supermarkets. I highly recommend the Pingvinstång,
with a filling of mint!
{{< /sidenote >}}


Such bars have many properties{{< note 2 >}}, but to illustrate my problem let us say that they have _diameter_
and the _type of filling_.
{{< sidenote >}}
2. Obviously, the "bar" had a lot more details attached to it, but I spared you those
details.
{{< /sidenote >}}


```csharp
record class Bar(decimal Diameter, string Filling)
```

The task was to group these bars based on their diameter and filling to better understand what sells the most.
Then take that list and save it elsewhere. When saving, the diameter should be rounded to _1 decimal_ which was
decided to be precise enought for our use-case.

I choose to implement a key that represented the unique properties to group on as follows:

```csharp
record class Bar(decimal Diameter, string Filling)
{
	string Key() => $"{Diameter:F1}_{Filling}";
}
```

There was a DTO for this bar as well, looking something like this:

```csharp
public class BarDto(uint count, Bar bar)
{
	public uint Count;
	public readonly string Filling = bar.Filling;
	public readonly decimal Diameter = Math.Round(bar.Diameter(), 1);
}
```


## The problem

The problem was that when I looked in the saved bars. There were duplicate rows where I expected none!

```
| Count | Diameter | Filling   |
|-------|----------|-----------|
|  12   | 11.0     | Mint      |
|   1   | 11.0     | Mint      |
|  10   | 12.5     | Lemon     |
```

Why are there two rows of mint liquorice bars with the same diameter!?

I looked closer at the data used. Examples of the diameters used were `11.0`, `11.04`, `11.05`, and `11.30`.

> Measure twice - cut once

Most of you have heard the expression _measure twice - cut once_. Whenever I do woodworking I measure, at least,
twice before I cut anything - something I have learned the hard way.

Another thing{{< note 3 >}} I have learnt that the measuring has to be performed by the same person and using the
same instrument. Me and my wife tend to get slightly different parallax errors when measuring, and different rulers
can yield different results.
{{< sidenote >}}
3. Yet another thing I have learnt is to STOP working when you are tired. When you, late in the
evening, miter cut the trim for a corner wrong the second time, you will not get it right the third time.
{{< /sidenote >}}

Turns out that the same thing was happening here. I rounded the decimal value twice, but I used different
instruments when doing so! And apparently C# implements this differently when using string interpolation and
when using `Math.Round`!

I wrote a quick test to better understand.

```csharp
[Test]
public void It_should_round_consistent()
{
    var offendingValue = 11.05m;

    Assert.Multiple(() =>
    {
        Assert.That($"{offendingValue:F1}", Is.EqualTo("11.1"));
        Assert.That($"{Math.Round(offendingValue, 1)}", Is.EqualTo("11.1"));
    });
}
```

The last assertion failed, `11.05` was rounded to `11.0` when using `Math.Round`. And to `11.1` when using `$”{11.5:F1}”`.

`Math.Round` is clear in [its documentation](https://learn.microsoft.com/en-us/dotnet/api/system.math.round?view=net-9.0)
that it uses what is called _bankers rounding_. It rounds midpoint values to the nearest even number.

While the string interpolation use C# number format strings, which states in
[its documentation](https://learn.microsoft.com/en-us/dotnet/standard/base-types/standard-numeric-format-strings) that it
use _fixed-point rounding_.


## The mistakes made

The API documentation for `Math.Round` describes how it rounds, and there are ways to control the algorithm used. But I did
not think of that, as the algorithm used was irrelevant for my use case, as long as it was consistent - which it was not.
**Assumption** is, once again, the mother of all fuck ups.

**Test data weakness**, this code was under test already. Tests that ensured that the values were rounded. But there was no test
with a midpoint decimal value (there is now!).

Not using the rounded value, but to **round it again**. The fix I made was to round the value once{{< note 4 >}}, and then use
that value in both places, which in hindsight, is what I should have done in the first place.
{{< sidenote >}}
4. Unlike humans, computers have very good memory and do not self
doubt their previous measurement.
{{< /sidenote >}}


```csharp
record class Bar(decimal Diameter, string Filling)
{
	string Key() => $"{DiameterRounded:F1}_{Filling}";
	decimal DiameterRounded() => Math.Round(Diameter, 1);
}

public class BarDto(uint count, Bar bar)
{
	public uint Count;
	public readonly string Filling = bar.Filling;
	public readonly decimal Diameter= thing.DiameterRounded;
}
```

So for programming, I guess the saying goes:

> Round once - use twice!
