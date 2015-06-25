---
title: "Preliminaries"
order: 3
---

Similar to existing CHR(Prolog), CHR(Java) and CHR(C) systems we built a CHR(JavaScript) application called *CHR.js*. Before we examine the requirements and implementation details, we want to present existing approaches to implement logic programming languages in general and CHR in particular in imperative host languages.

### Existing Tools

CHR can be used in a great many programming languages of different programming paradigms, including Prolog ([Schrijvers, Demoen; 2004](#ref-schrijvers2004ku)), Java ([Van Weert, Schrijvers, Demoen; 2005](#ref-van2005ku)), C ([Wuille, Schrijvers, Demoen; 2007](#ref-wuille2007cchr)) and Haskell ([Lam, Sulzmann; 2007](#ref-lam2007concurrent)). In general the given CHR rules are compiled to its host language. Although there a real interpreters like [ToyCHR](http://www.comp.nus.edu.sg/~gregory/toychr/), the most common approach is to build a transpiler.

There are already several Prolog implementations in JavaScript, for instance [Yield Prolog](http://yieldprolog.sourceforge.net/) and [JScriptLog](http://jlogic.sourceforge.net/), but there is currently no way to use CHR in web applications. [Pengines](#ref-lager2014pengines) (project homepage: [pengines.swi-prolog.org](http://pengines.swi-prolog.org/docs/index.html)), a recent approach to use Prolog on websites, communicates over HTTP with a server running a SWI-Prolog instance and would therefore be a possibility to call CHR constraints in the browser too. Nevertheless it does not solve the original problem as there is still a server necessary that accepts these Prolog remote procedure calls. Therefore Pengines is very similar to [WebCHR](http://chr.informatik.uni-ulm.de/~webchr/), that also dispatches calls on CHR constraints via HTTP to a backend server, in this particular case running Sicstus Prolog 4.

[J. F. Morales et al.](#ref-morales2012lightweight) presented a lightweight compiler of (constraint) logic programming languages, (C)LP, to JavaScript in 2012. It does not support CHR and is based on a special module system which makes it necessary to implement (C)LP rules in a JavaScript dialect. In contrast to that our aim was to embed CHR rules in existing JavaScript source code and compile this constraint solver to standard JavaScript.

### CHR in Imperative Programming Languages

Although the most popular host language for CHR, Prolog, is a representative of the declarative programming paradigm, there has been a lot of research about implementing CHR in imperative programming environments recently. As a consequence, CHR extensions for [C](#ref-wuille2007cchr) and [Java](#ref-van2005ku) evolved, as presented in the [previous subsection](#existing-tools).

Peter Van Weert explained some advantages of adopting CHR in imperative host languages in his [PhD thesis](#ref-vanweertphd) as follows:

> Moreover, imperative host languages are better suited for efficient evaluation of rule-based programs. The indexing data structures and complex nested loops required can be implemented more effectively in an imperative language.

The basic compilation scheme presented in this PhD thesis and [Peter Van Weert et al., 2008](#ref-vanweert2008) has been the basis for CHR.js. Besides that the compilation scheme presented by [Gregory J. Duck](#ref-duckphd) in 2005 was inspiration for the parsing and head and guard normalisation process.
