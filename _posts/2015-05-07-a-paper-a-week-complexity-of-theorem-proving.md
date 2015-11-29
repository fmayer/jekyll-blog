---
title: "A Paper a Week-ish #7: The Complexity of Theorem-Proving Procedures"
layout: post
date: 2015-05-07 23:00:00 BST
categories: apaw
---
First of all, sorry for the delay! I have moved this weekend (with a few complications) and am currently without Internet at home, and Three are actively trying to prevent me tethering from my phone (and I don't want to violate the contract).

This is actually, for computer science, a fairly old paper, written by no less than Stephen Cook, about whom you might have heard, for instance regarding the Cook(–Levin) theorem, which in fact is closely related to the content of this paper. Essentially, Cook's theorem states that the propositional case of satisfiability is NP-complete.

For those not familar with complexity theory (very roughly the theory for determining the relative hardness in terms of computational effort of different problems), NP-complete problems are those that are mostly believed to be hard to solve (that is, there exists no polynomial algorithm), but no proof exists for that statement. Solving – as in coming up with an algorithm of polynomial running time, one would immediately solve the whole class. This is due to the observation that is also expressed in the paper that there exist polynomial reductions between the problems (a reduction is a transformation of an instance of one problem into an instance of another, whereby one can solve the initial problem).

The first half of the paper talks about the complexity of theorem proving in the propositional case, which is essentially the same as satisfiability in classical logic (something is a tautology iff its negation is unsatisfiable)

The second half of the paper concerns itself with the complexity of theorem-proving first-order (i.e. predicate) logic and presents lesser known ideas. It is generally known that theorem proving is undedicidable, that is, there is no algorithm that can proof all valid theorems of first-order logic. Thus, the definition of complexity is a bit more evolved than in the propositional case. How can one analyze an algorithm that can possibly go on forever without halting? Cook essentially proposed setting an order on the Herbrand instances (essentially instantiating every function and predicate in every possible way) to analyze the complexity relative to the highest index of the instances used in a proof (this is a bit of a wandwavey explanation). He presents two bounds on this measure.

Find the paper [here] [1] (actually, this is a re-typeset version, because the [original one] [2] was written on a typewriter and this is, in my opinion, nicer to read).

[1]: http://www.chell.co.uk/media/product/_master/1/files/cook_complexity_of_theorem_proving_procedures_19711.pdf "Paper"
[2]: https://www.cs.toronto.edu/~sacook/homepage/1971.pdf "Original Paper"
