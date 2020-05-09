---
title: "Circular Dependencies in Software (and Lockdowns)"
date: 2020-05-08T15:33:43+01:00
---

A few weeks ago, I was watching some informative system design talks at work. One of the topics discussed was circular dependencies in distributed systems. In particular, I learnt something I found fascinating: if your system has a hard circular dependency, _you can never reboot it!_
<!--more-->
Okay, that's a bit of an overgeneralisation. The system needs to have a specific kind of hard circular dependency. But before we get into that, let's discuss what a circular dependency is.

> "In software engineering, a circular dependency is a relation between two or more modules which either directly or indirectly depend on each other to function properly." -- [Wikipedia](https://en.wikipedia.org/wiki/Circular_dependency)

Okay, sure. So let's say I have two components, A and B. If component A depends on component B and component B depends on component A, that's a circular dependency. Looks something like this:

<div style="text-align:center">

![simple circular dependency](/img/circular-dependencies/simple.png)
</div>

But it doesn't have to be that straightforward, as the Wikipedia definition suggests. You can also have an _indirect_ circular dependency. Service A could depend on service B, service B could depend on service C and service C could depend on service A. That's a circular dependency too.  

<div style="text-align:center">

![indirect circular dependency](/img/circular-dependencies/indirect.png)
</div>

And this graph can, of course, get much larger and more complex. For example, if you have hundreds or thousands of microservices, it could get very hard to know whether you have any circular dependencies.

Alright, cool. We know what a circular dependency is and what it looks like. How does this stop us from rebooting the system?

Well, let's say we didn't have a circular dependency. Let's say we just had a service A that depended on service B.

<div style="text-align:center">

![no circular dependency](/img/circular-dependencies/one-direction.png)
</div>

If you turned this system off, what would be the first step when turning it back on? You don't want to start with service A because service A depends on service B and something could go wrong if service B wasn't running. So you want to start with service B first and _then_ turn on service A. You probably see where I'm going with this.

If you have a circular dependency where A depends on B and B depends on A, then you can't turn on either service first! If you turned on A, something might go wrong because B isn't running and you can't turn on B because A isn't running.

Now, as I said in the beginning, this is a bit of a overgeneralisation. It needs to be a hard circular dependency that actually gets in the way of startup. To make up a more believable example, let's say your services do health checks on their dependencies and fail to start up if their dependencies are not healthy. That would be a realistic scenario where you would have this problem. Or, for a vague example, you might have a service managing quotas and a distributed key-value store service that depended on each other to start up in some way.

This was super interesting for me to learn for several reasons. For one thing, rebooting a system is something so fundamental to my daily life as a software developer. "Have you tried turning it off and on again?" is the usual suggestion when something isn't working. Applies to laptops, servers, IDEs, browsers, smartphones, pretty much anything! And to realise that it's possible to build a system that can run in production _but can't be rebooted_ was eye-opening. You have to make sure that you can recover from the system's partial failures because if there's a total failure, you can't turn it off and and on again! For another, this felt like something that seems "obvious" with hindsight -- of course you can't turn on a system if it has parts that need each other to already be running -- but it's not something that had ever occurred to me before.

But also, this topic popped up in my head again recently because I've been coming across stories about cities reopening after their pandemic-induced lockdowns. I realised that in some ways the same principle applies to cities, or any other complex real-world system. If you turn off a city and then need to turn it back on, one of the many problems you're going to have is circular dependencies.

Here's another contrived but believable example. Some cities have turned off their public transportation systems. Presumably, these systems need people in control rooms to keep the system running. The control rooms are likely to be in central locations in the city. How do operators generally get to their control rooms every day? Maybe... public transport? So when the city starts opening up after a lockdown, how do you get the operators to their control rooms to start the public transport system without the public transport system? (Someone drives them there in a car, of course. But still!)

Side-note: On May 6th, apparently for the first time in history, NYC deliberately shut down its entire subway system and there was this <a target="_blank" href="https://www.reddit.com/r/news/comments/gej9kq/for_the_first_time_in_its_history_new_york_city/fpo1wr5/">very interesting Reddit comment</a> about how it's actually quite complicated to shut down a subway system that was designed for 24/7 use only.

And I'm sure there are other cases. For example, how do bus drivers and train drivers get to their buses and trains? I also think I read a tweet recently which described how shop workers in some city were having trouble going back to work because the street food they normally ate during work hours was no longer available. (But I can't find the tweet anymore.)

It's interesting to think about how people are having to come up with innovative solutions to new challenges due to this unique situation and how existing challenges in other disciplines map to these ones. I'm not very familiar with the topics I've written about here and would love to learn more if you are -- in the comments below or [on Twitter](https://twitter.com/vghaisas/status/1258816214434021376)!