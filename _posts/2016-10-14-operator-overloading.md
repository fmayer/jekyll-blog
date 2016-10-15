---
title: "Fun with Operator Overloading"
layout: post
date: 2016-10-14 17:00:00 CET
categories: python
---
The other day I was asked to take a look at code I had written years ago that basically
allowed the user to build up logical formulas while always keeping them in CNF (conjunctive
normal form – that is, a logical formula where the outermost connective is always a conjunction).

This was done by building a class hierarchy with an abstract class `Expr` on the top level,
two classes `ExprAnd` and `ExprOr` that represent conjunction and disjunction respectively and
both contain lists of `Expr`. Various other classes where used to represent various atoms of
the formulas, the particular code was about solar physics so the atoms happen to be things like
`Wavelength(x, y)` for selecting observations of a particular wavelength. 

The base class `Expr` defines `__and__` and `__or__`, that is the methods that need to be
implemented to overload the `&` and `|` operations that just return `ExprAnd` and `ExprOr`
of the object and the left hand operand. 

`ExprAnd` and `ExprOr` also implement the operation corresponding to their node type
by just returning a new `ExprAnd` or `ExprOr` with the left hand operand added to the
list, respectively. They also implement `__rand__` and `__ror__`, so their special implementation
is also applied if they are used at the right hand side of the operation – or that was the plan.
`ExprAnd` commutes `|` operations into the conjuction to keep conjunction as the top level
connective.

The problem that I asked to help with was that `foo & (bar & baz)` did not seem to evaluate into
`ExprAnd(foo, bar, baz)`, but rather `ExprAnd(foo, ExprAnd(bar, baz))`, that is, the automatic
transformation to CNF was not working.

Python actually implements logic that when you have an operation `foo & bar` where the type of
`bar` is a subclass of the type of `foo` and implements its own version of `__rand__`, the
expression will be evaluating using that. So when you read `__rand__` (or any of the operators)
is used of the left hand side does not implement the operator, that is wrong. For instance,

{% highlight python %}
In [18]: class Foo(object):
    ...:     def __add__(self, x):
    ...:         print "Foo ", self, x
    ...:         

In [19]: class Qux(Foo):
    ...:     def __radd__(self, x):
    ...:         print "Qux ", self, x
    ...:         

In [20]: Foo() + Qux()
Qux  <__main__.Qux object at 0x112f9f750> <__main__.Foo object at 0x112f7e510>
{% endhighlight %}

This behaviour was what I was aiming to exploit by making `ExprAnd` (for instance) implement
both `__and__` and `__rand__` and make Python use the most specific implementation. Sadly, it
does not work this way. If you now introduce a third class that inherits from Foo and does not
implement either of the methods itself, the base class' will be used, for instance


{% highlight python %}
In [21]: class Bar(Foo):
    ...:     pass
    ...: 
In [22]: Bar() + Qux()
Foo  <__main__.Bar object at 0x112f80890> <__main__.Qux object at 0x112f93a90>
{% endhighlight %}

The semantics of Python operator overloading will *only* prefer the `__rand__` method of the
right hand side if it is an object of a subclass of the *type of the right hand side object*.
Note that this does not take into account in which class the implementation of the left hand side
was actually done, which explains the behaviour above.

If you look at the class layout described above, this was exactly the case that I was hitting.
Disclaimer: What follows below is not how it was actually fixed in the code.

# Going Crazy
I realized that what I had intended to implement back then was something like commutative
[multimethod](http://clojure.org/reference/multimethods). Python's dynamicity, for better or
worse, gives you the tools to do most of such things, so I decided to give it a go. 

The gist of the idea is to try to find the most specific implementation of the operation,
considering both operands. The operation of the left hand side operand is more specific if it overrides the definition of the operation on the right hand side (and the other way
round).

This can conveniently be expressed in terms of Python: The operation of the left hand side
operand is more specific if the operation defined by the right hand side is in the MRO (method
resolution order – the list of classes, in order, that get consulted when looking up a method
in an object).

To implement this, first we define a functions to get all functions. It is important to know
that the function is different to the unbound and bound method object and does not contain
information about which class it is contained in; the function object can be obtained from
an unbound method object by using `im_func`.

{% highlight python %}
import inspect

def fn_mro(cls, fn):
    mro = inspect.getmro(cls)
    return [getattr(x, fn).im_func for x in mro if hasattr(x, fn)]
{% endhighlight %}


This can then be used to implement the logic as described above as a decorator that wraps
a method. Note that we have to compare whether the left and right side operator is actually the
same to prevent an infinite loop.

{% highlight python %}
def most_specific(fn):
    def func(self, other):
        name = fn.__name__
        self_fn = getattr(self, name).im_func
        other_fn = getattr(other, name).im_func
        if self_fn == other_fn:
            return fn(self, other)
        if self_fn not in fn_mro(other.__class__, name):
            # We did not inherit this from something in other's MRO.
            if other_fn not in fn_mro(self.__class__, name):
                # They didn't either. PANIC!
                raise TypeError
            return fn(self, other)
        else:
            return getattr(other, name)(self)
    return func
{% endhighlight %}

If we use the decorator on a slightly modified version of the previous example
(we use `__add__` instead of `__radd__` because the decorator assumes commutativity
and uses the same operations on both sides).

{% highlight python %}
In [7]: class Foo(object):
    @most_specific
    def __add__(self, x):
        print "Foo ", self, x
   ...:         

In [8]: class Qux(Foo):
    @most_specific
    def __add__(self, x):
        print "Qux ", self, x
   ...:         

In [9]: Foo() + Qux()
Qux  <__main__.Qux object at 0x103103690> <__main__.Foo object at 0x103103f50>

In [10]: class Bar(Foo):
   ....:     pass
   ....: 

In [11]: Bar() + Qux()
Qux  <__main__.Qux object at 0x103103e50> <__main__.Bar object at 0x10310b050>
{% endhighlight %}
