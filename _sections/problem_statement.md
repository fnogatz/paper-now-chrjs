---
title: "Implementation Goals"
order: 4
---

The aim of CHR.js was to implement a CHR constraint solver in JavaScript. It allows to run Constraint Handling Rules in a web environment as well as in served-based JavaScript runtime environments like [node.js](http://nodejs.org/).

Referring to the implementation goals specified for [K.U.Leuven JCHR](#ref-van2005ku) and [CCHR](#ref-wuille2007cchr), we have set the following requirements for our CHR.js prototype implementation:

1. CHR.js closely resembles other CHR systems

    The syntax used by the CHR(JavaScript) application should feel familiar for CHR- as well as JavaScript-experienced developers. One should be able to use regular JavaScript variables and expressions. On the other hand it should be possible to easily adapt existing CHR source code, written for example for CHR(Prolog) systems, by only changing syntactic elements specific for the host language.

    The implementation should follow the [refined operational semantics](#ref-duck2004operationalsemantic) of CHR. Due to its prototype character, efficiency is currently not a requirement.

1. CHR.js can be used both in browser and server environments

    Transpiling programming languages into JavaScript is not new at all. There are currently [more than 350 tools that compile to JavaScript](https://github.com/jashkenas/coffeescript/wiki/List-of-languages-that-compile-to-JS). To also work in the browser it is necessary to focus on a small file size of the compiler and its generated JavaScript source code. The compressed source code of the transpiler should not exceed several hundred kilobyte.

### The CHR.js Language

The syntax of CHR.js, i.e. JavaScript with embedded CHR, is a good compromise of both worlds. It uses the well-known Prolog CHR syntax to represent the CHR rules. Instead of predicates we use function-like representations for the constraints, that means that a constraint call looks like a function call in JavaScript. While Prolog supports nullary predicates by simply call `pred`, we modellize the call to nullary constraints in CHR.js as function calls with zero arguments: `pred()`; only within CHR rules the shorter notation `pred` is supported.

The CHR rules follow closely the Prolog CHR syntax with the following notable exceptions:

- A rule ends in a semi-colon. Although in JavaScript it is not necessary to end expressions and declarations by a semi-colon, it is common practice and helps avoiding edge-cases in the [parsing process](#parser).
- Variables are always locally scoped for the current particular rule.
- There is no kind of pattern matching (so called `destructuring` in JavaScript) in the rule's head.
- There is currently no support for logical variables, that means only ground terms are allowed.

In this prototype implementation we omit the declaration of constraints. The consequential disadvantages of this simplification of design are discussed later in the [summary](#summary). The most notable consequence is that it is not possible to call user-defined JavaScript functions in the rule's body.

### Example Program

We introduce the CHR.js language by a small example program for the bottom-up computation of the Fibonacci numbers upto a given maximum value, as it has been used to present the syntax of CCHR in [Pieter Wuille et al.; 2007](wuille2007cchr) too:

    begin  @ upto(A) ==> fib(0,1), fib(1,1);
    calc   @ upto(Max), fib(N2,M2) \ fib(N1,M1) <=> 
               N2 === N1+1, N2 < Max | fib(N2+1, M1+M2);

We will assume familiarity with both CHR and JavaScript at this point, and therefore only want to refer to the used built-in JavaScript comparison `===` in the rule `calc`.

More examples can be found at the [interactive online demo](http://chrjs.net/) and in the [CHR.js repository](#source-code-chrjs).
