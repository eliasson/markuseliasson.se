+++
Categories = ["Code"]
Description = "Starting the bittorrent implementation in go"
Tags = ["go", "bittorrent"]
draft = false
date = "2014-11-10T20:32:02+01:00"
title = "Bencoding in Blackbird"
+++

As I previously blogged about (months ago), I decided to try to learn go by implementing a
small bittorrent client. As usual, it turns out that you have far less time
to learn new stuff than you wish. But nonetheless I managed to get a few hours
to read up on and to start implement support for bencoding.


For some reason I was not to fond of the name *lox* for my little project
so I renamed it to *Blackbird*. It's nothing macho about the rename, the thing
is that Blackbirds (Koltrast in Swedish) is a flocking bird known for flying
in big, sometimes huge, swarms.


Blackbird is far from complete, the only supported actions at the moment is
to open and parse a .torrent file and print the meta info to stdout.


    ./blackbird info --torrent=ubuntu-14.10-server-amd64.iso.torrent
    INFO:   2014/11/10 09:49:18 info.go:36: Opening .torrent file: ubuntu-14.10-server-amd64.iso.torrent
    Torrent:
        Announce: http://torrent.ubuntu.com:6969/announce
        Pieces length: 524288
        Name: ubuntu-14.10-server-amd64.iso
        Length: 610271232


The code, which is still work in progress, is available at
[github.com/eliasson/blackbird](https://github.com/eliasson/blackbird)

The next task is to connect to the Tracker and read some stats on the torrent.