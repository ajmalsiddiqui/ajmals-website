---
title: "Mastering Your Tools: Becoming a Better Programmer Without Any Programming"
date: 2020-12-20T18:35:15+05:30
lastmod: 2020-12-20T18:35:15+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2016/03/09/09/17/computer-1245714_960_720.jpg
categories:
  - miscellaneous
tags:
  - dotfiles
  - misc
showcase: true
draft: false
---
This post talks about how you can be a better programmer without actually learning any programming! I want to make a case for actively and consciously trying to master your tools as a way to improve your ability as a programmer while also looking like a wizard-hacker.

<!--more-->

Programming is still a very artisanly job. Sure, we have things like code linters, boilerplate generators, and capabilities to automatically refactor large codebases, but most of the code that actually matters - new features, complex bugfixes, etc - is still handcrafted with love. A master artisan is a master of his toolbox, and that's what we're talking about today. If you aren't convinced that efficiency and speed matter, [this article][19] might help change your mind.

> At risk of stating the obvious, I am definitely not saying that you should stop learning actual programming skills and devote yourself to this stuff. Being a good programmer is all about, well, programming. This stuff will just make the process more efficient and more rewarding.

Let's get started!

## The Keyboard

The keyboard is exceedingly simple. It has a bunch of keys which you press to see (fairly) predictable effects on your screen. However, this tool puts the "hand" in "beautifully handcrafted code", and thus deserves to be mastered. But what is there to master in this simple device?

In my eyes, there are three facets to this:
1. Being fast enough to not make typing cumbersome.
2. The keyboard isn't getting in your way when you work.
3. The keyboard or the act of typing isn't something that you are conscious about.

Considering that as a programmer the only thing you do is type (or attend meetings on zoom but let's not talk about that) all day, it is easy to see why being fast will make you noticeably more productive. 50WPM+ is considered decent by most standards, but most standards include people who don't type as much or as often. I suggest aiming for a speed of 80WPM or higher, though anything over 70 is still pretty good.

You might be surprised at the thought of the keyboard getting in your way as you work. But if you alternate between looking at the keyboard and the screen as you type, it is getting in your way. Typing should be muscle memory, not a conscious act that that requires attention and cognitive resources.

Finally, if your thought process while working involves _realising_ that you must type and you _think_ about the typing, you're in trouble. Especially if you are a programmer (or someone who has to type a lot in their job).

Fortunately, these issues are both easy and quick to fix. In case you haven't noticed yet, yes, I'm advocating that you learn how to touch type.

In all honesty, touch typing was a random thing I picked up during my university finals. But I quickly realised how useful this skill was. In fact, during my first week as an intern at Platform.sh, I was still waiting for my laptop and the interim one I was given had a French (AZERTY) keyboard (the keymap for which was set to the usual QWERTY), and I had **zero** trouble using it because I didn't need to look at the keyboard at all even while typing at 70+WPM!

To get started, I suggest using a typing tutor like [this one][12] and spending 15-20 minutes a day actively practicing. More importantly, use touch typing during your work. It's gonna suck initially. You'll be slower than your grandma with a touchscreen. But with a week or two of focussed effort, you'll be as fast as you were earlier. A little more effort and you'll be even faster! A more fun way is competing on typeracer.com, which I occasionally enjoy.

![Ajmal's TypeRacer Stats][13]

## The Text Editor

When you're not hunting for the copy-paste-able StackOverflow snippet to fix the cryptic stacktrace you're looking at, you're most likely editing code. And pay attention to my choice of words here. I didn't say that you're "writing code". Because contrary to what most people might think, we don't spend much time writing new code. Most of it is spent editing existing code to improve or fix it.

And that's where a text editor comes in. You may not buy into my pitch for mastering some of the other things that I mention here, but you don't wanna pass on this one. Your efficiency with your editor should reach the point where your fingers translate thoughts into edits.

Most new programmers consider text editors to be like Word processors that are customized for coding. And this is, at best, a misappropriation of the venerable text editor. At worst, it is hours of wasted time and a world of productivity enhancements undiscovered. So take your time to explore the various editors out there, pick one that you like, and tailor it to your style. Consider vim, emacs, or, if you want a GUI, I used to love VSCode when I used it.

When you master your editor, not only will you be weaving code out of what look like 3 keystroke incantations, but you will also enjoy a richer and more productive experience as a programmer. I talk about my editor of choice (spoiler: vim) in the last section, in case that is something you want to know.

## The Shell

The Unix shell is perhaps one of the greatest advancements in computer history. It gives you a level of communication with the computer that nothing can match. It sounds both daunting and annoying to have to remember and type out commands to do things, but when you see how efficient it is, you'll find GUIs a lot more annoying. I personally don't leave the terminal unless I really have to. I won't talk about why this is important or showcase the power of the shell here because [this post][4] already does a great job of doing both, so check it out!

I will, however, emphasize the importance of investing effort in making your shell environment better. Even if it involves writing a little code, do it. You'll thank yourself! Start with simple things - making your prompt fancy, using a nice colour theme, etc. And then move on to useful shell [aliases][8] and [functions][7] that'll make your shell experiences even more rewarding. Automate boring stuff. You can look at [my dotfiles][3] or the dotfiles of the many legends out there to see how they've set their shell environments up. I have a two part [tutorial about dotfile management][9] which might give you more ideas.

## Misc

This area is for tools that don't necessarily warrant a full section. Don't be misled into thinking that mastering these is less important though. In fact, many of these things might be *more* important than the ones we talked about, in part because knowledge of this stuff may be an explicit job requirement. So bear with me for a little longer.

First, please learn how to use a debugger. No really, you need this stuff. If you're the kind of prodigy who can debug very complicated bugs by littering the codebase with `print` statements, using a debugger will let you do that but much faster. If you're like the rest of us, well, do I really need to say any more?

Second, master your VCS. And if you're thinking, "oh but I use git all day every day I know this well", stop. It's hard to find someone who _doesn't_ know the basics of git. But learn about the less common things. Have you tried tracking down bugs using `git bisect`? Backporting fixes using `git cherry-pick`? Don't just _learn_ how to use a tool, _master_ it.

Finally, be good at finding, navigating through, and understanding documentation using man pages and the `help` (or equivalent) in REPLs instead of instinctively googling something like "scp remote to local transfer syntax". The man pages are comprehensive (sometimes a little too comprehensive, but [there's tldr][5] to solve that), and you'll find things here that usually aren't in tutorials. The more advanced/niche the thing you're doing is, the less likely it is that you'll find a video course for it. The "how" for this is simple: practice. As a bonus treat, if colourless manpages kill your joy, here's [a snippet from my dotfiles][6] that adds some colour to them!

## My Rig

"So what tools have you mastered, Ajmal?" Glad you asked. The answer is "None". But if you want to know which ones I am in the processing of mastering, read on.

The keyboard part is simple. I am a touch typist who uses QWERTY like most people. As for the shell, I use [zsh][14] with [oh-my-zsh!][15] to make it nice. I might ditch this in favour of something less bloated and more handcrafted though. I also use [tmux][16] for managing sessions and splitting panes. Touch typing + vim is a dangerously powerful combo, as you can find in [this article][17].

While I enjoy spectating (and sometimes indulging in) the text editor wars, this post (sadly) isn't about that. But I'll let you know that I exclusively use vim. This post isn't a sales pitch for vim (not going to lie, it almost was one), but I can tell you that skilled vimmers do make it look like they can edit text at the speed of thought. I'm writing this post using vim. [This introductory article to vim][2] doubles as a pretty good sales pitch.

You can find my shell setup, vim/tmux configuration and the plugins I use in [my dotfiles][3].

## Parting Words

I hope this gave you some direction towards being at your craft. You may not agree with the importance of every little thing I've mentioned here, but a modicum of effort towards a few of these things will pay big dividends down the line.

Find me on [Twitter][10] and check out some of my code adventures on [GitHub][11]!

*Big thanks to [Rounak Vyas][18] for the reviews that made this post more useful and less narcissistic.*

  [1]: https://data.typeracer.com/misc/badge?user=ajmal21 "Ajmal's Typeracer Stats"
  [2]: https://danielmiessler.com/study/vim/ "Learn Vim for the Last Time"
  [3]: https://github.com/ajmalsiddiqui/dotfiles "Ajmal's Dotfiles"
  [4]: https://drewdevault.com/2020/12/12/Shell-literacy.html "Becoming Shell Literate"
  [5]: https://tldr.sh/ "tldr pages"
  [6]: https://github.com/ajmalsiddiqui/dotfiles/blob/master/.exports#L6-L14 "More colour to less!"
  [7]: https://github.com/ajmalsiddiqui/dotfiles/blob/master/.functions "Ajmal's dotfiles: functions"
  [8]: https://github.com/ajmalsiddiqui/dotfiles/blob/master/.aliases "Ajmal's dotfiles: aliases"
  [9]: /blog/dive-into-dotfiles-part-1/
  [10]: https://twitter.com/_ajmalsiddiqui "Ajmal's Twitter"
  [11]: https://github.com/ajmalsiddiqui/ "Ajmal's GitHub"
  [12]: https://www.keybr.com/ "Keybr"
  [13]: https://data.typeracer.com/misc/badge?user=ajmal21 "Ajmal's Typeracer Stats"
  [14]: https://www.zsh.org/ "Zsh"
  [15]: https://ohmyz.sh/ "Oh my Zsh!"
  [16]: https://www.ocf.berkeley.edu/~ckuehl/tmux/ "Tmux Guide"
  [17]: https://nullprogram.com/blog/2017/04/01/ "Touch Typing and Vim"
  [18]: https://rounakvyas.me/ "Rounak Vyas on Twitter"
  [19]: https://scattered-thoughts.net/writing/speed-matters/ "Speed Matters"
