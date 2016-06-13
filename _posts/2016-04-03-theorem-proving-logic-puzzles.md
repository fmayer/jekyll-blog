---
title: "Using theorem proving to cheat in logic puzzles"
layout: post
date: 2016-04-03 23:40:00 BST
categories: logic
---

I recently got the book [What is the Name of this Book?] [1] by the excellent Raymon Smullyan, who is also the author of a book about [Gödel's Incompleteness Theorems] [2] I could not praise highly enough. The book I purchased is a collection of logical riddles. While recreationally solving logical puzzles oneself can be very rewarding, when I got the book I considered that it would be interesting to try and solve these using formalized logic. Letting a computer find the solution given the formal specification is the obvious next step, which is what I will attempt in this blogpost. To do so, I will first formalize the specification in first-order logic, and then use the [Z3 SMT solver] [3] to find satisfying models.

A lot of the book is about a world in which there are two types of people, knights and knaves. Knights always speak the truth, and knaves always lie. This particular puzzle is about finding out information about the one werewolf among people A, B and C that make the following statements.

* A: C is a werewolf.
* B: I am not a werewolf.
* C: At least two of us are knaves.

We are trying to find answers to the following two questions:

* Is the werewolf a knight or a knave?
* If you have to take one of them as a travelling companion, and it is more important that he be not a werewolf than that he not be a knave, which one would you pick?

First we will formalize the fact that there is exactly one werewolf as the two statements “there exists at least one werewolf” and “the werewolf is unique” (we assume that the universe contains only persons, because they are the only objects in this riddle, so we do not need a predicate for “is person”):

\begin{equation}\label{firsteqn}
\exists x\, Werewolf(x)
\end{equation}

\begin{equation}\label{onewerewolf}
\forall x\, (Werewolf(x) \implies (\forall y\, (x \ne y \implies \neg Werewolf(y))))
\end{equation}

Then, we introduce the Knight predicate (and an equivalence of negative Knight to Knave for convenience)

\begin{equation}\label{knave}
\forall x\, (Knight(x) \equiv \neg Knave(x)) 
\end{equation}

Next we can formalize the three statements, which are true if and only if the person that says them is a Knight.

\begin{equation}\label{astate}
Knight(A) \equiv Werewolf(C) 
\end{equation}

\begin{equation}\label{bstate}
Knight(B) \equiv \neg Werewolf(B) 
\end{equation}

\begin{equation}\label{lasteqn}
Knight(C) \equiv \exists x\, \exists y\, (x \ne y \land Knave(x) \land Knave(y)) 
\end{equation}

If we call the set of equations (\ref{firsteqn}) to (\ref{lasteqn}) \\( \Pi \\), then we want to find out whether

\begin{equation}\label{sol1}
\Pi \models \exists x\, (Knight(x) \land Werewolf(x))
\end{equation}

\begin{equation}\label{sol2}
\Pi \models \exists x\, (Knave(x) \land Werewolf(x))
\end{equation}

Then, to answer the second question, we want to check

\begin{equation}\label{sol3}
\Pi \models \neg Werewolf(A)
\end{equation}

\begin{equation}\label{sol4}
\Pi \models \neg Werewolf(B)
\end{equation}

\begin{equation}\label{sol5}
\Pi \models \neg Werewolf(C)
\end{equation}

So, with that out of the way, let us introduce the tool that will make sure we do not need to do any thinking: the Z3 SMT solver. SMT stands for satisfyability modulo theory and means that you can give it first-order formulas of some theories (like equality, integers, computer arithmetic, …) and it will try to find a satisfying model. Given infinite domains there is no guarantee that will be found (especially the negative answer is particularly challenging – if there is a positive one it will be found by enumeration eventually). If you want to prove \\( \Pi \models A \\) using an SMT solver, in general what you do is apply the deduction theorem to obtain \\( \models \pi \implies A \\) (where \\( \pi \\) is the conjunction of the elements of \\( \Pi \\)). All possible models satisfy this if and only if there is no model satisfying the negation, i.e. \\( \neg ( \pi \implies A) \\), which can be rewritten as \\(\pi \land \neg A \\). In other words, we ask the SMT solver if  \\(\pi \land \neg A \\) is satisfyable, and if the answer is no, \\( \Pi \models A \\) holds.

Now all that's left is translating this specification to Z3's language. First thing we need to do is define a datatype for our objects, that are Persons. By using `declare-datatype` we get objects that are only equal to themselves, so we can use the equality predicate as above.

{% highlight text %}
(set-option :timeout 600)
(declare-datatypes () ((Person A B C)))
{% endhighlight %}

Then, we need to define our three predicates Knight, Knave and Werewolf.

{% highlight text %}
(declare-fun Knight (Person) Bool)
(declare-fun Knave (Person) Bool)
(declare-fun Werewolf (Person) Bool)
{% endhighlight %}

Then we can formalize the formulas of \\( \Pi \\) into Z3 syntax pretty much verbatim. The statement that there is exactly one werewolf can be expressed with the two assertions that a model has to satisfy 

{% highlight text %}
(assert (exists ((x Person)) (Werewolf x)))
(assert (forall ((x Person)) (implies (Werewolf x) (forall ((y Person)) (implies (not (= x y)) (not (Werewolf y)))))))
{% endhighlight %}

The three statements by A, B and C become the following. Note that `=` is used for both logical equivalence and equality between objects of the universe in Z3.


{% highlight text %}
(assert (forall ((x Person)) (= (Knight x) (not (Knave x)))))
(assert (= (Knight A) (Werewolf C)))
(assert (= (Knight B) (not (Werewolf B))))
(assert (= (Knight C) (exists ((x Person) (y Person)) (and (not (= x y)) (and (Knave x) (Knave y))))))
{% endhighlight %}

Now we have asserted all we know about the problem, we can begin trying to prove (\ref{sol1}) – (\ref{sol5}) to see which one the solutions is correct. We use the `push` and `pop` operations of Z3 to temporarily add add assertions between the two operations. I simplified \\( \neg \neg Werewolf(…) \\) to \\( Werewolf(…) \\)

{% highlight text %}
(push)
(assert (not (exists ((x Person)) (and (Werewolf x) (Knave x)))))
(check-sat)
(pop)

(push)
(assert (not (exists ((x Person)) (and (Werewolf x) (Knight x)))))
(check-sat)
(pop)

(push)
(assert (Werewolf A))
(check-sat)
(pop)

(push)
(assert (Werewolf B))
(check-sat)
(pop)

(push)
(assert (Werewolf C))
(check-sat)
(pop)
{% endhighlight %}

Z3's answer to the input we gave it is

{% highlight text %}
unsat
sat
unsat
sat
sat
{% endhighlight %}

This means that (\ref{sol1}) and (\ref{sol3}) hold in all models, and the others do not. This means that the werewolf is a Knave, and we should take person A as our travelling companion.

While a rather simple example, I think this is a nice demonstration of what one can do with an SMT solver.

[1]: http://www.amazon.co.uk/What-Name-This-Book-Recreational/dp/0486481980 "What is the name of this book?"
[2]: http://www.amazon.co.uk/Godels-Incompleteness-Theorems-Oxford-Guides/dp/0195046722 "Gödels Incompleteness Theorems"
[3]: https://z3.codeplex.com "Z3 SMT Solver"
