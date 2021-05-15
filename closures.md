<img src="https://i.imgur.com/qI5itXg.jpg">

# Walk-thru of JavaScript Closures

## Roadmap

- Intro
- Review of Scope
- Inner Functions
- BAM! We have Closure
- Definition of a Closure
- Examples of Closures 
- Closing

## Intro

Closures are one of my three **JavaScript Ninja** topics:

1. `prototypal inheritance` and the 	`prototype chain`: **How JS implements object inheritance and property lookup**
2. `this`: **A keyword in JS that represents a function's "context" during its execution** <br>

> Sam's note for #2: basic functions look to the left of it to find what "this" is. Arrow functions change the context of the scope to look UP.

3. `closures`: **A feature in the JS programming language that enables a function's free variables to exist even after its enclosing function has completed execution**  

> Sam's note for #3: After you run a function, all of the variables inside it are still available to any other function that is returned by the parent.

This discussion on closures is an important one because they are a key feature of languages with first-class functions such as JavaScript and Python.

More importantly, you will likely be asked about them in technical interviews.

To start, realize that closures are **not** explicitly created; code like this<br>`var c = new Closure();`<br>does not exist.

Basically, closures come into and out of existence when JavaScript programs execute.

Let's see how...

## Review of Scope

Understanding scope is fundamental to the concept of closures.

❓ **What is scope?**

> Sam's answer: what you have access to at any particular point of time

❓ **How is a new scope created in JavaScript?**

> Sam's answer: whenever you create a variable

❓ **What happens when a function's code accesses a variable or function that is not in its local scope?**

> Sam's answer: it will go up the scope chain (unless it's a function expression, where the variable needs to be defined before the function)

❓ **When is a scope destroyed and marked for garbage collection?**<br>**_unless..._**

> Sam's answer: after the execution of a function for function scopes. For global scopes, reload the page. Garbage collection is the process in which that happens (looks to see what's not in use anymore and gets rid of it).

## What's a "free" Variable?

The variables accessible by a function that are not defined locally are called _free variables_.  They are called free variables because the function gets them for "free" 🙂

## "Persisting" Scope Using Inner Functions

Let's first look at a very basic function:

```js
function f() {
  var x = 5;
  console.log(x);
}
```

❓ **When we invoke this function, what will the lifetime of the variable `x` be?**

> Sam's answer: sets x to 5, logs it out, marks it for garbage collection.

Now let's refactor this code by including an inner (nested) function inside of function `f()`:

```js
function f() {
  var x = 5;
  
  function inner() {
    console.log(x);
  }
  
  inner();
}
```

❓ **Have we changed the functionality of `f()` by adding the nested function?**

> Sam's answer: no

❓ **What scope is the variable `x` in?**

> Sam's answer: local scope in the f function, and it's being used in the inner function

❓ **What are the free variables of the `inner()` function above?**

> Sam's answer: `x` because it's outside the inner() function.

Now let's see how we can invoke the function `inner()` from outside of function `f()`:

```js
function f() {
  var x = 5;
  
  function inner() {
    console.log(x);
  }
  
  return inner;
}

var fun = f();  // fun now references the "inner()" function
fun();
```

> Sam's note: you can also do `f()()` to run the inner function!

❓ **Since a function's scope is destroyed after a function has finished executing, should the scope of `f()` exist by the time we invoke `fun()`?**

> Sam's answer: no but the reference will be maintained in the inner function.

## BAM! We have Closure

Why does the `console.log(x)` work as expected if the scope that variable `x` lives in has been destroyed?

Because a **closure** has been created by the fact that the nested function, `inner()`, references a free variable `x`, and `inner()` is still in existence thanks to the external reference to it by the variable `fun`. 

❓ **If the variable `fun` went out of scope, or was reassigned so that it did not reference function `inner()` anymore, what do you think would happen to the closure?**

> Sam's note: it would still exist (??) sorry I wasn't listening this bit

## Definition of a Closure

So, when you are asked, "What the heck is a closure?"

> Sam's answer: Information in memory that we have access to at a particular point in time.

The description in the intro above is highly accurate, however, here's a viable answer that's not too technical:

> **A closure is a way for JavaScript to "persist" scope.**

A follow-up question might be, "When is a closure created?"

> Slightly more technical answer:<br>**When a variable holds a reference to a function and that function references one or more free variable(s).**

Again...

**Closures come into existence when a function continues to reference its free variable(s) after its enclosing function has finished execution.**

Closures have their use cases and are being created and destroyed as JavaScript executes.

## Examples of Closures

Closures typically are more important to the creators of libraries and frameworks, however, let's take a look at a few examples of how we might create them.

### Example 1 (factory pattern #1)

```js
function adderFactory(fixedOperand) {
  
  return function(secondOperand) {
    return fixedOperand + secondOperand;
  };

}

var add5 = adderFactory(5);
add5(2) // returns 7

var addTwenty = adderFactory(20);
addTwenty(100) // returns 120
```

Note here that `adderFactory()` returns an anonymous function expression, there's no need that the inner function be a function definition.

### Example 2 (factory pattern #2)

```js
function elapsedTimeFactory() {
  var startTime = Date.now();

  return function() {
    return Date.now() - startTime;
  };
}
```

> Sam's note: the difference will be miniscule (a couple milliseconds), unless you add a setTimeout like:

```js
setTimeout(() => {console.log(test())}, 1000)
```

❓ **How many of these timers could we create?**

> Sam's answer: as many as we want

❓ **What is the inner function's free variable(s)?**

> Sam's note: startTime

### Example 3 (module pattern)

As you know, an object holds information in properties. There are no variables allowed within an object.

Also, as you know, there is no easy way to "hide", or as developers say, make "private", an object's properties.

Now let's see how closures can be used to implement "private" variables in an object or "module":

```js
// The IIFE (Immediately Invokable Function Expression) will run and assign the object being returned
var fourLetterWord = (function() {
  var _fourLetterWord = 'love';
  
  return {
    // property getter
    get word() {
      return _fourLetterWord;
    },
    // property setter
    set word(newWord) {
      if (newWord.length === 4) _fourLetterWord = newWord;
    },
    version: 1.0
  };
})();

fourLetterWord.version
// 1
fourLetterWord.word
// love
fourLetterWord.word = 'six';
fourLetterWord.word
// love
fourLetterWord.word = 'four';
fourLetterWord.word
// four
```
> Sam's note: Objects, if statements and functions all share BLOCK scope. The other kind is LEXICAL scope.

> Sam's note: the pro is that any code you create inside this module's library will never interfere with any other applications.

Note that the above code is returning an object that has ES5's under utilized [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) and [setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set) properties.

Note that there's no way to access the "private" variable `_fourLetterWord `.

## Summary

Closures are not complicated, however, they are a mystery to many programmers and even those familiar with them have a hard time explaining them (myself included).

Be sure to refresh your memory of their definition and use before a JS-heavy tech interview.