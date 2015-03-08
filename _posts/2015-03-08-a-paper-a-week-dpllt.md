---
title: "A Paper a Week #2: DPLL(T): Fast Decision Procedures"
layout: post
date: 2015-03-08 12:00:00 CET
categories: apaw
---

If you are not interested in reading about I paper I meant to read and cover here but eventually gave up doing so, skip the next two paragraphs.

Because I was interested in reading about Z3, the SMT (satisfiability modulo theory) solver developed by Microsoft Research, I originally tried to read ["Tractibility and Modern Satisfiability Modulo Theories Solvers"] [2]  because I – wrongly, as it would sadly quickly turn out – assumed to would explore some deep connection between tractibility and SMT solvers I have not previously heard of. I started reading and it would offer a fairly good reasonably high level introduction into SMT and DPLL and some of its applications. 

Then it turns to listing examples for tractible and intractible theories, elaborating them to various degrees; I am not sure how this adds to the rest of the paper, but I stopped reading the paper about 2 thirds in, so I might have missed something. Especially the section about the theory of arrays was impossible to understand, as from all I can tell, they define \\( C_1 = a \lor \neg b \\), only to later use it as \\( read(C_1, \ldots) \\), which looks like applying a function to a formula, which is odd. I am really not sure about the notation there; maybe I don't understand "We will use the array M to encode a propositional model, and for each clause \\( C_i \\) use a corresponding array to encode selecting a literal that evaluates to true in the model" correctly, because I am not sure what it is supposed to mean.

The paper referenced "DPLL(T): Fast Decision Procedures", which introduces the general algorithm used by Z3, and because I have been running out of time trying to understand the Microsoft Research paper, I decided to go for that one for this post instead. It introduces the DPLL(T) method, which is basically propositional DPLL extended by some theory. I should probably elaborate  a bit more on that  at this point. DPLL is (*very* roughly) based on the observation that, imagine you have \\( a \lor b \\) and you know that \\( \neg a \\), then you can conclude \\( b \\). Now, if you have a formula in CNF (conjunctive normal form) and want to  check its satisfiability, you can branch on any atom arbitrarily (i.e., you just assume that some atom \\( a \\) is true, and see if that leads to a satisfying interpretation, if it does not, assume it is false and do the same), then propagate the information as in the example before (this process is called unit propation), hopefully get the value for more atoms, repeat this process on them until you can no longer propagate and then branch on another unset atom.

Countless improvements and heuristics have been thought up, but will not be covered in this post as to keep this post at a sane length. The paper is not about those, anyway, but rather a new way to adapt the system to be able to handle formulas over a theory. Basically, a theory is a set of symbols with related axioms that force them to behave the way they are supposed to. The paper considers the theory EUF, which stands for equality and uninterpreted functions, which essentially makes sure that equality signs are interpreted with the usual meaning (also in the presence of functions, such that \\( f(a) \ne f(b) \\) implies \\( a \ne b \\)). Techniques predating the paper achived that by adding applicable axiom instantiations (e.g. add \\( a = a \\), \\( b = b \\), … to make sure equality is reflexive) either at once or incrementally until a model consistent with the theory was found; hybrid approaches were also proposed. 

The technique introduced in the paper, on the other hand, solves the problem by defining a modular interface that interfaces solvers for a theory to the general DPLL algorithm. This can be thought of this way: to the DPLL part, all atoms are purely syntactic as in the propositional case. For instance \\( a = b \land b = c \land a \ne c \\) is of the form \\( A \land B \land \neg C \\) and is satisfiable by \\( v(a = b) = 1, \\) \\( v(b = c) = 1 \\), \\( v(a = c) = 0 \\). Of course, this model is inconsistent to the theory of equality, which the solver for the theory will detect; it will also take into account the semantics of the theory for unit propagation.

So, I guess this post is long enough as it is, so without further ado, a link to the paper: [[PDF]] [1].


[1]: https://www.cs.upc.edu/~oliveras/espai/papers/dpllt.pdf "DPLL Paper"
[2]: http://research.microsoft.com/en-us/people/nbjorner/tractability.pdf "MSFT Paper"
