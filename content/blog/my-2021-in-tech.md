---
title: "My 2021 in Tech"
date: 2022-02-19T13:54:42+05:30
lastmod: 2022-02-19T13:54:42+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
categories:
  - miscellaneous
tags:
  - misc
  - programming-languages
showcase: true
draft: false
---

It is that time of the year again. Or, to be precise, it is 51 days after that time of _last_ year. I've been remiss in my schedule for these posts. But it is the time when I reflect upon my growth in tech over the past year. Like [last years post][1], this one goes over what I learned, some of the opinions I formed, what I could have done better, and what I think I should focus on in 2022.

While I do this primarily to track my progress, I do hope that those who read about my growth find it instructive, and walk away having gained something from it. With that happy thought, let's get started.

<!--more-->

## On Programming Languages

### Getting Better At Rust, Go and Clojure

A disproportionately large section of [the 2020 edition of this post][1] was devoted to talking about the four languages I picked up then: Go, Rust, C and Clojure. While becoming familiar with these languages did serve me well, I realized that my knowledge of these languages was quite shallow. Sure, I could implement trivialities or simple algorithms in any of the languages but I was far from the kind of expertise that one gains while working with a language extensively. So for 2021, I decided that instead of ticking off more languages from my to-learn list, I'd become better acquainted with the ones I had in my repertoire already.

I honestly didn't care about learning any more C, and the language that made me feel the most uncomfortable then was Rust. So I did a fair bit of Rust in 2021. I read (and often coded along) the entirety of [Tim McNamara's Rust in Action][3], and did a few side projects (notably, I implemented a [simple FUSE][4] in Rust). One of the most profound things about Rust (apart from the obvious) is the quality of the toolchain. It is easily one of the most sophisticated I've ever come across, and it empowers the developer. It is a solid language ecosystem, one that I hope to see more of.

However, I must confess that I don't find Rust a pleasant language for my exploratory adventures. It clearly isn't a panacea. The features that back Rust's incredible correctness guarantees are often the ones that make lengthy wars of attrition with the compiler a rite of passage for the novice Rust developer. Which is fine if your goal is to get better at Rust, but derails you if Rust is merely a means to an end for you.

The language that I do reach for whenever I am exploring things is Go. Go is a small, easy language. And it has enough useful primitives to make it acceptable for most things that you throw at it. Most importantly, it has a low barrier to productivity. Sure, it has rough edges, but what doesn't?

One thing that Go does well is concurrency. The language is, unsurprisingly, a boon for writing network applications for example. In 2020, I implemented a proof-of-concept malware payload in C that uses SSH to communicate with the C2 server. In 2021, I reimplemented it in Go, and the difference in developer experience was phenomenal! It was easier by orders of magnitude to implement the SSH client/server, make it concurrent (something which I hadn't achieved in the C implementation), and the Go toolchain made it delightfully simple to cross compile the binary for Windows. Thus, it is no surprise that Go is my language of choice for PoCs, side projects, and even more serious projects. We also use Go extensively at Platform.sh for network related things, our observability stack, and other performance critical code.

Regretfully, I didn't do anything beyond a little side project in Clojure in 2021. My (very high) opinion of the language hasn't changed since [the 2020 edition][1]. I hope to do a lot more Lisp (not necessarily Clojure) this year though.

### Enter Elixir

One language that has been on my list since 2020 but didn't make the cut that year was Elixir. In 2021, I finally got around to learning it. Though I didn't use it for any projects, I perused and coded along with the excellent [Elixir in Action by Saša Jurić][6], and I was quite impressed!

"Elixir makes the easy things hard and the hard things easy". The adage is true in a literal sense (where the hard things are distributed systems). Elixir is a powerful functional language with an ecosystem that is tailor-made for DistSys. I find it to be an instructive introduction to the Actor Model as well. And it has Lisp style macros I hear!

I recommend everyone interested in or working with Distributed Systems to at least acquaint themselves with the language, and I'll add that [Elixir in Action][6] serves as a good introduction to DistSys in addition to Elixir.

## Journey To Systems Engineering

As the [job listing on the Platform.sh website][7] (if it is still around) states, my role is close to that of a systems engineer. When I assumed this role in 2020, guess what was the one thing I wasn't? A systems engineer. Thus, I had to become one. During my time at Platform.sh, I steadily acquired knowledge and skill that one might associate with the venerated Systems Engineer. I had to - my work demanded it. So can I call myself one now? Probably not. [Like Newton][8], I am but a child playing on the beach of the ocean that is knowledge. But am I closer than I was before? By a massive margin.

Over the past year, I consciously focussed on the low level things that most programmers I know tend to stay away from. Lots of things related to systems and networking. As mentioned before, I wrote [a FUSE in Rust][4] and implemented an SSH client/server for a malware payload in Go. Additionally, in the course of authoring an internal article on how we do container networking at Platform.sh, I became intimately familiar with containers and their networking, routing and encapsulation protocols, kernel IPC subsystems etc. Knowledge that was further solidified while debugging network issues in our software (though, admittedly, these issues were so painful to debug that I might've aged a little each time I fixed one). Furthermore, my knowledge of Linux is now leagues better.

Needless to say, I understand computers a lot better now. Also needless to say, they are far more mysterious to me now than they were in those days of blissful ignorance when the lowest level of abstraction I had to deal with was a TCP connection. Honestly, sometimes I am truly surprised anything works at all.

## Certified Cloud Enchanter

I mentioned earlier that my role is close to a Systems Engineer. But my job title - Cloud Software Engineer - likely fails to drop any hints pertaining to that.

The Cloud. One of the few tech buzzwords that has truly earned its place as a buzzword. Mysterious. Enchanting. Making things that were impossible two decades ago trivially easy. What a thing! And I am to be amongst those who engineer The Cloud? Remarkable!

I should state here that I graduated in 2020 with a bachelors degree in computer science. As a requisite for that, I spent 5 months in the enchanting city of Paris as a Platform.sh intern. After the internship, I was offered a full time role, and thus here I am. In all this time, I had neither reason nor opportunity to do a whole lot with The Cloud. Consequently, I wasn't very familiar with any cloud platforms when I started full time. I picked up a fair bit of AWS for my internship project, but it was pretty minimal. And so I was plagued with an odd variety of the imposter syndrome: "How are you a Cloud Engineer though you don't know a whole lot about clouds?".

In 2021, I decided to remedy this by acquiring the [AWS Certified Solutions Architect (Associate)][11] cert. This exposed me to a diverse array of AWS services, and gave me a good degree of familiarity with the most ubiquitous ones (e.g. EC2, S3, IAM to name a few). It was a good overview of the potential that The Cloud holds. It also left me in deep appreciation for some of the impossible sounding things that AWS manages to pull off. The tech that goes into their storage and networking offerings is fascinating. And I also learned about some of the crazier services that AWS offers. For example, did you know that they'll [send a 45 foot truck to your data center][10] if you ask nicely? Wow! To my disappointment, this service is not included in the AWS free tier. Anyways, an exam later I was an AWS Certified Cloud Guy (or something)!

And thus, one of the many imposters that I harbour was put to rest.

## Ramblings Of A Young Man

If you read both my [introspections for 2020][1] and 2021, you'll realise that this one was quite different from the previous one. I only picked up one language instead of four. I didn't explore as many new domains as I did in 2020. Heck, I didn't even ramble as much in this post as I did in that one! (Right? _Right_?)

This is a result of a profound realization that I had last year: that I was too "breadth oriented" in my learning. I learned too little of too much. A jack of some trades, master of none. I must state here that breadth oriented learning isn't necessarily a bad thing, and I still do a great deal of it. In fact, I think specializing too much (and too early) - what I call depth oriented learning - can often be a costlier mistake. But one must find the balance. Specialize too much, and you'll get left behind when your specialization (inevitably) becomes obsolete. Be too superficial in your knowledge and you'll never become an expert. I realized I was dipping my toes into everything, so this time I decided to go deeper and dip my feet as well instead of seeking new waters to sate my toes.

And I think it is paying off. My understanding of software systems is a lot more coherent now. More importantly, I now have increased responsibilities at work. They gave my clumsy fingers root access to prod! And that demands a certain (and non-trivial) degree of expertise. Expertise that will come, in part, from carefully choosing how I invest my learning time. In that sense, I am closer to the balance that I seek. Or so I believe.

## 2022: Head In The Clouds, Grounded Feet

Okay, now that I'm done pretending to be Master Oogway, I'll pen down what I _think_ I'll be doing this year.

The focus will be on acquiring expertise. To go from being a master of none to a master of some, while also remaining the jack of most. Thanks to my job, this will naturally be in the areas of systems engineering, networks, etc. Outside my job, I want to become a stronger Rust/Go/Clojure programmer, and strengthen my roots in some of the domains that I already have some familiarity with.

Simultaneously, being a "breadth oriented learner" is one of my strong suites and I believe this quality will pay rich dividends if I use it well. So this year I hope to broaden my horizons a little more. Currently, embedded systems and the engineering of unmanned vehicles seem like fun. I want to also walk the bridge where the world of 1s and 0s intersects with whatever it is that is spun into the fabric of reality, and acquire knowledge about interdisciplinary applications of computer science.

Or perhaps an entirely different thing will catch my fancy, and I'll chuckle at myself when I reread this post. Either way, it is going to be a fun ride.

## Parting Words

Thank you for indulging me, and I hope you gained something from this post. If you have interesting ideas related to what I talked about here, hit me up on [Twitter][12] (and also find me on [GitHub][13])!

Until next time, à bientôt!

  [1]: /blog/my-2020-in-tech/ "My 2020 in Tech"
  [2]: https://dystroy.org/blog/how-not-to-learn-rust/ "How Not to Learn Rust"
  [3]: https://www.manning.com/books/rust-in-action "Rust in Action"
  [4]: https://github.com/ajmalsiddiqui/siftfs "SiftFS"
  [5]: https://platform.sh "Platform.sh"
  [6]: https://www.manning.com/books/elixir-in-action-second-edition "Elixir in Action"
  [7]: https://platform.sh/company/careers/job/?gh_jid=4770789002 "Cloud Software Engineer at Platform.sh"
  [8]: https://www.goodreads.com/quotes/592171-to-myself-i-am-only-a-child-playing-on-the
  [9]: https://www.credly.com/badges/b432abe5-d08e-4398-b749-be33d4b51222/public_url "Ajmal's AWS Certified Solutions Architect (Associate) Certificate"
  [10]: https://aws.amazon.com/snowmobile/ "AWS Snowmobile"
  [11]: https://aws.amazon.com/certification/certified-solutions-architect-associate/ "AWS Certified Solutions Architect (Associate)"
  [12]: https://twitter.com/_ajmalsiddiqui "Ajmal's Twitter"
  [13]: https://github.com/ajmalsiddiqui "Ajmal's GitHub"
