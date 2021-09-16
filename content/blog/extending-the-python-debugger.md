---
title: "Extending The Python Debugger"
date: 2021-09-16T20:01:59+05:30
lastmod: 2021-09-16T20:01:59+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
categories:
  - python
  - workflows
tags:
  - python
  - pdb
  - debugging
---

For a little while now I've had this itch with pdb that I've been wanting to scratch. I wanted a limited version of [the hooks feature from gdb][1] so that I can automatically execute debugger commands each time execution stops. Today, I ended up scratching that itch by extending Pdb and coding in the functionality I needed. This post talks about how I did so and also covers a loosely organized collection of related Python debugger tricks that you may find useful (or at least fascinating).

<!--more-->

> Since this post is about the python debugger, I'm naturally assuming that you have a degree of familiarity with Python and at least basic competence in using pdb. If you aren't familiar with pdb (but understand the point of debuggers in general), [the official pdb docs][2] are quite good and deserve a reading. Or [there's a really good tutorial by RealPython][4] on the subject.


## The Problem

The [gdb user defined hooks feature][1] is quite powerful. It let's you define a "hook" that executes before (or after, if you so wish) either after a command or when execution stops and you drop into the debugger. Check out the docs linked for more detail on that. If you've spent any time in a debugger, you'll know that you inevitably have times when you need to repeat the same sequence of commands again and again, and the hooks feature lets you automate this quite well. I wanted something much smaller - the ability to automatically execute debugger commands every time code execution stops.


## The Stuff That Didn't Quite Work (But Is Cool To Know)

### commands

The closest feature that I could find for my use case was the `commands` command. It allows you to execute a series of commands when a breakpoint of a particular breakpoint number is hit. Certainly useful (consult [the official docs][2] for some usage ideas), but clearly falls short of our requirements because a) it wouldn't work with breakpoints defined in the code using a `breakpoint()` or `pdb.set_trace()` call because it only works with breakpoint numbers and b) even if it did, it wouldn't execute our commands every time execution stops. It only executes commands when a breakpoint is hit, not when you step over a single line of code using `n(ext)` for example.

### alias

Another technique that one can use to make the debugging process more bearable is the `alias` feature. It is kinda similar to the `alias` command of bash and other shells. This lets you define an alias for **anything that be legally typed on the pdb prompt**. This includes using other aliases you have defined. Here's an example (taken straight from [the official pdb docs][2]):

```python
# Print instance variables (usage "pi classInst")
alias pi for k in %1.__dict__.keys(): print("%1.",k,"=",%1.__dict__[k])
# Print instance variables in self
alias ps pi self
```

Once again, quite useful. The biggest issue with this is that unlike a bash alias, *you cannot put multiple commands in an alias* since you can only legally execute one command/expression in the pdb prompt. Pretty serious limitation for our use case. And the other problem is that this isn't automated - you need to type and execute the alias manually in the pdb prompt.

### pdbrc

Not many people know that pdb supports a `.pdbrc` file, which is quite similar to a `.bashrc`. This file lets you execute any debugger commands you want at the startup of the debugger. It can either be present in the home directory or in the directory from where you fire up the python process (the one in the home directory is executed first if you have `.pdbrc` files in both places).

This is great for initializing useful aliases, and if you find yourself executing the same cumbersome sequence of commands at startup again and again, you might find it worthwhile to put this sequence in a `.pdbrc` file in your current working directory. But clearly this doesn't help us much, since these commands are only executed at startup.

### display

The `display` debugger command takes an expression as an argument and displays the value of the exception if it changed each time execution stops in the current frame. I mention this because before learning about this command, I could only think of an implementation of [the gdb hooks feature][1] to kinda pull this off, and it is a very common use case.


## Building Your Own Pdb

> Disclaimer: while the information here is useful, the code example is a hacky little thing I pulled off in a couple of hours. It likely breaks other pdb things and has weird edge cases. If you find problems, let me know. I might update/improve this if I see the need to, but no promises!

So we see that pdb has a featureset that doesn't cover this use case. Depending on how you use it, you may have your own use cases that pdb isn't adequate for. One way forward is to build your own pdb. No, not from scratch. It turns out that the pdb module of the standard library exports a `pdb.Pdb` class, which you can subclass. And then you can override various methods of the original `Pdb` class in your subclass in order to customize them to your liking.

As for which methods of the `Pdb` do what and which ones you should override, that is something I haven't found much documentation for. The easiest thing for me was to read [the pdb source code][5] and find the parts that I needed to tinker with. I am far from an experienced Pythonista, but I found the code to not be too difficult to grok at. As an example (and to present a solution to the original problem posited above), here is my crude Pdb variant that has support for a `hook` command. It is far from perfect, but the less than 50 lines of fairly straightforward code should be a testament to how easy it is to get your own pdb variant up and running.

{{< gist ajmalsiddiqui a9f50594c1d3d858a9685aa74e61f2da >}}


## Using Your Pdb With The breakpoint Builtin

As you might already know, we can hardcode a breakpoint into a python program using the following idiom:

```python
import pdb; pdb.set_trace()
```

Or, if you're using a debugger other than the stdlib pdb (such as `ipdb` or `gtools.pdb`), you might be writing something like this:

```python
import ipdb; ipdb.set_trace()
```

You might also know that Python 3.7 implements a new `breakpoint` builtin, which lets you set a breakpoint using:

```python
breakpoint()
```

For all intents and purposes, `breakpoint` is an alias to `pdb.set_trace`. Except it is a little more than that. Skimming through the PEP which introduced this builtin - [PEP 553][7] - leads to some interesting discoveries that we can leverage to make it so that a call to `breakpoint` results in our custom pdb being used instead of the stdlib one.

Internally, the `breakpoint` builtin is basically an alias for `sys.breakpointhook`. This function, by default, imports `pdb` and executes `pdb.set_trace`. However, by overriding the value of the `PYTHONBREAKPOINT` env var, you can instruct it to use a debugger of your choice. For example, doing `export PYTHONBREAKPOINT=ipdb.set_trace` in your shell before executing python code with `breakpoint` calls results in `ipdb` being used as the debugger! The `sys.breakpointhook` function also deals with the importing of the module that has the breakpoint function. So as long as the module is in your Python path, just setting the env var will work.

For our `HookPdb`, the `hookpdb.py` file exports a `set_trace` function. So assuming that this is in your Python path (easiest way to test it is to have it in your current working directory), the following happens:

```
❯ PYTHONBREAKPOINT=hookpdb.set_trace python
Python 3.9.5 (default, May 14 2021, 00:00:00) 
[GCC 10.3.1 20210422 (Red Hat 10.3.1-1)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> breakpoint()
Welcome to HookPdb
--Return--
> /home/ajmal/playground/gevent-hang/hookpdb.py(75)set_trace()->None
-> mypdb.set_trace()
(Pdb) 
```

Notice the `Welcome to HookPdb` banner when we drop into the debugger!


## Concluding Words

 As I said earlier, what I have so far is a fairly hacky thing that possibly breaks other features of pdb or has unknown side-effects/consequences. And there is a lot about python, pdb and debuggers that I don't know, so there might be other solutions to my problem that are easier/better/more robust. Regardless, this little dive into the pdb ecosystem was a fun learning experience for me, and I hope you also found it as fun (and hopefully learned a thing or two as well).

If you have comments, want to point out flaws in my approach, point out mistakes I made, or you just have better ideas, please reach out to me on [Twitter][8]!

  [1]: https://sourceware.org/gdb/onlinedocs/gdb/Hooks.html "gdb User Defined Hooks"
  [2]: https://docs.python.org/3/library/pdb.html "Python 3 Pdb Docs"
  [3]: https://github.com/python/cpython/blob/9fd87a5fe5c468cf94265365091267838b004b7f/Lib/pdb.py#L245-L255
  [4]: https://realpython.com/python-debugging-pdb/ "RealPython Pdb Tutorial"
  [5]: https://github.com/python/cpython/blob/main/Lib/pdb.py "Pdb Source Code"
  [6]: https://gist.github.com/ajmalsiddiqui/a9f50594c1d3d858a9685aa74e61f2da "HookPdb Gist"
  [7]: https://www.python.org/dev/peps/pep-0553/ "PEP 553: Builtin Breakpoint"
  [8]: https://twitter.com/_ajmalsiddiqui
