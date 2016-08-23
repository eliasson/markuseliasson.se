+++
date = "2016-08-22T22:58:17+02:00"
title = "A BitTorrent client in Python 3.5"
draft = true
Categories = ["Code"]
Tags = ["Python", "BitTorrent"]
Description = "Python 3.5 comes with support for asynchronous IO, which seems like a perfect fit when implementing a BitTorrent client. This article will guide you through the BitTorrent protocol details while showcasing how a small client was implemented using it."
+++
When Python 3.5 was relesed together with the new module asyncio I was curios to give it a try. Recently I decided to implement a simple BitTorrent client using asyncio - I have always been interested in peer-to-peer protocols and it seemed like a perfect fit.

The project is named **Pieces**, all of the source code is available at [GitHub](https://github.com/eliasson/pieces) and released under the Apache 2 license. Feel free to learn from it, steal from it, improve it, laugh at it or just ignore it.

I previously posted a short introduction to Python's async module. If this is your first time looking at `asyncio` it might be a good idea to read through that one first.


## An introduction to BitTorrent

BitTorrent has been around since 2001 when [Bram Cohen](https://en.wikipedia.org/wiki/Bram_Cohen) authored the first version of the protocol. The big breakthrough was when sites as _The Pirate Bay_ made it popular to use for downloading pirated material. Streaming sites, such as Netflix, might have resulted in a decrease of people using BitTorrent for downloading movies. But BitTorrent is still used in a number of different, legal, solutions where distribution of larger files are important.

* [Facebook](https://torrentfreak.com/facebook-uses-bittorrent-and-they-love-it-100625/) use it to distribute updates within their huge data centers
* [Amazon S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/S3Torrent.html) implement it for downloading of static files
* Traditional downloads still used for larger files such as [Linux distrubutions](http://www.ubuntu.com/download/alternative-downloads)

BitTorrent is a peer-to-peer protocol, where _peers_ join a _swarm_ of other peers to exchange pieces of data between each other. Each peer is connected to multiple peers at the same time, and thus downloading or uploading to multiple peers at the same time. This is great in terms of limiting bandwidth compared to when a file is downloaded from a central server it is also great for keeping a file available as it does not rely on a single source.

TODO: Give slightly more details on that each torrent is split into pieces.

While going through the implementation it might be good to have read, or to have another tab open with the [Unofficial BitTorrent Specification](https://wiki.theory.org/BitTorrentSpecification). This is without a doubt the best source of information on the BitTorrent protocol. The official specification is vague and lacks certain details so the unofficial is the one you want to study.


### Parsing a .torrent file

The first thing a client needs to do is to find out what it is supposed to download and from where. This information is what is stored in the `.torrent` file, a.k.a. the _meta-info_.

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
In *pieces* the encoding and decoding of _bencoded_ data is implemented in the `pieces.bencoding` module ([source code](https://github.com/eliasson/pieces/blob/master/pieces/bencoding.py)).

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

The parser implementation is pretty straight forward, no asyncio is used here though, not even reading the `.torrent` from disk.

Using the parser from `pieces.bencoding`, let's open the `.torrent` for the popular Linux distribution Ubuntu:

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

Notice how the keys used in the `OrderedDict` are _binary_ strings. Bencoding is a binary protocol, and using UTF-8 strings as keys _will not work_!

A wrapper class `pieces.torrent.Torrent` exposing these properties is implemented abstracting the binary strings, single vs. multiple file away from the rest of the client. This class only implements the attributes used in pieces client.

I will not go through which attributes that is available, instead the rest of this article will refer back to attributes found in the `.torrent` / _meta-info_ where used.


### Connecting to the tracker

Now that we can decode a `.torrent` file and we have a Python representation of data within we need to get a list of peers to connect with. This is where the tracker comes in. The tracker is a central server keeping track on available peers for a given torrent. The tracker does **NOT** contain any of the torrent data, only which peers that can be connected to and their statistics.


#### Building the request

The `announce` property in the _meta-info_ is the HTTP URL to the tracker to connect to using the following URL parameters:

Parameter   | Description
------------|------------
info_hash   | The SHA1 hash of the info dict found in the `.torrent`
peer_id     | A unique ID generated for this client
uploaded    | The total number of bytes uploaded
downloaded  | The total number of bytes downloaded
left        | The number of bytes left to download for this client
port        | The TCP port this client listens on
compact     | Whether or not the client accepts a compacted list of peers or not

The peer_id needs to be exactly 20 bytes, and there are two major conventions used on how to generate this ID. Pieces follows the [Azureus-style](https://wiki.theory.org/BitTorrentSpecification#peer_id) convention generating peer id like:

````python
>>> import random
# -<2 character id><4 digit version number>-<random numbers>
>>> '-PC0001-' + ''.join([str(random.randint(0, 9)) for _ in range(12)])
'-PC0001-478269329936'
````

A tracker request can look like this using [httpie](https://github.com/jkbrzt/httpie):

````http
http GET "http://torrent.ubuntu.com:6969/announce?info_hash=%90%28%9F%D3M%FC%1C%F8%F3%16%A2h%AD%D85L%853DX&peer_id=-PC0001-706887310628&uploaded=0&downloaded=0&left=699400192&port=6889&compact=1"
HTTP/1.0 200 OK
Content-Length: 363
Content-Type: text/plain
Pragma: no-cache

d8:completei3651e10:incompletei385e8:intervali1800e5:peers300:£¬%ËÌyOkÝ.ê@_<K+Ô\Ý Ámb^TnÈÕ^AËO*ÈÕ1*ÈÕ>¥³ÈÕBä)ðþ¸ÐÞ¦Ô/ãÈÕÈuÉæÈÕ
...
````

_The response data is truncated since it contains binary data that screws up the Markdown formatting._

From the tracker response, there is two properties of interest:

* **interval** - The interval in seconds until the client should make a new announce call to the tracker.
* **peers** - The list of peers is a binary string with a length of multiple of 6 bytes. Where each peer consist of a 4 byte IP address and a 2 byte port number (since we are using the compact format).

So, a successful announce call made to the tracker, gives you a list of peers to connect to. This might not be all available peers in this swarm, only the peers the tracker assigned your client to connect. A subsequent call to the tracker might result in another list of peers.

Python does not come with a built-in support for async HTTP and my beloved [requests library](https://github.com/kennethreitz/requests) does not implement asyncio either. Scouting around the Internet it looks like most use [aiohttp](https://github.com/KeepSafe/aiohttp), which implement both a HTTP client and server.


#### Async HTTP

Pieces use aiohttp in the `pieces.tracker.Tracker` class for making the HTTP request to the tracker announce url. A shortened version of that code reveil this:

````python
async def connect(self,
                    first: bool=None,
                    uploaded: int=0,
                    downloaded: int=0):
    params = { ...}
    url = self.torrent.announce + '?' + urlencode(params)

    async with self.http_client.get(url) as response:
        if not response.status == 200:
            raise ConnectionError('Unable to connect to tracker')
        data = await response.read()
        return TrackerResponse(bencoding.Decoder(data).decode())
````
The method is declared using `async` and uses the new [asynchronous context manager](https://www.python.org/dev/peps/pep-0492/#asynchronous-context-managers-and-async-with) `async with` to allow being suspended while the HTTP call is being made. Given a successful response, this method will be suspended again while reading the binary response data `await response.read()`.

The result of this is that our event loop is free to schedule other work while we have an outstanding request to the tracker.

See the module `pieces.tracker` [source code](https://github.com/eliasson/pieces/blob/master/pieces/tracker.py) for full details.


### The loop

Everything up to this point could really have been made synchronously, but now that we are about to connect to multiple peers we need to go asynchronous.

The main function in `pieces.cli` is responsible for setting up the asyncio event loop. If we get rid of some `argparse` and error handling details it would look something like this (see [cli.py](https://github.com/eliasson/pieces/blob/master/pieces/cli.py) for the full details).

````python
import asyncio

from pieces.torrent import Torrent
from pieces.client import TorrentClient

loop = asyncio.get_event_loop()
client = TorrentClient(Torrent(args.torrent))
task = loop.create_task(client.start())

try:
    loop.run_until_complete(task)
except CancelledError:
    logging.warning('Event loop was canceled')
````

We start off by getting the default event loop for this thread. Then we construct the `TorrentClient` with the given `Torrent` (meta-info). This will parse the `.torrent` file and validate everything is ok.

Calling the `async` method `client.start()` and wrapping that in a `asyncio.Future` and later adding that future and instructing the event loop to keep running until that task is complete.

Is that it? No, not really - we have our own loop (**not** event loop) implemented in the `pieces.client.TorrentClient` that sets up the peer connections, schedules the announce call, etc.

`TorrentClient` is something like a work coordinator, it starts by creating a [async.Queue](https://docs.python.org/3/library/asyncio-queue.html) which will hold the list of available peers that can be connected to.

Then it constructs _N_ number of `pieces.protocol.PeerConnection` which will consume peers from off the queue. These `PeerConnection` instances will wait (`await`) until there is a peer available in the `Queue` for one of them to connect to (_not blocking_).

Since the queue is empty to begin with, no `PeerConnection` will do any real work until we populate it with peers it can connect to. This is done in a loop inside of `TorrentClient.start`.

Lets have a look at this loop:

````python
async def start(self):
    self.peers = [PeerConnection(self.available_peers,
                                    self.tracker.torrent.info_hash,
                                    self.tracker.peer_id,
                                    self.piece_manager,
                                    self._on_block_retrieved)
                    for _ in range(MAX_PEER_CONNECTIONS)]

    # The time we last made an announce call (timestamp)
    previous = None
    # Default interval between announce calls (in seconds)
    interval = 30*60

    while True:
        if self.piece_manager.complete:
            break
        if self.abort:
            break

        current = time.time()
        if (not previous) or (previous + interval < current):
            response = await self.tracker.connect(
                first=previous if previous else False,
                uploaded=self.piece_manager.bytes_uploaded,
                downloaded=self.piece_manager.bytes_downloaded)

            if response:
                previous = current
                interval = response.interval
                self._empty_queue()
                for peer in response.peers:
                    self.available_peers.put_nowait(peer)
        else:
            await asyncio.sleep(5)
    self.stop()
````

Basically, what that loop does is to:

1. Check if we have downloaded all pieces
2. Check if user aborted download
3. Make a annouce call to the tracker if needed
4. Add any retrieved peers to a queue of available peers
5. Sleep 5 seconds

So, each time an announce call is made to the tracker, the list of peers to connect to is reset, and if no peers are retrieved, no `PeerConnection` will run. This goes on until the download is complete or aborted.


### The peer protocol

After receiving a peer IP and port-number from the tracker, our client will to open a TCP connection to that peer. Once the connection is open, these peers will start to exchange messages using the peer protocol.

First, lets go through the different parts of the peer protocol, and then go through how it is all implemented.


#### Handshake

The first message sent needs to be a `Handshake` message, and it is the connecting client that is responsible for initiating this.

Immediately after sending the Handshake, our client should receive a Handshake message sent from the remote peer.

The `Handshake` message contains two fields of importance:

* **peer_id** - The unique ID of either peer
* **info_hash** - The SHA1 hash value for the info dict

If the `info_hash` does not match the torrent we are about to download, we
close the connection.

Immediately after the Handshake, the remote peer _may_ send a `BitField` message. The `BitField` message serves to inform the client on which pieces the remote peer have. Pieces support receiving a `BitField` message, and most BitTorrent clients seems to send it - but since pieces currently does not support seeding, it is never sent, only received.

The `BitField` message payload contains a sequence of bytes that when read binary each bit will represent one piece. If the bit is `1` that means that the peer _have_ the piece with that index, while `0` means that the peer _lacks_ that piece. I.e. Each byte in the payload represent up to 8 pieces with any spare bits set to `0`.

Each client starts in the state _choked_ and _not interested_. That means that the client is not allowed to request pieces from the remote peer, nor do we have intent of being interested.

* **Choked** A choked peer is not allowed to request any pieces from the other peer.
* **Unchoked** A unchoked peer is allowed to request pieces from the other peer.
* **Interested** Indicates that a peer is interested in requesting pieces.
* **Not interested** Indicates that the peer is not interested in requesting pieces.

_Consider **Choked** and **Unchoked** to be rules and **Interested** and **Not Interested** to be intents between two peers._

After the handshake we send an `Interested` message to the remote peer, telling that we would like to get _unchoked_ in order to start requesting pieces.

Until the client receives an `Unchoke` message - it may **not** request a piece from its remote peer - our `PeerConnection` will be choked (passive) until either _unchoked_ or _disconnected_.

The following sequence of messages is what we are aiming for when setting up a `PeerConnection`:

````
              Handshake
    client --------------> peer    We are initiating the handshake

              Handshake
    client <-------------- peer    Comparing the info_hash with our hash

              BitField
    client <-------------- peer    Might be receiving the BitField

             Interested
    client --------------> peer    Let peer know we want to download

              Unchoke
    client <-------------- peer    Peer allows us to start requesting pieces
````


#### Requesting pieces

As soon as the client gets into a _unchoked_ state it will start requesting pieces from the connected peer. The details surrounding which piece to request is detailed later, in [Managing the pieces](#managing-the-pieces).

If we know that the other peer have a given piece, we can send a `Request` message asking the remote peer to send us data for the specified piece. If the peer complies it will send us a corresponding `Piece` message where the message payload is the raw data.

This client will only ever have one outstanding `Request` per peer and politely wait for a `Piece` message until taking the next action. Since connections to multiple peers are open concurrently, the client will have multiple `Requests` outstanding but only one per connection.

If, for some reason, the client do not want a piece anymore, it can send a `Cancel` message to the remote peer to cancel any previously sent `Request`.


#### Other messages

##### Have

The remote peer can at any point in time send us a `Have` message. This is done when the remote peer have received a piece and makes that piece available for its connected peers to download.

The `Have` message payload is the piece index.

When pieces receive a `Have` message it updates the information on which pieces the peer has.


##### KeepAlive

The `KeepAlive` message can be sent at anytime in either direction. The message does not hold any payload.

#### Implementation

The `PeerConnection` opens a TCP connection to a remote peer using `asyncio.open_connection` to asynchronously open a TCP connection and returns a tuple of `StreamReader` and a `StreamWriter`. Given that the connection was created successfully, the `PeerConnection` will send and receive a `Handshake` message.

Once a handshake is made, the PeerConnection will use an asynchronous iterator  to return a stream of `PeerMessages` and take the appropiate action. Remember that _async_ & _await_ is _coroutine_ & _yield_ which is implemented using a _generator_, which in turn is a special kind of _iterator_ - it make.

Using an async iterator separates the `PeerConnection` from the details on how to read from sockets and how to parse the BitTorrent binary protocol. The `PeerConnection` can focus on the semantics regarding the protocol - such as managnig the peer state, receiving the pieces, closing the connection.

This allows the main code in `PeerConnection.start` to basically look like:

````python
async for message in PeerStreamIterator(self.reader, buffer):
    if type(message) is BitField:
        self.piece_manager.add_peer(self.remote_id, message.bitfield)
    elif type(message) is Interested:
        self.peer_state.append('interested')
    elif type(message) is NotInterested:
        if 'interested' in self.peer_state:
            self.peer_state.remove('interested')
    elif type(message) is Choke:
        ...
````

An [asynchronous iterator](https://www.python.org/dev/peps/pep-0492/#asynchronous-iterators-and-async-for) is a class that implements the methods `__aiter__` and `__anext__` which is just async versions of Pythons standard iterators that have `__iter__` and `next`.

Upon iterating (calling next) the `PeerStreamIterator` will read data from the `StreamReader` and if enough data is available try to parse and return a valid `PeerMessage`.

The BitTorrent protocol uses messages with variable length, where all messages takes the form:

````
<length><id><payload>
````

* **length** is a 4 byte integer value
* **id** is a single decimal byte
* **payload** is variable and message dependent

So as soon as the buffer have enough data for the next message it will be parsed and returned from the iterator.

All messages are decoded using Python's module `struct` which contains functions to convert to and from Pythons values and C structs. [Struct](https://docs.python.org/2/library/struct.html) use compact strings as descriptors on what to convert, e.g. `>Ib` reads as "Big-Endian, 4 byte unsigned integer, 1 byte character. All messages uses Big-Endian.

This makes if easy to create unit tests to encode and decode messages. Let's have a look on the tests for the `Have` message:

````python
class HaveMessageTests(unittest.TestCase):
    def test_can_construct_have(self):
        have = Have(33)
        self.assertEqual(
            have.encode(),
            b"\x00\x00\x00\x05\x04\x00\x00\x00!")

    def test_can_parse_have(self):
        have = Have.decode(b"\x00\x00\x00\x05\x04\x00\x00\x00!")
        self.assertEqual(33, have.index)
````

From the raw binary string we can tell that the Have message have a length of 5 byets `\x00\x00\x00\x05` an id of value 4 `\x04` and the payload is 33 `\x00\x00\x00!`.

Since the message length is 5 and ID only use a single byte we know that we have four bytes to interpret as the payload value. Using `struct.unpack` we can easily convert it to a python integer like:

````python
>>> import struct
>>> struct.unpack('>I', b'\x00\x00\x00!')
(33,)
````

That is basically it regarding the protocol, all messages follow the same procedure and the iterator keeps reading from the socket until it gets disconnected. See the [source code](https://github.com/eliasson/pieces/blob/master/pieces/protocol.py) for details on all messages.

### Managing the pieces

So far we have only discussed pieces - pieces of data being exchanged by two peers. It turns out that pieces is not the entire truth, there is one more concept - _blocks_. If you have looked through any of the source code you might have seen code refering to blocks, so lets go through what a _piece_ really is.

A _piece_ is, unsurprisingly, a partial piece of the torrents data. A torrent's data is split into _N_ number of pieces of equal size (except the last piece in a torrent, which might be of smaller size than the others). The piece length is specified in the `.torrent` file. Typically pieces are of sizes 512 kB or less, and should be a power of 2.

Pieces are still too big to be shared efficiently between peers, so pieces are further divided into something refered to as _blocks_. Blocks is the chunks of data that is actually requested between peers, but pieces are still used to indicate which peer that have which pieces. If only blocks should have been used it would increase the overhead in the protocol greatly (resulting in longer BitFields, more Have message and larger `.torrent` files).

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

Exachanging these blocks between peers is basically what BitTorrent is about. Once all blocks for a piece is done, that piece is complete and can be shared with other peers (the `Have` message is sent to connected peers). And once all pieces are complete the peer transform from a _downloader_ to only be a _seeder_.

Two notes on where the official specification is a bit off:

1. _The official specification refer to both pieces and blocks as just pices which is quite confusing. The unofficial specification and others seem to have agreed upon using the term block for the smaller piece which is what we will use._

2. _The official specification is stating another **block size** that what we use. Reading the unofficial specifcation, it seems that 2^14 bytes is what is agreed among implementers - regardless of the official specification._


### The implementation

When a `TorrentClient` is constructed, so is a `PieceManager ` the resposibility to:

* Determine which block to request next
* Persisting received blocks to file
* Determine when a download is complete.

When a `PeerConnection` successfully _handshakes_ with another peer and receives a `BitField` message it will inform the `PieceManager` which peer (`peer_id`) that have which pieces. This information will be updated on any received `Have` message as well. Using this information, the `PeerManager` knows the collective state on which pieces that are available from which peers.

When the first `PeerConnection` goes into a _unchoked_ state it will request the next block from its peer. The next block is determined by calling the method `PieceManager.next_request`.

The `next_request` implements a very simple strategy on which piece to request next.

1. When the `PieceManager` is constructed all pieces and blocks are pre-constructed based on the piece length from the `.torrent` meta-info
2. All pieces are put in a missing list
3. When `next_request` is called, the manager will do one of:
    - Re-request any previously requested block that has timed-out
    - Requst the next block in an ongoing piece
    - Request the first block in the next missing piece

This way the blocks and pieces will be requsted in order. However, multiple pieces might be ongoing based on which piece a client have.

Since pieces aims to be a simple client, no effort have been made on implementing a smart or efficient strategy for which pieces to request. A better solution would be to request the rarest piece first, which would make the entire swarm healthier.

Whenever a block is received from a peer, it is stored (in memory) by the PieceManager. When all blocks for a piece is retrieved, a SHA1 hash is made on the piece. This hash is compared to the SHA1 hashes include in the `.torrent` info dict - if it matches the piece is written to disk.

When all pieces are accounted for (matching hashes) the torrent is considered to be complete, which stops the `TorrentClient` looping closing any open TCP connection and as a result the program exits with a message that the torrent is downloaded.

### Future work

Seeding is not yet implemented, but it should not be that hard to implement. What is needed is something along the lines of this:

* Whenever a peer is connected to, we should send a `BitField` message to the remote peer indicating which pieces we have.

* Whenever a new piece is received (and correctness of hash is confirmed), each `PeerConnection` should send a `Have` message to its remote peer to indicate the new piece that can be shared.

In order to do this the `PieceManager` needs to be extended to return a list of 0 and 1 for the pieces we have. And the `TorrentClient` to tell the `PeerConnection` to send a `Have` to its remote peer. Both `BitField` and `Have` messages should support encoding of these messages.

Having seeding implemented would make Pieces a good citizen, supporting both downloading and uploading of data within the swarm.

Additional features that probably can be added without too much effort is:

* **Multi-file torrent**, will hit `PieceManager`, since Pieces and Blocks might span over multiple files, it affects how files are persisted (i.e. a single block might contain data for more than one file).

* **Resume a download**, by seeing what parts of the file(s) are already downloaded (verified by making SHA1 hashes).

* **UDP** support and not only HTTP when connecting to a tracker.


## Summary

It was real fun to implement a BitTorrent client, having to handle binary protocols and networking was great to balance all that recent web development I have been doing.

Python continous to be one of my favourite programming language. Handling binary data was a breeze given the `struct` module and the recent addition `asyncio` feels very pythonic. Using _async iterator_ to implement the protocol  turned out to be a good fit as well.

Hopefully this article inspired you to write a BitTorrent client of your own, or to extend pieces in some way. If you spot any error in the article or the source code, feel free to open an issue over at [GitHub](https://github.com/eliasson/pieces/).
