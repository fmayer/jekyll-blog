---
title: "A Paper a Week #6: Differential Privacy"
layout: post
date: 2015-04-12 12:00:00 CET
categories: apaw
---
First, I will, for the coming few weeks, reduce the frequency of these posts to one a fortnight – for one, fortnight is an amazing word, and also I am rather busy with moving (and working), so I will resume to the usual frequency after I have settled a bit.

With this out of the way, now on to talking about papers! This week's paper was published by Cynthia Dwork (the co-inventor of differential privacy) of MS Research and is about differential privacy.

What is that, you ask? Differential privacy is a measure introduced to reason about privacy leaks (affecting individuals) caused by statistical databases. Differential privacy aims to make certain guarantees of protecting individuals' privacy even in the presence of auxilliary information to an attacker. This concept is introduced in the paper by an example (which is an excellent thing to do in a paper if you want people to actually understand what you are talking about): imagine an attacker knows a person is 2 inches smaller than the average citizen of Urugay, then a query giving the average height of a citizen of Urugay, in combination with this auxilliary knowledge, reveals (more or less) personal information about the person.

This also nicely illustrates that it is impossible to design a database that, combined with arbitrary auxilliary information, never reveals any personal information. Thus, the aim of this paper is more modest: to design databases where participating in the database does not significantly increase the chance that personal information is revealed. 

Essentially, this is achieved by fuzzing the output enough so that it does not differ for any two database configurations only differing in one entry. I will skip the details and refer to the paper, which presents them in a quite understandable fashion (assuming you are concentrating while reading it).

And yes, the paper stops as randomly and abrupt as this blog post.

Find the paper here: [PDF] [1].

[1]: http://www.msr-waypoint.com/pubs/64346/dwork.pdf "Paper"
