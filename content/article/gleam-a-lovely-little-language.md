---
title: "Gleam a lovely little language"
date: 2025-11-19T09:00:00+02:00
description:
  This is the story...
---

A few months back I stumbled upon this language called [Gleam](https://gleam.run/). I have been
playing around with it since and I have really been enjoying myself, it is a lovely little
language!

I have been enjoying it so much that I have gone up early{{< note 1 >}} during the weekends to
get some quality coding time before the rest family wakes up.
{{< sidenote >}}
1. I have reached that age where it hurts less to get up early than to stay up too late. Also
my kids are teenagers, so getting up a few hours before them it not really that hard.
{{< /sidenote >}}

So what is it that makes this languange so plesant to work with? I think it is a combination
of things, and of course, personal preference. Before we dive in let's see what some Gleam
code looks{{< note 2 >}} like!
{{< sidenote >}}
2. Mind you that I am not a Gleam expert! This piece of code might not be idiomatic, or the most
efficient way of doing this. But I think it serves as a good example of Gleams capabilites and
syntax.
{{< /sidenote >}}

```gleam
/// Try to identify the content-type for the given path.
/// Defaults to application/octet-stream.
pub fn identify_content_type(path: String) -> String {
  let seq =
    path
    |> string.split(on: ".")
    |> list.reverse()
    |> list.map(string.trim)

  case seq {
    ["html", ..] -> "text/html"
    ["css", ..] -> "text/css"
    ["js", ..] -> "text/javascript"
    _ -> "application/octet-stream"
  }
}
```

I am not going to step through that example just yet! It was an appetizer. If you think that
the code looks awful feel free to close this tab and move on, the Internet might have something
else for you up the road.

## My road to Gleam

What drew me to Gleam in the first place was actually as an alternative to [Elm](https://elm-lang.org/)!
I have been lurking around in the Elm community for a while and I really like The Elm Architecture
{{< note 3 >}}, the philosophy behind the language, etc. But, Elm is an ML-family language and the
syntax never really clicked with me.
{{< sidenote >}}
3. I think that Evan and the Elm community made a real dent in the programming world. The Elm
architecture, the friendly compiler errors, the focus on correctness, etc. If you have not checked
it out, I highly recommend you to have a look at it.
{{< /sidenote >}}

Scala, is another language I like (while influenced by ML, I find that the Scala syntax is more to
my liking{{< note 4 >}}).
{{< sidenote >}}
4. I know that some people argue that syntax is superficial and that the semantics is what matters.
   To me, both matters a great deal.
{{< /sidenote >}}

One of Scala's strengths is that it allows you to mix functional programming with object-oriented
programming. Having bee. But it also allows me to switch to OOP when FP does not come naturally.

In Gleam, there is no OOP, and the language shares some of the features I enjoy in Scala, like
pattern-matching, expressions, result types, etc.

The thing that pushed me over the edge was [Lustre](https://github.com/lustre-labs/lustre). An
Elm-inspired web framework for Gleam.

## Gleam itself

As I mentioned, Gleam is a small language. It has few reserved keywords, a concise syntax and only
a small number of language features.

Gleam compiles to Erlang{{< note 5 >}}, and is run on the BEAM virtual machine. Gleam also compiles
to JavaScript which then can run either in the browser or usin a JavaScript runtime such as Node
or Bun.
{{< sidenote >}}
5. Funny thing regarding being superficial and syntax. I just cannot stand the Erlang syntax! I have
been interested in Erlang and BEAM for a long time, but the syntax has always kept me away.
{{< /sidenote >}}

Here is my way of describing Gleam:

- **Functional language**, immutable data structures and mostly pure functions.
- **Type-safe**, types, exhaustive checks, no nulls or exceptions.
- **Concurrent**, built on top of BEAM and OTP.
- **Simple**, small grammar, no type classes, etc.
- **Fun**, having fun with functions!

Two things that initially stands out when you start working with Gleam is the _lack of_
`if` statements, and the _absence of_ loops.

### What, there is no if?

No, it has something much better, expressions and pattern-matching!

```gleam
let number: Int = 33

let result = case number {
  x if x > 10 -> "large"
  _ -> "small"
}

io.println("The number is " <> result)
```

Here the case expression will match the value of `number` against the patterns in each branch. The
first branch binds the value of `number` to `x` which is used to evaluate the expression `> 10`
(also known as a _guard expression_).

The second branch use the wildcard pattern `_` to match any value, in this case we know that the
value must be less than 10.

As the entire case is an expression, the result is assigned to the variable `result`, which we
later use to print{{< note 99 >}} out the result.
{{< sidenote >}}
99. <> is the string concatenation operator in Gleam.
{{< /sidenote >}}

The case expression is how you do conditional branching in Gleam, it is both powerful and safe. The
compiler forces you to cover all possible cases, and warns you if there is any unreachable code.

In the first example, we used the case expression to match against a list of strings:

```gleam
case seq {
  ["html", ..] -> "text/html"
  ["css", ..] -> "text/css"
  ["js", ..] -> "text/javascript"
  _ -> "application/octet-stream"
}
```

Here the variable `seq` is a list of strings, let's say that the value if seq is
`let seq = ["js", "example/filename"]`. That will match the third branchm, as it match any list
that has "js" as the first element and any number of other elements (the `..` syntax, similar
to what sometimes called _tail_ in other languages).

Matching lists can be more advanced as well:

```gleam
let result = case lst {
  [] -> "List is empty"
  [_] -> "List have exactly one item"
  [_, _] -> "List has exactly two items"
  [_, _, ..] -> "List has at least three items"
}
```

It takes some getting used to, to only use case expressions. Sometimes you have something
you want to add or perform based on a condition. Then you come up with what the "else branch"
should return. Often you fall back on an empty list, or a representation of none.

For example, in one of my hobby applications I have a menu that might have an extra line of
information.

```gleam
fn dropdown_item(text: String, appendix: Option(String)) {
  let appendix_elm = case appendix {
    option.Some(t) -> html.div([], [html.text(t)])
    option.None -> element.none()
  }

  html.div([], [
    html.div([], [html.text(text)]),
    appendix_elm,
  ])
}
```

Don't worry too much about the HTML stuff, that is just{{< note 99 >}} Lustre's way of producing HTML.
{{< sidenote >}}
99. "just" is not really fair, working with HTLM as code rather than a template is very powerful,
and is now my preferred way. I hope to describe how well this compose in a future article.
{{< /sidenote >}}
Here we either create an DIV element with one or two child elements. The first is a DIV containing The
`text` argument, and the other is either another DIV with the appendix text, or none.

While a few more lines of code, I find this code to more clearly communicate what is happening.

### Loops, what loops?

That's right, there are no loops in Gleam. The `list` module contains the usual `fold`, `map`, and
even an `each` function. Those are you options, so for to sum the values in a list you can fold
over it:

```gleam
fn sum(lst: List(Int)) -> Int {
  list.fold(lst, 0, fn(acc, v) { acc + v })
}
```

If you want to calculate the factorial value of an integer you have to use recursion. In languages
like JavaScript or Python you could have done this either using recurion or by using a for-loop. In
Gleam, there is only recursion.

```gleam
pub fn factorial(x: Int) -> Int {
  case x {
    0 -> 1
    1 -> 1
    _ -> x * factorial(x - 1)
  }
}
```

If you are unfamiliar with recursion (I was a bit rusty myself, due to mainly programming in C# at work),
the source code of the Gleam standard library is easy to read and understand. They often have an internal
function named with a `_loop` suffix that hides the accumulator. E.g. [`list.take`](https://github.com/gleam-lang/stdlib/blob/main/src/gleam/list.gleam#L614)

### Types and records

You have built in types, custom types and records in Gleam. They all share the same construct.

You can alias a type{{< note 99 >}} like this.
{{< sidenote >}}
99. TBD
{{< /sidenote >}}
```gleam
type Number =
  Int
```

A type can have different variants:

```gleam
// Season is a type
pub type Season {
  // Spring is a variant of that type
  Spring
  Summer
  Autumn
  Winter
}
```

A type can be a record, carrying data:

```gleam
// A record is a type that can hold data
pub type User {
  // Convention is to use the same name for type and variant.
  User(name: String)
}
```


## Toolchain

TODO.

## Community

I have not interacted with the Gleam community that much yet. I am an introvert that tends to
observe rather than participate in the beginning. However, my impression from the side of the
road is that the community is helpful and friendly. I have not observed any elitist
Monad-warriors seen in other communities.{{< note 99>}}
{{< sidenote >}}
99. Parts of the Scala community suffers from this unfortunately (although it has become better!).
{{< /sidenote >}}.

There is the subreddit [/r/gleamlang](https://www.reddit.com/r/gleamlang/) which is fairly
quiet. And there is the much more active [Discord](https://gleam.run/community/) where I mainly
read the "questions" and "sharing" topics.

## Getting started

If you want to try Gleam for yourself, I recommend you to head over to and read through the
[Gleam language tour](https://tour.gleam.run/) and start to build something!

TODO Describe what I am currently working on.
