+++
date = "2017-06-11T18:20:35+03:00"
draft = false
tags = ["dev", "balloon"]
title = "Hoisting and Locality"

+++

Here's a puzzle. What do you think this Javascript code does?

```js
f();
var f = 5;
function f() {
    console.log("Hello!");
}
f();
```
<!--more-->
The first call to `f` succeeds and prints "Hello", but the second one fails with `TypeError: f is not a function`. This is because of function hoisting. In Javascript, all [functions are "hoisted"](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/function#Function_declaration_hoisting) to the top of the scoped section (top-level/function).

This is in addition to [`var` hoisting](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/var#var_hoisting) in which the "declaration" of the variable is hoisted (but not the initialization). This means that in the following code, `x` is a global variable:

```js
function f() {
    x = 5;
}
f();
console.log(x); // prints 5
```

but in the following code, it isn't:

```js
function f() {
    x = 5;
    var x;

    /* equivalent to:
    var x;
    x = 5;
    */
}
f();
console.log(x); // ReferenceError: x is not defined
```

As programmers, we're used to reading things top to bottom. Programs execute sequentially, and that's how we try to follow the execution too. However, we don't generally read the whole file, because we have some expectation of how certain code will affect code around it and so when we read this snippet:

```js
// some code
function f() {
    console.log("Hello!");
}
f();
// more code
```

it's very difficult to imagine that it won't work (as in the first example).

This question has the "balloon" tag because I ended up thinking about this while wondering whether to [implement function hoisting in Balloon](https://github.com/polybuildr/balloon-lang/issues/27). Since Balloon does not currently have any kind of module system (or anything at all that allows you to split scripts across multiple files), hoisting definitely has some use, such as in:

```
setup();
request_credentials();
make_request();
cleanup();

fn setup() { ... }
// and other long declarations follow
```

where it allows you to declare intent initially, and then give details of how you want to achieve it.

However, Balloon will definitely get some kind of import system in the future (and you can't request credentials or make requests as of now anyway), and the appropriate solution can be deferred to then. Meanwhile, the loss of locality in code is harmful enough that I would prefer not to include this in the language.
