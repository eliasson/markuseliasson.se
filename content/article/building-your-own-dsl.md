+++
Categories = ["Software Development", "Code"]
Description = "The need for different types of computer languages is growing rapidly — luckily it turns out that creating your own Domain Specific Language does not have to be all that hard."
Tags = ["Software Development", "Sequence", "DSL"]
date = "2018-03-23T18:48:00+01:00"
title = "Building your own DSL does not have to be hard"
draft = false
+++
Currently there are a lot of initiatives saying that everybody should learn how to program, and schools are making programming part of the curriculum. For enterprises there are numerous of low-code platforms popping up, all with the sole purpose of enabling more people to construct programs. And it's all happening at a rapid pace.

Meanwhile, as developers, we get exposed to multiple languages daily, be it build-tools, web-frameworks or query language. Not only are we skilled in our languages of choice, we are also trained to be able to pick up new languages fairly quickly, sometimes withouth even being aware of it. If you stop to think about it, how many computer languages have you worked with during your career? 5? 10? 20? More?

In short, programming is growing tremendously, it is finding its way into new domains and new people are finding their way into programming - I think it is safe to say that we will see an increasing need for many different types of programming languages in the near future.


## Not hard - really?

I think many engineers mistakenly assume that creating your own language, and especially the dreaded compiler, is a huge undertaking and a very complex one. If the language is a General Purpose Language such as Java, C or Rust yes the task is daunting, to say the least. There must be a reason why some argue that the C compiler is one of the most complex pieces of software out there.

Thankfully, a Domain Specific Language is not a General Purpose Language per definition. A DSL can be, and often is much simpler, and it might not even need a complex compiler.

A General Purpose Language is generic by definition. Its domain is not the domain in which it is being used, but rather the domain _computer science_. Scala is a great example of a GPL. It leans heavily on theories from both computer science and math (such as category theory) and can be used to write programs either in functional or object-oriented style - one cost of this is that both the language and the compiler are well known for their complexities.

A DSL, as I see it, has a few different characteristics:

**Unambiguous grammar** - The grammar should be kept as simple and unambiguous as possible, or as [The Zen of Python](https://www.python.org/dev/peps/pep-0020/) puts it _"There should be one - and preferably only one - obvious way to do it."_. Still the grammar can be both big and complex, just as a domain can be big and complex (similar to defining a _Ubiquitous Language_ in DDD).

**A single abstraction level** - A DSL should be designed to operate on a single abstraction level; its purpose is specific and not general. If you have a wide spectrum, maybe it is better to go with multiple DSLs, choose a GPL, or maybe even build an embedded DSL. HTML and CSS are an examples of two different languages being used; one for describing the semantics of a document and the other for styling it (including graphical transitions). Although there is an overlap in domain, and HTML can inline style properties, they are two separate languages.

**Tooling** - A DSL is only as good as its tooling. Putting an untrained programmer in front of a great language, but with poor tooling, will lower the adoption rate. Likewise, a well-trained programmer will most likely fall back and use one of the other languages he or she knows to get the job done. I do think that, for some reason, people are more forgiving when using a GPL when it comes to tooling (or at least I am).

**No machine-code** - Traditional programming languages compile to machine code, or to some sort of byte code like the JVM or CLR. For a DSL that is not always the case, a DSL can very well be used for configuration, modeling, etc. Thus, there might not be a need for a generator / compiler, it might be enough to interpreted a language (such as a Makefile).

This does not automatically mean that a DSL is simple, but for some use-cases you might get away with a smaller scope than with a traditional language. Also, there are plenty of high quality, and free, tools available for you to use to get a pretty good tooling.

To prove my point, I intend to illustrate this by developing an example language with associated tooling and publishing both articles and code to follow up on that work.

Finally, not only do I think this skill will be more valuable in the future; developing a language is a lot of fun too!


## Illustrating with an example

I often find myself creating sequence diagrams, whether when designing a system, explaining some details to a colleague or just when trying to understand something for myself. The last few years I have used the excellent service [http://websequencediagrams.com](http://websequencediagrams.com) but I have two problems with that service:

1) Version control, I needed to keep my source files under version control and manually copy paste into the web editor.

2) Online only, I do a fair amount of work while travelling and in the deep forests of Sweden you’re not always connected to the Internet.

This will result in a blog series about creating a custom language where you can express a sequence flow and produce image representations out of these, both using a command line tool as well as Visual Studio Code.

The upcoming posts are planned to cover the following topics:

* Defining a language
* Parsing source files
* Analyzing an AST
* Producing the target image
* Syntax highlight
* Showing diagnostics
* Producing images in VS Code

The work for the tool is ongoing, and while not complete, there is a command line tool that is in a usable state.

The source code is available on GitHub at http://github.com/eliasson/sequence

In the next post I will discuss the language, the reasoning behind why I designed it the way I did and how to use a tool called _parser generator_ to create a parser for your language.