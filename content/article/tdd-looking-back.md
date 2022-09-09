+++
title = "Test-Driven Development - looking back"
date = 2022-09-09T19:21:31+02:00
description = "For the last five years I have been test driving pretty much all the code I have written. This is a retrospective on why it took so long to get started, where I see the greatest benefits and where it is still hard."
tags = ["Software Development"]
+++

I have been working in the software industry for more than 15 years. I started my career with Symbian C++, an obscure programming language used by the smartphones of that era. At that company there were almost no unit-tests, most of the testing was done manually (which included the time consuming flashing of ROM-image) both by developers and a huge test department. From there I moved on to web and general back-end development, still relying on manual testing. At this moment in time, web development was all about server-side rendering, CSS and jQuery. There were a lot of moving parts like CMS, networking, browsers and most of it was about look and feel - that is too hard to test I argued. At the back-end, I wrote the occasional test, for the simple stuff at least, and so it continued for quite a few years.

I was well aware of Test-Driven Development and that it was popular in some communities such as Ruby. I also bought in on the rationale and motivation that you should test your code, at least your application logic. But whenever I tried to go from a tutorial on testing (typically testing some maths functions) to my application I failed and then I gave up, crawling back to manual testing.

Then, about five years ago I started working at my current company. This is a high profile company of highly skilled developers that are advocates of eXtreme Programming in general, TDD in particular and producing quality software overall. Not writing tests was never an option, using TDD was (and still is) considered hygiene by us. Before I joined this company I met with a few of them at a conference where one of them said something along the lines of:

> “Yes writing tests are hard but so are a lot of things in computer science. Just because it is hard does not mean we stop doing it does it?”

## Five years later

It has been five years since I forced myself to use TDD for pretty much all the code I write and I absolutely love it. I would never go back to what I consider, my dark ages.

Was it hard? Yes and no. First off I got the opportunity to build a transpiler as one of my first tasks, I think this is an ideal project for test driven development given its pure nature. Then I used that project to **practice, practice and practice**. Every time I got carried away I commented out the code and started all over again, this time with tests first.

Besides practice I leaned on my colleagues, asking them “how would you design the test for this problem” and reading through their test code. Having mentors like this really accelerated my journey. Learning established patterns for both test setup and how to produce well described failures was key to get the most out of my testing early on.

## My favourite things

During my earlier attempts I failed to experience the benefit of testing. I understood the quality aspect from a theoretical perspective but I never experienced it. Most of my early tests were for simpler things that maybe did not change or were trivial to implement, then I gave up too soon when it became more complicated.

Now, after years of test driving various projects and types of applications I see several benefits. 

The quality aspect I was looking for all those years ago is present, I do produce higher quality code using TDD. This is of course a great thing, quality code is what the client pays us for after all. However great, it is not the greatest benefit.

**The greatest benefit** for me is to have the confidence to change code over time, knowing that if I break existing behaviour there will be a test there to catch it. This does not just keep the regression risk to a minimum, it enables me to keep my working set as small as possible, knowing that I can always extend or change the code in a safe way tomorrow.

That in turn **enables me to move fast and be agile**, something which enables my customers to **respond to their users and market quicker**.

I believe that TDD makes me a **better team player** as well. It is much easier for a colleague to work together with me when there are existing tests they can start from when adding or changing behaviour. While I do not think tests should replace documentation it is a very good tool to learn existing behaviour (e.g. comment out something and see what fails).

My second favourite thing with TDD is that it **reduces my cognitive load**. I break down a complex feature into smaller parts and then, one by one, add tests that go red and then add the code to make it green. It is a great feeling when the tests are all green, then I know my current increment is ready to be committed and after that my brain can take a break and then move on to the next increment.

## The hard parts

In the beginning I thought the hard parts about testing were technical. And sure, there are technical challenges such as:

**Look and feel** like a web page’s layout and disposition is hard to automate (from my understanding raster comparison is still too fragile). Here I tend to still rely on manual verification.

**Concurrency and time** can be quite tricky, it requires you to have control over scheduling and to be able to advance time programmatically.

**Third party integrations** might not offer testable versions or be implemented using techniques that are not easily mocked.

My experience is that you can get decent tests for the latter two by using a combination of wrappers / proxies and dependency injection (there I said it, DI, it is pretty awesome).

Technical challenges aside, the hardest part with test-driven development is to write tests that **clearly communicate the intent of the test**. This might sound like it is a matter of naming, but I think it is more to it than just the name.

To communicate intent a test must have an easy-to-understand test setup and a crystal clear assertion on what is expected. If the expectation fails, the test should be as precise as possible on why it failed and what happened instead.

Writing high quality tests takes time to master. A few principles I tend to use is:

- Use hieracial test suites to share common setup
- Use view object / test wrappers to encapsulate interaction with the system under test
- A single assertion per test


## What is the cost?

Is it expensive to TDD? Yes.

Is it worth it? Yes.

When I usually have these discussions, it is comparing the effort of writing tests versus not writing tests. In which the latter obviously takes zero effort. But if we consider the effort to maintain a piece of code for 5-10 years I am confident that it’s not more expensive - in fact I do believe **it is cheaper than the alternative**.

If we put the quality aspect aside again, the two greatest benefits of using TDD from my perspective is **to move fast and be agile** and to be a **better team player**. Aspects that I know are super important to most organisations. The fact that it reduces cognitive load and makes you sleep better at night are just added benefits.

## My recommendations

So if you are about to learn TDD or maybe you are experiencing the same frustration I did when going from a tutorial to your own application, here are some recommendations:

- Find a mentor to guide you on how to write good tests or testable software.
- Don’t rely too much on katas and exercises, practice in your daily code base. It does not matter if the tests are not that great to begin with, you can always improve them later.
- It takes time to establish new behaviour - do not give up! Once you get over the hill, writing tests is both technically and emotionally rewarding.
