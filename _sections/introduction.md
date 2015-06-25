---
title: "Introduction"
order: 2
---

[Constraint Handling Rules](#ref-fruhwirth1998chr) (CHR) is a declarative programming language extension, that "has become a major declarative specification and implementation language for constraint-based algorithms and applications" ([Frühwirth, Raiser; 2009](#ref-fruhwirth2009constraint)) in the last 20 years. Although there are implementations for popular host languages like Java and Prolog, the distribution of CHR is currently restricted almost entirely to the research community.

One of the reasons that CHR is not widely used in a non-academic environment is the lack of an easy to use programming and software environment: Due to its orientation as a language extension it is always necessary to install its host language first - most common are Prolog, Java and C. By implementing a CHR constraint solver in JavaScript it is possible to use CHR constraints natively in the browser and in server-based JavaScript runtime environments like [node.js](http://nodejs.org/).

### Context of this Work

The aim of this project was to create a prototype implementation of a CHR transpiler for JavaScript, executable directly in the browser. It was realized as individual project as part of the Masters programme in Computer Science at the [University of Ulm](http://uni-ulm.de). It has been supervised by [Prof. Dr. Thom Frühwirth](https://www.uni-ulm.de/in/pm/mitarbeiter/fruehwirth.html) and [M.Sc. Daniel Gall](https://www.uni-ulm.de/in/pm/mitarbeiter/gall.html) of the [Institute of Software Engineering and Compiler Construction](http://www.uni-ulm.de/in/pm/).
