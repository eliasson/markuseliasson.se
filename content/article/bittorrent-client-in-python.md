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


### The concept

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


### Python's asyncio

As stated in the beginning of this article, one of the purposes with building **Pieces** was to get acquainted with Python's asyncio.

In Python 3.4 a new module, `asyncio` was introduced, this module allows you to write _concurrent_, _single threaded_ code in Python without relying on any third-party libraries (such as Twisted, or Tornado).

Remember, **concurrency** is not the same as **parallellism**.

* **Concurrency** is when more than one function can be started and finished, overlapping of each other, without having to be executed at the exact same time. This is possible with a single-core CPU.

* **Parallellism** is when one or more functions run at the same time, this requires multi-core CPU.

_As a metaphor, consider when you are alone in the kitchen making dinner. You put your potato cake in to the oven, setting your kitchen timer for 15 minutes. Meanwhile, you start frying some pork to go with it. After 15 minutes, the timer goes off with a beep, you put away the frying pan and take the cake out of the oven._

Your are being concurrent, there is only one person (CPU) in the kitchen, doing multiple tasks at the same time.

Single threaded, asynchronous programming if far simpler than using multi-threaded programming. The reason is that you don't need to coordinate routines, and shared mutable state. Rather you write single-threaded programs that feels quite sequential.

`asyncio` gives us asynchronous IO, thus it is suitable for file and network operations, where the process will be schedule to wait for data being available. It is **not** suitable for CPU-bound programming - here you need to fallback to threading or multi-processing.

It turns out that BitTorrent is loaded with IO operations, thus this should be a perfect match.

As it turns out, BitTorrent have plenty of IO and not so much CPU-bound work to do - it should match `asyncio` perfectly.


#### await and async

In Python 3.4 the asyncio used the decorator `@coroutine` for a special type of generator, and `yield from` to pause the the generator while waiting for something to happen (e.g. a read being ready to consume). In Python 3.5 this was replaced by the expressions `async` and `await`.

Brett Cannon have written an excellent post on the detail and history regarding generators, coroutines, and these new expressions in his post [How the heck does async/await work in Python 3.5](http://www.snarky.ca/how-the-heck-does-async-await-work-in-python-3-5) you do not need to read this post in order to grasp this article, but it gives a greater understanding on what goes on underneath the surface.

Basically, it works like this. `await` will pause the current execution waiting for the call to be ready to be resumed (e.g. when a stream of data is read, a timeout occurred, etc). All methods that can be resumed should use the expression `await`. As an example, consider the code below:

    async def get_foo() -> str:


#### The event loop

Python `asyncio` comes with an _event loop_ that manages the concurrent tasks and notifies a task whenever something of interest happened. E.g. whenever there is data read from a socket ready for consumption.

There is a single _event loop_ running all tasks in a _cooperative_ multi-tasking environment. _Cooperative_ means that a running task cannot be interrupted (unlike processes in an Operating System), it will run until it calls a function causing it to be suspended.

The _event loop_ can either be run until all tasks are complete, or it can be run forever. In our case, the _event loop_ will run until the download is either finished or aborted.

    https://github.com/eliasson/pieces/blob/master/pieces/cli.py#L52



#### Bencoding

Bencoding is the binary encoding format used in BitTorrent. It is used for the _meta-info_ stored inthe `.torrent` file, as well as some of the results when communicating with the tracker.

Bencoding supports four different data types, *dictionaries*, *lists*, *integers* and *strings* - it is fairly easy translate to Python's _object literals_ or _JSON_.

Below is bencoding described in *Augmenterad Backus-Naur Form* ([ABNF](https://en.wikipedia.org/wiki/Augmented_Backus–Naur_Form)).

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
