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

`asyncio` gives us asynchronous I/O, thus it is suitable for file and network operations, where the process will be schedule to wait for data being available. It is **not** suitable for CPU-bound programming - here you need to fallback to threading or multi-processing.s

As it turns out, BitTorrent have plenty of I/O and not so much CPU-bound work to do - it should match `asyncio` perfectly.


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

This was a short introduction to asyncio in Python. If you have done async programming in another language (JavaScript, C#) you might feel just at home, if not there are great articles presenting Python's implementation in greater details and with a proper walkthrough from iterator, generator, corouties to async/await.

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


### Pieces and blocks

Before we dive into the implementation, let us go through how a torrent is split into pieces as this is significant in a few places.

A _piece_ is, unsurprisingly, a partial piece of the torrents data. A torrent's data is split into _N_ number of pieces of equal size (except the last piece in a torrent, which might be of smaller size than the others). The piece length (_N_) is specified in the `.torrent` file. Typically pieces are of sizes 512 kB or less, and should be a power of 2.

Pieces are used to acknowledge which pieces a given client have, both to the tracker and to a remote peer (this will be described further down). However, pieces are further divided into something refered to as _blocks_.

A _block_ is 2^14 (16384) bytes in size, except the final block that most likely will be of a smaller size.

Consider an example where a `.torrent` describes a single file `foo.txt` to be downloaded.

```python
    name: foo.txt
    length: 135168
    piece length: 49152
```

That small torrent would result in 3 pieces:

```python
    piece 0: 49 152 bytes
    piece 1: 49 152 bytes
    piece 2: 36 864 bytes (135168 - 49152 - 49152)
          = 135 168
```

Now each piece is divided into blocks in sizes of `2^14` bytes:

```python
    piece 0:
        block 0: 16 384 bytes (2^14)
        block 1: 16 384 bytes
        block 2: 16 384 bytes
              =  49 152 bytes

    piece 1:
        block 0: 16 384 bytes
        block 1: 16 384 bytes
        block 2: 16 384 bytes
              =  49 152 bytes

    piece 2:
        block 0: 16 384 bytes
        block 1: 16 384 bytes
        block 2:  4 096 bytes
              =  36 864 bytes

    total:       49 152 bytes
              +  49 152 bytes
              +  36 864 bytes
              = 135 168 bytes
```

The *blocks* is the data that is requested from the connected peers, and when all blocks for a given piece is retrieved the piece is complete.


_The official specification refer to both pieces and blocks as just pices which is quite confusing. The unofficial specification and others seem to have agreed upon using the term block for the smaller piece which is what we will use._

_The official specification is stating another **block size** that what we use. Reading the unofficial specifcation, it seems that 2^14 bytes is what is agreed among implementers - regardless of the official specification._


## The implementation

As stated in the ingress of this article, the sole purpose of writing my own BitTorrent client, called **Pieces** was to try out Python's asyncio and at the same time scratch an old itch of implementing a BitTorrent client. I have always been fashinated with peer-to-peer protocols, it seems like lots of fun.

Thus, the implementation is not focused on performance, features or efficiency - instead it is focused on being simple and readable. The implementation is guaranteed to contain bugs and mistakes - after all, it is implemented late at nights during my summer holiday.

All of the source code is available at [GitHub](https://github.com/eliasson/pieces) and released under the Apache 2 license. Feel free to learn from it, steal from it, improve it, laugh at it or just ignore it.

While going through the implementation it might be good to have read, or to have another tab open with the [Unofficial BitTorrent Specification](https://wiki.theory.org/BitTorrentSpecification). This is without a doubt the best source of information on the BitTorrent protocol. The official specification is vague and lacks certain details so the unofficial is the one you want to study.


### Parsing the meta-data

The first thing a client needs to do is to find out what it is supposed to download and from where. This information is what is stored in the `.torrent` file, a.k.a. the _Metainfo_.

There is a number of properties stored in the _meta-info_ that we need in order to successfully implement a client. Things like:

* The name of the file to download
* The size of the file to download
* The URL to the tracker to connect to

All these properties are stored in a binary format called _Bencoding_.

Bencoding supports four different data types, *dictionaries*, *lists*, *integers* and *strings* - it is fairly easy translate to Python's _object literals_ or _JSON_.

Below is bencoding described in [Augmented Backus-Naur Form](https://en.wikipedia.org/wiki/Augmented_Backus–Naur_Form) courtesy of the [Haskell library](https://hackage.haskell.org/package/bencoding-0.4.3.0/docs/Data-BEncode.html).

```
<BE>    ::= <DICT> | <LIST> | <INT> | <STR>

<DICT>  ::= "d" 1 * (<STR> <BE>) "e"
<LIST>  ::= "l" 1 * <BE>         "e"
<INT>   ::= "i"     <SNUM>       "e"
<STR>   ::= <NUM> ":" n * <CHAR>; where n equals the <NUM>

<SNUM>  ::= "-" <NUM> / <NUM>
<NUM>   ::= 1 * <DIGIT>
<CHAR>  ::= %
<DIGIT> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```
In *pieces* the encoding and decoding of _bencoded_ data is implemented in the `pieces.bencoding` module [source](https://github.com/eliasson/pieces/blob/master/pieces/bencoding.py)

Here are a few examples decoding bencoded data into a Python representation using that module.

```python
>>> from pieces.bencoding import Decoder

# An integer value starts with an 'i' followed by a series of
# digits until terminated with a 'e'.
>>> Decoder(b'i123e').decode()
123

# A string value, starts by defining the number of charactes
# contained in the string, followed by the actual string.
# Notice that the string returned is a binary string, not unicode.
>>> Decoder(b'12:Middle Earth').decode()
b'Middle Earth'

# A list starts with a 'l' followed by any number of objects, until
# terminated with an 'e'.
# As in Python, a list may contain any type of object.
>>> Decoder(b'l4:spam4:eggsi123ee').decode()
[b'spam', b'eggs', 123]

# A dict starts with a 'd' and is terminated with a 'e'. objects
# inbetween those characters must be pairs of string + object.
# The order is significant in a dict, thus OrderedDict (from
# Python 3.1) is used.
>>> Decoder(b'd3:cow3:moo4:spam4:eggse').decode()
OrderedDict([(b'cow', b'moo'), (b'spam', b'eggs')])
```

Likewise, a Python object structure can be encoded into a bencoded byte string using the same module.

```python
>>> from collections import OrderedDict
>>> from pieces.bencoding import Encoder

>>> Encoder(123).encode()
b'i123e'

>>> Encoder('Middle Earth').encode()
b'12:Middle Earth'

>>> Encoder(['spam', 'eggs', 123]).encode()
bytearray(b'l4:spam4:eggsi123ee')

>>> d = OrderedDict()
>>> d['cow'] = 'moo'
>>> d['spam'] = 'eggs'
>>> Encoder(d).encode()
bytearray(b'd3:cow3:moo4:spam4:eggse')
```

These examples can also be found in the [unit tests](https://github.com/eliasson/pieces/blob/master/tests/test_bendoding.py).

The parser implementation is pretty straight forward, no asyncio is used here currently, not even reading the `.torrent` from disk.

_If you read through the bencoding module carefully you will notice that it does not support negative numbers (by specification). I have not seen any negative numbers in use yet, but for correctness the `pieces.bencoding.Encoder` and `pieces.bencoding.Decoder` should be extended with this support._


### Torrent structure

A `.torrent` normally contains more properties than we need for our simple client. Also the properties are slightly different between single- and multi-file torrents - and we only support single-file currently.

Let's decode a real world torrent and see what it contains:

```python
>>> with open('tests/data/ubuntu-16.04-desktop-amd64.iso.torrent', 'rb') as f:
...     meta_info = f.read()
...     torrent = Decoder(meta_info).decode()
...
>>> torrent
OrderedDict([(b'announce', b'http://torrent.ubuntu.com:6969/announce'), (b'announce-list', [[b'http://torrent.ubuntu.com:6969/announce'], [b'http://ipv6.torrent.ubuntu.com:6969/announce']
]), (b'comment', b'Ubuntu CD releases.ubuntu.com'), (b'creation date', 1461232732), (b'info', OrderedDict([(b'length', 1485881344), (b'name', b'ubuntu-16.04-desktop-amd64.iso'), (b'piece
length', 524288), (b'pieces', b'\x1at\xfc\x84\xc8\xfaV\xeb\x12\x1c\xc5\xa4\x1c?\xf0\x96\x07P\x87\xb8\xb2\xa5G1\xc8L\x18\x81\x9bc\x81\xfc8*\x9d\xf4k\xe6\xdb6\xa3\x0b\x8d\xbe\xe3L\xfd\xfd4\...')]))])
```

_I have truncated the `pieces` attribute heavily in order to make this article more readable._

The properties here we are interested in is only five:


```python

# The tracker URL is the HTTP URL we need in order to retrieve a
# list of peers we can connect to.
>>> torrent[b'announce']
b'http://torrent.ubuntu.com:6969/announce'

# The 'info' dict describe the file(s) in this torrent and is slightly
# different between single and multi files. We will only bother about
# single file properties here

# The info's length is the total size (in bytes) for the torrent.
# I.e. this is the number of bytes we need to download.
>>> torrent[b'info'][b'length']
1485881344

# The info's name is the suggested filename for the file we are about
# to download.
>>> torrent[b'info'][b'name']
b'ubuntu-16.04-desktop-amd64.iso'

# The piece length the size of byte per piece this torrent is distributed
# in - more on this later.
>>> torrent[b'info'][b'piece length']
524288

# The info's pieces is *NOT* the actual pieces, but a sequence of SHA1
# hash values for the pieces.
>>> torrent[b'info'][b'pieces']
b'\x1at\xfc\x84\xc8\xfaV\xeb\x12...'
```

Notice how the keys used in the `OrderedDict` are _binary_ strings. Bencoding is a binary protocol, and using UTF-8 strings will not work!

A wrapper class `pieces.torrent.Torrent` exposing these properties is implemented [source](https://github.com/eliasson/pieces/blob/master/pieces/torrent.py) abstracting the binary strings, single vs. multiple file away from the rest of the client.


#### Meta-data's info dict

One important piece of functionality the `pieces.torrent.Torrent`class has, is the SHA1 hasing of the `info` dict. This hash is used when communicating with the tracker and other peers as an identifier for this torrent.

It is the entire contents of the torrents _meta-info's_ info object that should be hased - not the entire torrent. This is why the use of `OrderedDict` is so _important_ when parsing the bencoded `.torrent`. If order is not respected, the hash might be invalid for this torrent.

The solution _pieces_ use for calculating the SHA1 is to encode the `info` dict to a bencoded sequence of bytes again, and then hash that value.

Look at [source](https://github.com/eliasson/pieces/blob/master/pieces/torrent.py) on how this is implemented.


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
