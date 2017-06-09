+++
date = "2017-06-08T15:03:06+03:00"
draft = false
tags = ["dev", "balloon-lang"]
title = "Introducing Balloon: A New Programming Language"

+++

Since February this year, I've been working on a project that I've wanted to work on for quite some time now - a new programming language. After about 200 commits, things are now in working condition!

<!--more-->
The project has just reached the 0.1.0 release and it doesn't do much yet - it certainly isn't _useful_. But it's got all the absolute basics of a programming language: values of different types, variables, conditionals, loops, functions (closures) and even a proof-of-concept HTTP server! :P

The project is located on GitHub at [polybuildr/balloon-lang](https://github.com/polybuildr/balloon-lang) and is open sourced (licensed under the interesting [MPL 2.0](https://www.mozilla.org/en-US/MPL/2.0/)). I would love to have some help on the project - please do contribute!

My motivation for the project was (and still is) two-fold. One was a desire to learn how to make a real-world, usable programming language. And the second was because there were things that I did not like in all the other programming languages I've worked with and I wanted to see if it was possible to solve those problems.

Last summer, I worked with Microsoft's [TypeScript](https://www.typescriptlang.org/), a "typed superset of JavaScript that compiles to plain JavaScript". The experience was lovely. With type annotations (and some inference), writing Javascript was quite pleasant with IDEs giving me "go to declaration" features as well as rich autocomplete. This new experience was refreshing - it gave me additional motivation to see if those problems I had with existing languages could actually be alleviated in some way.

The general complaint I have about dynamically typed scripting languages is that code written in them seems somewhat (for the lack of a better word) "weak". Sure, sometimes the overhead of type annotations and similar things isn't worth it (behold [Python's duck typing](https://stackoverflow.com/a/4205396/2141058)), but sometimes, you'd prefer to have some kind of checking for your code. On the other hand, statically typed languages tend to be overly verbose with all their type declarations (though inference has gotten pretty darn good), even for code that should ideally be very concise. And both of these _tend_ to have `null`s (or `None`, or even crazy old `undefined`).

The compromise that TypeScript provides seemed great for handling types. Don't explicitly use them when you don't want to - either by preference or because you're prototyping - and use them if you want to - because you want some type safety, or because you're done prototyping and now want production-ready code. And sum types like `Maybe`/`Option` seem great for handling cases where you'd otherwise use `null`.

This is the direction along which I'm trying to build Balloon. Similar to Rust's (which Balloon is written in) "[choosing your guarantees](https://doc.rust-lang.org/book/choosing-your-guarantees.html)", I think it's a great idea to let the programmer choose what level of safety - or any other guarantee - they want in their program and try to provide the best guarantees you can anyway. Along those lines, Balloon does not currently have anything like `null` at all (but it also does not have declaration without initialization yet, nor does it have sum types or `Maybe`/`Option`, there's definitely lots more work to do :P) and it already has a typchecker that type checks programs without being given any type annotations or declarations (similar to what Facebook's [Flow](https://flow.org/) does).

There's definitely quite some way to go before Balloon becomes real-world "usable" and a long way to go before it becomes "useful", but I'm happy with where it's gotten to right now. I'll probably be writing more about the kind of things I want to see in Balloon in the future, but for now, I look forward to someone wanting to help me out with the project given this very brief introduction.
