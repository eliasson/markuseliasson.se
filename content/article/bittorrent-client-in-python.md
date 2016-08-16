+++
Categories = ["Code"]
Tags = ["Python"]
date = "2016-08-12T00:28:13+02:00"
title = "A BitTorrent client in Python 3.5"
draft = true
Description = "Python 3.5 comes with support for asynchronous IO, which seems like a given fit when implementing a small BitTorrent client. This article will guide you through both the BitTorrent protocol details as well as diving in to asyncio for better or worse."
+++
BitTorrent has been around since 2001. The big breakthrough was when sites as _The Pirate Bay_ made it popular to use for downloading pirated material. Streaming sites, such as Netflix, might have resulted in a decrease of people using BitTorrent for downloading movies. But BitTorrent is still used in a number of different, legal, solutions where distribution of larger files are important.

* [Facebook](https://torrentfreak.com/facebook-uses-bittorrent-and-they-love-it-100625/) use it to distribute updates within their huge data centers
* [Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/S3Torrent.html) implement it for downloading of static files
* Traditional downloads still used for larger files such as [Linux distrubutions](http://www.ubuntu.com/download/alternative-downloads)

We will go through the details in the BitTorrent protocol in a moment, but first let us switch topic and have a look at Python's new module, `asyncio`.


## Python's asyncio

In Python 3.4 a new module, `asyncio` was introduced, this module allows you to write _concurrent_, _single threaded_ code in Python without relying on any third-party libraries (such as Twisted, or Tornado).

Remember, **concurrency** is not the same as **parallellism**.

* **Concurrency** is when more than one function can be started and finished, overlapping each other, without having to be executed at the exact same time. This is possible with a single-core CPU.

* **Parallellism** is when one or more functions run at the same time, this requires multi-core CPU.

_As a metaphor, consider when you are in the kitchen making dinner. You put your potato cake in to the oven, setting your kitchen timer for 15 minutes. Meanwhile, you start frying some pork to go with it. After 15 minutes, the timer goes off with a beep, you put away the frying pan and take the cake out of the oven._

Your are being concurrent, there is only one person (CPU) in the kitchen, doing multiple tasks at the same time.

Single threaded, asynchronous programming is considered simpler than using multi-threaded programming. The reason is that you don't need to coordinate routines, and shared mutable state. Rather you write single-threaded programs that feels quite sequential. This is partially what made NodeJS as popular as
it is - the async nature is built in to NodeJS and async is often default and a synchronous API is made an option.

`asyncio` gives us asynchronous IO, thus it is suitable for file and network operations, where the process will be schedule to wait for data being available. It is **not** suitable for CPU-bound programming - here you need to fallback to threading or multi-processing.s

As it turns out, BitTorrent have plenty of IO and not so much CPU-bound work to do - it should match `asyncio` perfectly.


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

Let's start with the main code block. First we get a reference to the default [event loop](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio-event-loop). Then create a list of two tasks calling the function `do_stuff` and tell the event loop to run until those tasks are complete.

The function `do_stuff` is declared with the `async def` statement, which makes it into something called a [coroutine](https://docs.python.org/3/glossary.html#term-coroutine). A _coroutine_ is a special kind of generator and to which you can send a value back. The nice thing about coroutines is that they can be suspended, and resumed at a later state - with the scoped variables intact.

Whenever a `await` is reached inside the `do_stuff` that coroutine will be suspended until the response is ready (the sending values back to a generator). When resumed, it will continue execution at the same position and keep going until the next `await` is there - or until the return statement is reached ( Python use implicit return statements).

Now, back to the _event loop_. Whenever a coroutine is suspended, the event loop will resume another coroutine (given that it is ready to be resumed). This is the _cooperative multi-tasking_ - coroutines will take turns running on the same thread, but two coroutines will **not** run in parallel.

Basically, any function that supports asyncio is either declared using `async def` _or_ by using the decorator `@asyncio.coroutine`. Any code calling such functions needs to use the `await` statement _or_ `yield from`.

In Python 3.4 the asyncio used the decorator `@coroutine` for a special type of generator, and `yield from` to pause the the generator while waiting for something to happen (e.g. a read being ready to consume). In Python 3.5 this was replaced by the expressions `async` and `await`.

_Awaitable_ functions (or coroutines) needs to be wrapped in a Future and handed
over to the _event loop_. And finally the _event loop_ needs to be instructed to run.


## Summary

This was a fairly short introduction to asyncio in Python. If you have done async programming in another language (JavaScript, C#) you might feel just at home, if not there are great articles presenting Python's implementation in greater details and with a proper walkthrough from iterator, generator, corouties to async/await.

Brett Cannon have written an excellent post [How the heck does async/await work in Python 3.5](http://www.snarky.ca/how-the-heck-does-async-await-work-in-python-3-5). And A. Jesse Jiryu Davis and Guido van Rossum gives great detail in their article [A Web Crawler With asyncio Coroutines](http://aosabook.org/en/500L/a-web-crawler-with-asyncio-coroutines.html). If you only read one, I recommend Brett Cannon's as I found it easier to digest.


## The BitTorrent concept

BitTorrent is a peer-to-peer protocol, where peers join a _swarm_ of other peers to exchange pieces of data between each other. Each peer is connected to multiple peers at the same time, and thus downloading or uploading to multiple peers at the same time. So if you are downloading a file you are downloading from multiple sources in parallel. This is great in terms of limiting bandwidth compared to when a file is downloaded from a central server it is also great for keeping a file available as it does not rely on a single source, but multiple sources.

A _torrent_ represents a set of files that should be shared. It can either be a directory of files or a single file - regardless it is distributed as a single `.torrent` file. This file is referred to as the _meta-info_ and all peers participating in the distribution of this _torrent_ need to have this file.

A BitTorrent client will execute these steps in order to download a torrent.

1. It begins by parsing the _meta-info_ (a `.torrent` file) to get information about the torrent to download.
2. The client connects to a _tracker_ to retrieve a list of _peers_ that can be connected to.
3. For each _peer_ the client tries to open a TCP connection and perform a handshake.
4. If the handshake is successful, the client will ask for permission to start requesting _pieces_.
5. The client starts requesting _pieces_ from multiple peers at the same time.
6. For each received piece, the client performs checksum control and persist the piece to disk.
7. Once all pieces are downloaded the peer transits from a _downloader_ to only being a _seed_.

These steps will be covered in more details together with how it is implemented in Python. But first, let us clarify those terms used (for a full list of terms, see [Wikipedia](https://en.wikipedia.org/wiki/Glossary_of_BitTorrent_terms#Seed))

Term        | Description
----------- | -----------------------------------------------------------------
Peer        | A node in the BitTorrent swarm, it can either be downloading and/or uploading pieces from other peers. |
Meta-info   | The `.torrent` file specifying URL to the tracker, information and SHA1 hashes for the torrent pieces. |
Tracker     | A server that keeps track of which peers that is part of a swarm for a specific _torrent_. |
Swarm       | The peers forming the BitTorrent network for a single torrent. |
Leecher     | A peer that is only downloading pieces and not contributing back by uploading to other peers |
Seed        | A peer that is uploading pieces of data for a torrent. A peer can |
Downloader  | A peer that is downloading pieces and does not have the entire torrent. |


## The implementation

As stated in the ingress of this article, the sole purpose of writing my own BitTorrent client, called **Pieces** was to try out Python's asyncio and at the same time scratch an old itch of implementing a BitTorrent client.

Thus, the implementation is not focused on performance, features or efficiency - instead it is focused on being simple and readable. The implementation is guaranteed to contain bugs and mistakes - after all, it is implemented in late nights during my summer holiday.

All of the source code is available at [GitHub](https://github.com/eliasson/pieces) and released under the Apache 2 license. Feel free to learn from it, steal from it, improve it, laugh at it or just ignore it.


### Parsing the meta-data

### Connect to the tracker

### Connect to peers

### Request pieces

### Persist pieces

### Future work

Seeding is not yet implemented in the pieces client, but it should not be that hard to implement. What is needed is something along the lines of this:

* Whenever a peer is connected to, we should send a `BitField` message to the remote peer indicating which pieces we have.

* Whenever a new piece is received (and correctness of hash is confirmed), each `PeerConnection` should send a `Have` message to its remote peer to indicate the new piece we have.

In order to do this the `PieceManager` needs to be extended to return a list of 0 and 1 for the pieces we have. And the `TorrentClient` to ask the `PeerConnection` to send the messages. Both `BitField` and `Have` messages should support encoding of these messages.

Having seeding implemented would make **Pieces** a good citizen, supporting both downloading and uploading of data within the swarm.

Additional features that probably can be added without too much effort is:

* **Multi-file torrent**, will hit `PieceManager`, since Pieces and Blocks might span over multiple files, it affects how files are persisted (i.e. a single block might contain data for more than one file).

* **Resume a download**, by seeing what parts of the file(s) are already downloaded (verified by making SHA1 hashes).

* **UDP** support and not only HTTP when connecting to a tracker.


-------------------------------------------------------------------------------


#### Bencoding

Bencoding is the binary encoding format used in BitTorrent. It is used for the _meta-info_ stored inthe `.torrent` file, as well as some of the results when communicating with the tracker.

Bencoding supports four different data types, *dictionaries*, *lists*, *integers* and *strings* - it is fairly easy translate to Python's _object literals_ or _JSON_.

Below is bencoding described in [Augmenterad Backus-Naur Form](https://en.wikipedia.org/wiki/Augmented_Backus–Naur_Form) courtesy of the [Haskell library](https://hackage.haskell.org/package/bencoding-0.4.3.0/docs/Data-BEncode.html).

    <BE>    ::= <DICT> | <LIST> | <INT> | <STR>

    <DICT>  ::= "d" 1 * (<STR> <BE>) "e"
    <LIST>  ::= "l" 1 * <BE>         "e"
    <INT>   ::= "i"     <SNUM>       "e"
    <STR>   ::= <NUM> ":" n * <CHAR>; where n equals the <NUM>

    <SNUM>  ::= "-" <NUM> / <NUM>
    <NUM>   ::= 1 * <DIGIT>
    <CHAR>  ::= %
    <DIGIT> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"




#### Tracker

#### Peers

#### Piece strategy


# References

ABNF https://hackage.haskell.org/package/bencoding-0.4.3.0/docs/Data-BEncode.html

Inofficiella BitTorrent specifikationen https://wiki.theory.org/BitTorrentSpecification

# TODO

- Link to Facebook, Amazon, Linux.
https://torrentfreak.com/facebook-uses-bittorrent-and-they-love-it-100625/

https://en.wikipedia.org/wiki/Bram_Cohen

http://docs.aws.amazon.com/AmazonS3/latest/dev/S3Torrent.html
