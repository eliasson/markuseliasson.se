+++
date = 2014-08-31T12:34:15Z
title = "Adventures in Go"
description = "I made my first baby steps in the world of Go. Will this little adventure lead me to a world where the grass is green, or will it lure me into the shadows?"
tags = ["go", "bittorrent"]
draft = false
+++

I am in my mid thirties and like many of my peers I need to take som action to stay(?) cool. Since I don't fancy running a marathon, nor do I have the money to buy me a brand new car - my only choice is to learn a new programming language.

My choices were Scala, Clojure or Go. I already done some work in Clojure (which I really enjoyed) so that was out. Scala gives me mixed feelings, it reminds me of Perl, SBT is anything but simple but it has Akka which I have always wanted to try.

Go on the other hand is known for being the love child of Python and C - and recently I have been doing a lot of Python programming. Python is a really nice language, it has a perfect abstraction level, it easy to read, great community - but it's not very good at concurrency. So I want to see if I can enjoy Go as I enjoy Python.


## lox

Deciding on what to implement was tricky, I wanted to try some concurrency features as well as regular systems programming. I ended up with the idea to implement a tiny bittorrent client. I don't use bittorrent that frequently anymore, but I have always been interested its technology, so this could be fun. I named this little experiment of mine to **lox** .


## Go's toolchain

Installing go and setting up the `GOPATH` did not take long. I decided to wrap the go commands using a traditional Makefile. Inspired by [Drone](https://github.com/drone/drone) I also added my dependencies in there, so I can run `make deps` to download the dependencies for lox.

How the dependencies work is quite strange when you're used to Maven or PIP. I still do not understand whether or not I am supposed to put the source code for my dependencies under source control or not. Given that there is no versioning of dependencies it would be a good thing, but I rather not put someone else's code in my repository.

I have not checkout out [Godep](https://github.com/tools/godep) but if my list of dependencies grows I might do that.

For now, my `Makefile` looks something like this [gist](https://gist.github.com/eliasson/e572b28c9a0eef0b2763):

    PACKAGES := \
        github.com/eliasson/foo \
        github.com/eliasson/bar
    DEPENDENCIES := github.com/eliasson/acme

    all: build silent-test

    build:
        go build -o bin/foo main.go

    test:
        go test -v $(PACKAGES)

    silent-test:
        go test $(PACKAGES)

    format:
        go fmt $(PACKAGES)

    deps:
        go get $(DEPENDENCIES)


## Logging

Go comes with a default logger, but you might want to have different loglevels, or print more detailed information. Then this blog post comes in super handy [Using The Log Package In Go](http://www.goinggo.net/2013/11/using-log-package-in-go.html)


## So far

My current thoughts on Go is that it is a quite simple language. While simple is better than easy (as [Rich Hickey](http://www.infoq.com/presentations/Simple-Made-Easy) puts it), there are a few things I find frustrating:

* Error handling. Go does not use exceptions, but return values which causes the code to be very verbose (or it is just me who has not got the hang of it).

* Add methods on non-local types. I was hoping I could use them similar to Clojures protocols.


What I find great is:

* The language feels nice so far, like I said the love child of Python and C. It is easy to read and well documented.

* The speed, both compilation and runtime.

* The separation of data and behaviour (structs and methods).


I'll try to add detailed posts on go along the road of my adventures. And I'll make sure to publish the source code for lox once it brings some value.

Markus