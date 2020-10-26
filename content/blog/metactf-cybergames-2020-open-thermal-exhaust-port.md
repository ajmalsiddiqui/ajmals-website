---
title: "MetaCTF CyberGames 2020: Open Thermal Exhaust Port"
date: 2020-10-25T21:04:16+05:30
lastmod: 2020-10-25T21:04:16+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2018/05/14/16/54/cyber-3400789_960_720.jpg
categories:
  - ctf writeups
tags:
  - security
  - forensics
  - wireshark
  - packet analysis
  - metactf cybergames 2020
showcase: false
draft: false
---

This is the write-up for the "Open Thermal Exhaust Port" forensics challenge from [MetaCTF CyberGames 2020][1], which was worth 275 points.

Challenge statement: Our TCP connect Nmap scan found some open ports it seems. We may only have a pcap of the traffic, but I'm sure that won't be a problem! Can you tell us which ones they are? The flag will be the sum of the open ports. For example, if ports 25 and 110 were open, the answer would be MetaCTF{135}.

<!--more-->

This is a basic forensics challenge intended to be an introduction to packet analysis tools like wireshark. The challenge is simple: we have a pcap (packet capture) file from a TCP connect nmap scan, and we need to figure out the sum of the open ports on the target that was scanned.

Let's dive in!

> Feel free to skip to the Analysing the Pcap section if you already know all the basics in the upcoming sections!

## Terms and Definitions

You should know a few (basic) things in order to understand and work out this problem.

1. Port scanning - scanning a machine (called a "host") or many hosts for open ports.
2. nmap - Perhaps the most popular port scanner out there. Extremely powerful, extensible, and very widely used.
3. pcap - Refers to packet capture files. These are files that contain a record of all the packets that went through a network interface. In other words, it's like a log of network activity.
4. wireshark - A popular and powerful tool for analysing pcap files that lets you filter through packet, construct powerful, fine grained queries to find packets, etc. Also used as a network packet sniffer.

## So What is a TCP Connect?

Before we even look at the network packets, let's go over some TCP knowledge that we'll need.

The challenge description tells us that we're looking at the pcap dump of a TCP connect scan done using nmap. This just refers to how a connection is established between two hosts via TCP. In TCP, a connection is established via a process called the TCP 3-Way Handshake. This is what is looks like when a client connects with a server:

1. The client sends a `SYN` message to the server. `SYN` refers to a Synchronize Sequence Number message. Without diving into the nitty gritty of this, it is a way of telling the server what sequence number the client packets will start with. This is important because the server will have to keep track of the order of packets.
2. The server responds with a `SYN + ACK` message. This is a combination of 2 messages in a single response (a technique known as piggybacking, which helps reduce the number of messages that need to be exchanged). The first message is a `SYN`, just like the previous one. This time, the server tells the client what sequence number packets coming from the server will start from. The second message is an `ACK`, for "Acknowledgement". This is how a server indicates to the client that it received the previous message.
3. The handshake completes with the client sending a TCP `ACK` message, which indicates to the server that the client also received the servers reply.

At this point, the client and the server can exchange data.

But what happens if the server does not want to connect?

In this case, during the second step, the server replies with `RST + ACK`, where the `RST` means "Reset". This is how the server says, "I acknowledge that I got your message but you cannot connect with me".

There's obviously a lot more to TCP, but this is all we need to solve this challenge. So let's move on.

## Setting Things Up

Setup is simple. You just grab the pcap file from the CTF challenge page, and install wireshark on your system.

## Analysing the Pcap

Let's start looking at what we have. Load the pcap into wireshark.

If you're starting wireshark from the terminal, like I did, you can give the pcap filename as an argument to wireshark. 

```
wireshark nmap_scan.pcapng
```

Wireshark starts with an interface that looks like this:

![Opening the pcap in wireshark](https://i.ibb.co/6Xv9qbK/wireshark1.png)

You can immediately see that we have the most crucial stuff in a nice table - source and destination IPs, the protcol, and the information about that packet. Since we're looking at TCP packets, you can see some of the messages we discussed about in the TCP connect primer section.

Now let's figure out what the open ports are. When nmap does a TCP connect scan, it just tries to connect to every port on it's list. When a connection succeeds, the server responds with a `SYN + ACK`. When it fails, it responds with an `RST + ACK`. So our task is to look at all the replies from the server where the source IP is the server IP and the info has a `SYN + ACK`. Whenever we find such a packet, we understand that the source port is open. And hence we add all the open ports up, and that's the flag.

At this point, you can start scrolling through the hundreds of packets looking for what you need...

...or you can be smart and use wireshark the way it is supposed to be used!

What we need to do is filter out the packets that we need. Let's see how we can do that.

Towards the top of the wireshark interface, right below the buttons, you'll see a long input box with the placeholder text, "Apply a display filter...<Ctrl-/>". You can use this to filter packets using a special query language within wireshark.

We can see that the client IP is `10.0.2.15` and the server IP is `10.0.2.6`.

For starters, we're interested only in server replies, so let's narrow down to only those packets which came from the server. Enter this in the display filter input and press enter. Note that the input box becomes green when a valid query is entered:

```
ip.src == 10.0.2.6
```

We now see this:

![querying based on the source IP](https://i.ibb.co/V3V8jkq/wireshark2.png)

This filters out all the packets which have a source IP of 10.0.2.6 (i.e. our server). The display now contains far fewer packets, and all the packets are either `SYN + ACK` or `RST + ACK`. This is actually good enough for us to scroll through and add up all the open ports (i.e. the packets with `SYN + ACK`), but let's refine our query a little to filter out the packets that are from closed ports.

Try this query:

```
ip.src == 10.0.2.6 and tcp.flags.syn == 1
```

![querying based on the source IP and the TCP SYN flag set](https://i.ibb.co/5s22wZy/wireshark3.png)

This query adds a new condition stating that the `SYN` flag (yes, it is technically a flag) should be one, and then combines it with our previous query using the `and` operator. Effectively, we are looking for packets which have a source IP matching that of the server and having the SYN flag set. We now see exactly what we need!

Whip out your calculator, add up all the source ports (be careful to avoid duplicates), and you get the flag!

For completeness, the sum is 80 + 443 + 23 + 21 + 53 + 22 + 3128 = 3770.
The flag is `MetaCTF{3770}`.

Which leaves us with only one task: figuring out why this challenge was called "Open Thermal Exhaust Port"...

  [1]: https://metactf.com/cybergames "MetaCTF CyberGames 2020"
