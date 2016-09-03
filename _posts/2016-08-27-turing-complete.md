---
title: "Turing Completeness"
layout: post
date: 2016-08-27 23:46:00 CET
categories: theoretical-cs 
---

This is one of my pet peeves: I have heard many people describe Turing-complete as “a language you can do everything with”. That is untrue and this is a very short post to set this right.

Turing-complete means a language can be used to express a (probably non-proper – if you manage to find a system that is more expressible than the Turing machine, I am sure there is a Turing award or two waiting for you) superset of the pure mathematical functions that a Turing machine can compute. This does not talk about I/O (which is an important part of an actual computer) or the complexity these functions are executed in. So, I am afraid we will not see an operating system written in the Game of Life.

This definition also means an actual computer is technically not Turing complete, given it cannot simulate a Turing machine's infinite tape.

So, concluding: Turing-complete does not mean a language is useful to do something on an actual computer, while an actual computer is not actually Turing-complete. The two concepts are quite orthogonal.

Elaborating more (it was late yesterday): I/O can be “simulated” with appropriate encoding in the output, but that is one of the points: arithmetic is Turing-complete, so I can, for all Turing-computable functions, produce an integer that *somehow encodes* its result. This is an extremely important theoretical construct in computation theory, but also useless in a practical context, because I want to write actual data into actual memory of an actual machine. 
