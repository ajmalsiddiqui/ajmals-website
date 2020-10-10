---
title: "The Toolkit of a Full Stack Javascript Developer"
date: 2018-01-15T22:23:18+05:30
lastmod: 2020-10-03T19:40:18+05:30
author: Mohammed Ajmal Siddiqui
authorlink: http://ajmalsiddiqui.me
cover: https://cdn.pixabay.com/photo/2016/03/27/18/54/technology-1283624_960_720.jpg
categories:
  - software development
  - JavaScript
tags:
  - javascript
showcase: false
draft: false
---

The JavaScript ecosystem of tools and technologies for building applications is neither small nor limited to the web. This article touches upon *some* of the options you have for technologies used to build web apps, cross platform applications, desktop apps, and more, while also talking about certain tools that aren't a strict requirement for a full stack developer, but one that most devs probably can't get very far without. Hop on!

<!--more-->

## What is a Full Stack Developer?

Simply put, a full-stack developer is a developer who is comfortable with both backend and front-end. Someone capable of making an entire functional application on his own. It is obvious why this skill has a lot of demand out there.

A full-stack JavaScript developer is exactly what it sounds like — a full stack dev who uses JavaScript. Duh.

Most of you may know how JavaScript is used at the client-side. A slightly smaller number may be aware of NodeJS, a server-side framework in JavaScript. While being comfortable with client-side JS and NodeJS for the server-side is enough to be called a full-stack developer, there is a lot more that goes into being a truly capable full-stack dev armed to the teeth with the tools to make stunning applications and deploy them online.

Before we go any further, note that the frameworks and technologies I have enumerated upon here are a few of many options available to you. Take your time to explore the limitless world of possibilities that JavaScript has to offer.

If you're in a hurry, just take a look at the minimal full-stack JS dev section towards the end for... well... the bare minimum.

Without further ado, let’s dive in and take a look at what a full-stack developer may use to achieve the incredible.

## Front-end Technologies

Okay lets face it — you can’t be a full-stack dev without being a front-end developer first. It’s a part of the definition of the term. The bare minimum you should be able to do is to make a functional, dynamic and responsive website/web app. If you’re not very comfortable with frontend, [here’s][1] something that might help. This can be achieved with your usual ingredients — HTML, CSS, JS. No surprises there. But in order to take your game up a notch, you’ll need to spice things up a little with a few frameworks and libraries.

There are a few well known CSS frameworks that can help you make a responsive website without much trouble. Here are a couple:

* [Bootstrap][2]
* [Materialize CSS][3]

But let's not underestimate the power of modern CSS, which provides some easy yet powerful ways to make responsive websites _without_ using a 3rd party framework. I personally feel that even if you prefer using a framework, you should be comfortable in using Grid/Flexbox, which form the crux of the CSS way of doing responsive websites without relying on an external library, to make websites.

Once you’re comfortable with JavaScript, it's time to dive into some of the frameworks! There was a time when learning [jQuery][4] was considered to be almost a rite of passage when diving into the JS world, but framework has seen its glory days and is now slowly losing importance. Regardless, if you intend to work with JS in a professional capacity, I recommend becoming familiar with it because a lot of enterprise code still relies on jQuery. Also, you should be comfortable with using [AJAX][5] for making API calls. There are some well designed libraries that simplify making network calls (such as [axios][6]), but you should still know how to use the lower level APIs for when you need them.

Moving on to the fun stuff. Consider learning at least one of these JavaScript frameworks thoroughly:

* [ReactJS][7]
* [AngularJS][8]
* [VueJS][9]

## Backend Technologies

Backend development is the other side of the coin that we call full-stack development. If you have no idea about backend development, [this article][10] may be a good place to start.

You may be aware of many technologies and frameworks used for backend dev, but since this article relates to JavaScript, lets talk about the bad boy for backend in the JS world — [NodeJS][11]. You might also be interested in [Deno][29] - a secure runtime for JavaScript and TypeScript written in one of my favourite langauges - Rust. Deno has been developed by the original creator of NodeJS himself, so expect some good things here!

The popularity of NodeJS is certainly not a surprise given it’s sheer versatility and community support. Add to this the huge amount of frameworks built on top of Node, and you can see why JavaScript technologies have created a complete ecosystem of their own in this area.

Once you’ve learned the fundamentals of NodeJS, definitely consider learning [ExpressJS][12], a lightweight routing and middleware framework built upon Node. This combo is sufficient to be a skilled developer, however, you may consider learning other frameworks and technologies built to work with Node. I’ll list two of these to help you get started:

* [HapiJS][13]
* [SailsJS][14]

## Am I Full-Stack Yet?

You’ve made it quite far, and you sure can call yourself a full-stack developer. But is the journey over? Nah. It only gets more exciting from here as you learn about the full extent of the power of JavaScript.

For starters, consider [ElectronJS][15], a framework to build desktop applications using JavaScript. Bet you didn’t see that coming, did you?

Next, let’s talk about hybrid and cross-platform apps.

Imagine an ideal universe where you could plug in your JavaScript skills into everything — websites, desktop apps, and even apps for smartphones. Crazy right? When I first learned about these mysterious JavaScript frameworks that could make Android or iOS apps, I was beyond excited. Turns out these mysterious frameworks aren’t very mysterious after all. In fact, they are quite intuitive and use JavaScript in more or less the same way that you’re used to.

Without further ado, here’s a list of such technologies:

* [PhoneGap][16]
* [Cordova][17]
* [React Native][18] (thanks to [Madhav Bahl][20] for adding this)
* [MeteorJS][19]

Back when I first wrote this article, I was a big fan of [MeteorJS][19]. I don't do much full stack dev anymore, and this landscape has changed significantly, so at this point I kinda recommend [React Native][18] a little more. As always, the choice is yours!

## Are Frameworks all I Need to Know About?

Yes and no.

Yes because you can whip up a lot of crazy cool stuff with the knowledge you already have. And without doubt you are a full-stack developer at this point. So in a crude sense, you know all you need to know to make stuff.

But there’s another aspect of being a developer designed to make your life and the lives of people you work with easier. A set of tools and technologies that can potentially save you hours of frustration and hair pulling. More importantly, if you intend to work with JavaScript in a professional capacity, you probably won't get very far without some of these technologies.

I’ll go over some of these real quick.

## Version Control

This is almost synonymous with [Git][30] and services like [GitHub][21]. And without doubt you should have mastery over these technologies if you intend to continue as a developer. However, other options exist. Two of those are:

* [SubVersion][22]
* [Mercurial][23]

## Package Managers

These often-overlooked tools help you quickly install and manage the packages and libraries your project needs. Most of you may know [NPM][24] — the famous [Node Package Manager][24]. However, again, other options exist. 

* [Yarn][25] is a package manager that is quickly gaining traction because of its speed and a few other advantages over NPM.
* [Bower][26] is another package manager for the web.

In the JS land (and in quite a few other areas as well), package managers often do a lot more than just install dependencies. For example, NPM lets you do a security audit of the packages your project depends upon using `npm audit`, and it'll also conveniently fix any security concerns that can be fixed without introducing breaking changes if you invoke it as `npm audit fix`. Another neat feature of NPM is NPM scripts, which are so cool that I wrote an entire article about them. [Check it out][31]!

Since package managers are central to your JS project, and because they come with such neat features, it is worthwhile to spend some time learning about their capabilities and identifying things that might help you.

## Other Tools

I didn’t want to have too many of these subsections, so here’s a quick list of tools that you will eventually have to get familiar with:

* [Babel][27] — a “transpiler” for JavaScript that lets you compile code into a format compatible with your target browsers
* [Webpack][28] — a bundler for your applications that has gained a lot of popularity in recent times

## Deploying Your Application

One final topic that I’d like to briefly go over here is how to get your applications online. Again, lets make this quick. This article isn't a place to explain the different classifications of cloud services, but I'll tell you that you're probably looking for something called a "Platform-as-a-Service (PaaS)" in order to deploy your application online. PaaS offerings take care of low level things like configuring/maintaining the underlying infrastructure of your server (such as the OS, the network, services that run on it, etc) and allow you to focus on the parts that you should focus on - deploying your application with the configuration and architecture that suits your needs. There is a massive catalogue of such services available to that cover all kinds of ground - from general PaaS systems to niche systems for things like IoT to PaaS systems deployed on private clouds to a lot more. When I started, I started with [Heroku][32], which, in addition to being an established company in this space, also lets you host 5 projects for free!

In the area of cloud technologies and deploying applications to them, you might have heard about buzzwords like AWS (Amazon Web Services) and GCP (Google Cloud Platform). These are Infrastructure-as-a-Platform offerings which place a much greater burden on you. While an application of sufficient complexity or performance requirements might benefit from being directly deployed on something like AWS, it is many levels more difficult to deploy on an IaaS than a PaaS. With PaaS offerings gaining maturity and offering an almost bloated featureset, you probably won't need to look at IaaS systems for your apps for a long time.

However, there were situations where I found it quite difficult to deploy to Heroku (and ended up spending tens of hours refactoring the system I was working on to make it fit there). So allow me to pitch another lovely PaaS to you - [Platform.sh][33]. Not only does Platform.sh give you a lot more control over your build processes and more freedom to do things as you wish, but they also have a lovely local development experience. For me, the biggest selling point is the Git driven workflow that let's you manage _entire copies of your application_ via git, so you can have a git branch representing your production system, and other _live applications_ built from your dev or staging branches. In fact, you can also build Pull Requests on Platform.sh, so you have a live copy of your system built using the codebase of the pull request, that you can happily test against before you merge it! Sweet huh?

If you think I'm a little _too_ obsessed with Platform.sh, you're probably correct. After all, I work here as a Cloud Engineer! :D


## A Minimal But Complete Full-Stack JS Developer

I've name-dropped way too many frameworks and libraries and other buzzwords in this article, and you're probably a little overwhelmed. So in this section, I'll present to you what I consider a _minimal_ full-stack JS dev. This may be a little opinionated, but if you've come this far, you already know a bunch of alternatives! ;)

So what does our minimal full stack JS dev look like? He/she can:

1. Spin up a responsive, mobile friendly website that fetches content via API calls using the good ol' trio of HTML, CSS and JS (Flexbox/grid to make it responsive).
2. Build a web app using ReactJS in a development environment where babel/webpack are conveniently leveraged to compile the react app.
3. Write an API that uses ExpressJS over Node, with MongoDB as a database.
4. Correctly use git to version control projects, and present a clean commit history and a good git workflow.
5. Deploy the application on a PaaS like Heroku or Platform.sh.

Yep. That's it. That's all you need. Now fire up your favourite editor and start mashing the keyboard!

## Concluding Words

That was a lot of information, wasn’t it? Take your time and gradually get familiar with the tools and frameworks you like. I know there is a lot more that can be added to this article, but I tried to limit each list to 2–3 entries for the sake of brevity. I’d appreciate comments adding to the article or the lists.

You can find me on [GitHub][34] and talk to me on [Twitter][35].

*Good luck and Happy Coding! :)*


  [1]: https://codeburst.io/a-practical-approach-to-web-development-1ee37a4ad829 "A Practical Approach to Web Development"
  [2]: https://getbootstrap.com/ "Bootstrap"
  [3]: http://materializecss.com/ "Materialize"
  [4]: https://jquery.com/ "jQuery"
  [5]: https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX "AJAX"
  [6]: axios-link "Axios"
  [7]: https://reactjs.org/ "ReactJS"
  [8]: https://angularjs.org/ "AngularJS"
  [9]: https://vuejs.org/ "VueJS"
  [10]: /blog/getting-started-with-backend-development/ "Getting Started With Backend Development"
  [11]: https://nodejs.org/ "NodeJS"
  [12]: https://expressjs.com/ "ExpressJS"
  [13]: https://hapijs.com/ "HapiJS"
  [14]: https://sailsjs.com/ "SailsJS"
  [15]: https://electronjs.org/ "ElectronJS"
  [16]: https://phonegap.com/ "PhoneGap"
  [17]: https://cordova.apache.org/ "Cordova"
  [18]: https://facebook.github.io/react-native/ "React Native"
  [19]: https://www.meteor.com/ "MeteorJS"
  [20]: https://madhavbahl.tech/ "Madhav Bahl's Website"
  [21]: https://github.com/ "GitHub"
  [22]: https://subversion.apache.org/ "Subversion"
  [23]: https://www.mercurial-scm.org/ "Mercurial"
  [24]: https://www.npmjs.com/ "NPM"
  [25]: https://yarnpkg.com/ "Yarn"
  [26]: https://bower.io/ "Bower"
  [27]: https://babeljs.io/ "Babel"
  [28]: https://webpack.js.org/ "Webpack"
  [29]: https://deno.land/ "Deno"
  [30]: https://git-scm.com/ "Git"
  [31]: /blog/introduction-to-npm-scripts/ "Introduction to NPM Scripts"
  [32]: https://www.heroku.com/ "Heroku"
  [33]: https://platform.sh/ "Platform.sh"
  [34]: https://github.com/ajmalsiddiqui/ "Ajmal's GitHub"
  [35]: https://twitter.com/_ajmalsiddiqui "Ajmal on Twitter"
