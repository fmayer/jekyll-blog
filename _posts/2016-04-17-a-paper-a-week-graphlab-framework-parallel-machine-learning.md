---
title: "A Paper a Week-ish #14: GraphLab: A New Framework for Parallel Machine Learning"
layout: post
date: 2016-04-17 01:10:00 BST
categories: apaw
---

It has been a  long time since I last posted one of those, apologies! I have been fairly busy with various things and the time I had for this blog I spent on non paper-related posts to mix things up a bit.

The paper I read for this post is about GraphLab which is a framework for expressing parallel computations based on a graph model that allows to exploit the sparse structure of machine learning algorithms, that is to say that steps of the computation only operate on a subset of the state. In this graph data is associated with each vertex and edge, and a user-supplied update function is used to compute new values. This update function can only access and modify data in the neighbourhood of the node, that is the data of the node, its adjacent edges and vertices.

In addition to the data stored in the graph, there is a shared data table, which contains data which, as the name suggests, can be accessed by all applications of the update function – it can not be modified by them though. To modify data in the shared data table, a fold function that iteratively calculates a value from the data in the vertices and the previous value is specified by the user. Optionally, the user can scale the result of the fold or supply functions to merge results of different fold operations.

By imposing the restriction that the update function only operate on the neighbourhood of a node, the graph encodes the data dependencies between different parts of the computation and allows to determine which updates can be applied concurrently. GraphLab offers different consistency models: *full consistency* makes sure no data can be accessed concurrently, *edge consistency* ensures no two updates that access shared edges are executed at the same time, and *vertex consistency* only ensures only one update is applied to a particular node at a given time. From all I can see, vertex consistency only works correctly for trivially parallelizable problems, that is problems consisting of independent threads of computation that do not interact.

A scheduler determines the sequence of sets of updates that should appear to be applied to the graph at the same time. So, for example, if the sequence is [\\( A \\), \\( B \\)] where \\( A \\) and \\( B \\) are sets of functions applied to vertices, it is ensured that (modulo the consistency model) it appears that all the operations in \\( A \\) were applied to the graph at the same time, then all operations in \\( B \\). Different machine learning algorithms require different schedulers.

Note how I said “appear to be applied”, which means that this is not necessarily how they are actually applied. As an example, if vertex consistency is used, and \\( A = \\{v_1, v_2\\} \\) and \\( B = \\{v_3, v_4\\} \\), then \\( f(v_1) \\), \\( f(v_2) \\), \\( f(v_3) \\), \\( f(v_4) \\) can be evaluated at the same time, while it still appears that \\( B \\) was applied after \\( A \\). There is a very strong connection to how databases handle transactions – if the update function does not modify values that are not exclusive to it by the consistency model, we get serializability – that is, the parallel execution produces the same result as some sequential one. Database transactions are also designed to be serializable, and two of them can commit concurrently if they involve disjuct sets of rows.

The paper concludes with case studies of machine learning tasks implemented in GraphLab, which I will skip in this post.

Find the paper here: [GraphLab: A New Framework For Parallel Machine Learning] [1].

[1]: http://arxiv.org/pdf/1006.4990v1.pdf "GraphLab: A New Framework For Parallel Machine Learning"
