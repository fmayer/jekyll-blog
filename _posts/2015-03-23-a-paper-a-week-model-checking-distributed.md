---
title: "A Paper a Week #4: Parameterized Model Checking of Fault-tolerant Distributed Algorithms by Abstraction"
layout: post
date: 2015-03-23 22:00:00 BT
categories: apaw
---

First of all, sorry for this post being delayed. But with moving to London and starting a new job, I guess it is not the worst week to give myself a break. Anyway, while there is a post, for this paper, as it is particularly involved (though very concentrated on 8 pages), I will start with a more high-level post and follow up in a later post with more technical aspects.

So I guess for the unfamiliar reader, the title needs some further elaboration. Model checking refers to a technique for automatically verifying properties of computer programs. This is very roughly done by restriciting the types of variables occuring in the program, checking for counter-examples in that simplified program, check whether it is possible for them to appear in the real program, and, if not, refine the simplification.

This paper introduces a method to make this also applicable for a kind of distributed algorithm. The paper concerns itself with algorithms which provide fault-tolerance up to a degree, i.e. they guarantee they are behaving correctly even if up to a predefined fraction of the processes contributing to it are misbehaving.

The problem with that is, though, that generally the number of processes and thus of mishaving processes can be arbitrary. Verifying the system for fixed parameters \\( n \\) of total processes and a maximum of \\( t \\) faulty ones is unfeasible for large values thereof due to state space explosion. This is where the parametrized model checking comes in. 

All processes are collapsed into a single large system where a state transition represents one of the processes taking a single step. This is then used to verify LTL formulas against the system, i.e. it is checked whether for all parameters compatible with the maximum amount of failing processes, the property holds on all possible paths.

The details on how this is done will be covered in a follow up post.

For those who want to skip ahead and find out themselves: [PDF] [1].

[1]: http://www.cs.utexas.edu/users/hunt/FMCAD/FMCAD13/papers/10-Model-Checking-Fault-Tolerant-Distributed-Algo.pdf "Paper"

