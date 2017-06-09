+++
date = "2017-06-09T18:23:26+03:00"
draft = false
tags = ["dev", "balloon-lang"]
title = "Longer Thoughts About Balloon"

+++

I recently wrote a [post about Balloon]({{< ref "post/introducing-balloon.md" >}}), a new programming language I've been working on. In this post, I'll try to put down some more of my thoughts on what I want to see Balloon become.

<!--more-->
Here's a table of contents if you're only interested in particular sections:

- [The Current State](#current-state)
    - [The Language](#language)
    - [The Typechecker](#typechecker)
    - [The Super-experimental LLVM Backend](#llvm-backend)
    - [The Binary](#binary)
- [The Future](#future)
    - [No Nulls](#no-nulls)
    - [Gradual Typing](#gradual-typing)
    - [Partial Compilation (or even JIT)](#partial-compilation)
    - [A Neat Standard Library](#neat-stdlib)
    - [Maybes](#maybes)
        - [Event loop](#event-loop)
        - [Multi-threaded (or even forking) web server](#multithreaded-server)

Let's look at what the language is currently capable of.

## The Current State {#current-state}

### The Language {#language}

Balloon has values of different types. Numbers, booleans, and strings along with basic operators on them.

```
var x = 5 + 5.5;
var y = true or false;
var z = "Hello, " + "world!";
```

(Unfortunately, there's nothing that can syntax highlight Balloon properly. There's an [issue filed for that](https://github.com/polybuildr/balloon-lang/issues/40) labelled "help wanted". :P)

It also has Python-like tuples.

```
var a = (1, true, "brace style");
```

And it has first-class functions (and they're closures).

```
var x = 5;

fn get_global_x() {
    return x;
}

fn curry_add(x) {
    return fn(y) {
        return x + y;
    };
}

curry_add(5)(10);
```

The language should already be Turing-complete, because [λ-calculus](https://en.wikipedia.org/wiki/Lambda_calculus), but it also has if/else conditionals, and a basic looping construct.

```
if false {
    // something
} else if false {
    // something
} else {
    // something
}

// an infinite loop that can be broken or continued
loop {
    if foo {
        break;
    }
    if bar {
        continue;
    }
}
```

### The Typechecker {#typechecker}

Balloon comes with a basic typechecker that also infers types (given how simple the language is, this is mostly just propagation of types and not modelled as constraints).

Running the typechecker on the following code from the [flow-test.bl test file](https://github.com/polybuildr/balloon-lang/blob/74fd840/tests/typecheck-fail/flow-test.bl)

```
fn square(n) {
  return n * n;
}

square("2");
```

gives

```
in flow-test.bl, line 5, col 1:
issue in function call:
5 | square("2");
    ^^^^^^^^^^^
in function in flow-test.bl, line 2, col 10:
type error: `*` cannot operate on types String and String
2 |   return n * n;
             ^^^^^

1 issue detected in flow-test.bl.
```

There are a lot of things that the typechecker does, but also lots that it doesn't yet. It currently has only two levels of types: a concrete type like `String` or `Bool`, or the I-give-up type `Any`.

In code like this, it will correctly "infer" the type (here, `Bool`):

```
var x = true;
if foo {
    x = false;
} else {
    x = true;
}
```

but in this case, it will fall back to `Any` because there is currently no concept of a union type:

```
var x = 5;
if true {
    x = true;
} else {
    x = 10;
}
```

but it will still complain with the following warning:

```
multiple types from branch: `x` gets different types in branches
  |
2 | if true {
3 |     x = true;
4 | } else {
5 |     x = 10;
6 | }
  |
```

I've also [been told](https://github.com/polybuildr/balloon-lang/issues/25#issuecomment-300622467) (by Siddharth, another IIIT student who has helped me with parts of the project) that I should use a tried-and-tested type checking algorithm instead of rolling my own and there's an [issue for that](https://github.com/polybuildr/balloon-lang/issues/26) too.


### The Super-experimental LLVM Backend {#llvm-backend}

Siddharth ([@bollu](https://github.com/bollu)) has been [working on an LLVM backend for Balloon](https://github.com/polybuildr/balloon-lang/pull/20). While he was initially trying to build a JIT, my development is proceeding too slowly to be able to properly integrate in a JIT, and so we've briefly discussed simply compiling certain parts of balloon code instead of actual JITing.

He would also love to have some help with the LLVM backend, so do ping him if you want to work on that stuff.

##### Siddharth writing this:

The current idea is to use the ["gradual lowering" ideas described in the talk by Azul](https://www.youtube.com/watch?v=sKIRIilZDnE) at EuroLLVM 2017. The `TL;DR` is that for each "virtual" instruction in the laguage bytecode, you create an LLVM function which has the same semantics. Then, you express your language optimisations in terms of LLVM passes. That way, the domain-specific language passes can help LLVM passes line inlining and dead code elimination. Similarly, LLVM passes could help language specific pass find optimisation opportunities.

A high level example is something like this:

```py
class Car:
   ...
   def run(self):
       print "car running"
       
class Spaceship:
   ...
   def run(self):
       print "spaceship running"
       
x = None

if 1 == 0:
   x = Car()
else:
   x = Spaceship()
   
x.run()
```

A language pass before dead code elimination cannot actually figure out much about the `x.run()`, which would compile to some sort of lookup table thing. But, if dead code elimination runs, the code transforms to this:

```py
class Car:
   ...
   def run(self):
       print "car running"
       
class Spaceship:
   ...
   def run(self):
       print "spaceship running"
       
x = None
x = Spaceship()
x.run()
```

Now, the language pass can kick in and notice that the function call doesn't depend on `x` at all, and so it transforms it into some sort of global call:

```py
class Car:
   ...
   def run(self):
       print "car running"
       
class Spaceship:
   ...
   def run(self):
       print "spaceship running"
       
x = None
x = Spaceship()
# x.run() -> spaceship_run
spaceship_run()
```

At this point, the `LLVM` inlinier can decide to inline the `print` to eliminate the overhead of a function call:

```py
class Car:
   ...
   def run(self):
       print "car running"
       
class Spaceship:
   ...
   def run(self):
       print "spaceship running"
       
x = None
x = Spaceship()
# spaceship_run() is inlined
print "spaceship running"
```

Clearly, this is a simplistic and exaggerated example, but the idea is solid. I wish to experiment with this in `balloon`, and I believe it can lead to interesting optimisation paths. This is even more so with a JIT, so we can use run-time information such as type information to specialise function calls, for instance.

As an aside, a shameless plug to my own compilers/language project: [simplexhc](http://github.com/bollu/simplexhc) is a custom backend for Haskell that uses similar ideas to generate LLVM code. Haskell compiles to an abstract machine called `STG`, which stands for the *Spineless Tagless G-machine* (I know, sounds badass). `simplexhc` aims to generate performant `LLVM` from the `STG` description, and maybe try to use ideas from [polyhedral compilation](http://polyhedral.info/) along the way. Anyway, that's all the salesmanship I'll do here. Interested readers are advised to consult the repo.


### The Binary {#binary}

Given a built binary (either using Cargo or by downloading a release from GitHub), a file can be run by passing it as an argument.

```sh
$ balloon file.bl
```

The basic REPL can be run by simply running the binary without any arguments.

```sh
$ balloon
```

The typechecker can be run on a file by passing the `--check` flag as follows:

```sh
$ balloon --check file.bl
```

## The Future {#future}

So now we have a good picture of how the language looks right now. Here are some of the things I wish to see in the future, in no particular order.

### No Nulls {#no-nulls}

Nulls are bad.

Java:

```
Exception in thread "main" java.lang.NullPointerException
```

C:
```
[1]    4561 segmentation fault (core dumped)  ./a.out
```

Javascript:

```
TypeError: Cannot read property 'foo' of null
```

and the similar, but more famous (and much more annoying):

```
Uncaught TypeError: undefined is not a function
```

and all the other examples in all the other languages.

The primary issue with nulls, at least in "strongly typed" languages is that they are effectively a violation of contract. If I have a variable of type `A` but it can be set to null, then effectively, it could not have the type `A`. In dynamically typed languages, this is somewhat less of an issue, since a particular variable can already have multiple types and null is just one more. However, Balloon will probably try to add gradual typing to the language and therefore, avoiding nulls is definitely a goal.

Instead, Balloon will try to add support for some kind of sum type like `Option<T>`. For example, Rust has an option type and uses pattern matching to explicitly handle the cases as shown below:

```rust
enum Option<T> {
    None,
    Some(T)
}

let x = None;
// or
let x = Some(5);

match x {
    None => { /* something */ }
    Some(val) => { /* something using val */ }
}
```

I think this is a rather elegant way of doing things and will attempt to add something _similar_ to Balloon.

### Gradual/Optional Typing {#gradual-typing}

What I mean when I say gradual typing (because the term is rather loaded) is that if you write a Balloon program with no type annotations, everything will run fine, like any other dynamically typed language. However, if you do add type hints/annotations/declarations then Balloon's typechecker will try as much as it can to identify errors before runtime.

This is useful for multiple reasons. For one, the code becomes significantly more self-documenting. For another, IDEs and other tools will be able to provide better help to programmers if libraries use type annotations. And finally, while I think full type inference would be possible, the feeble typechecker I wrote could certainly do with some help from the programmer. :P

This means that code could eventually look like this:

```
fn square(n: Num): Num {
  return n * n;
}

square("2");
```

As I mentioned in my previous post about the topic, this is part of an attempt to let the programer "choose their own guarantees", as is the next section.

### Partial compilation (or even JIT) {#partial-compilation}

I'd like balloon to be reasonably fast, but not necessarily super-fast, at least as an early goal. However, one thing I'd like to see is compiling certain parts of the code, such as specific functions. Opting in to this feature could maybe require full type annotations in that function. Type inference could suffice in many cases, though I'm not sure how safe that would be. And even if there were no type annotations, it would be possible to generate machine code that would then have to use tagged unions to represent values.

However, one particularly complex part of this would be interop between compiled code and the interpreter. There are difficult questions to answer, such as "How should Balloon functions along with the closed environment be represented in the machine code?"

### A Neat Standard Library {#neat-stdlib}

A useful general purpose scripting language would need a comprehensive, "neatly" designed standard library. Starting from functions/methods on inbuilt types, to file I/O and webservers, there's a lot to think about, design, and implement. The availability of the Rust ecosystem is particularly useful, though, because one can provide Balloon APIs to well designed and well tested crates from [crates.io](https://crates.io).

### Maybes {#maybes}

#### Event-loop {#event-loop}

Asynchronous code is hard to get right. Data races are a pain. But I quite like Javascript's event-loop model. External operations can still be performed asynchronously, but the interpreter itself is single threaded. Node.js has [libuv](https://github.com/libuv/libuv) for this and while I could call into that from Rust, Rust has [mio](https://github.com/carllerche/mio) which has an event-loop too, so I could use that instead. I also want to see if this event-loop feature can be made opt-in somehow.

#### Multi-threaded (or even forking) web server {#multithreaded-server}

There are a lot of problems with PHP, but there's at least one thing I appreciate about it. Allowing each request to be handled by a web server (Apache/Nginx) that then runs some PHP code independently (multithreaded, forking, whatever) for each request is a neat abstraction. It ensures a great deal of scalability and also removes internal shared state (you could still use something external, like a database or Redis).

---

There are a **lot** of things I want to see in Balloon eventually, some of which are mentioned here. :P I don't know how much will actually happen, but I hope these ideas excite some people who end up helping out with the project. Once again, the project is at [github.com/polybuildr/balloon-lang](https://github.com/polybuildr/balloon-lang). Please do contribute!
