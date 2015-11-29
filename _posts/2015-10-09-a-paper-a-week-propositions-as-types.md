---
title: "A Paper a Week-ish #13: Propositions as Types"
layout: post
date: 2015-10-09 00:10:00 BST
categories: apaw
---

Even though week-ish is an intentionally vague concept, I should apologize for this week-ish to take particularly long. The last month has been very busy and exhausting for me, so even though I read a paper, I never got to finish this post until now.

This paper presents no original insights, but is a very readable introduction into the Curry-Howard isomorphism and concepts related to it. It does not go into the complex technicalities but manages to convey a general intuition and interesting historical context. The appendix of the online edition (the edition I read) also includes an insightful correspondence between the author (Philip Wadler) and Howard about the various people involved in its history.

It introduces the concept in three simple statements.

* *propositions* as *types*

* *proofs* as *programs*

* *simplification of proofs* as *evaluation of programs*

What does this mean? This means that there is a bijection between proofs in a formal proof system
and programs in a formal system of computation, where propositions in the proof correspond to
types in the program.

This is a very general statement, and the paper in question shows it for one particular pair
of formal proof system and formal system of computation, in particular natural deduction
for intuitionistic logic and singly typed lambda calculus. It then notes that it indeed holds
in various other cases, the list of which I will not enumerate here.

If one thinks about the Brouwer–Heyting–Kolmogorov interpretation of intuitionistic logic, which is also
introduced in the paper, this correspondence becomes intuitively plausible. The BHK interpretation defines
the logical connectives by the existence of procedures to turn proofs of a proposition into another proof.

For example, a proposition \\( A \rightarrow B \\) holds if there is a procedure that turns
proofs of \\( A \\) into proofs of \\( B \\).  So if we look at a bijection between propositions
and types, this means that there is a procedure that turns an object of type \\( A \\) into one
of type \\( B \\). Also other propositions are expressed as objects in the BHK interpretation.
For example, \\(A \land B \\) holds if there is a tuple \\( \langle a, b \rangle \\), where
\\( a \\) is a proof for \\( A \\) and \\(b \\) is a proof of \\( B \\), so it can be seen as a tuple.
\\( A \lor B \\) can similarly be seen as the option type for an object that is either of type \\( A \\) or \\( B \\).

The paper does a much better job of explaining this all in a bit more detail than I ever could in the
course of a blogpost here, so I will not attempt to, but I hope this will bring across the very
basic idea behind the concepts.

I also recommend browsing the titles in the reference section if you are interested in the content
provided by this paper, I have highlighted [23], [33], [40], [42], [54], [56] and [59] (which does not mean
the other ones are worse, just that they did not immediately catch my eye) for further study (probably I will only get
to the papers, not the textbooks).

Find the paper here: [Propositions as Types] [1].

[1]: http://homepages.inf.ed.ac.uk/wadler/papers/propositions-as-types/propositions-as-types.pdf "Propositions as Types"
