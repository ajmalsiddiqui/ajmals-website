---
title: Getting Started With Backend Development
date: 2018-01-15T22:23:18+05:30
lastmod: 2020-11-28T22:08:18+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2018/01/22/21/34/sever-3100049_960_720.jpg
categories:
  - backend development
tags:
  - backend
  - API development
  - server side
showcase: true
draft: false
---

A code-free introduction to what backend (i.e. server side) development is all about, what backend devs do, some prerequisite knowledge and a few guideposts to help you get your hands dirty!

<!--more-->

> This article does not dive into code. It is an introduction to what backend is, the prerequisite knowledge you’ll need as a backend developer and some of the languages and frameworks you can learn. If you’re looking for coding tutorials, please consider other resources.

## What is Backend Development?

An app or a web service can have two broad parts: the frontend and the backend. The roles of these parts can be guessed from their names. The frontend is the part that you can “see”. It is the part that lets you interact with the app or service. The colours, animations, layout, and all the other cool stuff that adds to your experience of using the app or website is the frontend. The frontend is generally called the User Interface (UI).

The backend is the part that you can’t “see”. It is the internal working of the application or website. This includes stuff like the server, the database, etc.

Consider this blog. You’re reading an article on the blog. The fonts, colours, designs, etc constitute the frontend of this page. However, the webpage and the content of this article were served by a server and fetched from a database. This is the backend part of the application.

## What Does a Backend Developer Do?

Some of the things taken care of by backend developers include:

* Writing server-side code
* Writing code to interact with a database (yes, I know, this falls under "writing server-side code")
* Coming up with system architectures, being clever about how the system does really well for the use-case at hand
* Ensuring that the server-side code is secure and free of vulnerabilities (in collaboration with security teams)
* Ensuring that the server-side code is optimised enough to handle large volumes of traffic
* Deploying the code online so that other people can use your service

Backend developers may work with other kinds of developers on a single project since the skill set required to complete an app or a web service is not limited to backend development.

## Prerequisite Knowledge

Before you delve into frameworks, languages and technologies used by backend developers, there are certain things you should be comfortable with.

First and foremost, understand what a server is and what the different kinds of servers are. Thanks to movies and popular culture, people are intimidated by the idea of a server. But it isn’t that complex at all. You’ll know once you start learning.

Second, learn about what a database is and the different kinds of databases out there.

Third, gain a fundamental understanding of what HTTP is. The communication between the server and the client (i.e. the “user” of the application — your web browser for example) happens by means of certain rules (a “protocol”) and you should have an understanding of how this protocol works. Eventually you'll encounter other protocols on your journey (such as WebSockets, RTC protocols, RPC protocols, etc).

Fourth, you should know what an Application Programming Interface (API) is. Further, understand the difference between REST APIs and SOAP APIs.

Fifth, know how to design resilient systems and write good software. Learn about various design patterns. Write good tests. Set up automation and CI/CD pipelines.

Where should you learn about all these? I recommend you start with a Google search. Read a few articles or watch videos relating to each topic. Different people have different ways of learning that work for them. Figure out what works for you.

It is important to point out that many believe you shouldn’t jump straight into backend development without having some experience with web frontend development first. This isn’t a hard and fast rule, but I personally recommend it too. Learn some HTML, CSS and JavaScript before you dive headfirst into backend. Or maybe try some mobile app dev. If you’re looking to start web frontend, [this article][1] might be a good place to start.

At this point, you should have a fair understanding of what you’ll be doing as a backend developer. Once your fundamentals are clear, you’re ready to start writing actual code.

## Technologies and Languages

> Note: It's November 2020 and I've come a long way since I wrote this article, and I've updated this section with a little of what I've learned!

Here’s the bit you’ve been waiting for — choosing a language/framework to learn for backend development.

The classic set of backend technologies (the “technology stack”) is called LAMP. It is an acronym that stands for Linux (operating system), Apache (server), MySQL (database), PHP (server-side language).

However, modern backend technologies are quickly replacing LAMP and other more traditional ones due to their power and simplicity, and due to their incredible community support. Lets talk about a few of these backend “frameworks” written in two incredibly popular languages: Python and JavaScript.

#### Python

Flask is a popular backend framework written in Python. However, the most well known and popular backend framework in Python is Django. I’ve seen many start with Flask and move over to Django once they gain some experience. I personally don't like working with Django because it generates a bunch of files that I won't understand unless I dive into the framework, and that I may not even need. I don't recommmend it as your first web framework because it can do magic that will lead to lapses in your knowledge of how a web server works.

In the python ecosystem, I personally recommend [the pyramid framework][2], which lets you get started with a very minimal codebase (much like Flask), but comes with batteries included for a variety of more advanced use cases. It does a good job of doing what it advertises - "Start small, finish big and stay finished".

#### JavaScript

The language that I personally use for backend is JavaScript. The corresponding framework for the same is perhaps a buzzword in the web development community — NodeJS. NodeJS sparked a revolution by bringing a client-side language, JavaScript, to the server.

NodeJS is a JavaScript server-side framework built on the Chrome V8 engine and written in C++. It is extremely powerful and has seen a huge rise in popularity in recent times.

One reason why I recommend NodeJS over other frameworks is that since JavaScript is also used on the client-side, you have a single language over the entire application (assuming you’re building a web app). This basically gives you more bang for your buck — you have one language less to learn. This is especially true if you’re moving into backend after gaining experience with web frontend.

An interesing development in this context is [the Deno framework][3], which is supposed to be a better JavaScript runtime than NodeJS. Written in Rust and with out of the box support for TypeScript, and also a strong emphasis on security, this framework seems to be a big step in addressing some of the shortcomings of NodeJS, and one that I am definitely excited about!

#### Databases and Misc

Next step: learn about a database. If you’re going for NodeJS, a natural choice is MongoDB — one of the most popular NoSQL databases in the world. SQL is an important skill, and RDBMS systems (relational databases) are probably here to stay. Postgres is a good choice.

You should also learn about things like key-value stores and data structure stores (I favour Redis), which are useful as caches or session stores.

Once you’ve decided upon which framework and database you want to use, there are a few more things you should be comfortable with as a developer in general and a backend developer in particular:

* Git and GitHub for version control
* Using database-as-a-service platforms (e.g. Mlab for MongoDB)
* Using server-as-a-service platforms(e.g. Heroku, Digital Ocean, or my personal favourite: [Platform.sh][4] (note: I work at Platform.sh 😛))

### November 2020 Update: Writing Servers in Go or Clojure

It's been over a year since I wrote a server in Python or NodeJS. I picked up a couple of new languages this year, and I've had interesting experiences writing HTTP servers in those, so I'll drop in a quick note about that here.

#### Go

Go is an excellent langauge for things that involve concurrency. It is also a top choice for network heavy stuff. And web servers are both network heavy and require good concurrency (even if this isn't immediately obvious to you when you use frameworks like NodeJS or Flask). Also, Go being a strongly typed compiled langauge means that your server is immediately MUCH faster than a Node or Python server (many times faster infact) and a lot more bug free as well!

Go is gaining a lot of traction, and it is already being used in production in a lot of places. We write a lot of Go at Platform.sh too!

I wrote a non-trivial slackbot in Go when I was learning the language and though the curve is steeper than for something like Flask, it is well worth the results. I've written servers in Node+Express, Flask, Pyramid and even Clojure (see next section) but right now Go is my go-to language for web servers (and anything network related).

#### Clojure

Surprise surprise! You weren't expecting to see this one here, were you? Clojure is a modern Lisp that runs on top of the JVM (and thus can interoperate with Java code). It also has excellent concurrency primitives and clever use of STM (Software Transactional Memory), and the crazy level of metaprogramming support that you'd expect from a Lisp.

I've only written a very trivial webserver in Clojure using the Compojure+Ring stack, but I thoroughly enjoyed the experience.

The only downside of Clojure is that the crazy amount of memory that the JVM hogs up.

Of course, my reason for choosing Clojure is very simple - IT IS SO MUCH FUN! ;)

## Getting Your Hands Dirty

Now that you have a rough idea of what you want to do, start learning. There are many popular blogs, vlogs, tutorial series, video series, courses, etc. that you can go for. Again, different people have different ways to learn, so find what works best for you.

Here's [a lovely roadmap][6] for going from novice to wizard as a backend developer.

One thing which applies to everyone is this: learn by doing. Do mini-projects. Make stuff. Unless you actually write code, you will not be able to grab the essence of what you’re learning. Most people make the mistake of learning a lot of theory but not doing much practical work. I made this mistake. A lot. And I realized it quite late, so I'm fixing it now. I can tell you that the difference that some hands-on work makes is quite significant.

Another great tip is to look at code written by other people. It exposes you to other techniques and solutions to the same problems that you are solving (or will have to solve) as a developer. GitHub is a great place to get started. You may not understand much (or even nothing at all!), but is a good exercise. Revisiting a codebase that you failed to understand earlier is also a good way to gauge progress.

## Next Steps

So you’ve learned the basics of backend development and you’ve worked on a couple of projects. What now? Your options are limitless.

Perhaps the most natural course of action is to learn frontend as well and become a full-stack developer — a legendary developer who can make an entire app/website on his own since they are comfortable with both frontend and backend. Full-stack development is a highly valued skill in todays world. If you're planning to choose the JS path, check [this article][5] out to get an idea of the roadmap to becoming a full stack developer.

Once you’ve gotten your feet wet and are looking for a change, consider learning frameworks like Ionic that let you build cross-platform apps using HTML, CSS and JavaScript. In simple words, the code you write will translate into an app for your phone. Incredible, isn’t it? You can also transition into a more backend-related thing like DevOps (like I did). Or build mobile apps maybe?

The whole wide world of building amazing things by hitting a keyboard until magic starts to happen awaits you!

## Concluding Words

That concludes this article. I hope you have gained some insight into what it means to be a backend developer and how to go about becoming one. All the best.

Liked this article? Didn't like it? Want to add something? Wanna talk about something that I should have mentioned here but didn't? [Let me know on Twitter][7]!

Want to check out the stuff I build? Here's [my GitHub][8]!


  [1]: https://codeburst.io/a-practical-approach-to-web-development-1ee37a4ad829 "A Practical Approach to Web Development"
  [2]: https://trypyramid.com/ "Pyramid Framework"
  [3]: https://deno.land/ "Deno"
  [4]: https://platform.sh/ "Platform.sh"
  [5]: /blog/the-toolkit-of-a-full-stack-javascript-developer/ "Toolkit of a Full Stack JS Developer"
  [6]: https://roadmap.sh/backend "Backend Roadmap"
  [7]: https://twitter.com/_ajmalsiddiqui "Ajmal's Twitter"
  [8]: https://github.com/ajmalsiddiqui "Ajmal's GitHub"
