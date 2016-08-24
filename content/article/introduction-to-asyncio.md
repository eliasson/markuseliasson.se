+++
date = "2016-08-24T21:00:00+02:00"
title = "An introduction to Python's asyncio"
Categories = ["Code"]
Tags = ["Python", "BitTorrent"]
Description = "This article gives a gentle introduction to the asyncio that arrived in Python 3.5. The purpose is to set the scene for a future article where I use asyncio to build a BitTorrent client in Python."
+++
In Python 3.4 a new module, `asyncio` was introduced, this module allows you to write _concurrent_, _single threaded_ code in Python without relying on any third-party libraries (such as Twisted, or Tornado).

I plan to take this new module for a test run, implementing a simple BitTorrent client. But first lets see how we can write _concurrent_ code in Python 3.5.

Remember, **concurrency** is not the same as **parallelism**.

* **Concurrency** is when more than one function can be started and finished, overlapping each other, without having to be executed at the exact same time. This is possible with a single-core CPU.

* **Parallelism** is when one or more functions run at the same time, this requires multi-core CPU.

_As a metaphor, consider when you are in the kitchen making dinner. You put your potato cake into the oven, setting your kitchen timer for 15 minutes. Meanwhile, you start frying some pork to go with it. After 15 minutes, the timer goes off with a beep, you put away the frying pan and take the cake out of the oven._

Your are being concurrent, there is only one person (CPU) in the kitchen, doing multiple tasks at the same time.

Single threaded, asynchronous programming is considered simpler than using multi-threaded programming. The reason is that you don't need to coordinate routines, and shared mutable state. Rather you write single-threaded programs that feels quite sequential. This is partially what made NodeJS as popular as
it is - the async nature is built in to NodeJS and async is often default and a synchronous API is made an option.

`asyncio` gives us asynchronous I/O, thus it is suitable for file and network operations, where the process will be schedule to wait for data being available. It is **not** suitable for CPU-bound programming - here you need to fallback to threading or multi-processing.

As it turns out, the intended example project - a BitTorrent client - have plenty of I/O and not so much CPU-bound work to do - it should match `asyncio` perfectly.


### await and async

If you run the code snippet below, you will open two TCP connections to two of Google's DNS servers. Once the connection is open, we'll pretend to do some I/O but rather sleeping. Once the fictive work is done, the connection will be closed.

This is all run on a single thread, yet two connections are open at the same time. If you run the program for a couple of times you will see that the order in which the connections are closed varies due to the randomized time to sleep.

````python
import asyncio
from random import randint

async def do_stuff(ip, port):
    print('About to open a connection to {ip}'.format(ip=ip))
    reader, writer = await asyncio.open_connection(ip, port)

    print('Connection open to {ip}'.format(ip=ip))
    await asyncio.sleep(randint(0, 5))

    writer.close()
    print('Closed connection to {ip}'.format(ip=ip))

if __name__ == '__main__':
    loop = asyncio.get_event_loop()

    work = [
        asyncio.ensure_future(do_stuff('8.8.8.8', '53')),
        asyncio.ensure_future(do_stuff('8.8.4.4', '53')),
    ]

    loop.run_until_complete(asyncio.gather(*work))
````

Let's start with the main code block. First we get a reference to the default [event loop](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio-event-loop). Then we create a list of two tasks calling the function `do_stuff` and tell the event loop to run until those tasks are complete.

The function `do_stuff` is declared with the `async def` statement, which makes it into something called a [coroutine](https://docs.python.org/3/glossary.html#term-coroutine). A _coroutine_ is a special kind of generator and to which you can send a value back. The nice thing about coroutines is that they can be suspended, and resumed at a later state - with the scoped variables intact.

Whenever a `await` is reached inside the `do_stuff` that coroutine will be suspended until the response is ready (the sending values back to a generator). When resumed, it will continue execution at the same position and keep going until the next `await` is there - or until the return statement is reached ( Python use implicit return statements).

Now, back to the _event loop_. Whenever a coroutine is suspended, the event loop will resume another coroutine (given that it is ready to be resumed). This is the _cooperative multi-tasking_ - coroutines will take turns running on the same thread, but two coroutines will **not** run in parallel.

Basically, any function that supports asyncio is either declared using `async def` _or_ by using the decorator `@asyncio.coroutine`. Any code calling such functions needs to use the `await` statement _or_ `yield from`.

In Python 3.4 the asyncio used the decorator `@coroutine` for a special type of generator, and `yield from` to pause the the generator while waiting for something to happen (e.g. a read being ready to consume). In Python 3.5 this was replaced by the expressions `async` and `await`.

_Awaitable_ functions (or coroutines) needs to be wrapped in a Future and handed
over to the _event loop_. And finally the _event loop_ needs to be instructed to run.


## Summary

This was a short introduction to asyncio in Python. If you have done async programming in another language (JavaScript, C#) you might feel just at home, if not there are great articles presenting Python's implementation in details and with a proper walkthrough from iterator, generator, corouties to async/await.

Brett Cannon have written an excellent post [How the heck does async/await work in Python 3.5](http://www.snarky.ca/how-the-heck-does-async-await-work-in-python-3-5). And A. Jesse Jiryu Davis and Guido van Rossum gives great detail in their article [A Web Crawler With asyncio Coroutines](http://aosabook.org/en/500L/a-web-crawler-with-asyncio-coroutines.html). If you only read one, I recommend Brett Cannon's as I found it easier to digest.
