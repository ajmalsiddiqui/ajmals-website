---
title: "Dive Into Dotfiles Part 1"
date: 2018-05-27T22:23:18+05:30
lastmod: 2018-05-27T22:23:18+05:30
author: Mohammed Ajmal Siddiqui
authorlink: https://ajmalsiddiqui.me
cover: https://cdn-media-1.freecodecamp.org/images/1BklMEVdm9uru1MpfsiUAHRjL-4aK8AvRtUo
categories:
  - workflows
  - dotfiles
tags:
  - dotfiles
  - bash
showcase: true
draft: false
---

This article looks at the very basic elements of customizing your dotfiles and stepping up your workflows. We look at defining custom aliases and functions, and the rationale behind checking your dotfiles into version control. The article concludes with a few references to good dotfile repositories and a taste of things to come.

<!--more-->

> Note: This is a very basic, introductory article. If you already know the fundamentals of dotfile management, I’d recommend you read my [second article](/blog/dive-into-dotfiles-part-2).

As developers, we strive to minimize the time we spend on redundant things, like setting up our environment, writing boilerplate code, and basically not doing anything that does not concern the fun part of coding - building new stuff.

In this context, imagine a perfect world where tiny commands carry out incredibly complex tasks tailored to your needs, where you could buy a new laptop today and install all the tools and packages you need and setup your development environment with nothing but a couple of terminal commands, and where everything is magic.

This digital fairyland can be made, and with ease. And there is a name for this magic: dotfiles.

Without further ado, let’s unravel the secrets of the dotfiles!

## Introduction

> **Note:** This article assumes that you’re working with a Unix-like operating system and it relies heavily on Unix terminal commands and shell scripting. If you’re not familiar with these, I recommend learning the basics and coming back here. [Here’s](https://medium.com/@madhavbahl10/your-perfect-kickstart-to-shell-scripting-857b81c0939b) a primer to shell scripting.

In UNIX-like systems, a lot of configuration files and the like are preceded with a dot(.). These files are hidden by default, and even the `ls` command doesn’t reveal their presence (we’ll get to finding these files in a bit). Since these files are preceded by a dot, they’re called dotfiles. Duh.

So how do we find these legendary files if they’re hidden by default? Pop open a terminal and do this:

> Note: The “$” sign is not meant to be typed in the terminal. It represents the fact that the text after it is supposed to be typed in a terminal prompt.

```bash
    $ cd ~
    $ ls -a
```

So what does this do?

The first command ( `cd ~` ) moves into the home directory (the “~” symbol represents the home directory). The home directory is where most of your config files are found. So we move there first.

The second command lists the files and folders in the current directory. But there’s some magic here. The -a flag instructs the command to include hidden files in the list.

Bingo! We can now see the dotfiles!

## Modifying The `.bashrc`

Usually, the first file that most people modify when they enter the world of dotfiles is the `.bash_profile` or the `.bashrc`. And for good reason. This file is loaded when you start your terminal, and its commands are executed at terminal startup.

One reason why you might want to modify your `.bashrc` is to customize the look of your terminal (to be specific, your terminal prompt). This is an art and a science in itself and probably should have an entire book dedicated to it, so I won’t cover this topic much in this article. You can get started with customizing your prompt with [this article](https://www.howtogeek.com/307701/how-to-customize-and-colorize-your-bash-prompt/).

Instead, let’s look at two common shell constructs that are perhaps among the most important and useful parts of dotfiles: aliases and functions.

## Aliases

Aliases are simply short names/acronyms that you can assign to a longer sequence of commands in order to reduce how long you take to type it and thus increase your speed. For example, almost every developer uses git. Anyone who uses the git CLI (and let’s face it - you should be using the git CLI), probably has used long commands like these:

```bash
    // commit changes
    $ git commit -m "some changes"

    // push changes to the master branch of origin
    $ git push origin master
```

These commands are quite a bit to type. If you think they’re not, you will change your mind after you start using aliases.

Type the following in your shell prompt:

```bash
    $ alias gpom='git push origin master'
```

Now when you type `gpom`, `git push origin master` is executed! You’ve gone from 4 **words** to 4 **characters**! 😲

But there’s a problem. Close your terminal, restart it, and try `gpom` again. Your alias is gone! This is because the alias is defined for the current terminal session.

So how do we get around this and make our aliases stick?

Remember we talked about a file whose commands are executed when a terminal is started? Bingo!

Add the following line to your `.bash_profile` or `.bashrc` and save it:

```bash
    alias gpom='git push origin master'
```

Now, whenever you start a bash terminal, the above alias is created. Life is already starting to get awesome!
> **Note:** You can use the nano text editor to edit your text files. When in the home directory, type nano `.bash_profile` to open the file using nano, make your changes, and save the file by hitting Ctrl+X and then y when prompted. Vim is another text editor you can use. It is my editor of choice for everything, but it has a steeper learning curve.

Since aliases essentially replace the full command, you can aliases as part of a common multi command CLI tool like git to make all its commands easier. Just add this to your `.bash_profile`:

```bash
    alias g='git'
```

And you can type “g” instead of “git” wherever you want to use “git”. Sweet!

Here are a few common aliases you might wanna use:

```bash
    alias home='cd ~'
    alias ..='cd ..'
    alias '?=man'
    # Git CLI aliases
    alias g='git'
    alias gi='git init'
    alias gra='git remote add'
    alias gs='git status'
    ...
    # Aliases for NPM
    alias nr='npm run'
    alias ni='npm install'
    alias nid='npm install -D'
    ...
```

### Functions

Aliases can go a long way in improving our workflow, but there’s one thing they can’t do: work with arguments.

Let’s say you were tired of executing two commands to make a new directory and cd into it:

```bash
    $ mkdir new_folder
    $ cd new_folder
```

And you wanted to make an alias for this. But you can’t, since both mkdir and cd take arguments, and you can’t pass arguments to aliases.

So what now? Remember, there’s a super common programming construct that takes arguments? Yup, functions! Shell scripts can have functions that can take arguments. Awesome! If you’re a little rusty with functions in shell scripts, [here’s](https://medium.com/@madhavbahl10/your-perfect-kickstart-to-shell-scripting-857b81c0939b) a little reminder.

You can turn the above sequence into a shell function like this (this example was taken from the [dotfiles of mathiasbynens](https://github.com/mathiasbynens/dotfiles), who has some of the most popular dotfiles around. Other people with excellent dotfiles to refer to are listed and linked to at the end of the article):

```bash
    # Create a new directory and enter it
    function mkd() { 
        mkdir -p "$@" && cd "$_";
    }
```

Again, you can put this in your `.bash_profile` and the function will be accessible during any terminal session.
> **Note:** You’ll have to restart your terminal for any changes to your `.bash_profile` to take effect. If this is a chore, run source .bash_profile to add your changes to the current terminal session. Even better, in the spirit of dotfiles, make an alias like `alias reload='source .bash_profile'`!

## Dotfiles and Sharing

Why do people commit their development environments — their dotfiles — to version control? Why do they put it up on GitHub for everyone to see? Same reason as always: to track how your dotfiles evolve over time and, most importantly, to **share your dotfiles and inspire other people**.

If you look at any mature dotfiles repo, you’ll realize that there are always snippets taken from other dotfiles repos and the like. They may even have multiple contributors and maintainers. We share dotfiles to collectively help each other build better environments and workflows.

This also allows people to use features of version control to make each others’ dotfiles better. One example of this is using the GitHub Issue Tracker to discuss issues and improvements.

I was inspired to work on my dotfiles from the dotfiles of [pradyunsg](https://github.com/pradyunsg/dotfiles), who has an impressive dotfiles repo of his own.

[My own dotfiles](https://github.com/ajmalsiddiqui/dotfiles) are fairly basic and very immature right now, and they’ll get better over time. But this also means that beginners in the world of dotfiles will be less intimidated when they check the repo out.

Like many other people, I’ve added some support for customization of [my dotfiles](https://github.com/ajmalsiddiqui/dotfiles), so it might be a good idea for people who are new to the idea of dotfiles to fork the repo and try making it their own. More details in the [repository](https://github.com/ajmalsiddiqui/dotfiles). Do check them out and give me feedback!

Here’s a list of a people whose dotfiles are much more expansive and might inspire you:

* [pradyunsg](https://github.com/pradyunsg/dotfiles)
* [mathiasbynens](https://github.com/mathiasbynens/dotfiles)
* [paulmillr](https://github.com/paulmillr/dotfiles)
* [holman](https://github.com/holman/dotfiles)

## Conclusion

These are the very fundamentals of creating your development environment using dotfiles. However, there is a lot more to this, which we’ll continue looking at in the [next article](/blog/dive-into-dotfiles-part-2) in this series.

Some of the topics we’ll look at in the next article in the series are:

* Creating an environment to organize, track and painlessly work with dotfiles
* Splitting our dotfiles to make managing them easier and more scalable (notice it is *dotfiles*, not *dotfile*)
* Writing scripts to setup (bootstrap) a new system with our dotfiles
* Making our dotfiles easy to share by adding support for customization

That concludes the first part of the series on dotfiles! [Here’s](/blog/dive-into-dotfiles-part-2) a link to the next one.

I loved the idea of dotfiles so much that it inspired me to create a basic dotfile management framework - [autodot](https://github.com/ajmalsiddiqui/autodot). The framework is in its infancy, so I’m looking for enthusiastic people who can give me feedback for the framework, contribute to it by telling me about bugs and making feature requests, and contribute to the code and documentation. Do take some time out for this! :)

[ajmalsiddiqui/autodot](https://github.com/ajmalsiddiqui/autodot)

Also, connect with me on [GitHub](https://github.com/ajmalsiddiqui/) and [Twitter](https://twitter.com/_ajmalsiddiqui).

*Good luck and Happy Coding! :)*
