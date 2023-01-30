# UW CS Teaching Talk

[Auxiliary slides](https://docs.google.com/presentation/d/1h4pEEX1NFbZl6DM_fcFgMuUCkR8ZGLTFOL36wiBuIUU/edit?usp=sharing)

## Preamble

This is a tentative script for Matthew Wang's (~ 50 min) teaching talk for UW CS, minus the introduction.

The lesson has two key goals:

1. learn a teensy bit about JavaScript - particularly, its type system
2. learn how to reason about language behaviour, even without all the information!
  - and, on a related note, how software tools are your friend :)

The end goal is to teach students a different approach to learning new languages and refining their mental models about software tooling.

By the nature of this talk, this script cannot be complete - much of the exploration is done open-ended by students. Instead, this script is a general outline of the topics that I (Matt) will touch on, with the suggestive nudges I give students.

### Baseline Knowledge

JavaScript is…
- C-like in syntax (like Java!)
- designed to be “error resistant”
(and many other things, but we don’t care!)

Base assumption: you’ve taken CS 1 and CS 2 (~ CSE 142/143 at UW), and have used a C-like language! **No JS or PL background required!**

### Mechanics

I teach this lesson with a split-screen. One half has a JavaScript REPL; I use the default one packaged with `node`. The other half is a scratchpad where the class writes down (and updates) their mental model of the language, given our experiments. In this document, I'll place the scratchpad within blockquotes, like so:

> I think JavaScript is strange *and* cool!

By default, Node's REPL shows preview hints, which can "spoil" the answer for students. You can disable it with the following startup script:

```js
#!/usr/bin/env node

const repl = require('repl');

repl.start({
  preview: false
});
```

## Warmup - `typeof`

Let's start by getting used to the *style* of this lecture. Our primary method of learning about the language is going to be trying statements in the REPL.

In other words, this is not really a "lecture"; I'm not going to tell you (many) facts about JavaScript.

My focus with this lesson is going to be about semantics, not syntax. Since I don't assume you know how to use JS, you can safely ask me to do a certain operation in JS, and I'll write it out in the REPL. I'll also occasionally introduce new keywords.

Let's start with the following example:

```js
> typeof 18
'number'
> typeof 18.0
'number'
```

As you've probably guessed, `typeof` tells you the type of a value in JS. `18` and `18.0` are both values.

I argue that something is strange about this behaviour, at least compared to other languages. What might that be?

> JavaScript...
> - seems to not differentiate between integers and floats

Astute students may note a couple of things:

- maybe `18.0` gets converted to `18`? In that case, we should try `typeof 18.1`. We'll still get `number`.
- maybe JavaScript only has floats?
- maybe JavaScript has a rational or decimal type (like ... COBOL?)
- maybe a special keyword is necessary to mark a value as atype

We'll test these hypotheses in the coming sections.

## Precision and Equality

JavaScript supports many of the basic math and boolean operations we'd expect in most languages.

What would we expect this to output?

```js
> 1 + 2 == 3
```

```js
> 1 + 2 == 3
true
```

Whew! This language isn't too strange.

What about

```js
> 0.1 + 0.2 == 0.3
```

(discussion)

```js
> 0.1 + 0.2 == 0.3
false
```

Hm. What could be going on?

Some students will immediately shout out "floating-point"! But, even if you didn't know how floating point really works, a good first step would be to simply inspect `0.1 + 0.2`.

```js
> 0.1 + 0.2
0.30000000000000004
```

Ah. Clearly, that isn't `0.3`. So, this behaviour is at least explainable - especially if you do know something about floating point.

> JavaScript...
> - seems to not differentiate between integers and floats
> - **"decimals" in JavaScript are floats by default**

The above example is pretty standard in many languages. Let's try another one:

```js
> 999999999999999999
1000000000000000000
```

What's going on here?

(discussion)

Great. Let's now update our model

> JavaScript...
> - ~~seems to not differentiate between integers and floats~~
> - **seems to not have integers - all numbers are floats**

Let's take a step back. What types of applications does this favour? And what is this a disadvantage for? Keep in mind - JavaScript was origianlly created for client-side browser code.

## Operators (and something else)

### Act 1

There's some other interesting simple code we can look at with operators.

What would you expect to happen here?

```js
> 3 - '3'
```

(discussion)

```js
> 3 - '3'
0
```

Type coercion. Not in every language, but certainly a known language feature.

Hm. What about...

```js
> 3 + '3'
```

```js
> 3 + '3'
'33'
```

Hold on. Why wasn't that `33`?

(discussion)

> JavaScript...
> - ~~seems to not differentiate between integers and floats~~
> - seems to not have integers - all numbers are floats
> - **coerces types in operations**
>   - **specifically, prioritizes string coercion**

### Act 2

JS has variables.

```js
> x = 3
3
```

So, what should this output?

```js
> '5' + x - x
```

(discussion)

```js
> '5' + x - x
50
```

Is this expected? Why or why not?

```js
> '5' - x + x
```

```js
> '5' - x + x
5
```

Is this expected? What does this tell us about operator commutativity?

> JavaScript...
> - ~~seems to not differentiate between integers and floats~~
> - seems to not have integers - all numbers are floats
> - coerces types in operations
>   - specifically, prioritizes string coercion
>   - **operations, and thus coercion, happens from left-to-right**

```js
> '5' + - '2'
```

```js
> '5' + - '2'
'5-2'
```

```js
> '5' - + '2'
```

```js
> '5' - + '2'
3
```

> JavaScript...
> - ~~seems to not differentiate between integers and floats~~
> - seems to not have integers - all numbers are floats
> - coerces types in operations
>   - specifically, prioritizes string coercion
>   - operations, and thus coercion, happens from left-to-right
> - **there is a unary `-` and `+` operator**
>   - **it seems to have a higher precedence than binary `+/-`**

### Act 3

And now, for the showstopper:

```js
> '5' + - + - - + - - + + - + - + - + - - - '2'
'5-2'
> '5' + - + - - + - - + + - + - + - - - '2'
'52'
```

This seems absurd! How is this valid syntax, and how does removing just one `+-` pair change the answer?

But, I argue we now have the tools to break this down.

Mentally, it's possible to figure this out without writing anymore code. But, why not use our REPL to our advantage? What can we start with to break this down into smaller problems?

(discussion)

Let's take stock.

> JavaScript...
> - seems to not have integers - all numbers are floats
> - coerces types in operations
>   - specifically, prioritizes string coercion
>   - operations, and thus coercion, happens from left-to-right
> - there is a unary `-` operator
>   - it seems to have a higher precedence than binary `+`

There is likely to be some discussion about why our previous examples didn't error; in most mainstream languages, at least one (if not all) of them would error.

Given what we know about how JavaScript was used (and what its purpose is), why would this decision be made? How does this otherwise inform how you'd use it?

## Interlude

### Spongebob

![Spongebob meme - code is annotated below](https://raw.githubusercontent.com/mattxwang/uw-cs-teaching-talk/main/spongebob.png)

```js
> 0 == "0"
true
> 0 == []
true
> "0" == []
false
```

At this point, some of you are probably grumbling. JavaScript *does* have a way to do equality without coercion, namely, **strict equality**.

```js
> 0 == "0"
true
> 0 === "0"
false
```

Some codebases mandate that you only use strict inequality. The dominant linter for the language, ESLint, [has a rule requiring it](https://eslint.org/docs/latest/rules/eqeqeq).

I'll also leave you with another example:

```js
> [] == ![]
true
```

### Made in a Weekend

![headline: JavaScript: Designing a Language in 10 Days](https://raw.githubusercontent.com/mattxwang/uw-cs-teaching-talk/main/js-10-days.png)

## Absurdity and Practicality

I now want to move us closer to language designers and also real-life applications. Let's examine a few different items that *feel wrong*, but likely have a pretty good reason for why they're true.

### Warmup: `NaN`

Many languages, including JavaScript, have a value called "not a number", or NaN.

```js
> 1919 / UW
NaN
> Number("UW")
NaN
```

In JS, what's its type?

```js
> typeof NaN
'number'
```

Hm. Why?

(discussion)

As an aside - this is a really common bug in code. Don't check if a number is `NaN` with `typeof` - instead. use `isNaN()`.

```js
> isNaN(1919)
false
> isNaN(1919 / "UW")
true
```

Another open item:

```js
> NaN === NaN
false
```

### Function Flyover

Inadvertently, we just saw the first use of functions in JS! I feel obligated to give a quick overview.

```js
> typeof isNaN
'function'
```

(optional: is this different from other functional languages? what does this tell us about JavaScript?)

We can, of course, define our own functions.

```js
> sub = function(a,b) { return a - b }
[Function: sub]
> sub(1,2)
-1
```

(optionally, there's a similar syntax -- arrow functions)

```js
> sub = (a,b) => a - b
[Function: sub]
> sub(2,1)
1
```

What does this tell us?

```js
> sub('3','3')
0
```

Or, this?

```js
> doubleCall = (x, f) => f(f(x,x), x)
[Function: doubleCall]
> doubleCall(10, sub)
-10
```

I'd devote a whole lecture to functions; but, given that we're likely wrapping up the lecture at this point, I'll take a pause.

### which `this`? this `this`?

First, JavaScript has first-class support for objects; its notation is now one of the de facto standards for serialization supporting the web (JSON). They look like objects in many other languages:

```js
> person = { name: 'matt wang', affiliation: 'ucla' }
{ name: 'matt wang', affiliation: 'ucla' }
> person.affiliation
'ucla'
> person.affiliation = 'uw'
'uw'
> person
{ name: 'matt wang', affiliation: 'uw' }
```

Objects in JS occupy a similar space as classes do in Java. They are the primary implementation method of object-oriented programming in JS.

One commonly-confusing area of OOP is the `this` keyword. JS is no exception. Let's look at an example that can help clarify *how* JS's OOP features are implemented.

```js
> person.getBio = function() { return this.name + ' is at ' + this.affiliation }
[Function (anonymous)]
> person.getBio()
'matt is at uw'
```

The `this` is evaluated at runtime.

```js
> person.affiliation = 'uw'
'uw'
> person.getBio()
'matt is at uw'
```

`this` has some dynamic semantics:

```js
> mattsFave = { name: 'amy ko', affiliation: 'uw' }
{ name: 'amy ko', affiliation: 'uw' }
> keeperOfTime.getBio = person.getBio
[Function (anonymous)]
> keeperOfTime.getBio()
'amy ko is at uw'
```

```js
> biology = person.getBio
[Function (anonymous)]
> biology()
'undefined is at undefined'
```

So, let's discuss. What's going on here?

### ternaries?

JavaScript supports a common language feature called the "ternary operator". It is often understood as shorthand for an if statement.

```js
> x = 3
3
> x > 0 ? "Positive" : "Not Positive"
'Positive'
```

```js
> x = -3
-3
> x > 0 ? "Positive" : "Not Positive"
'Not Positive'
```

Many production JavaScript codebases have an absurd amount of ternary operators. Here's one, taken from the codebase for react-router (one of the most popular routing libraries for React, a UI framework)[^1]:

```jsx
<RouterContext.Provider value={props}>
  {children && !isEmptyChildren(children)
    ? children
    : props.match
      ? component
        ? React.createElement(component, props)
        : render
          ? render(props)
          : null
      : null}
</RouterContext.Provider>
```

Why would this be necessary? Why couldn't we just use if statements here?

## True, True Absurdity

I'll leave with this:

```js
> (!+[]+[]+![]).length
9
```

```js
> (![] + [])[+[]] + (![] + [])[+!+[]] + ([![]] + [][[]])[+!+[] + [+[]]] + (![] + [])[!+[] + !+[]]
'fail'
```


## Appendix

There are some other things that I don't think would fit in a standard ~ 50-minute teaching lecture, but are interesting rabbit holes if you found this material interesting.

### Null and ununequality

```js
> null > 0
false
> null < 0
false
> null == 0
false
> null >= 0
true
```

What does this tell you about `null`? What about how `>=` is implemented?

### Strange Language Additions

```js
> true+true+true
3
```

```js
> [] + []
''
> [] + {}
'[object Object]'
> {} + []
'[object Object]'
```

```js
> 1 + [2]
'12'
> 1 + [2,3]
'12,3'
```

```js
> [1, 2] + [3,4]
'1,23,4'
```

### Plus Plus is a Minus

```js
> '5' + + '5'
'55'
> 'foo' + + 'bar'
'fooNaN'
```

## Attribution

*This is very similar to the [teaching talk I gave at UCLA](https://github.com/mattxwang/ucla-cs-teaching-talk)*.

I've written the entire content of this talk. It's an amalgamation of various teaching talks I've given about JavaScript over the past 6 years, starting from a web development class I taught in high school, to two years of running ACM Teach LA's dev team, to most recently a talk I gave in CS 495 at UCLA. The last talk also inspired the *format* for this talk, which leans entirely into a REPL and student-driven discussion; previous iterations were more didactic.

I'm not explicitly releasing this talk with a license, but you're generally free to use it how you'd like. I would prefer if you let me know - mostly to give me feedback!

The spongebob meme is hard to attribute; [Know Your Meme](https://knowyourmeme.com/memes/patrick-stars-wallet) has an interesting discussion on the overall meme's history; I got this version of the image from [this Reddit post](https://www.reddit.com/r/ProgrammerHumor/comments/cvjx7x/oh_javascript/), but it's certainly not the original source.

The screenshot of "JavaScript: Designing a Language in 10 Days" is from the [IEEE Digital Library](https://www.computer.org/csdl/magazine/co/2012/02/mco2012020007/13rRUy08MzA).

---

[^1]: this code example is from https://github.com/prettier/prettier/issues/5814, one of the more controversial features in prettier, a large open-source pretty-printer.
