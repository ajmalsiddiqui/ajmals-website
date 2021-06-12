---
title: "MetaCTF Cybergames 2020: Password Here Please"
date: 2020-10-27T10:38:30+05:30
lastmod: 2020-10-27T10:38:30+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2018/05/14/16/54/cyber-3400789_960_720.jpg
categories:
  - ctf writeups
tags:
  - security
  - reverse engineering
  - metactf cybergames 2020
showcase: false
draft: false
---

This is a write-up for the "Password Here Please" reverse engineering challenge from [MetaCTF CyberGames 2020][1], which was worth 325 points.

Challenge statement: I forgot my bank account password! Luckily for me, I wrote a program that checks if my password is correct just in case I forgot which password I used, so this way I don't lock myself out of my account. Unfortunately, I seem to have lost my password list as well... Could you take a look and see if you can find my password for me? Part 3 requires some math skills. To solve it, think about what is being done by the exponentiation step. Try rewriting the large number in base 257.
<!--more-->

We are given a small python program that validates the password. Here it is:

```python
def ValidatePassword(password):
    print("Password Validator v1.0")
    print("Attempting to validate password...")

    if(len(password[::-2]) != 12 or len(password[17:]) != 7):
        print("Woah, you're not even close!!")
        return False

    pwlen = len(password)
    chunk1 = 'key'.join([chr(0x98 - ord(password[c]))
                             for c in range(0, int(pwlen / 3))])
    if "".join([c for c in chunk1[::4]]) != '&e"3&Ew*':
        print("You call that the password? HA!")
        return False

    chunk2 = [ord(c) - 0x1F if ord(c) > 0x60
                  else (ord(c) + 0x1F if ord(c) > 0x40 else ord(c))
                  for c in password[int(pwlen / 3) : int(2 * pwlen / 3)]]
    ring = [54, -45, 9, 25, -42, -25, 31, -79]
    for i in range(0, len(chunk2)):
        if(0 if i == len(chunk2) - 1 else chunk2[i + 1]) != chunk2[i] + ring[i]:
            print("You cracked the passwo-- just kidding, try again! " + str(i))
            return False

    chunk3 = password[int(2 * pwlen / 3):]
    code = 0xaace63feba9e1c71ef460e6dbf1b1fbabfd7e2e35401440ac57e93bd9ba41c4fbd5d437b1dfab11fe7a1c6c2035982a71765fc9a7b32ccef695dffb71babe15733f5bb29f76aae5f80fff
    valid = True
    for i in range(0, len(chunk3)):
        if(ord(chunk3[i]) < 0x28):
            valid = False
        code -= (257 ** (ord(chunk3[i]) - 0x28)) * (1 << i)

    if code == 0 and valid:
        print("Password accepted!")
        return True
    else:
        print("Quite wrong indeed!")
        return False

print("Please enter password")
while not ValidatePassword(input()):
    pass
```

Right, lets reverse this!

## A High Level Overview

Since we have the luxury of having the source code at our disposal (and in a language as readable as Python!), let's get an overview of what's going on here. The easiest way to do so is to just run the validator. I've saved the program in a file called `validator.py`. Start with running

```
python validator.py
```

Note that you'll have to enter the password surrounded with quotes or python tries to evaluate the password as a symbol and crashes.

The program prompts us to enter the password. Let's try something simple, like `"helloworld"`. It responds with a rather condescending `Woah, you're not even close!!`, following which we get another shot at entering the password.

Okay, not very nice. Let's take a cursory glance at the code and get an idea of what wer're dealing with.

There's a `ValidatePassword` function which seems to do all the heavy lifting here, and it returns `True` if the password was accepted, and `False` otherwise (and boy, it returns `False` a lot!). This function is run in a `while` loop that runs until a valid password is provided (or until you stop the program with `Ctrl+c`).

```python
print("Please enter password")
while not ValidatePassword(input()):
    pass
```

Now let's start breaking down the `ValidatePassword` function. At a glance, you can tell that it applies various validations one after another, and if a validation fails, it returns `False`. We see references to `chunk1`, `chunk2` and `chunk3`, each of which are validated independently. And towards the start, there seems to be something that validates the length of some convoluted subsection of the password. That's 4 parts in total!

Let's work through these in order!

> Note: Each stage of the reversing process is under it's own heading. So if you've already figured something out, feel free to skip ahead. This is a long read!

## Stage 1: The Length of the Password

Our validation function kicks of with this bit:

```
if(len(password[::-2]) != 12 or len(password[17:]) != 7):
    print("Woah, you're not even close!!")
    return False
```

It checks for two conditions, and if either one is satisfied, the password is invalid, and we get the `Woah, you're not even close!!` response we saw earlier. The second one looks easier, so we'll start with that. The `password[17:]` part is python syntax for "get me all the elements in this iterable starting from index 17". The condition is true if the length of the part of the password starting from the 17th character is not 7. Since we want this to be false, we realise that our password has a length of 17 + 7, which is 24!

The first part of the condition also checks for the same thing, but in a much more convoluted way. The `password[::-2]` is python speak for "Give me all the characters in this list, from start to finish, but start at the end and take 2 steps at a time". Let's break that down for the sake of clarity. Fire up a python interpreter using `python` and enter these commands:

```
# Define the password
password = "mysupersecretpass"

# password[:] just gets all the characters in the password, which returns the password itself
password[:] # mysupersecretpass

# password[::-1] gets all the characters in the password, but starts from the end
password[::-1] # ssaptercesrepusym

# password[::-2] gets all the characters starting from the end, but skips every alternate character
password[::-2] # satrerpsm

```

The condition is that `len(password[::-2])` should not be 12. Since we want it to be false, we realise that the length of the string returned by taking every alternate character of the password should be 12, which means that the length of the password should be `12 * 2 = 24`. Perfect!

Run `validator.py` again, but this time enter a string of length 24. I just entered `"aaaaaaaaaaaaaaaaaaaaaaaa"`.

This time the validation script says `You call that the password? HA!`. Sweet, we've gotten past the first stage!

## Stage 2: Deobfuscation and Inversion

Okay, here's the code for the second stage:

```python
pwlen = len(password)
chunk1 = 'key'.join([chr(0x98 - ord(password[c]))
                         for c in range(0, int(pwlen / 3))])
if "".join([c for c in chunk1[::4]]) != '&e"3&Ew*':
    print("You call that the password? HA!")
    return False
```

At this point, we'll have to do some computations. Most people would write a script to solve this, but I preferred an interactive method where I could try my ideas in an interpreter. So that's what we'll be doing here. Fire up a python interpreter using `python` and let's get started!

It looks like this stage uses a subsection of the password to create the first chunk, `chunk1`. We see that it operates on the characters of the password from index 0 to `pwlen / 3`, where `pwlen` is the length of the password. Since we already know that the password is 24 characters long, `pwlen / 3 = 8`. The first chunk operates on the first 8 characters, so that's what we have to figure out.

Let's look at what `chunk1` looks like. In your interpreter, enter the code for finding `chunk1` verbatim:

```python
password = "abcdefghijklmnopqrstuvwx"
pwlen = len(password) # 24

# returns '7key6key5key4key3key2key1key0'
chunk1 = 'key'.join([chr(0x98 - ord(password[c])) for c in range(0, int(pwlen / 3))])
```

Looking up the `chr` and `ord` functions tells us that `ord` converts a single character into it's integer ordinal, and `chr` does the opposite - convert an integer ordinal into a single character.

> Tip: You could google the `chr` and `ord` functions, but a real hacker would use the `help` function of the interpreter! Try typing `help(chr)` or `help(ord)` in the interpreter.
>
> Okay, I lied. A _real_ hacker would use `chr.__doc__` and `ord.__doc__`. ;)

So it looks like this is constructed by:
1. Converting each character of the password into an integer using `ord`
2. Subtracting each of these integers from `0x98`
3. Converting the resultant back into a character using `chr`
4. Constructing a string by putting the word "key" between each character returned

Whew! Okay, let's touch this when we need to. Let's look at the next part, which has our condition.

```python
if "".join([c for c in chunk1[::4]]) != '&e"3&Ew*':
    print("You call that the password? HA!")
    return False
```

Here, we use another string constructed from `chunk1` by taking every 4th character of `chunk1`, starting from the first one. This is what `chunk1[::4]` does. Type this into the interpreter:
```python
"".join([c for c in chunk1[::4]]) # '76543210'
```

It looks like... this removes the `key` between each character? That makes the whole adding "key" part from earlier pointless! Why would you ever do that?

Obfuscation. They're trying to make our job harder and send us down rabbit holes. But it'll take more than that to confuse us!

Let's simplify this stage a little.

```python
pwlen = len(password)
chunk1 = ''.join([chr(0x98 - ord(password[c]))
                         for c in range(0, int(pwlen / 3))])
if chunk1 != '&e"3&Ew*':
    print("You call that the password? HA!")
    return False
```

Okay, that is so much easier to understand! All we need to do is find an 8 character string that, when transformed into `chunk1`, results in `&e"3&Ew*`.

In order to pull this off, let's try and convert the `chunk1` that we expect back into the string. In other words, let's write a function that can _invert_ the transformation that is applied to obtain `chunk1`. It's time for some reversing!

So how do we figure this out? For starters, let's say that the integer returned by `ord(password[c])` is `x`. Then the transformation becomes `chr(0x98 - x)`. Let's forget the `chr` for a second. The crux of the transform is the `0x98 - x`. Let's say that this results in an integer `y`. We then have the equation `0x98 - x = y`. Now here's the question: Given `y`, how do you find `x`? Yep, we can rewrite that as `x = 0x98 - y` and solve for `x`.

In order to convert that back to a character, we apply the `chr` function, thus resulting in `chr(0x98 - y)`. Remeber that `y` is the integer ordinal of a character from `chunk1`, so we need to convert a character `c` from `chunk1` into it's ordinal before we use our little reverse-function. So, for a character `c` of `chunk1`, we can figure out the corresponing character of the password using `chr(0x98 - ord(c))`. This looks... familiar? This is exactly what we're doing to calculate `chunk1` in the first place! This is a function whose inverse is... itself!

That makes our job easy. We don't even need to write any code to figure out the first 8 characters. Since we know that chunk1 should be `&e"3&Ew*`, and that the function that calculates `chunk1` is its own inverse, we can just give `&e"3&Ew*` as an input to the function itself and figure out the first 8 chars of the password!

Let's modify the `validator.py` file to print `chunk1`. Add this line right after `chunk1` is calculated:

```python
print(chunk1)
```

Let's run `validator.py` again, but this time we'll set the first 8 characters of the password to `&e"3&Ew*`, and the remaining 16 to something random like "aaaaaaaaaaaaaaaa". **Make sure that you've simplified the code as per the instructions above, and replaced the `"key".join` with `"".join`.**  Here's the input you need:

```
'&e"3&Ew*aaaaaaaaaaaaaaaa'
```

And that prints `r3verS!n`! So the the first 8 characters of the password must be `r3verS!n`. Let's try this input next to confirm:
```
'r3verS!naaaaaaaaaaaaaaaa'
```

And this time it tells us that `You cracked the passwo-- just kidding, try again! 0`! Looks like we've gotten past this stage and figured out the first 8 characters correctly!

## Stage 3: Reverse and Inverse

We move on to the next snippet, which deals with `chunk2`. Judging by the range in the list comprehension which creates `chunk2`, it looks like it operates upon the next 8 characters of our 24 character password.

```python
chunk2 = [ord(c) - 0x1F if ord(c) > 0x60
              else (ord(c) + 0x1F if ord(c) > 0x40 else ord(c))
              for c in password[int(pwlen / 3) : int(2 * pwlen / 3)]]
ring = [54, -45, 9, 25, -42, -25, 31, -79]
for i in range(0, len(chunk2)):
    if(0 if i == len(chunk2) - 1 else chunk2[i + 1]) != chunk2[i] + ring[i]:
        print("You cracked the passwo-- just kidding, try again! " + str(i))
        return False
```

We see that the list comprehension being used to construct `chunk2` returns the result of some arithmetic operations on the ordinal value of each character (which is obtained using the `ord` function). So `chunk2` is a list of 8 integers.

And then there seems to be a list of seemingly random numbers called `ring` which is used to validate `chunk2`.

Finally, we have a for loop that iterates over the elements of `chunk2` and applies a condition to each of them in order to validate them. In essence, we will need the second group of 8 characters of the password to generate an array of integers `chunk2` that passes this validation. It makes sense that we want to know `chunk2` first (and also, this is a _reversing_ challenge, hehehe), so let's go in reverse and figure out the integer array `chunk2`.

Here is the condition (where `i` is the index):

```python
(0 if i == len(chunk2) - 1 else chunk2[i + 1]) != chunk2[i] + ring[i]
```

This might seem complex at first glance (especially because of the ternary operator), but it really isn't. It just says that for all values of `i` except the last (i.e. except when `i` equals `len(chunk2) - 1`), `chunk2[i] + ring[i]` must equal `chunk2[i+1]` (i.e. the value at the next index of `chunk2`. For the last value of `i`, `chunk2[i] + ring[i]` must equal 0.

Since we know the exact value of `chunk2[i] + ring[i]` for the last element (`i=7`), we can start with that and work our way backwards. Lot's of reversing here!

Fire up your python interpreter and let's play with this. The comments will explain what's happening.

```python
# let's first grab the ring from the source code
ring = [54, -45, 9, 25, -42, -25, 31, -79]

# let's create a list of 8 integers to represent chunk2
# we initialize the elements of chunk2 to 0
chunk2 = [0 for i in range(8)] # [0, 0, 0, 0, 0, 0, 0, 0]

# we know that chunk2[7] + ring[7] = 0
# therefore...
chunk2[7] = 0 - ring[7] # [0, 0, 0, 0, 0, 0, 0, 79]

# we work our way backwards to calculate the rest of the elements of chunk2
# using the knowledge that since chunk2[i+1] = chunk2[i] + ring[i]
# chunk2[i] = chunk2[i+1] - ring[i]
for i in reversed(range(7)):
  chunk2[i] = chunk2[i+1] - ring[i]

# print chunk2 (note that if you are in the interpreter, you can just enter the variable name instead of using the print function)
chunk2 # [72, 126, 81, 90, 115, 73, 48, 79]
```

You can confirm that this is the right value of `chunk2` by placing this line of code right before the for loop that validates it and checking if you are able to get past this stage.

```python
chunk2 = [72, 126, 81, 90, 115, 73, 48, 79]
```

Okay, now that we know our target value of `chunk2`, it is time to figure out the characters that generate it. Time to reverse the part that generates `chunk2`!

This source code seems to make rather generous use of the ternary operator which (possibly intentionally) hurts readability. To make matters worse, we have a list comprehension! So let's refactor the whole thing into a vanilla for loop which uses a function to get the right value for each char. Here's what that might look like:

```python
def chunk2_char(c):
    if ord(c) > 0x60:
        return ord(c) - 0x1F
    else:
        if ord(c) > 0x40:
            return ord(c) + 0x1F
        else:
            return ord(c)

chunk2 = []
for c in password[int(pwlen / 3) : int(2 * pwlen / 3)]:
    chunk2.append(chunk2_char(c))
```

In this form, the code becomes far easier to analyse. Refactoring code to improve readability and mitigate obfuscation is more important than you might expect during reverse engineering.

Now lets try to invert the `chunk2_char` function.

Let's start with the return values. The easiest one is the `return ord(c)` in the very end. This just converts the character to the integer ordinal, so to invert that, we can use the `chr` function.

For the condition where `ord(c) > 0x60`, the function returns `ord(c) - 0x1F`. To invert this, let's say that `x = ord(c)`. We then solve the equation `x - 0x1F = y` for `x`, which gives us `x = y + 0x1F`. Since `x = ord(c)`, we can get `x` by using the inverse of the `ord` function, which, as you are well aware by now, is the `chr` function. Thus, the inversion of `ord(c) - 0x1F` becomes `chr(x)` which boils down to `chr(x + 0x1F)`.

By a similar chain of reasoning, you'll find that the inverse of `ord(c) + 0x1F` is `chr(x + 0x1F)`.

Now let's look at the conditions. In the original function, the first condition is `ord(c) > 0x60`. Let's think of this as follows: let's say that our inverse function takes an integer ordinal `x` and returns a character `c`. When we apply the _original_ function to `c`, we should get `x` back. So when we check whether `ord(c) > 0x60` is true in the _original_ function, the `c` here is the value _returned_ by our inverse function. Which means that for each of the conditions, we can replace the `ord(c)` with the integer ordinal of the value returned if that condition is true.

For example, if the first condition (`ord(c) > 0x60`) is true, the returned value is `chr(x + 0x1F)`. Replacing the `c` with this value, we get the condition `ord(chr(x + 0x1F)) > 0x60`. But we can simplify this knowing that the `ord` function is the inverse of the `chr` function. So `ord(chr(x)) = x`. Applying this, we get the inverse condition `x + 0x1F > 0x60`. We can rearrange terms and make this `x > 0x60 - 0x1F` which simplifies to `x > 65` (you can do all these calculations in your interpreter).

By using similar reasoning, we figure out that the inverse of the second condition is `if x - 0x1F > 0x40`, which simplifies to `x > 95`.

Armed with the inverses of each of the pieces, we can construct our inverse function as follows:

```python
def inverse_chunk2_char(x):
  if x > 65:
    return chr(x + 0x1F)
  else:
    if x > 95:
      return chr(x - 0x1F)
    else:
      return chr(x)
```

And apply it to the `chunk2` value we calculated earlier like this:

```python
result = ''.join(list(map(inverse_chunk2_char, chunk2)))

# if you don't understand the map syntax very well, this list comprehension also works
result = ''.join([inverse_chunk2_char(x) for x in chunk2])
```

But if you actually look at the value of `result`, you'll see that though it looks like we got _some_ things right, some of the values are non-printable. Looks like there is an issue with our inverse function.

The issue lies in the order in which we check our conditions. Think about what would happen for the value `x = 126` (the second item in the `chunk2` array). The first condition checks out because obviously `126 > 65`. But so does the second condition (`126 > 95`). In fact, if you think about it, you'll realise that _the first condition is true whenever the second condition is because 95 > 65_. So we don't ever reach the second condition when the first one is true. Even when you need to. To fix this, we just reorder things a little (note that all conditions are independent, so we don't need the nested if here anyways).

```python
def inverse_chunk2_char(x):
  if x > 95:
    return chr(x - 0x1F)
  elif x > 65:
    return chr(x + 0x1F)
  else:
    return chr(x)
```

We now retrieve the 8 characters we need using

```python
result = ''.join(list(map(inverse_chunk2_char, chunk2)))
# 'g_pyTh0n'
```

That looks legit. So the first 16 characters of our password seem to be `r3verS!ng_pyTh0n`. Reversing python huh?

Entering `'r3verS!ng_pyTh0naaaaaaaa'` in the password prompt results in `Quite wrong indeed!`. Nice, we've gotten past this stage!

## Stage 4: Doin' the Math

We're finally at the last stage. Let's look at what we have:

```python
chunk3 = password[int(2 * pwlen / 3):]
code = 0xaace63feba9e1c71ef460e6dbf1b1fbabfd7e2e35401440ac57e93bd9ba41c4fbd5d437b1dfab11fe7a1c6c2035982a71765fc9a7b32ccef695dffb71babe15733f5bb29f76aae5f80fff
valid = True
for i in range(0, len(chunk3)):
    if(ord(chunk3[i]) < 0x28):
        valid = False
    code -= (257 ** (ord(chunk3[i]) - 0x28)) * (1 << i)

if code == 0 and valid:
    print("Password accepted!")
    return True
else:
    print("Quite wrong indeed!")
    return False
```

This stage follows the same pattern of taking an 8 character long chunk and applying some validations on it. If these pass, you will (presumably) get the coveted `Password accepted!`.

We start with a really, _really_ long hexadecimal number called `code`. And the condition to make it work is quite straightforward: `code` must equal 0 and `valid` must be `True`.

All the processing happens in this for loop:

```python
valid = True
for i in range(0, len(chunk3)):
    if(ord(chunk3[i]) < 0x28):
        valid = False
    code -= (257 ** (ord(chunk3[i]) - 0x28)) * (1 << i)
```

For starters, we are looking at the `ord` values of the characters. So we're dealing with numbers here.

The first thing is a condition that checks if `ord(c) < 0x28` (where `c` is a character of chunk3), and if so, sets `valid` to `False`. We don't want that, so we obviously don't want to trigger this condition. It is thus obvious that all the characters of this chunk will have integer ordinals greater than 0x28. Easy enough.

The next line does some strange maths. In each iteration, it subtracts from the current value of `code` the result of this strange calculation: `(257 ** (ord(chunk3[i]) - 0x28)) * (1 << i)`. In the end, the value of `code` must be 0. Since we are dealing with 8 iterations (corresponding to the last 8 characters of the password), we are doing this subtraction 8 times. In other words, we are subtracting 8 values from `code`, and this must result in a `0`. Let's say that these values are `x0` to `x7`. We can then express the intent of the for loop and condition as this equation:

```
code - x0 - x1 - x2 - x3 - x4 - x5  -x6 -x7 = 0
=> code = x0 + x1 + x2 + x3 + x4 + x5 + x6 + x7
=> x0 + x1 + x2 + x3 + x4 + x5 + x6 + x7 = code
```

Where `x0...x7` are the various values of the convoluted expression `(257 ** (ord(chunk3[i]) - 0x28)) * (1 << i)`.

It's time to break this expression down!

To make things easier, let's say that `ord(chunk3[i]) - 0x28 = yi`. This is easy to invert and lets us focus on the right things. The expression then becomes `(257 ** yi) * (1 << i)`. That's much better.

The `(1 << i)` represents a bitwise left shift. If you don't know how these work, I suggest doing a google search at this point. All you need to know for this part is that a bitwise left shift by `i` places is equivalent to multiplying the given number by `2^i`. Since we are shifting `1` in all the cases, the `(1 << i)` turns into `1 * 2^i` which is simply `2^i`. Finally, just to use more comfortable mathematical notation, let's replace the python exponentiation operator `**` with the more familiar `^`. We now get something that looks a lot more manageable:

```
(257 ^ yi) * (2 ^ i)
```

Our problem now reduces to finding 8 values (`y0 to y7`) such that plugging each of them into the above expression and summing them up results in `code`. That's troublesome. We have 8 unknowns and only one equation. And it isn't even a polynomial one!

I was stuck at this point for a bit, but that's when the hint from the challenge came to my rescue! To recap, "Part 3 requires some math skills. To solve it, think about what is being done by the exponentiation step. Try rewriting the large number in base 257."

Yes! We can convert this to base 257! To understand why, let's take a quick look at what it means to be a number in a certain base `b`. Let's say we have the octal number 176. Let's convert this into the more familiar base 10 (decimal). To do this, we run the following calculation (your interpreter is a good place to play with this):

```
6 * (8 ** 0) + 7 * (8 ** 1) + 1 * (8 ** 2)
# 126
```

There is a pattern here. Starting from the _end_, we take each digit and multiply it by the base raised to the power of the position of the digit (starting with 0, which is the position of the _last_ digit).

So if we convert `code` into base 257, we will have a resultant that looks something like:

```
a * (257 ** 0) + b * (257 ** 1) + .... + x * (257 ** k)
```

Where `k` is one less than the number of digits in base 257 that code has. This matches with the pattern we saw in the previous expression! Let's try this and see where it takes us.

I found [this base conversion python script][2] online which was well written and up to the task, and the article accompanying it is quite good too.

Copy the code from the article into a python file and name it `bconvert.py`. Since we need to get a list of what would be the digits in base 257, we'll take a quick look at the code and print this list out. It turns out that this is quite simple. Most of the work happens in the `convert_number_system` function. Right after the main `while` loop (the `while sum_base_10 > 0` condition) completes, add this line:

```python
print(remainder_list)
```

Run the converter using `python bconvert.py`. Note that this one requires python3. When asked for the base to convert from, enter 16 (since `code` is in hexadecimal). For the base to convert to, enter 257. Finally, for the number to convert, enter the value of `code`.

The script should print a list containing our digits (ignore the final output; you don't have 257 digits to represent base 257) that looks like this (note that you'll see an array of strings, but I've converted it into numbers):

```
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 32, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0, 0, 0, 0, 0, 64, 0, 0, 0, 0, 0, 0, 0, 0, 0, 17, 0, 0, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 128, 0, 0, 0, 8]
```

This is interesting! It looks like most of our "digits" are zero! We also seem to have _seven_ non-zero digits: 32, 4, 64, 17, 2, 128, and 8. You'll have to take a quick look at the base conversion script to realise that it conveniently prints out the digits _from the end_. So the digit at index 0 of this list is the last digit, and so on. Recall that we can convert this into the familiar decimal system by multiplying each of these digits with `257 ^ index`.

Let's quickly find the indices of the interesting non-zero digits. Here's a Python one liner. We'll also make a new list with the non-zero digits:

```python
digits = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 32, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0, 0, 0, 0, 0, 64, 0, 0, 0, 0, 0, 0, 0, 0, 0, 17, 0, 0, 0, 0, 0, 0, 2, 0, 0, 0, 0, 0, 0, 0, 128, 0, 0, 0, 8]

non_zero_digits = filter(lambda x: x, digits)
# [32, 4, 64, 17, 2, 128, 8]

indices = [digits.index(i) for i in non_zero_digits]
# [30, 39, 45, 55, 62, 70, 74]
```

Since most of our numbers are zero, `0 * (257 ^ index)` will just be 0, so we can ignore them. We can now express `code` mathematically as:

```
code = (257 ^ 30) * 32 + (257 ^ 39) * 4 + (257 ^ 45) * 64...
```

We immediately see that all the digits except 17 are powers of two, and they are being multiplied by something that is in the form `257 ^ yi`! This looks _exactly_ like our equation from earlier:

```
(257 ^ yi) * (2 ^ i)
```

Except we have two problems:
1. There are 8 terms in our equation (for 8 characters of the password), but we only have 7 non-zero digits.
2. The 17 is problematic because it isn't a power of 2.

The solution to both of these issues is the same. It is the subtle realization that `17 = 16 + 1`!

So we can rewrite the term of the equation involving the digit 17 as `(257 ^ 55) * 17 = (257 ^ 55) * 16 + (257 ^ 55) * 1`. We now have exactly 8 terms, all of which are in the form `(257 ^ yi) * (2 ^ i)`. Perfect!

We can now write each of the 8 digits we have in the form `(257 ^ yi) * (2 ^ i)`, so let's do that. I've ordered this based on increasing values of i:

```
(257 ^ 55) * (2 ^ 0) # 1 = 2 ^ 0, the index corresponding to 1 is the one for 17, which is 55
(257 ^ 62) * (2 ^ 1) # 2 = 2 ^ 1
(257 ^ 39) * (2 ^ 2) # 4 = 2 ^ 2
...
(257 ^ 70) * (2 ^ 128) # 128 = 2 ^ 7
```

With this ordering, we can construct a list of `yi` values, which are just the powers that `257` is raised to.

Now recall that we had set `yi = ord(chunk3[i]) - 0x28` earlier. To get `chunk3[i]` values, we need to invert this function. This is easy if you've gotten past the previous stage. We realise that `chunk3[i] = chr(yi + 0x28)`. Knowingthis, we can construct `chunk3` as follows:

```python
# Our list of yi values
yis = [55, 62, 39, 74, 55, 30, 45, 70]

chunk3 = [chr(yi + 0x28) for yi in yis]
# ['_', 'f', 'O', 'r', '_', 'F', 'U', 'n']

# We convert the character list into a string 
password_chunk = ''.join(chunk3)
# '_fOr_FUn'
```

And we have the last 8 characters! You can enter this in the password prompt to finally feel the euphoria of `Password accepted!`.

```python
'r3verS!ng_pyTh0n_fOr_FUn'
```

The flag is `MetaCTF{r3verS!ng_pyTh0n_fOr_FUn}`!


  [1]: https://metactf.com/cybergames "MetaCTF CyberGames 2020"
  [2]: https://towardsdatascience.com/how-to-write-a-number-systems-calculator-in-python-b172557cb705 "Number systems calculator in Python"
