---
title: Architecture
order: 5
---

In this section we introduce the components which form the [CHR.js module](#source-code-chrjs). The architecture of the [CHR.js-website](#source-code-chrjs-website) repository, that provides an online compiler and in-browser CHR playground, is out of the scope of this project report. 

The [CHR.js module](#source-code-chrjs), that is used to transpile and execute JavaScript with embedded CHR rules, consists of the following components:

- The [*Parser*](#parser), that performs a lexical and syntactical analysis of the source code to return a semantic representation.
- The [*Compiler*](#compiler), that takes this representation and generates the JavaScript code.
- The [*Runtime*](#runtime), which provides data structures needed for the execution of the compiled source code, in particular handling the constraint store and propagation history.

With the help of these three components we created a module that can be used in [node.js](http://nodejs.org/) or as command line tool. To support the usage as a browser script we assemble thes components into a single JavaScript file that can be used in existing websites.

In the following subsections we elaborate the separate components in detail.

### Parser

Given a JavaScript source code with embedded CHR rules like in the [previous example](#example-program), we have to parse its components first. Our parser is based on [PEG.js](#dependency-pegjs), a parser generator for JavaScript that is based on the parsing expression grammar (PEG) formalism. We extended the existing [PEG for the JavaScript standard](https://github.com/pegjs/pegjs/blob/master/examples/javascript.pegjs) by grammars for CHR. Without getting into detail of the PEG syntax, we present an extract of the added rules:

    CHRRule
      = PropagationRule
      / SimplificationRule
      / SimpagationRule

    PropagationRule
      = RuleName? CHRConstraints "==>" Guard? Body

    SimplificationRule
      = RuleName? CHRConstraints "<=>" Guard? Body

    SimpagationRule
      = RuleName? CHRConstraints "\" CHRConstraints "<=>" Guard? Body

    RuleName
      = Identifier "@"

    Guard
      = BuiltInConstraints "|"

    BuiltInConstraints
      = BuiltInConstraint "," BuiltInConstraints
      / BuiltInConstraint

    BuiltInConstraint
      = AssignmentExpression       /* JavaScript expression    */

    Body
      = Constraint "," Body
      / Constraint

    Constraint
      = CHRConstraint
      / AssignmentExpression       /* JavaScript expression    */
      / FunctionExpression         /* JavaScript function call */

    CHRConstraint
      = ConstraintName "(" Parameters ")"
      / ConstraintName

*For the sake of simplicity we omitted some rules, the whitespace-handling and transformation steps in the presented Parsing Expression Grammar. The PEG syntax is similar to EBNF or Regular Expressions: `/` separates alternatives, `?` indicates optional terms. The complete grammar can be found in the [CHR.js repository](https://github.com/fnogatz/CHR.js/blob/master/compiler/parser.pegjs).*

In the previous listing we presented only the parsing grammar. PEG.js allows us to specify a transformation process for every single rule in a code block `{ ... }` to create some kind of semantic representation. For the `PropagationRule` the exact grammar with transformation code looks like this:

    PropagationRule
      = name:RuleName? headConstraints:CHRConstraints
          "==>" guard:Guard? bodyConstraints:Body
        {
          var desc = {
            type: 'PropagationRule',
            kept: headConstraints,
            removed: [],
            body: bodyConstraints,
            guard: guard || []
          };
          if (name) {
            desc.name = name.name;
          }
          desc.constraints = getConstraints(desc);

          desc = headNormalForm(desc);
          desc = addProperties(desc);

          return desc;
        }

In this way we create JavaScript objects that contain all the information about the rules, in particular the rule name, kept and removed constraints and rule body. The `headNormalForm()` function applies the head and guard normalisation step as presented in section 4.2.2 of Gregory Duck's [PhD thesis](#ref-duckphd), that means:

- variables occuring more than once in the rule's head are renamed,
- terminals in constraints of the rule's head are replaced by fresh variables and appropriate guards are added.

Given the above Parsing Expression Grammar, [PEG.js](#dependency-pegjs) provides a function that can parse JavaScript source code with embedded CHR and return its semantic representation as JavaScript object.

### Compiler

This object is passed to the compiler which creates the corresponding JavaScript constraint solver. The compilation scheme is based on [Van Weert et al.; 2008](#ref-vanweert2008) without any optimizations.

For every constraint in we create three kinds of rules:

- A *Caller* function, that is first called if a constraint is manually added or called in the body of a rule. For the `fib/2` constraint of the [Fibonacci example](#example-program), this is a generalized `CHR.fib()` JavaScript function that would be used for all `upto` constraints regardless of its arity:

      CHR.prototype.fib = function fib() {
        var arity = arguments.length
        var callConstraint = "_fib_"+arity+"_activate"
        // for example: "_fib_2_activate"
        
        // Create new Constraint and add it to the Store
        var constraint = new Runtime.Constraint("fib", arity, arguments)
        this.Store.add(constraint)

        // Execute the (arity-depending) activator function
        this[callConstraint](constraint)
      }

- An *Activator* function for every (name,arity)-pair. In case of the Fibonacci example we only have two different Activators, namely `CHR._upto_1_activate()` and `CHR._fib_2_activate()`. The activators get the recently created constraint and try to execute the application of every CHR rule that contain this (name,arity)-pair in its head. Therefore the occurences of these pairs are passed through from top to bottom (in the order of the CHR rules) and from right to left (in the order of a single CHR rule). This sequence is chosen in [Van Weert et al.; 2008](#ref-vanweert2008) to ensure the [refined operational semantic](#ref-duck2004operationalsemantic) which states that the removed constraints of a simpagation rule are tried for application first.

    Therefore the `fib/2` constraints in the `calc`-rule are numbered in that way, that `fib(N1,M1)` is the first and `fib(N2,M2)` the second occurence. The Activator tries to execute the application in this order, that means for the `fib/2` constraint the Activator function simply looks like this:

      CHR.prototype._fib_2_activate = function (constraint) {
        this._fib_2_occurence_1(constraint)
        this._fib_2_occurence_2(constraint)
      }

- An *Occurence* function for every occurence of a constraint in a rule's head. This function performs a lookup in the constraint store to find all appropriate constraints that are also in the rule's head and therefore needed for the rule's application. Depending on the rule's body and type, new constraints are added, others removed or kept. This Occurence function also performs a check in the propagation history to prevent the repeatedly execution of a rule.

    The generated Occurence function `CHR._fib_2_occurence_1` for the first occurence of the `fib/2` constraint can be found in the [Appendix](#example-occurence-function).

### Runtime

In the previous code examples we already used the three Runtime components that are necessary for all CHR.js applications:

- `Runtime.Constraint`, a JavaScript object that represents a single instantiated constraint.

      // Create a fib(1,0) constraint
      var c = new Runtime.Constraint('fib',2,[1,0])

      // Properties
      c.name                // 'fib'
      c.arity               // 2
      c.args                // [1,0]
      c.id                  // is unique
      c.alive               // true or false

      // Methods
      c.toString()          // 'fib(1,0)'
                            //   used for debug output

- `Runtime.Store`, the Constraint Store that manages the addition and removal of constraints.

      // Create a new instance
      var s = new Runtime.Store()

      // Methods
      s.add(c)              // add a constraint
      s.kill(c)             // remove a constraint
      s.alive(c)            // check if constraint
                            //   is still alive
      s.lookup(name,arity)  // search for matching
                            //   constraint

      // Events emitted
      s.on('add', function(constraint) {})
                            // fired on constraint
                            //   addition

      s.on('remove', function(constraint) {})
                            // similar on removal

- `Runtime.History`, the Propagation History to save already executed combinations of constraint identifiers.

      // Create a new instance
      var h = new Runtime.History()

      // Methods
      h.add(rule, ids)      // add an application
                            //   to the history
      h.notIn(rule, ids)    // check for record

