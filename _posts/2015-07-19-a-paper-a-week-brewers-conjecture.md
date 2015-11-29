---
title: "A Paper a Week-ish #11: Brewer’s Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services"
layout: post
date: 2015-07-19 22:00:00 BST
categories: apaw
---

This paper was more discovered than chosen. I decided to write this week-ish's post sitting in a train without Internet connection, so I scanned my Download folder for the random papers I have accumulated over the months. I discovered this paper, which seemed interesting and brief enough so I could finish reading it on the train.

It is, as is fairly obvious, about Brewer's conjecture. To be honest, I have never actually heard of this before (and I might just have mistaken it for Brouwer, the logician, back when I downloaded this paper, but I guess I will never know). It seems to state the fact that it is impossible to achieve consistency, availability and partition-tolerance at the same time.

The first section of this paper talks about these properties in a fully asynchronous system – that is a system where every agent must function only based on the messages it has received. Most statements made are fairly self-evident. The main theorem of this part states that if the network is partitioned into two or more parts, a write made in one of them will not be reflected in reads in others'. This isn't a very deep observation, in my opinion, as the side condition that every request be served eventually is obviously inconsistent with partition-resistant consistency. It cannot even be both available and consistent even in the case where all messages are delivered, because a node has no way of delaying the response to the client until a point where it knows it would have received all messages preceding the request by the client.

The second section is a bit more interesting. It analyzes these properties in a what it calls partially synchronous network. These are networks in which every node has a clock running at the same rate and there are bounds on network latency – mind the difference to systems having the absolute result of clocks synchronized to some degree, like TrueTime used in Google's Spanner. I am sure a physicist would tell me that having two clocks run at exactly the same rate is technically impossible, but we can probably get close enough for practical purposes (and this is a *model* anyway).

Anyway, in these networks the main theorem of the previous section obviously still holds. Even such a probably-physically-impossible system is incapable of magically bridging arbitrarily long network partitions in finite time. However, the variant where all messages are delivered does not hold, because assuming a central node that manages the system, a node can always serve a request by waiting for the appropriate amount of time before returning because there are bounds on the network latency (of course in practice there is not much point in having a distributed data-base managed completely by a central node).

A weaker form of consistency is then proposed, which is based on consistency while all messages are delivered, and a bound on re-convergence time after a interval in which not all messages have been delivered. This notion is formalized and then an algorithm that achieves this is shown.

Find the paper here: [Brewers Conjecture paper] [1].

[1]: https://github.com/papers-we-love/papers-we-love/blob/master/distributed_systems/brewers-conjecture.pdf "Brewers Conjecture Paper"
