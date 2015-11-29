---
title: "A Paper a Week #1: OpenJDK's java.util.Collection.sort() is broken: The good, the bad and the worst case"
layout: post
date: 2015-02-27 01:18:35 CET
categories: apaw
---
I'm planning to read at least one paper a week about computer science or logic related topics and comment a bit on this blog. This is the first post in the series (let's see how long I am able to keep this up).

The paper thus discussed has caught my attention because while I am really interested in and like logic and theoretical computer science, I am always ambiguous about the practicability of the field. This makes me happy about a paper that shows real world problems being solved using tools (theorem proving) that would not be possible without extensive research in both fields.

So, what is the paper actually about? It documents the findings of a group of researchers that have uncovered a bug in Java's TimSort implementation (and, actually, the algorithm TimSort itself) that results in Collection.sort throwing an index out of bound exception, which is the last thing one would expect it to do. As the authors point out in the introducing paragraphs, this is particularly interesting because the bug occurs in OpenJDK, a well-tested and well-studied piece of code. This shows that the bug occured despite unit-testing practices.

The authors then go on formally verifying the properties that make sure the exception can never be raised in their fixed version. While one could argue that actually, the authors did not present a *practical* application of theorem proving, as the counter-example is obviously very carefully crafted to trigger the bug (and apparently wasn't triggered as no one was aware of the bug), one has to consider two things. Firstly, the bug possibly offers an exploit for malicious attackers to remotely crash an application that accepts user data, and, secondly by Murphy's law, by throwing enough data at `Collection.sort`, eventually one will hit a bad case.

While the paper certainly shows that it is possible to use formal methods to an advantage in the software industry (a practice that is common in the hardware industry nowadays, following the extremely expensive [Intel FPU bug] [2]), this would, unless theorem proving methods improve, require an almost impossible amount of change in the industry. One must consider that it took experts in the field guiding the steps of the interactive verification where, quote, approximately 5-10% required ingenuity, such as introducing crucial lemmas and finding suitable instantiations of quantifiers (“Q-inst”). I do not see this happening in a lot of contexts.

So, without further ado, here is a link to the paper for those interested: [PDF] [1]

[1]: http://envisage-project.eu/wp-content/uploads/2015/02/sorting.pdf "Paper"
[2]: http://www.csl.sri.com/papers/computer96/computer96.html "FPU bug"
