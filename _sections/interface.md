---
title: Interface
order: 6
---

### Code Generation

The CHR.js transpiler can be used in different ways. Either as a command line tool or programmatically as [node.js](http://nodejs.org/) module or browser script.

The easiest way to use the CHR.js transpiler is the executable `chrjs` provided by its [npm package](https://www.npmjs.org/package/chr):

    $ npm install -g chr
    $ chrjs /path/to/fib.chr > transpiled.js

Additional options can be found in the [CHR.js repository](#source-code-chrjs). There is also an example how to use the transpiler programmatically in node.js or the browser.

### CHR.js Usage

Once the CHR code is transpiled as mentioned before, we can bind the generated code to a variable `CHR` and access the constraints by calling for example `CHR.upto(4)`. In the following we demonstrate the usage of the generated JavaScript constraint solver with the help of the [Fibonacci example](#example-program). For a more verbose output we use the supplied `chr/console`-module, which is based on [tconsole](#source-code-tconsole).

    $ node
    > var CHR = require('./transpiled.js');  // load the generated source
    > var konsole = require('chr/console');  // load the graphical console
    > konsole(CHR.Store);                    // print the current store
    ┌────────────┐
    │ Constraint │
    ├────────────┤
    │ (empty)    │
    └────────────┘
    > CHR.upto(4);                           // generate fibs upto 4
    > konsole(CHR.Store);                    // print updated store
    ┌────────────┐
    │ Constraint │
    ├────────────┤
    │ upto(4)    │
    ├────────────┤
    │ fib(3,3)   │
    ├────────────┤
    │ fib(4,5)   │
    └────────────┘

*Cf. [Screenshot 4](#screenshot-4)*

The constraint calls are chainable, that means `CHR.upto(4).upto(10)` would be possible too. Because the constraint store instance `CHR.Store` is an EventEmitter, it is very easy to implement special loggers, activities etc.:

    CHR.Store.on('add', function(c) {
      console.log('Constraint '+c.toString()+' has been added.')
    })

    CHR.Store.on('remove', function(c) {
      console.log('Constraint '+c.toString()+' has been removed.')
    })
