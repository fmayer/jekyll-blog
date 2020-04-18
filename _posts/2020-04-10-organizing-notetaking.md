---
title: "Automatic Org-Mode"
layout: post
date: 2020-04-16 00:40:00 CEST
---
With the unprecedented en vogue[^1] right now, I have taken the unprecedented
step of trying out Emacs after having been a loyal Vim user for years. This is
not going to be another charge in the endless Editor War[^2]. Rather more
uncontroversially, this is going to focus on note-taking and how technology can
(and cannot) help.

If how this relates to Emacs is a mystery, just trust me that this will make
sense. But first to more elementary things.

# Pen and Paper

On my desk at work[^3] I have a notebook (of nice dotted paper[^4]) and a fountain
pen[^5]. I use them to take notes in meetings, while working, draw diagrams,
write TODO lists. I clearly mark items that are tasks, and tick them off once
done. Once a notebook is full, I scan it for archival. 

It works fairly well, but there are some limitations. An obvious drawback is
the lack of searchability[^6]. This is not helped by that the way I keep the
notebook, it is sorted by time; this also makes it hard to amend or extend
previous entries. Legibility occasionally is an issue, especially when I take
notes in a hurry, for instance in a meeting.

Paper and fountain pens are technologies *invented 2000 and 200 years ago*.
With all our fancy tools of the supposed information age, can we do better?

# Maybe?
This has been my main note-taking system for about three years now. Before that,
calling my note-taking at work a system would have been an euphemism[^7]. How
come I, as a computer engineer with terrible handwriting, still use a physical
piece of paper?

Paper, it turns out, is quite good at what it does. No one, to my knowledge,
has been able to reproduce the freedom and convenience of writing on paper. It
is trivial to interleave drawings and text, you can split the page into as many
sections or columns as you want. I regularly take some space I have free on the
right to designate to some related thought that does not really fit into the
main narrative.

Does this mean digital notetaking is a lost cause?

# Beating at its own game 

Some programs, like [OneNote], have attempted to replicate the experience of
taking notes on paper. I have tried using OneNote for a bit, and would say
it has not succeeded. But that, I think, is also the wrong approach: you should
not try to beat paper at its own game. Rather, you should focus on what digital
technologies are good at.

When you do that, paper's strength also becomes a weakness. From an information
theory point of view, a hand-written piece of paper holds a lot more bits of
information than the same text as an ASCII text file. Sometimes this lets you
express more exactly what you want, but mostly it is unnecessary. When writing
a word, chances are the exact ways you write the letters is not relevant and
mostly determined by chance.

Having less noise in the data, as you have in digital notes, makes it easier to
automatically process them. You can search within them, automatically find
relationships, easily replicate, among other things.

# Relationships

At least for me, one of the most important part of notes is linking different
kinds of artifacts. As I work in software engineering, these artifacts might be
a bug, a pull request, some mathematical concept or algorithm, things like that.
A bug I am fixing might relate to an algorithm, and the fix is done as a pull
request. In addition to all of this, I might want to add my own comments.

This is a use-case very poorly served by paper notes, as you'd have to
physically bring these things to the same place. This is why, so far, I
have mostly used notes as a *thinking aid*, rather than a *memory aid*. I would
take down notes more as a form of [rubber ducking] than anticipating I will
revisit them later – it was often prohibitively hard to find them again at the
right time.

But if I could revisit those notes that were intended as *thinking aid* later,
it would help me remember why certain decisions were taken. If this was
fairly close to when it was written, this has already proven useful.
I might not remember why I had dismissed an alternative approach, but sometimes
my notes do. They would say something like "~~overload flux capacitor -> bad idea,
brings us back to dinosaur age.~~" Then I can immediately dismiss that approach
again, without wasting cognitive effort on it again.

# Roam

The [Roam] note taking app has a simple but really cool way of dealing with
those relationships between concepts. Every time you refer to another concept
in a note, you get a back reference from the page about that concept. So, if
I write a note about, say, move semantics in C++, and mention C++ in that page,
the C++ page will show me a back reference to "move semantics". It also lets
you see these relationships in a graph.

That goes some way of making notes more discoverable later, as you automatically
get a sort of index built. But what I think is missing there is how this
interacts with artifacts that are not notes. Take for instance this blog post.
Say you have a great idea that is related to it, and you want to remember it.
You could write a note that has a link to it. Wouldn't it then be nice to have
a backreference *from this post*? Without that, how likely are you to remember
that you had even written that note in a year's time?

# OrgMode

[OrgMode] is a package for Emacs that is designed for note-taking. It is
superficially similar to Markdown: you can have headings, lists, tables, etc.
It is somewhat more sophisticated than that, even as a text processor: you can
insert blocks of code that can be executed in line, and produce data to be used
by another block of code. But this is somewhat beside the point here.

What's more interesting are its cross-reference abilities. You can put TODO
items with deadlines in your notes, and then have OrgMode generate an agenda
(from all your files). You can tag your headings (and files) and then search
according to that.

# Org-Roam
[Org-Roam] takes the ideas of Roam (mainly backreferences) and adapts them
to OrgMode. The main feature it allows is to display the back-references to
the current file. It also allows to to generate a graph of your references
using GraphViz, but that is much less pretty and useful than the interactive
graph Roam-proper offers.

That's nice, but it's still a walled garden. Back-references can only be
created from other notes (specifically ones within your Org-Roam directory).
But this time, the walled-garden is within an open-source software, so maybe
we can do something to jump the wall.

# Instant-Org
These ideas have led me to write a [simple prototype] of how a system that can
jump that wall might look. The idea is very simple: whenever we switch to a new
window[^8], we analyze whether the currently focused window relates to any
entity that we have a note about: for instance, if we switch to a browser
window, I parse the title to see whether we are currently looking at a bug,
a pull request, a GitHub repository (sadly, each of those is a hardcoded
regex). If it's a terminal window, I look at the current working directory
to see if I have any notes about that.

It then checks the Org-Roam database to see whether we have a note about that
entity, and if we do, automatically opens it in the background. 
This, being a prototype, is still an approximation on what I ultimately
envision: we can link notes to entities outside of the walled garden this way,
but we don't collect back-references if other notes link to the entity, not
the associated note. I recorded a [short demo] on how that looks.

Once I get more comfortable with elisp, I will attempt to teach Org-Roam to
handle more sorts of links. That would make the resulting graph much richer and
more useful: whenever you visit a website, you can see all the time you
referred to it in any note.

Is this better than paper notes? Maybe. But only once the knowledge graph has
grown enough for the overhead in taking notes to pay dividends.

# Endnotes

[^1]: If anyone happens to read this in the future: this was written at the
      time of the global COVID-19 lockdowns.

[^2]: Vim keybindings are better.

[^3]: If you have never tried dotted paper, do it. It's like squared paper with
      less visual clutter. I like the [Rhodia A4 notebook](
      https://www.amazon.co.uk/gp/product/B00BCH03Z2).

[^4]: Which I, inconveniently, have no access to right now.

[^5]: JinHao make [insanely affordable](https://www.amazon.co.uk/dp/B07PQ2RXF7)
      ones. I promise I do not get paid for these Amazon links.
      
[^6]: There will need to be some more breakthroughs in machine learning before
      I can subject my handwriting to OCR software, and until then I'll have to
      continue to search manually.

[^7]: University is easier. Things can be neatly arranged by subjects.

[^8]: Actually, I just poll the active window every 100 ms to see if it changed.

[rubber ducking]: https://en.wikipedia.org/wiki/Rubber_duck_debugging

[Roam]: https://roamresearch.com/

[OneNote]: https://onenote.com

[OrgMode]: https://orgmode.org

[Org-Roam]: https://org-roam.readthedocs.io/

[short demo]: https://drive.google.com/file/d/19v5u2AmO27IMI-H9mx-Q0TqKmzrgzVGT/view

[simple prototype]: https://gist.github.com/segfaulthunter/f56ec6b8fd579b5bf1a0298a0f8cc175
