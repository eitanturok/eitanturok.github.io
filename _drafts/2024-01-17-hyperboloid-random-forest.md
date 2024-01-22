---
layout: distill
title: Random Forest Algorithm in Hyperbolic Space
date: 2024-01-09 11:59:00-0400
description: I explain my paper "Tensor Ranks and the Fine-Grained Complexity of Dynamic Programming".
tags: comments
categories: explain-paper, hyperbolic-space, machine-learning
giscus_comments: true
related_posts: true

toc:
  - name: Background
  - name: The Paper
---


The random forest algorithm is one of the classic ML algorithms, partitioning Euclidean space into smaller and smaller subsets until each partition represents a single class. But what do you do when your data does not lie in Euclidean space? How do you partition this kind of space?

My paper <PAPER-TITLE> with <COAUTHOR-NAMEs> seeks to answer this question: how do you design an *efficient* random forest algorithm in hyperbolic space?

Here is the data from BLANK. We can try to partition it with the good old Euclidean random forest algorithm.

<INSERT-ANIMATION>

This algorithm poorly partitions the data. The Euclidean random forest algorithm creates linear splits in the space. But this data is not linearly separable. Moreover, it might be better captured by a different geometry: hyperbolic geometry.

In this post, we'll explain what kinds of data lie in  the motivation (what kinds of data lie in hyperbolic space and when you'd use this algorithm) and explain the algorithm itself.

