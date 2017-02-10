---
title: "More Turing Completeness, or: Somebody is wrong on the Internet"
layout: post
date: 2016-11-23 23:40:00 CET
categories: computer-science
---
Hello, I am Florian and you might remember me from posts such as [Turing Completeness](http://bitsrc.org/blog/theoretical-cs/2016/08/27/turing-complete.html). Today I am going to return to the topic of Turing Completeness, in a post in similar vein. Today's post explores the arguments Zed Shaw presents in [Learn Python The Hard Way](https://learnpythonthehardway.org) on why [Python 3 is the inferior version](https://learnpythonthehardway.org/book/nopython3.html) of the language; in particular, how the concept of Turing Completeness is abused to make a point that is completely unrelated to it. Note that I explicitly do not want to take any position in the Python 2 vs. Python 3 argument, I merely want to expose the flawed logic in this particular line of reasoning.

Let's go explore the argument presented and why it is flawed.

> Python 3 Is Not Turing Complete

With the right definition of Python 3, this might actually be technically correct. Any computer language executed on a real physical computer can never be truly Turing complete. Because all resources in a computer are finite, you could theoretically enumerate all possible states. If you have studied a bit of computation theory, Turing Machines are more powerful than that (also, you will know that functions that you can enumerate are usually "boring" from a computation theory point of view).

> In computer science a fundamental law is that if I have one Turing Machine I can build any other Turing Machine.

I am not completely sure what that even means. If you have a Turing Machine that just does nothing, I can not use that to build any Turing Machine. There is a Turing Machine (also known as the Universal Turing Machine) that can simulate all other Turing machines. Note that this requires the Turing Machines it simulates to be *encoded* in some suitable way.


> If I have COBOL then I can bootstrap a compiler for FORTRAN (as disgusting as that might be). If I have FORTH, then I can build an interpreter for Ruby.

If you have Python 3, you can *definitely* use that to write an interpreter of Python 2 in it, if you really really want to. But even that has no direct correspondence to the point of Turing Completeness. Note how this talks about specific encodings of a program (i.e. programming languages), while the concept of Turing Completeness concerns itself with mathematical (partial) functions.

A mathematical function is just a mapping from input to output, while a program is a procedure to produce said output. It is completely possible that a Turing Complete system is unable to express a *program* another one can. The Game of Life is Turing Complete, but it will definitely will never be able to parse your Python program from your hard-disk.

> Currently you cannot run Python 2 inside the Python 3 virtual machine. Since I cannot, that means Python 3 is not Turing Complete and should not be used by anyone.

No, no. It really really does not mean either of these things.
 
> Note
> 
> Yes, that is kind of funny way of saying that there's no reason why Python 2 and Python 3 can't coexist other than the Python project's incompetence and arrogance. Obviously it's theoretically possible to run Python 2 in Python 3, but until they do it then they have decided to say that Python 3 cannot run one other Turing complete language so logically Python 3 is not Turing complete. I should also mention that as stupid as that sounds, actual Python project developers have told me this, so it's their position that their own language is not Turing complete.

I would really like to see these conversations. Either these Python developers are misquoted here, or were wrong (or, as explored above: technically correct – but that has nothing to do with any of the arguments that were made).

The rest of the argument goes on in similar fashion, incorrectly combining concepts of theoretical computer science with implementations, citing how other virtual machines manage to be more general purpose, and so on, and so forth; the latter of which, of course, is true – but invariantly true for CPython 2 and CPython 3. 
