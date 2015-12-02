---
title: "Theorem Proving in Haskell"
layout: post
date: 2015-12-02 01:00:00 BST
categories: programming
---
[Code on GitHub] [2]

For some reason that is not completely clear to me either, I decided to go about and implement the [sequent calculus] [1] technique for theorem proving this weekend. Because that alone would be a rather dull exercise, I decided to do it in a language I do not know very well. I first thought about trying to implement it in OCaml, but then decided to go with Haskell because there I at least know some basics.

Before going into the implementation, it is probably worth spending a few words on the sequent calculus. The basic construct of the sequent calculus is a sequent, which looks like this: \\( A, B, \ldots \vdash C \\). This is the assertion that, from the set of assumptions \\( A, B, \ldots \\), we can prove \\( C \\). The sequent calculus consists of rules that allow to simplify the sequents in a way that the simplified sequents are valid iff the original one is. These rules can be repeatedly applied until only atoms exist in the sequent, at which point it is obvious whether it is valid or not (e.g. \\( A \vdash A \\) is obviously correct, while \\(A \vdash B \\) is not, with \\( A \\) and \\( B \\) being logical atoms).

One detail I have omitted so far is that I initially wanted to write a prover that works for intuitionistic instead of classical logic. The difference between the two logics is that, intuitionistic logic is stricter than classic logic in that it requires proofs to be constructive. This means that the proofs can be used to construct an object that satisfies the assertion. For instance, \\( P = NP \lor P \ne NP \\) is obviously true in classical logic, but intuitionistically one cannot prove this without knowing which of the two sides hold. If you are only here for the Haskell, it is probably not important that you completely understand this.

My initial attempt (I am very inexperienced in Haskell) was to define two functions to encode the proof with the *first operation* being on left or the right side of the sequent. It would then call the respective other function on the simplified sequents. At this point I only wanted to encode intuitionistic proofs, so I had the left hand side of the proof as a list of expressions, and the right hand side as a single expression. I had the constructor Nil, only to be used on the right side, to express that it is empty. `proveLeft` tried to apply a rule to the first expression in the list.


{% highlight haskell %}

data Expr = Implication Expr Expr | Var String | Neg Expr | Nil deriving (Eq,Show)
data Sequent = Sequent [Expr] Expr deriving (Eq,Show)

proveLeft :: Sequent -> Bool
proveRight :: Sequent -> Bool
{% endhighlight %}

I ran into all sorts of problems, especially with the negation operator on the left side, which can only be applied when the right hand side is empty. Atoms had to be explicitly skipped by putting them to the end of the list. Because of this it was hard to figure out when to terminate, or, as with intuitionistic logic one has the drop the right hand side if the proof is “stuck”, when to do that.

I tried to make this work for a bit, but then realized if I ever got it to work, it will be extremely brittle and probably broken in one corner case or another. I then tried attacking the problem in smaller parts – the first one of which was transforming a sequent into simpler sequents.  I also thought always using the first expression in the list as the one to operate on was not helpful, so I came up with the following functions (with rather poor names). I also decided to do classical logic first – in a sense, it is simpler because the left and the right hand side are both sets of expressions.

{% highlight haskell %}
expandLeft :: Sequent -> Expr -> Maybe [Sequent]
expandRight :: Sequent -> Expr -> Maybe [Sequent]
{% endhighlight %}

The return value is `Maybe [Sequent]` so the function can return `Nothing` if the `Expr` cannot be operated on, or `Just [Sequent]` if it can be. For optimization purposes I decided to replace the `[Expr]` from before with a custom type containing three `[Expr]` – one for atoms (we need not bother trying to apply rules to them), one for negation (for classical logic, this is not needed, but later for intuitionistic we will want to skip those if there is something on the right hand side), and other composite expressions that we can always apply rules to. This is not strictly needed because of the `Maybe [Sequent]`, but it felt a bit wasteful to call a function that only returns `Nothing` for every atom – would be interesting to do profiling on this.

{% highlight haskell %}
data Expr = Implication Expr Expr | Var String | Neg Expr deriving (Eq,Show)
data ExprSet = ExprSet [Expr] [Expr] [Expr] deriving (Eq,Show)
data Sequent = Sequent ExprSet ExprSet deriving (Eq,Show)
{% endhighlight %}

The rest of the program involved a lot of using functions from `Data.Maybe` to put the pieces together.

{% highlight haskell %}
stepRight :: Sequent -> Maybe [Sequent]
stepRight s@(Sequent lhs rhs) = listToMaybe $ mapMaybe (expandRight s) ((getComposite rhs) ++ (getNeg rhs))
{% endhighlight %}

This runs `expandRight` for all expressions on the right hand side, and returns the result first one of those that returns a `Just [Sequent]`, or `Nothing` if all of them return `Nothing`. The right hand side is the same idea. 

Then, the same idea can be used to apply steps on the left side until there are none more to be taken, then on the right hand side. LK stands for the classical version of the sequent calculus there.

{% highlight haskell %}
stepLK :: Sequent -> Maybe [Sequent]
stepLK s = listToMaybe $ catMaybes [stepLeft s, stepRight s]
{% endhighlight %}

Even though I start with a single sequent, a step on a sequent can result in more than one. We need to apply `stepLK` to all sequents in a list until it returns `Nothing` for all of them. To do so a function `steps` is  defined that returns either the derived list of Sequents after taking a step, or `Nothing` if no steps can be taken any more. 

{% highlight haskell %}
getFirst :: (Maybe a, a) -> a
getFirst ((Just x), _) = x
getFirst (_, y) = y

steps :: [Sequent] -> Maybe [Sequent]
steps xs = if all (isNothing . fst) nextIter then Nothing
                                             else Just $ concat $ map getFirst nextIter 
        where nextIter = map (\x -> (stepLK x, [x])) xs
{% endhighlight %}

Then, to bring this to the logical conclusion, this function is applied until it returns `Nothing`, and then it is checked whether all sequents in the call before that were axioms.

{% highlight haskell %}
iterToNothing :: (a -> Maybe a) -> a -> a
iterToNothing fn x = case fn x of Nothing -> x
                                  Just y  -> iterToNothing fn y

solve :: Sequent -> Bool
solve s = all isAxiom $ iterToNothing steps [s]
{% endhighlight %}

Turning this into a solver for LJ (the intuitionistic version of the calculus) is fairly easy at this point. The rules operating on the negation on the left side have to be guarded by a condition that the right hand side be empty, and the `stepsLK` function has to be changed to drop the right hand side if there are no more steps for a sequent but it is not an axiom yet. In the implementation this leads to two functions being different between the LK and LJ cases, but I have yet to find a way to properly select which of the two to use at runtime: plumbing the functions through the computation by passing them as arguments does not seem the nicest way.

[1]: https://en.wikipedia.org/wiki/Sequent_calculus "Wikipedia: Sequent Calculus"
[2]: https://github.com/segfaulthunter/prove.hs "Code"
