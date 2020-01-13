---
layout: post
author: Koffiebaard
title: Testing-framework based on bash one-liners
description: Using bash one-liners for validation testing makes it easy, powerful, flexible, easy to learn and lightweight. It can be as simple as one file.
tags: [bash]
page_class: post
---

**tl;dr:** using bash one-liners for validation testing makes it easy, powerful, flexible, easy to learn and lightweight: it can be as simple as one file. Any single test can be copy-pasted and run by anyone. [See a working example here](https://github.com/wisc/bash-oneliner-testing-example/blob/master/validate.sh).

I was looking for a way to easily verify the end result when deploying or migrating my website + API (it’s a [webcomic](https://consolia-comic.com/) btw). I wouldn’t call it integration testing, it’s more like validation testing.

For the website-part i’d like to verify things like:

- Are the html pages / json responses being compressed properly?
- Is https working? Using the right tls protocol negotiation?
- Is Varnish caching properly? Is it skipping the cache wherever it should?
- Does the homepage show the latest comic? Is that comic image returning a 200?
- Is the Cache-Control header set properly for all assets in the pages?
- Are all other pages returning 200's? Are requests to the assets folders returning 403's? Are redirect endpoints returning 303's? Are non-existent pages returning 404's?

Etc, etc. I could go on for a while. These things are a hassle to test manually, which means i don’t.


## Bash one-liners to the rescue

Doing validation testing through bash means you have the entirety of Bash at your disposal. Anyone who knows bash can immediately work with it, any test can be shared with anyone, in order to be reproduced. Considering Bash’s flexibility, a lot of things can be thoroughly tested in straightforward one-liners.

I’m a back-end developer. Whenever i need to pass along a test case for a bug to a third party, or even another developer in my team, i send them a curl command. They’re easy to write, and can be passed to anyone by chat or mail.

Once you have that you’re almost there. To turn a curl command into an actual test, just pipe the response / req. headers into some bash that verifies whatever you’re looking for.


## The code

Take a simple Varnish test for example, verifying the homepage is cached properly:

```bash
# Verify 200 OK on homepage, through Varnish
curl -X GET "http://consolia-comic.com/" -si | grep HTTP | awk '{print $2}' # returns the response code, hopefully 200

# Verify cache hit on homepage
curl -X GET "http://consolia-comic.com/" -si | grep "X-Cache:" | grep HIT | wc -l # returns 1 on cache hit, 0 on cache miss
```

So i wrote a simple function that compares the result of a bash one-liner to the value it should be.

A very simple validate function could be:

```bash
validate () {
  is=$2
  should_be=$3
  if [[ $is -eq $should_be ]]; then
    echo -e "\e[92m[$1]\tPassed.\e[0m"
  else
    echo -e "\e[31m[$1]\tFailed. Should be $should_be, is $is\e[0m"
  fi
}
```

with its syntax being:

```bash
validate "description" <bash one-liner> <expected result>
```

If a test passes, say so in a pretty little green color. If not, mention the value difference in an evil red. Of course you can extend this with the return status for automation.

You could write the tests like this:

```bash
printf "\nVarnish\n"

validate "200 OK on /" $(curl -X GET "http://consolia-comic.com/" -si | grep HTTP | awk ‘{print $2}’) 200

validate "cache hit on /" $(curl -X GET "http://consolia-comic.com/" -si | grep "X-Cache:" | grep HIT | wc -l) 1
```

An example result could be:

```
Varnish
[200 OK on /] Passed.
[cache hit on /] Failed. Should be 1, is 0
```

**Mind you, there are some disadvantages.** First off, this is solely for validation testing. You can also use it to validate json schema’s for example, but as soon as you need a test database you should add a second solution for your testing needs.

Also, the endpoints are (of course) environment-specific. You can switch the hostname with an environment variable, though that does lose some of the interoperability, however slightly, since you’d need to switch back to a hostname if you’re passing on a single test to a 3rd party.


## Summarizing

It’s powered by Bash, so it’s:

- Flexible.
- Easy to learn.
- Easily shared, even to 3rd parties.
- Powerful (anything bash can do, can be used to test).
- No dependencies (except curl).
- Lightweight and transparent. It can be one file, you don’t need any libraries.


And, best of all, they’re /bash one-liners/. How awesome is that?