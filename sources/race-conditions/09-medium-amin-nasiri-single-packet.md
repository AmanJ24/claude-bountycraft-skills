---
title: "Dive into Single Packet Attack"
author: Amin Nasiri (nxenon)
url: https://infosecwriteups.com/dive-into-single-packet-attack-3d3849ffe1d2
date_published: 2023-11-24
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

(Medium navigation/footer chrome trimmed. Full article body was publicly rendered; not paywalled. Images omitted, captions noted.)

Published in InfoSec Write-ups. Tags: Single Packet Attack, Race Condition, Http2, Web Security, Web Security Tools. 6 min read.

# Dive into Single Packet Attack

An Overview and a Tool for Exploiting Single Packet Attack

## Overview

After watching [James Kettle's presentation at DEF CON 31](https://youtu.be/tKJzsaB1ZvI?si=W6trjxYI9-BIFnTq), I started to research more on Race Conditions and specifically on Single Packet Attack. In the end, I created a tool(Library) called [H2SpaceX](https://github.com/nxenon/h2spacex) to exploit this type of race condition on HTTP/2 in addition to James Kettle implementation which is [SpikeEngine.kt](https://github.com/PortSwigger/turbo-intruder/blob/master/src/SpikeEngine.kt) in Burp Turbo Intruder extension. My goal was to research and understand HTTP/2 and Single Packet Attack more and have a tool to test new techniques and more research on the topic. It also allows you to do some automation, for example, read and process the response of sent requests and then decide what to do based on responses. You can also integrate the tool into your environment.

## Network Packets

So what exactly is a Packet?

[TechTarget](https://www.techtarget.com/searchnetworking/definition/packet):

> A network packet is a basic unit of data that's grouped together and transferred over a computer network, typically a packet-switched network, such as the internet.

Packets in Wireshark (each row is **a packet**): I am showing decrypted HTTP/2 request packets in this picture:

[Image: Packets in Wireshark]

## HTTP/2 Multiplexing

You can see [RFC 9113](https://httpwg.org/specs/rfc9113.html) which is for HTTP/2 to get more information about H2 and multiplexing itself. The descriptions here are from the RFC mentioned above.

In HTTP/2 we can have multiplexing with a new feature called [Frames](https://httpwg.org/specs/rfc9113.html#FramingLayer), each frame has an ID called [Stream Identifier](https://httpwg.org/specs/rfc9113.html#StreamIdentifiers). Requests are separated by Stream IDs and for this reason, we have no Head of Line Blocking at HTTP/2 compared to HTTP/1.

[Image: Request Frames and Streams in HTTP/2]

You can see in this picture that multiple Frames with different Stream IDs are in a **single** packet in Wireshark:

[Image: Frames in a single packet]

## Nagle's Algorithm

Nagle's algorithm helps **have more bytes(requests) in a single packet** to improve the efficiency of TCP/IP networks by removing the "small packet problem".

[Wikipedia](https://en.wikipedia.org/wiki/Nagle%27s_algorithm):

> **Nagle's algorithm** is a means of improving the efficiency of TCP/IP networks by reducing the number of packets that need to be sent over the network. It was defined by John Nagle while working for Ford Aerospace. It was published in 1984 as a Request for Comments with title _Congestion Control in IP/TCP Internetworks_ in RFC 896.
>
> The RFC describes what Nagle calls the "small-packet problem", where an application repeatedly emits data in small chunks, frequently only 1 byte in size. Since TCP packets have a 40-byte header (20 bytes for TCP, 20 bytes for IPv4), this results in a 41-byte packet for 1 byte of useful information, a huge overhead. This situation often occurs in Telnet sessions, where most keypresses generate a single byte of data that is transmitted immediately. Worse, over slow links, many such packets can be in transit at the same time, potentially leading to congestion collapse.

To enable Nagle's Algorithm, you can just disable TCP_NODELAY option. In Python, simply disable TCP_NODELAY on your TCP socket object:

[Image: Disable Nagle's Algorithm]

## Last-byte Sync

In last-byte technique, you send the whole request except the last byte, and in the end, you send the last byte of all requests together.

[James Kettle](https://x.com/albinowax/status/1110202126196719618?s=20):

> Synced last-byte is a technique to mitigate network congestion and ensure a group of requests get processed by the target server simultaneously. You send everything but the final byte of each request, wait till that's done, then send a single packet to complete each request.

## Single Packet Attack

By combining these techniques on HTTP/2, we can have a great **Race Condition**. But before all, I want to explain why the name is like that. Why Single Packet Attack? Single Packet?

### What Is Jitter?

Network jitter refers to the **variation** in the **delay of received packets** in a network. In other words, it measures the variability of packet arrival time.

OK, we said that it means: a variation in the delay of received **packets,** not a single packet. So when we send multiple packets, they do not arrive at the same time because there is network jitter and we can not easily and without trying many times to send packets that are arrived together.

### Single Packet

If we have a single packet, network jitter is not a problem, because there are no other packets that need to be sent together. **All requests are in a single packet**. The "Single Packet Attack" is named like that because in this attack we do not have network jitter and all requests are in a single packet and not multiple packets.

## How to Put Multiple Requests in a Single Packet

Thanks to HTTP/2 Multiplexing feature we can send multiple frames in a single packet without Head of Line Blocking(Mixing up requests wrongly). But we have a limit! In a TCP Packet, we can have approximately 1500 bytes of data. So we have a limit, for example, how many POST requests with body we can send in a single packet! If the body is so big maybe we can just put 1 request in a packet! So what is the solution?

With the last-byte sync method + enabling Nagle's Algorithm mentioned above, we can do Single Packet Attack more easily and put about 20 requests in a single packet. We can send all requests without the last byte for each request, wait some time, and send the remaining byte of each request all together.

Normal POST request with data is like this:

[Image: A Normal Request]

We keep the last byte of the last DATA frame and remove the END_STREAM flag. New Request frames are like this:

[Image: Frames in Single Packet Attack]

I show the steps:

1. Create all [HEADERS](https://httpwg.org/specs/rfc9113.html#HEADERS) frames with [DATA](https://httpwg.org/specs/rfc9113.html#DATA) frames (each request has a Stream Identifier. HEADERS frame and DATA frames for a request have the same Stream ID)
2. Remove the END_STREAM flag from the last DATA frame of each Stream and **create** new DATA frame with last byte in it and END_STREAM flag set to 1
3. Wait some time e.g 100 ms
4. Send a [PING](https://httpwg.org/specs/rfc9113.html#PING) frame to warm up the HTTP/2 connection
5. Send all remaining new created Data frames containing the last remaining byte

## Single Packet Attack Tool H2SpaceX

I created a tool(library) called [H2SpaceX](https://github.com/nxenon/h2spacex) for exploiting this type of race condition for two reasons:

1. Research more on Single Packet Attack & HTTP/2
2. Customize and do some automation based on responses if it is needed

I explained all in the [Wiki](https://github.com/nxenon/h2spacex/wiki/Quick-Start-Examples) if you would like to use my tool and also have some [examples](https://github.com/nxenon/h2spacex/tree/main/examples) for using the tool and solve Portswigger race condition labs.

[Image: Example of H2SpaceX]

## About Me

I'm Mohammad Amin Nasiri (Xenon), a Application Security Engineer with 2+ years of hands-on security assessment and auditing experience, trying to expand my hacking skills and research on web security. Find me on Github (nxenon), LinkedIn, and Twitter (0xnxenon).
