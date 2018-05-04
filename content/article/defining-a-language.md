+++
Categories = ["Software Development", "Code"]
Description = "Defining a language requires you first need to consider the semantics and then the syntax. Even for a tiny language such as Sequence it is an iterative process where the definition was changed multiple times early on."
Tags = ["Software Development", "Sequence", "DSL"]
date = "2018-04-19T14:55:00+01:00"
title = "Defining a language"
draft = true
+++


When discussing languages, especially computer language, I have noticed that people often start
with expressing their view on syntax and grammar.  Just like it is easy to have opinions about
the appearance of an application, they are both _user interfaces_. What is far more important
is the semantics, what the language allows to be expressed. I do not think look and feel or syntax
is superficial - it is still very important, but second to semantics.

### Start with the semantics

When I drafted the Sequence language I sat down and tried to express sequence diagrams
as natural as I could. I tried hard to completely ignored technical aspects, like complexity
of parsing or the "correctness". As this is a small language and one of the use-cases is to
describe the process in which it was developed, I tried to not overthink or over-engineer the
language.

What I knew I wanted:

* **Fluent language** - I wanted anyone be able to read and understand the sequence of messages
* **Focused** - I did not want to support all use-cases for sequences, all I wanted was to cover the basic situations
* **Not UML** - I had no ambition to fulfil UML or any other notation, unless it made sense to do so

This was one of the first drafts I came up with:

```
Actor Alice:
    Alices description in plain text

System Acme

Sequence Login:
    Alice -> Acme: Make me a sandwich
    Acme -> Acme: Housekeeping work
        Indentend block under a single interaction becomes a
        description for that interaction.
```

First off, I wanted to have a global definition of Actors and Systems, clearly defining any participant
taking place in the upcoming message exchange. Participants should have an optional description to allow
for shorter names, but still allow for a detailed description within the document.

Next, I think having multiple sequences in a single file, and later in a single diagram is quite useful
for describing alternative or very similar sequences involving the same participants.

Each sequence should then define the list of messages containing source, destination and the actual
message.

### Words over symbols

I thought this was pretty good, it allows me to describe actors and systems separately, where an actor
is a human and a system is a system. And it's clear that **Alice** sends a message to **Acme**. The
arrow notation was also something I was used to from websequencediagrams.com

Then I quickly realized that having different symbols for the type of message was problematic,
would I render an asynchronous message as `~>`, a reply as `<-`? I came to the conclusion that
using words are better than symbols. Symbols are only meaningful if the user has previous experience
of those. Which in my case would require symbols to be established in the _domain_ of sequence diagrams.

This is a well-known fact among [usability experts](https://www.nngroup.com/articles/icon-usability/),
which says that an icon needs a label and standalone icons can be problematic.

Thus, I replaced the arrows with the wording of **ask** or **tell** to better communicate
the nature of the message (asynchronous or synchronous). I also changed **System** to **Object** since I though
**System** was a bit to narrow.

My draft was now:

```bash
Actor Alice
    Alices description in plain text

Actor Bob
Object Acme

Sequence Hello
    Alice tell Bob
        | Sync call to Bob

    Alice ask Bob
        | Async call to Bob
```

### Talk to your users

When I showed this for my collogue ([@provegard](https://twitter.com/provegard)), he immediately pointed out that
I swapped the semantic meaning of `tell` and `ask` - at least compared to how [Akka](https://doc.akka.io/docs/akka/2.5/actors.html#send-messages)
(a JVM actor implementation) defines them.

This lead to an interesting discussion on the semantics of both **ask** and **tell** and was very valuable
and made it obvious that I had mixed things up.

* **Tell** - should be asynchronous since you when tell someone something you do not expect an answer.
* **Ask** - should be synchronous since when you ask someone something, you wait for the answer.

Lesson learned here is that it's easy to be blind to flaws if you're the only one using the language.
This, however, is not to say that multiple people should define a language, I am a firm 
believer that a language should have a benevolent dictator.

I also removed the free form text from both descriptions and message text. The reason was a combination
of my familiarity of using strings combined with the fear of parsing inline text fragments.

These changes resulted in the latest, and present, version of Sequence:

```bash
Name "Sequence demo"

Actor User
Object Alpha
Object Bravo

Sequence Test
    User ask Alpha "A synchronous message"
    Alpha tell Bravo "An asynchronous message"
    Bravo replies Alpha "A response message"
    Alpha replies User """
        This is a multi-line
        message text!
        """
```

My primary takeaways from defining the Sequence language so far is:

* **Begin with the semantics** - Start with what do you want to express and then move into grammar and syntax
* **Words over symbols** - Icons and symbols are only valuable if they are established in the domain
* **Talk to your users** - Avoid a self-centered language, is your choice of words shared by your users?


Next up we will dive into the technical aspects of defining and parsing the language. I will show how to
create a parser for the Sequence language and how you can use TDD your way through the grammar definition.
