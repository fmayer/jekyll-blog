---
title: "A Paper a Week #5: Temporal Synchronization of multiple Audio Signals"
layout: post
date: 2015-03-31 01:00:00 BST
categories: apaw
---
This week's paper was published by Google and concerns itself with finding methods of combining overlapping audio streams that were recorded from different locations, which is often the case, for instance in concert recordings. It suggests use-cases such as 3D scene reconstruction, which could be done by combining videos that were first synchronized using their audio stream.

The paper starts by explaining that existing techniques for matching audio streams basically work by finding a locally good match and then trying to expand that match further. This local approach is contrasted by a global one presented here.

In fact, the paper shows two different implementations of what is roughly the same general idea. The first method proposed is, at least to me, very appealing in its simplicity and elegance. It goes as follows: first, an offset is calculated between every pair of input signals.  This can be done in several ways, one of which is maximizing the cross-covariance (think of it as a measure of correlation) of “spectral flatness”, which is described as being “the variation of tonality over time. It is defined by the ratio of the geometric mean and arithmetic mean of the frequency domain coefficients”.

Then, a complete graph containing each signal as nodes is constructed, and a measure of “how good” the matching between any two signals is (according to some metric) is used as the edge weight between the two. On this, MST (minimum spanning tree) is applied to find a global solution.

The second method (very roughly) proposes a probabilistic distribution for the offset between two signals, based on the offsets calculated as above. Then, a method called Belief Propagation (which I hope I can cover in greater detail at a later point in this series) is used to calculate the marginals of that distribution. Because this not necessarily converges to a global solution, different base hypothesis have to be tried. I find the notation in this part of the paper a bit confusing and I had to make a few assumptions about what they are trying to say when reading it (so I might be summarizing that incorrectly!).

Then follows an experimental evaluations that shows both methods perform well.

Without further ado, here is the paper: [PDF] [1]. 

[1]: http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/42193.pdf "Paper"
