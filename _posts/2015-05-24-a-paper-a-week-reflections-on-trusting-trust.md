---
title: "A Paper a Week-ish #8: Reflections on Trusting Trust"
layout: post
date: 2015-05-24 16:00:00 BST
categories: apaw
---

So, I feel like cheating a bit again as  this paper is only three pages long, but I guess it *is* an ACM Turing Award lecture paper, so it deserves a spot here, at least for historical significance. Also, the paper is written by the highly influential (well – which goes without saying considering the Turing Award) Ken Thompson.

In the introduction he argues that while people may expect him to talk about UNIX, because this is what he is most famous for, he will not because he feels that he is getting undeserved credit for the work of a multitude of people where for his contributions luck and fortuate timing played as much part as skill. He also promises to show the “cutest program” he has written, but I am not sure how that is followed up on in the paper.

It talks about self-printing programs before concerning itself with self-hosting compilers, that is, compilers for a language that are written in that same language, for instance a C compiler written in C. This reminded me a bit of assembling an IKEA desk yesterday that required a table for assembly, so you have the same chicken-and-egg problem. The solution, of course, is relatively straightforward and consists of writing an initial version of the compiler in another language (or getting an table that is not this IKEA desk – or indeed in this case not following the instructions, but that is irrelevant to a discussion about computers), and then using existing binaries of the compiler to compile subsequent versions.

The example he shows is a bit more interesting. He considers a high-level idealized section of C compiler that treats string escape sequences, basically as a sequence of ifs like

{% highlight c %}
if (c == 'n') {
	return '\n';
}
{% endhighlight %}

This is interesting for itself, as once again this is a self-reference that expects this to be compiled by a thing that already understands what `'\n'` means. Now consider the case when you want to add a new escape sequence that is not available in your current compiler. Obviously the above will fail, as your compiler will complain that it does not know what to do with `'\n'`. However, what you can do is add the following the the compiler, creating a version that understands the meaning of `'\n'`.
'
{% highlight c %}
if (c == 'n') {
	return 10; // 10 is the ASCII code for \n.
}
{% endhighlight %}

You can now, in any subsequent version of the compiler that is compiled by this (or a newer one), refer to `'\n'` instead of 10. He claims that ‘It is as close to a "learning" program as I have seen’, which I would take with a grain of salt (especially honouring the fact that this paper was written in the 70s) considering recent advances in machine learning, whereas this is more of a matter of where a particular piece of data was introduced (the compiler, the compiler's compiler, …) than anything that has genuinely to do with learning – the fact that it should return 10 for `'c == 'n'` is in the compiler's binary, either way.

Then he, as is quite obvious, applies these ideas to information security. How, in face of this ridiculous matryoshka doll situation, can one be sure what a program does – in particular, that it does not do anything malicious? The only way would be to go up the chain of compilers until the root, which does not appear tractible. He suggests the alternative is trusting the people that wrote the program, thus the title of the paper (as the trust, indeed, has to be recursive – you implicitely trust the compiler that was used to compile your compiler).

He then ends with a few political comments about persecution of “hackers” and the two-facedness of considering them wiz kids and criminals at the same time, which sadly still seem current now over 40 years later.

Find the paper [here] [1].

[1]: https://www.ece.cmu.edu/~ganger/712.fall02/papers/p761-thompson.pdf "Paper"