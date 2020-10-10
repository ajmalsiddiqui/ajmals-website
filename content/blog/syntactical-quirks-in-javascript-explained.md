---
title: Syntactical Quirks Of JavaScript Explained
date: 2018-02-07T22:23:18+05:30
lastmod: 2020-10-10T06:57:00+05:30
author: Mohammed Ajmal Siddiqui
authorlink: https://ajmalsiddiqui.me
cover: https://cdn.pixabay.com/photo/2019/10/03/12/12/javascript-4523100_960_720.jpg
categories:
  - software development
  - JavaScript
tags:
  - javascript
showcase: false
draft: false
---

Tag along for a quick tour of some of the weirder things about JavaScript syntax that, once you pick them up, will (hopefully) become indispensible parts of your JS codebases. We'll look at the spread operator, IIFEs, and implicit returns in arrow functions.

<!--more-->

> Note: JavaScript is considered to be a very complex language. It is filled with things (read: pitfalls) that defy all reason, and require an understanding of the innards of the language to figure out why it works the way it does. Hence, I **strongly** recommend you go over the articles linked in the Suggested Reading section, especially if you are expecting to be interviewed for your JS skills.

Every programming language has some pieces of syntax that can confuse (read: scare) a developer looking at them for the first time. And given a language with a pace of evolution that is incredibly fast, such instances of weird syntax are inevitable. This article talks about 3 of such syntactical quirks that intimidated me when I first encountered them:

* the Spread Operator
* Immediately Invoked Functional Expressions
* Implicit return arrow functions

> Note: This article was written in 2018, and while the contents still hold true as of now, the language has evolved even more and this article barely scratches the surface of what JS can look like. The Suggested Reading section at the end of this article is a good place to start if you're left wanting for more!

These syntactical “quirks” might have intimidated me the first time I saw them, but now I use them all the time.

Such quirks are definitely not a bad thing. Every language has certain things that it simplifies, and different languages have different things that they simplify. Once you understand the syntax that is designed to make your life easier, you start to appreciate it. If you don’t, don’t worry. Once you switch to a different language that doesn’t make things as simple, you will definitely come to appreciate these quirks.

A good example of this is the idea of list/dictionary/set comprehensions in Python. Once you become comfortable with the syntax, your life as a Python developer will be incomplete without using these.

Without further ado, let’s dive into some of the aspects of JavaScript syntax that confused me when I first saw them in other people’s codebases.

## Spread Operator

The spread operator looks something like this:

```javascript
    const arr = [1, 2, 3];    // initialize an array
    const something = someFunction(...arr);
```

See the `...arr` in the argument to the function someFunction?

The `...` represents the spread operator. The spread operator allows you to **expand an iterable** in places where you need the individual elements laid out. Basically, if your function took the elements of an array as an argument (and not the array itself) you would use the spread operator. Consider the operator to simply replace the iterable it operates on with its elements.

So writing `someFunction(...arr)` is equivalent to writing `someFunction(1, 2, 3)`. Pretty neat, huh?

Where will you use this operator? There are various use cases. One of the most common ones is when using the `Math.max` or `Math.min` functions. These functions return the maximum of their arguments, and you can provide as many arguments as you like. But they don’t work with lists. So if you tried something like `const max = Math.max(arr)` it wouldn’t really work. However, since we know that `Math.max(1, 2, 3)` would return the desired value, we can try this:

```javascript
    const max = Math.max(...arr);
```

Which would “spread” the elements of arr and thus be equivalent to:

```javascript
    const max = Math.max(1, 2, 3)
```

Which is super neat!

[Here’s][3] the documentation for the spread operator.

## Immediately Invoked Functional Expressions (IIFE)

This is something you’ll see in the JavaScript files of many web front-end codebases:

```javascript
    (function(){
        //do some stuff
    })();

    // And here's the ES6 variant with arrow functions
    (() => {
       //do some stuff
    })();
```

When I first saw this, I was definitely intimidated. Despite the unusual syntax (at least in my opinion), IIFEs are fairly simple to understand. However, before we talk about this aspect of JavaScript sorcery, let’s understand the difference between functional declarations and functional expressions first.

Consider these three snippets of code:

```javascript
    // First snippet
    const myFunction = () => {
        console.log("hello world");
    }
    myFunction();

    // Second snippet
    function myFunction() {
        console.log("hello world");
    }
    myFunction();

    // Third snippet
    (() => {
        console.log("hello world");
    })();
```

All of these snippets do the same thing — they print “hello world” to the console. However, in the first two snippets, we explicitly have to call myFunction in order to execute it. This is clearly not the case in the second function.

The difference is that the first two snippets demonstrate a functional declaration — you are declaring a function and defining it as well, but unless you call it (using the `myFunction()` call) the function won’t execute. This is fairly obvious and it is what you are used to.

In the third snippet, which is a functional expression and an IIFE notice the following:

* The definition of the function is wrapped between parentheses.
* There is a pair of parentheses () after the definition, which is what you normally see when calling a function.

And here’s what happens:

1. JavaScript evaluates the contents between the parentheses as an *expression*. This works almost analogously to normal arithmetic expressions.
2. When the closing parenthesis of the function definition is reached, JavaScript has essentially *evaluated* the expression (i.e. the function, in this context).
3. The extra pair of parenthesis in the end *call* the function that was just evaluated.

This has the exact same effect as the following code:

```javascript
    const myFunction() {
        console.log("hello world");
    }

    myFunction();
```

The functional expression is just some syntactical icing on the cake that lets you call a function the moment the JavaScript begins execution.

The most common and most obvious use case for this would be when initialising the JavaScript part of a web app. You can attach event handlers and do all the usual stuff within the IIFE.

Oh and by the way, IIFEs are pronounced “iffys”. JavaScript can get weird sometimes.

## Implicit Return Arrow Functions

Those of us who use ES6 are quite comfortable with arrow functions. We’re all used to seeing this:

```javascript
    const myCoolFunction = () => {
        // do cool stuff
    }
```

And we’re also used to anonymous arrow functions. Let’s take an example of anonymous arrow functions in order to delve into arrow functions with implicit returns. Let’s say we want to take an array of integers and get an array with their elements doubled from it. A natural approach to doing this is to use the map method of arrays.

The map method takes a function with one argument as an argument and applies the operations within it to the element, and returns the result. The argument to the arrow function represents an element of the array — hence the map method maps each element of an array to another and returns the resultant array.

The usual way would be something like this:

```javascript
    const myArr = [1, 2, 3, 4];

    const doubledArr = myArr.map((element) => {
        return element*2;
    });

    console.log(doubledArr);  // Outputs [2, 4, 6, 8]
```

As you can see, we’re returning twice the element’s value, and this is what the elements of the array map to. There is an explicit (i.e. obvious) return statement in this anonymous function.

However, this can work just as well without a return statement. Here’s how:

```javascript
    const myArr = [1, 2, 3, 4]

    const doubledArr = myArr.map(element => element*2);

    console.log(doubledArr); //[2, 4, 6, 8]
```

Right off the bat, this approach is much cleaner. However, this is weird.

* There are no curly braces around the function definition.
* There is no return statement.

This is an inline implicit return arrow function. In this case, since there is only one line, the return statement is implied. Thus, even though there is no return statement in the code, the code behaves as if there is one. This is what we mean by an “implicit” return arrow function. The above code has an arrow function that “implicitly” returns `element*2`.

This syntax is much cleaner than the one previously used, and it is a method of choice when using methods like map, reduce, etc.

## Suggested Reading

This section links to articles that I think are a good complement for this one. It is important. This article went over just 3 arbitrarily selected pieces of syntax, but JS is a far bigger beast. Check em out!

1. [The JavaScript Hiring Guide][4] details tricky JS interview questions, and explains the reasoning behind the strange behaviour of JavaScript in certain situations. Some of these tripped me up as well!
2. [The Complete Beginners JavaScript Handbook 2020][5] is a good walkthrough and reference for the basics of modern JS.

I hope this article helped you understand some of the more quirky aspects of JavaScript and hopefully they wont intimidate you any further.

Also, do connect with me on [GitHub][1] and [Twitter][2].

*Happy Coding! :)*

  [1]: https://github.com/ajmalsiddiqui/ "Ajmal's GitHub"
  [2]: https://twitter.com/_ajmalsiddiqui "Ajmal's Twitter"
  [3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator "Spread Operator documentation"
  [4]: https://www.toptal.com/javascript#hiring-guide "JavaScript Hiring Guide"
  [5]: https://www.freecodecamp.org/news/the-complete-javascript-handbook-f26b2c71719c/ "The Complete JavaScript Handbook"
