---
title: "My 2020 in Tech"
date: 2021-01-01T14:23:46+05:30
lastmod: 2021-01-01T14:23:46+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2014/05/27/23/32/matrix-356024_960_720.jpg
categories:
  - miscellaneous
tags:
  - misc
  - programming-languages
showcase: true
draft: false
---

2020 has easily been the most profound year for me as a tech enthusiast. I graduated with a CS degree, got an amazing job, picked up 2 new languages (Go and Clojure), revisited 2 other langauges that I _thought_ I knew but clearly didn't (C and Rust), and explored many new domains (most notably cybersecurity and cloud engineering). This post is a loosely organized collection of my thoughts and experiences related to each of these items, as well as some insight into how I achieved some of the things I achieved this year.

So if you're going to be kind enough to indulge me, let's start talking!

<!--more-->

## Life Updates: Graudating, Interning and Getting My First Job

As of July 2020, I was officially a graduate of Vellore Institute of Technology (VIT), Vellore with a degree in CS. While I don't hold the academic side of CS education (at least here) in high regard, VIT became the place where I found many likeminded peers, inspiring seniors and friends for life. I'll take this chance to say, with utmost honesty, that I find an academic education in CS to be insufficient, and one of the (many) things I credit for what little ability I have as a CS engineer is learning on my own and getting involved with the extra-curricular tech culture on campus.

Finding a solid internship on your own is hard. Even finding a not-so-solid one is hard, especially if your talents don't align with the hiring process. I was a full stack developer and self-taught DevOps guy who was writing backends for and managing prod servers of start-ups, but I certainly wasn't the most charming person to interview (mostly thanks to not spending hours on competitive coding just to clear interviews).

However, some companies are different, and a lot more practical in their approach to hiring. Platform.sh was looking for people who weren't necessarily the type to write an optimized implementation of self-balancing binary trees on the whiteboard in C++ in under 20 minutes, but who understood computers well, were brave enough to work with things that most programmers would shy away from, were comfortable with Linux and familiar with modern software engineering. They were looking for engineering interns. I applied, and after multiple rounds of intense but really fun interviews conducted by people who I would come to respect beyond measure, I managed to get through!

This meant that my final semester, which started in Jan 2020, was spent in the beautiful city of Paris. However, I arrived in Paris in the middle of political unrest leading to city-wide strikes, and when those calmed down, COVID-19 hit us like a freight train. You might think my internship experience was ruined, but it wasn't. Facing all this (and more) alone endowed me with immense confidence as an individual, and I got to experience the culture of Platform.sh in the middle of a crisis.

And not only did Platform.sh come through on the business front, but it also showcased that the fundamental human values that we profess to live by weren't mere words. The company was immensely supportive, and thus I didn't hesitate one bit when I was offered a full time job with Platform.sh as a Cloud Engineer. I decided to work remotely so that I could stay with my family during these trying times, and started working full time in August.

The job is, at my current level of experience, enjoyably challenging. Thanks to solid management and supportive colleagues, I was able to keep up with my work. Personal growth aside, Platform.sh and the people I work with there have become the compass of most of my technical growth, which is what I'll be talking about for the rest of this article.

If you're going to take one thing away from this post, let it be this: when evaluating a potential employer, make sure you pay attention to the kind of people, work culture, (the lack of) blame culture, and the challenge that the job offers.


## Polyglot Programmer: Go, Clojure, C and Rust

Four lanuguages in one year? Not really given I (thought I) knew a little C and Rust. But read on!

### Go

A lot of the code at Platform.sh is Python, but we use Go for anything that requires high performance or has strong concurrency elements. Additionally, most of our network-level stuff is in Go too. Though my work was in Python, I absolutely could not finish an internship with Platform.sh and not learn Go! So I did. I picked it up in my spare time in late January, and I found it to be a solid language.

There are things that I dislike about it (such as how Go code can sometimes get tediously repetitive), but it does certain things quite well. Concurrency in Go is a universally appreciated feature, and I am no different here. Additionally, Go is performant and the compiler is blazing fast. I also like how readable and clean-looking Go code is. I strongly recommend it for network related or concurrency heavy applications.

As a side project to learn Go, I wrote a multi-purpose, plugin based Slack + Telegram bot that could, among other things, keep track of karma points and give you Zoom meeting links on demand. It turned out quite well, and though it is currently closed-source, I might open source it soon. I also read most of the book ["Network Programming in Go" by Jan Newmarch][1], which is a useful intro to Go for low level network programming.

### Clojure and Common Lisp

Working in Platform.sh can be risky - if you're not careful you suddenly find yourself learning and loving Lisp. And I am by no means a careful person, which meant that I obviously went down the Lisp rabbit hole. The Platform.sh engineering team is filled with Lisp hackers of all levels - from nascent Lispers (i.e. me) to seasoned professionals. Which means advice and well-informed opinions are never scarce.

I decided to pick up Clojure - a modern Lisp that runs on top of the JVM and provides excellent concurrency primitives that let you use STM (Software Transactional Memory) instead of locks. Once I started to see the [intuition behind lisp][2] and the insane [power of Lisp metaprogramming][3], I was addicted! I also wrote some Common Lisp (most notably a patch to an internal Slackbot that - like most slackbots at Platform.sh - is written in SBCL).

Picking up some Lisp is a strong recommendation from me to those who want to expand their CS horizons. You'll see why when you give it a go. For practice, I wrote a small PoC log parsing pipeline in Clojure for Platform.sh. I also wrote an internal Slackbot in Clojure, which was a fun chance to explore Compojure. My ability as a Lisper is still quite limited, but I know that this is one of the languages I want to get really good at.

### C

So many of us put "C/C++" in our resumes because it is among the first languages we're introduced to, and we implement simple algorithms in them, and consider that we "know" these languages. And I will admit that I was guilty of the same crime. But C is a whole other beast, and one that takes ages to learn and probably a lifetime to master. However, instead of removing C from my list of skills in my resume, I decided to learn it instead.

I'll say that I am not a big fan of the language. It lacks modern toolchains, the syntax can be quite confusing, the error handling is horrible, and it is far too easy to write bad (or worse, vulnerable) code. So why did I decide to learn it? Because C is universal. The Linux kernel is in C. Most libraries and software you use depend on C, other things interop with or even compile down to C (Nim is a great example), and it is also important from a cybersecurity perspective (as you will see in the section where I talk about cybersec).

It has strengths though - it is crazy fast and crazy powerful. Almost too powerful. But in the hands of a competent programmer, that power translates into some very incredible things. It is also peerless when it comes to embedded programming. It is definitely a language worth learning if you are into systems or network level things, or into embedded programming.

My approach to learning C was quite interesting. I found this lovely book - [Build Your Own Lisp][4] - which teaches you C from the ground up by building your own variant of Lisp! This was an excellent read. Not only did I learn C, but I learned how programming langauges are designed and how Lisp works.

I also built what I consider to be the most complex side project of this year using C - a proof-of-concept malware payload that uses SSH to securely communicate with a command and control (C2) server. More on that when I talk about my foray into cybersecurity.

### Rust

Ah yes, this beauty of a language. Technically, I learned some Rust a couple of years ago, though back then I couldn't completely understand why the language was how it was. Now, with a far more solid (and yet still quite basic) understanding of what influences performance of languages, memory management, safety, etc, I am starting to see why Rust gets so much love. After having almost completely forgotten the language, I decided to pick it up again.

Rust is definitely a solid language. It is [crazy performant][5] and memory safety is guaranteed at compile time (unless you use `unsafe`). It also has a solid type system, and the toolchain is quite nice. Also, after having witnessed the terrible 200+ line long unhelpful stack traces that Clojure throws at you, the friendly way in which the Rust compiler tells you about errors and nudges you towards possible oversights or fixes is both refreshing and endearing.

One of the (potentially irrelevant) downsides of Rust is that it has quite a steep learning curve. It is quite a rewarding learning curve though. And rust can be difficult to work with because of all the strange things it does to make those amazing compile time guarantees, but it might (or might not) help to know that all this difficulty is the language forcing you to write bug-free code. Finally, rust is among the "strangest" langauges I have ever learned, which can make a lot of things unintuitive in Rust though I consider this to be a good thing overall.

I only got back to Rust in December, so I don't have much under my belt. The official Rust Book is a wonderful learning resource. I am also working on an interesting side project in Rust, a part of which involved [implementing the Levenshtein Algorithm][6].


## Broadening Horizons: Cybersecurity, Cloud Computing, Network/Systems Programming, Linux

I almost always end up exploring 2-3 new domains every year, if not more, and this year was no different. However, this year was better because I actually dove in quite deep into each of the domains I picked this year.

For starters, given that I am working in Platform.sh, which is (in a simplified way) a PaaS, it is natural that I work extensively on the cloud. My job taught me a lot of AWS, and a lot of the code that I write orchestrates things in the cloud. To call this technology complex would be a major understatement, but I find it mind blowing how it works. Needless to say, this is a fun job!

Though I have been using a UNIX based OS (macOS) for a while, my internship required that I switch to Linux. Exciting! I chose to start with Debian Buster, and thanks to the nature of my work, I have had to dive deep into the workings of this wonderful piece of tech. I am by no means an expert. Far from it, in fact, but I am starting to see how this works under the hood, and it is exhilirating! It did lead me to completely nuking my system though, which provided an opportunity to switch to Fedora.

Trying out low level network programming has been on my list for a while now, and once again, I seem to have landed the perfect job for that. We write tonnes of low level network code at Platform.sh, usually in Go, and taking it apart has been a crazy learning experience.

And finally, the one thing that I don't play with on my job: cybersecurity. This is something that I have been enthusiastic about for around 5 years now, but I never got into it seriously. Not until now at least. Thanks to a friend who was making a CTF team and was kind enough to invite me, I had strong reasons to sharpen my hacker skills. I went on to learn a lot about web app pentesting, binary exploitation, network pentesting, and the like. In the process of doing so, our CTF team managed to get crazy high rankings in many CTFs!

I also decided to go a step further and take a crack at writing my own malware payload in C that uses SSH to provide a reverse shell to the target while encrypting all communications with the C2 (command and control) server. The idea and architecture were inspired by the amazing book [Advanced Penetration Testing by Wil Allsopp][7], and I managed to get something working. This involved low level socket programming and working with (the grossly under-documented :( ) libssh to write both a custom SSH server and client, etc. At one point, I really thought I wouldn't be able to pull this off, but I did!

I want to add that I **strongly** recommend anyone who wants to become a good software engineer to learn basic cybsersec related to their field. Not only does this make you a much better engineer who is conscious of the security of their code, but I also find that trying to hack into things teaches you about the internals of a system unlike anything else.

## Parting Words

You've actually listened to me ramble for this long! You have my thanks. I wrote this mostly so that later in life I can look back and see how I felt about all these things, but I hope you found something of value here.

As always, share your thoughts on [Twitter][8] and check me out on [GitHub][9]!

*Big thanks to [Rounak Vyas][10] for proofreading this article.*

  [1]: https://tumregels.github.io/Network-Programming-with-Go/ "Network Programming with Go - Jan Newmarch"
  [2]: https://stopa.io/post/265 "An Intuition for Lisp Syntax"
  [3]: https://aphyr.com/posts/353-rewriting-the-technical-interview "Rewriting the Technical Interview"
  [4]: http://buildyourownlisp.com/ "Learn C - Build Your Own Lisp"
  [5]: https://medium.com/@dexterdarwich/comparison-between-java-go-and-rust-fdb21bd5fb7c# "Comparison Between Java, Go and Rust"
  [6]: https://github.com/ajmalsiddiqui/levenshtein-diff "Levenshtein's Algorithm"
  [7]: https://www.amazon.com/Advanced-Penetration-Testing-Hacking-Networks/dp/1119367689 "Advanced Penetration Testing by Wil Allsopp"
  [8]: https://twitter.com/_ajmalsiddiqui "Ajmal's Twitter"
  [9]: https://github.com/ajmalsiddiqui "Ajmal's GitHub"
  [10]: https://rounakvyas.me/ "Rounak Vyas on Twitter"
