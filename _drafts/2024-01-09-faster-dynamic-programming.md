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

When most students learn about dynamic programming (DP) they think it is all about filling in tables. This is not true! The most important part of DP is finding the recurrence relation -- a recursive equation that gives a solution for the problem in terms of simpler sub-problems. This is the heart of dynamic programming.

In our paper, we proved that if a DP problem has a certain kinds of recurrence relations we can **solve it polynomially faster**. The takeaway: how a DP problem breaks down into its smaller sub-problems affects how quickly we can solve it.


Let's dive in.

## Recurrence Relations

Let's find the recurrence relations for two DP problems. First, the longest increasing subsequence problem:

> Given an integer array $x_1, \dots, x_n$,
> return the length of the longest strictly increasing subsequence in this array.

If our array is $[10,9,2,5,3,7,101,18]$, our longest increasing subsequence is $[2,3,7,101]$. So we'd return $4$.

Suppose we wish to compute $dp[j]$, the longest strictly increasing subsequence of $x_1, \dots, x_j$. We can see if $x_j$ is greater than one of the numbers that come before it, call it $x_i$.

* If $x_j > x_i$, we can add $x_j$ to the longest increasing subsequence ending at $x_i$, $dp[i]$. Because $x_j$ is greater than $x_i$, the sequence will still be strictly increasing. So $dp[j] = dp[i] + 1$.
* If $x_j \leq x_i$, we cannot add $x_j$ to the longest increasing subsequence ending at $x_i$. So we can just make longest increasing subsequence of $x_1, \dots, x_j$ not include $x_j$ and be the longest increasing subsequence of $x_i$, $dp[i]$. So, $dp[j] = dp[i]$.

Formally, we can use this intuition to define the recurrence relationship

$$
dp[j]
    =
    \begin{cases}
        0 & \text{if $j == 0$} \\\\
        \max_{0 \leq i < j} dp[i] + \mathbb{1}[x_j > x_i] & \text{otherwise.} \\
    \end{cases}
$$

where $\mathbb{1}[b]$ is an indicator variable that equals $1$ if $b$ is True and 0 if $b$ is false.

Let's consider another DP problem:
