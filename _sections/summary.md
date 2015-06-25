---
title: Summary
order: 7
---

In this project we created a prototype implementation of CHR in JavaScript. The module, called [*CHR.js*](#source-code-chrjs), allows the transpilation of JavaScript with embedded CHR into standard JavaScript. With [chrjs.net](http://chrjs.net) a first version of a CHR playground, similar to online tools like [JSFiddle](https://jsfiddle.net/) and [RequireBin](http://requirebin.com/), has been created. This is a first step to run Constraint Handling Rules directly in the browser, without having to install a host language like Prolog oder Java first.

Because of its prototype character the current implementation has several missing features. For CHR developers experienced with Prolog as CHR's host language, the absence of logical variables and pattern matching in the rule's head constraints might be most notable. Because of the missing constraint declarations it is currently near to impossible to interact with built-in or user-defined functions from CHR rules.

With reference to the research done by [Peter Van Weert](#ref-vanweertphd) in 2010, there is a wide range of optimization techniques that should be implemented, too. It also mentions extensions of CHR that can be implemented in imperative application of CHR.

The further development of the CHR.js transpiler as well as its related tools like the CHR.js website will be part of the author's Master Thesis. The following goals can be stated as of yet as conclusions of the prototype implementation:

- Reject the usage of [PEG.js](http://pegjs.org/) in favor of [Babel](http://babeljs.io/).

    This allows to concentrate on the compilation part of the [current process](#architecture). Although Babel is a very new JavaScript transpiler, it is used by a [large number of companies](http://babeljs.io/users/), among others Apple and Facebook. The latter one established [JSX](https://facebook.github.io/jsx/), a syntax extension for JavaScript which gets transpiled to standard JavaScript with the help of Babel.

- Use the new [ECMAScript 2015](http://www.ecma-international.org/publications/standards/Ecma-262.htm) JavaScript standard, which has been approved in June 2015. Some new language features might be also a starting point for optimizations in performance or resulting file size, most notable:

    - Destructuring, for easier pattern matching in constraint arguments
    - Tail Calls, to increase the performance
    - Template Strings, to embed CHR natively in JavaScript without having to use new keywords
    - Generators, for the result set of constraint store lookups
    - Promises, which optimizes the current constraint chaining mechanism like `CHR.upto(4).upto(10)`
    - Proxies, which would allow the call of nullary constraints in the form of `pred` (cf. [The CHR.js Language](#the-chrjs-language))
    - Module Systems, to combine multiple CHR modules

    Although these new features aren't available in all JavaScript environments as of yet, Babel provides polyfills for most of them.

- Provide a mechanism to interact with HTML elements in the DOM. CHR seems to be a very suitable and declarative approach to create new HTML elements depending on existing ones.

- Implement the optimization techniques mentioned in [Peter Van Weert; 2010](#ref-vanweertphd). Today's JavaScript runtime engines are highly optimized. That means a CHR(JavaScript) application could benefit by the high performance of runtime engines like Google's V8, which is the basis for [node.js](http://nodejs.org).
