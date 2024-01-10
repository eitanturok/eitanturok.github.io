---
layout: distill
title: Can You Solve Dynamic Programming Problems Faster?
date: 2024-01-09 11:59:00-0400
description: I explain my paper "Tensor Ranks and the Fine-Grained Complexity of Dynamic Programming".
tags: comments
categories: explain-paper dynamic-programming cs-theory
giscus_comments: true
related_posts: true

toc:
  - name: Background
  - name: The Paper
---

I want to explain my recent paper with the amazing Josh Alman, and Hantao Yu: [Tensor Ranks and the Fine-Grained Complexity of Dynamic Programming](https://arxiv.org/abs/2309.04683).

In our paper, we set out to answer the question: **can you solve dynamic programming problems faster?**

Let's dive in.

## Background

Dynamic programming (DP) is a technique to solve discrete optimization problems. It works by breaking down a problem into simpler sub-problems and storing the solution to each sub-problem so that each sub-problem is only solved once. DP appears everywhere from computational biology to matrix multiplication.

Consider the famous DP problem, the longest strictly increasing subsequence problem:

> Given an integer array $$x_1, \dots, x_n$$,
> return the length of the longest strictly increasing subsequence.

If our array is $$[10,9,2,5,3,7,101,18]$$, our longest increasing subsequence is $$[2,3,7,101]$$. So we'd return $$4$$.

To find $$dp[j]$$, the longest strictly increasing subsequence of $$x_1, \dots, x_j$$, we see if $$x_j$$ is greater than one of the numbers that come before it, $$x_i$$. If $$x_j > x_i$$, then we can increase the length of the subsequence by one. make $$dp[j]$$ the

Formally, we can define the recurrence relationship as
$$
    dp[j]
    =
    \begin{cases}
        0 & \text{if $j == 0$} \\\\
        \min_{0 \leq i < j} dp[i] + \mathbb{1}[x_i < x_j] & \text{otherwise.} \\
    \end{cases}
$$

