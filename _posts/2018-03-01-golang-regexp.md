---
title: "Analysing regexp with Golang"
layout: post
date: 2018-03-01 01:00:00 BST
categories: programming
---
**DISCLAIMER**: Let me say this first. Google's regular expression implementations are known for not implementing features that make them, well, not regular. Both `re2` and Golang's `regexp` do not support backreferences. Otherwise, the things done here would be hard, or impossible.

I may not be Golang's biggest fan in general (lack of generics, verbose syntax, simplistic type system, etc), but I've written a bunch of it in the last couple of years and found an unexpectedly useful feature. Golang provides a package called `regexp/syntax` that proves to be as useful as its documentation is sparse.

In essence, this package exposes the user to the finite state machines built by the regexp compiler. This can be used to do analyses on regular expressions such as "does this regexp ever match a particular character after matching `n` characters?" or "does this regexp match any strings starting with a particular prefix?". These might sound like constructed examples, but both of them actually popped up in my dayjob.

# Toy Example: Loop detection

For sake of simplicity, let's explore a constructed example first in this post: does a regexp match strings of arbitrary length? Or, in more technical terms: is there a loop in the finite state machine? Let's get right started with the boilerplate of compiling a regular expression into something called a `Prog`.

{% highlight go %}
r, err := syntax.Parse(`.*`, syntax.Perl)
if err != nil {
	panic(fmt.Sprintf("Parse: %v", err))
}

p, err := syntax.Compile(r.Simplify())
if err != nil {
	panic(fmt.Sprintf("Compile: %v", err))
}
fmt.Println(p)
{% endhighlight %}

This actually produces a nice and readable representation of the generated finite state machine. Note the * next to 2 which tells us that this is the initial state.

{% highlight text %}
  0	fail
  1	anynotnl -> 2
  2*	alt -> 1, 3
  3	match
{% endhighlight %}

What we see here is the textual representation of the `Prog` object. It's a struct containing

* `Inst`: list of instructions
* `Start`: initial instruction 

`Inst` is the type used to represent an instruction. It's a struct containing:

* `Op`: type of instruction
* `Out`: next instruction (except for InstMatch and InstFail, which are terminal instructions)
* `Arg`: additional argument to some instructions

For the purposes of this toy example, all we care about is which instructions can follow from a given instruction. For most instructions, that is the instruction referred to in `Out`. Let's introduce the odd ones here:

* `InstMatch`: successfully match input string. Does not have successor instruction.
* `InstFail`: reject input string. Does not have successor instruction.
* `InstAlt` / `InstAltMatch`: either `Out` or `Arg` are the successor instruction. A string matches if either of the branches arrives at `InstMatch`.


If you are curious about the difference between `InstAlt` and `InstAltMatch`: From all I could determine, `InstAltMatch` is an optimisation where it is known that one branch leads to a match while the other branch consumes characters. I do not see the compiler or any rewriting actually using this instruction, so it does not seem to be in use. Most of the implementation treats them interchangeably, while [backtrack.go](https://golang.org/src/regexp/backtrack.go?h=InstAltMatch#L181) in the regex evaluator appears to use it to determine which branch to take.

This information allows us to implement a helper function to determining the successor instructions, given an instruction.

{% highlight go %}
func GetSuccessors(i *syntax.Inst) []uint32 {
	if i.Op == syntax.InstMatch || i.Op == syntax.InstFail {
		return []uint32{}
	}
	o := []uint32{i.Out}
	if i.Op == syntax.InstAlt || i.Op == syntax.InstAltMatch {
		o = append(o, i.Arg)
	}
	return o
}
{% endhighlight %}

Now we can implement loop-detection by a simple breadth-first search, keeping track of already visited nodes in a set (i.e. a `map[uint32]bool`, because Golang does not have a set type).

{% highlight go %}
func HasLoop(p *syntax.Prog) bool {
	var tovisit []uint32
	tovisit = append(tovisit, uint32(p.Start))
	seen := make(map[uint32]bool)
	for len(tovisit) != 0 {
		i := tovisit[0]
		if seen[i] {
			return true
		}
		seen[i] = true
		tovisit = tovisit[1:]
		tovisit = append(tovisit, GetSuccessors(&p.Inst[i])...)
	}
	return false
}
{% endhighlight %}

# A regex engine

Now let's try to build a very inefficient regex engine based on this. To do so, let us first introduce the various rune instructions (rune is Golang for Unicode codepoint). There is `InstRune`, `InstRune1`, `InstRuneAny` and `InstRuneAnyNotNL`. Most of them (except `InstRune` and `InstRune1`) should be self-explanatory, but here's the whole list:

* `InstRuneAny` matches any rune.
* `InstRuneAnyNotNL` matches any rune except newlines.
* `InstRune` has a `MatchRune` method to determine whether it matches a rune.
* `InstRune1` matches the rune provided in `i.Rune[0]` (obviously.)

This leaves us with two remaining useful instructions:

* `InstCapture`: capture a match into a capture group. We won't bother with this here.
* `InstEmptyWidth`: match constrains on the current position in the string. This has a `MatchEmptyWidth` to determine whether it matches.

There's also `InstNop` which, well, does nothing.

Of course, the easiest way to do this is a recursive evaluator. We pass in a program, the current instruction, and input, and the current position in the input.

{% highlight go %}
func Match(p *syntax.Prog, pc uint32, input []rune, idx int) bool {
{% endhighlight %}

Let's first determine the previous and current rune at the current position, and use `-1` for the borders of the string (to be consistent with `MatchEmptyWidth`).

{% highlight go %}
	var prev rune = -1
	if idx > 0 && idx <= len(input) {
		prev = input[idx-1]
	}
	var cur rune = -1
	if idx < len(input) {
		cur = input[idx]
	}
{% endhighlight %}

And now all that's left is one fairly large switch (if you look at actual implementations of regex engines, there are also often giant switches, so it's legit).

`InstAlt` and `InstAltMatch` are the same, so let's use the `fallthrough` statement for go switch statements (this is much more sane than C-style fallthrough by default).

{% highlight go %}
	i := p.Inst[pc]
	switch i.Op {
	case syntax.InstAlt:
		fallthrough
	case syntax.InstAltMatch:
		return Match(p, i.Out, input, idx) || Match(p, i.Arg, input, idx)
{% endhighlight %}

We don't care about `InstCapture` or `InstNop` here.

{% highlight go %}
	case syntax.InstNop:
		fallthrough
	case syntax.InstCapture:
		return Match(p, i.Out, input, idx)
{% endhighlight %}

For `InstEmptyWidth` we use the method that was given to us for this.

{% highlight go %}
	case syntax.InstEmptyWidth:
		if !i.MatchEmptyWidth(prev, cur) {
			return false
		}
		return Match(p, i.Out, input, idx)
{% endhighlight %}

(I could get used to this). `InstMatch` and `InstFail` are obvious.

{% highlight go %}
	case syntax.InstMatch:
		return true
	case syntax.InstFail:
		return false
{% endhighlight %}

Then there come the various ways of matching runes. Note that this is the only time we have to increment the index into our input, as this is the only time we actually consume any runes.

{% highlight go %}
	case syntax.InstRune:
		if cur == -1 || !i.MatchRune(cur) {
			return false
		}
		return Match(p, i.Out, input, idx+1)
	case syntax.InstRune1:
		if cur == -1 || cur != i.Rune[0] {
			return false
		}
		return Match(p, i.Out, input, idx+1)
	case syntax.InstRuneAny:
		if cur == -1 {
			return false
		}
		return Match(p, i.Out, input, idx+1)
	case syntax.InstRuneAnyNotNL:
		if cur == -1 || cur == '\n' {
			return false
		}
		return Match(p, i.Out, input, idx+1)
{% endhighlight %}

That's it. Now some due diligence against us being bad programmers, and we're done.

{% highlight go %}
	default:
		panic("Invalid instruction.")
	}
	panic("Fell off the switch.")
}
{% endhighlight %}

Well, that was fun. But not terribly exciting. But what this gives is as good mental model of what exactly the different instructions mean, which can be used to build more exciting things.

# Do we only match even-length strings?

Now that we can proudly proclaim we have written a regular expression engine (well, maybe we were *slightly* cheating and someone else helped *a bit*), let's take up a bigger challenge. Given a regular expression that possibly matches arbitrarily length strings, determine whether all strings matched have an even size. Sounds like an interview question? A bit, but I also hope I'll never get this as an actual interview question.

Let's start with a similar prototype as for our `Match` function, but instead of our input let's have something that flips around whether we are at an even or odd step. For reasons that will make sense later, let's encode this as an integer that flips between `0` and `1` (so it's `idx % 2`). We also need to keep track of which nodes we have seen before, or we will wait for a long time. But if you think about it a bit, we need to keep track of this for even and odd steps, an even visit and an odd visit are not the same. That results in the beautiful type of `map[int]map[uint32]bool`, or a map from an integer to a set of `uint32`.

{% highlight go %}
func Even(p *syntax.Prog, pc uint32, idx int, map[int]map[uint32]bool) bool {
{% endhighlight %}

Now let's start with the easy part. I literally copy & pasted the `Match` code, removed all the stuff that actually does any … matching, plumbed through the visited map and made `idx` mod 2. That gives us all the rune instructions and `InstCapture`, `InstNop` and `InstEmptyWidth` (which are all, for all intents and purposes, noop).

{% highlight go %}
	i := p.Inst[pc]
	switch i.Op {
	case syntax.InstNop:
		fallthrough
	case syntax.InstCapture:
		fallthrough
	case syntax.InstEmptyWidth:
		return Even(p, i.Out, idx, visited)
	case syntax.InstRune:
		fallthrough
	case syntax.InstRune1:
		fallthrough
	case syntax.InstRuneAny:
		fallthrough
	case syntax.InstRuneAnyNotNL:
		return Even(p, i.Out, (idx+1)%2, visited)
{% endhighlight %}

`InstMatch` is straightforward, as we just have to check whether we are at an even step. `InstFail` confusingly returns `true`, as we do not care about branches that do not lead to matches.

{% highlight go %}
	case syntax.InstMatch:
		return idx == 0
	case syntax.InstFail:
		return true 
{% endhighlight %}

Now to one of my least favourite parts of Golang, copying nested maps. But here we go. Let's introduce a helper method, as when we branch for `InstAlt` we will need a separate ropy of the visited map for both branches.

{% highlight go %}
func copyVisited(visited map[int]map[uint32]bool) map[int]map[uint32]bool {
	n := make(map[int]map[uint32]bool)
	for k, v := range visited {
		if n[k] == nil {
			n[k] = make(map[uint32]bool)
		}
		for k2, v2 := range v {
			n[k][k2] = v2
		}
	}
	return n
}
{% endhighlight %}

OK, with that out of the way, we can tackle the `InstAlt` instructions.

{% highlight go %}
	case syntax.InstAlt:
		fallthrough
	case syntax.InstAltMatch:
		branchvisited := copyVisited(visited)
		return Even(p, i.Out, idx, visited) && Even(p, i.Arg, idx, branchvisited)
{% endhighlight %}

This one is different, as we want to make sure we can *only* match even length strings, so we need to `&&` the conditions, rather than `||` them.

Due diligence again.

{% highlight go %}
	default:
		panic("Invalid instruction.")
	}
	panic("Fell off the switch.")
{% endhighlight %}

If we stop for a second to see what we've built, it's obvious we've built a potential infinite loop. Let's rectify this. If we arrive back at an instruction that we've been before with the same `idx % 2`, we can just prune this branch and return `true` as it is exactly the same.

So let's prepend our switch statement with the following.

{% highlight go %}
	if visited[idx][pc] {
		return true
	}
{% endhighlight %}

And then, of course, we need to keep track of what we've seen before, which again involves one of my least favourite parts of Golang, nested maps.

{% highlight go %}
	if visited[idx] == nil {
		visited[idx] = make(map[uint32]bool)
	}
	visited[idx][pc] = true
{% endhighlight %}

And that's done! To see the whole code used in this post head of to this [Github Gist](https://gist.github.com/segfaulthunter/25a8fb5502ebddd413c00eada4ff18bf).

If you have made it this far, it is probably also worth noting that the Z3 SMT solver has a [Regex Theory](https://rise4fun.com/z3/tutorialcontent/sequences#h23). The last time I've played with it it was giving obviously incorrect answers, but that seems to have been rectified since. The nice thing about `regexp/syntax` is that it uses the same library your application uses if you write it in Golang, and can be used in programs with more predictable performance and fewer dependencies than Z3.
