---
title: "Google CTF 2022: Treebox"
date: 2022-07-17T13:00:05+05:30
lastmod: 2022-07-17T13:00:05+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
categories:
  - ctf writeups
tags:
  - sandbox escape
  - python
  - google ctf 2022
draft: false
---

This is the write-up for Treebox, one of the easier Sandbox Escape challenges from Google CTF 2022.

The challenge statement says, "I think I finally got Python sandboxing right.", and we're provided with a hostname/port pair and (conveniently) [the source code of the sandbox][2]. The flag is in a file called `flag` in the current working directory of the sandbox process.

<!--more-->

The kind folks at Google have said that they won't take the challenge servers down, so if they're still up and not broken, you can connect to the challenge via:

```bash
nc treebox.2022.ctfcompetition.com 1337
```


## The Challenge

This Python sandbox is implemented in an interesting way. Upon connecting to the challenge server, you are prompted to enter arbitrary Python source code, and add `--END` as the last line. The sandboxing is implemented by first compiling your source code to a python AST, and then walking through the entire AST to check if any of the nodes in the tree are either imports or function calls. If so, the sandbox refuses to execute your code.

In other words, your python code cannot have imports or function calls, and you somehow have to read a file and print it's contents.

So you somehow have to execute `print(open("./flag").read())` while not being able to call any functions explicitly!

Again, [here's][2] the original source code of the sandbox (though it is possible to solve it without the source).


## TL;DR

There are a ton of ways break out of this sandbox, and many are both more elegant and less verbose than my solution (for example, check out [the challenge author's solution][1]). Most of these involve executing the `eval` function via some sorcery. My approach does not involve using `eval` at all, and is a little more roundabout, but I find that it is quirky and creative in it's own right. So I'll share it here:

```python
sys.stderr = sys.stdout

FileIO = sys.modules['io'].FileIO

class FlagIO(FileIO):
    def __init__(self, fn):
        pass

FlagIO.__eq__ = FileIO.__init__

@FlagIO
def hello():
    pass

hello == "./flag"

flag = [a for a in hello]

assert False, flag
--END
```

The tricks are:
1. Use `sys.modules` to import a module without an explicit import statement.
2. Use a class subclassing `io.FileIO` in order to create something that can open the file and let us read it.
3. Set it's `__eq__` method to the `__init__` method of the original `io.FileIO` class, so that it can be invoked by just doing an equality test.
4. Instantiate the class by using it as a decorator.
5. Doing an equality test to implicitly run the `io.FileIO.__init__` function, which opens the `flag` file.
6. Iterate over the open `io.FileIO` object to read the flag.
7. Use the `AssertionError` raised by an `assert` statement as a `print` function to print the flag (after setting stderr to stdout because the challenge server doesn't print stderr).

The explanations for all these tricks and some insight into how I came upon them follows.


## The Sandbox Environment

The aim is to execute `print(open("./flag").read())` (or equivalent) without any explicit function calls. By "explicit function call" here I mean that nothing in your code should qualify as an `ast.Call` node. This includes normal functions like `print`, but also includes things like instantiating a class (and thus raising an exception as well, since you have to instantiate an `Exception` class to do so).

However, this does _not_ mean that it is impossible to call functions. It only means that it shouldn't _look_ like you're calling a function.

Of course, everything else that is legal python - assignments, operations on things, etc - is fair game.

Notice that our code executes in the context of the sandbox environment itself. This means that it implicitly has access to any variables/functions defined in `treebox.py`. For example, you don't need to import `os`, `ast` or `sys` - since `treebox.py` imports them already, you can use them straight away. Similarly, you have a variable called `dname` defined for you already because `treebox.py` defines it and sets it to the current directory. This comes in quite handy.

Other solutions exist which let you completely bypass the restrictions of the sandbox (e.g. see [the challenge author's solution][1]) and call whatever you like, but my approach here was to creatively do the equivalent of calling the three functions we need: `print`, `open` and `read`, and chaining the tricks used to do so to leak the flag.


## The Tricks

### Printing

Since just reading the file containing the flag isn't enough, let's first figure out how to print something and actually see the result.

Raising an exception with a custom message was one of the first approaches to occur to me, but we have two problems:
1. The challenge server doesn't send stderr back to you (and exceptions are printed to stderr, of course).
2. Raising an exception usually involves instantiating an `Exception` subclass, which results in a function call like thingy, and the sandbox blocks us.

First problem is easy to solve. We already have `sys` imported in the execution context of our code, so we have control over `sys.stderr`. Doing `sys.stderr = sys.stdout` makes it so that our exceptions are now printed to stdout, which we can see!

Second problem is trickier, and took me a while. Eventually, we realized that an assert statement takes an optional second argument that becomes the message of the `AssertionError` raised when the assertion fails. Combining the two, we can print things! Here's an example that prints the environment of the sandbox process (sadly, nothing "fun" was found there):

```python
sys.stderr = sys.stdout

assert False, os.environ
--END
```


### Opening A File

After considering a bunch of options for things to try and use to solve this challenge, I found a useful class in the `io` library: the `io.FileIO` class. The very useful thing about this class is that if you instantiate it with a filename (e.g. `f = io.FileIO("./flag")`, it implicitly opens the file for you.

So the challenge now is to instantiate an `io.FileIO` object which has `"./flag"` as it's first argument.

#### Step 1: Importing The io.FileIO Class

Whelp. Unlike `sys` or `os`, the `io` module is not in scope in the execution environment of our sandbox. And the sandbox prevents any import statements.

Luckily, `sys` is in scope. And it maintains a reference to all modules available for import in `sys.modules`, which is a dictionary! So `sys.modules["io"]` is the `io` module, and thus this assignment allows us to access things exported by the module:

```python
FileIO = sys.modules["io"].FileIO
```

#### Step 2: Instantiating The FileIO Class

So it turns out that you can easily call any arbitrary function in python without it being an explicit function call. And that's by using it as a [decorator][3]. And since instantiating a class basically works like a function call, you can use classes as decorators too. Decorators are one of the neatest things about python, and this challenge made me like them even more.

The tricky part, however, is that when trying to use a decorator to call functions in this sandbox, you have very little control over the arguments that the decorator can work with. Although it is possible for decorators to have arguments (in this case, they must return another decorator), using decorators this way involves calling it like a function, which turns out to be an `ast.Call` node that is banned by the sandbox. So we're limited to using decorators without additional arguments, and such decorators can naturally only accept a function as an argument.

So for example, this attempt to use a decorator to open a file named `flag` via implicitly calling the `FileIO.__init__` method **won't** work and results in a `SyntaxError`:

```python
@FileIO
"./flag"
```

So let's define our own subclass of `FileIO` that can accept a function when instantiated. We can then decorate a random function with it to instantiate the class without an explicit function call.

```python
class FlagIO(FileIO):
    def __init__(self, fn):
        pass

@FlagIO
def hello():
    pass
```

Unfortunately, this does not work. That's because the part that opens the file - which is what we're running this whole circus for - happens in the `__init__` method of the original `FileIO` class. Which we override by necessity to allow it to be used as a decorator. Bummer.

But we at least managed to instantiate our custom `FlagIO` class, that's progress.

#### Step 3: Calling FileIO.__init__

One of the coolest things about python is that everything is an object, and these objects have magic methods that allow you to control the behaviour of the object. For example, if you define your own `__len__` method for an object, then you control what gets returned when the `len` function is called on an object.

And python lets you patch almost anything. That often includes magic methods. And we can use this to our advantage to initialize our class. My trick was to set the `__eq__` magic method to the `FileIO.__init__` method.

The `__eq__` magic method defines how your object behaves when it is being compared to another object using the `==` operator.

Why did I choose to do this? Because the `FileIO.__init__` method has two required arguments: `self` and `name`. The `__eq__` magic method also takes two arguments: `self` and `other`, where `other` is the thing you're comparing with.

```python
class FlagIO(FileIO):
    def __init__(self, fn):
        pass

FlagIO.__eq__ = FileIO.__init__

@FlagIO
def hello():
    pass

hello == "./flag"
```

This is what happens here:
1. `FlagIO` is used to decorate `hello`. This is the equivalent of calling `hello = FlagIO(hello)`, but since the `__init__` method of `FlagIO` is empty, nothing happens other than the assignment.
2. We do a comparison using `hello == "./flag"`. This invokes `FlagIO.__eq__(hello, "./flag")`. Which is the same as `FileIO.__init__(hello, "./flag")`.
3. `FileIO.__init__` implicitly opens the file corresponding to the name it is supplied.
4. We now have an open file aliased to the `hello` variable, and we profit.

And thus we've opened the file! Now onto reading it...

### Reading A File

Squinting at the methods and properties of `io.FileIO` using `dir(io.FileIO)` (for longer than I'd like to admit) made one method stand out: the `__iter__` magic method. This implies that `FileIO` objects support iteration. And iterating over an `io.FileIO` object (and, from what I know, over any of these stream wrappers from the `io` package) is an interface to read it's contents! In other words, with our open file in the `hello` variable, we can now do `flag = [a for a in hello]` in order to read it's contents into `flag`. Since no functions are called here, the sandbox allows this!


## Profit!

Lo and behold!

```
gctf-2022/sandbox/treebox via 🐍 v3.10.4 on ☁️  (us-east-1) 
❯ exploit
== proof-of-work: disabled ==
-- Please enter code (last line must contain only --END)
-- Executing safe code:
Traceback (most recent call last):
  File "/home/user/treebox.py", line 51, in <module>
    exec(compiled)
  File "input.py", line 20, in <module>
AssertionError: [b'CTF{CzeresniaTopolaForsycja}\n']
```


  [1]: https://github.com/google/google-ctf/blob/master/2022/sandbox-treebox/healthcheck/solution.py "Treebox Author's Solution"
  [2]: https://github.com/google/google-ctf/blob/master/2022/sandbox-treebox/challenge/treebox.py "The Treebox Source Code"
  [3]: https://realpython.com/primer-on-python-decorators/ "Primer on Decorators"
