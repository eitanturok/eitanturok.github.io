---
layout: distill
title: Can You Solve Dynamic Programming Problems Faster?
date: 2024-01-09 11:59:00-0400
description: We prove you can solve dynamic programming problems polynomially faster if you have a simple function for the cost of transitioning from one DP state to another.
tags: comments
categories: explain-paper dynamic-programming cs-theory
giscus_comments: true
related_posts: true

toc:
  - name: Background
  - name: The Paper
---

Dynamic programming (DP) is one of the most common algorithmic paradigms, used everywhere from DNA sequencing and financial modeling to matrix multiplication and shortest path algorithms. When one solves a problem using DP, a natural question arises: *is this the fastest algorithm to solve the problem?*

Recently, fine-grained complexity has been used to show that for many important problems, the standard DP algorithm *is* optimal. For example, researchers proved that for the edit distance problem there is no algorithm (whether or not it uses DP) that is faster than the standard DP algorithm by a polynomial factor (assuming $\text{SETH}$).

On the other hand, there are some notable examples where a natural DP formulation is *not* optimal. For example, researchers showed that for the polygon triangulation problem the standard DP algorithm results in an $O(n^3)$ time solution while geometric techniques lead to an $O(n \log n)$ time solution, a polynomial speedup.

So, we wonder:
> For which problems can we achieve a polynomial speedup over the standard DP algorithm? And when is the standard DP algorithm already optimal?

In our [latest work](https://arxiv.org/abs/2309.04683), we provide an answer to this question for certain kinds of DP problems, specifically for $k$-dimensional Least Weighted Subsequence ($k\text{D}\hspace{1mm}\text{LWS}$) DP problems.

We prove that achieving a polynomial speedup for $k\text{D}\hspace{1mm}\text{LWS}$ depends on the cost of transitioning from one DP state to another. This cost is expressed as a matrix in two dimensions or, analogously, a tensor in higher dimensions. If this cost tensor has constant rank, then it *is possible* to achieve a polynomial speedup but if the rank is slightly greater than constant, a polynomial speedup *is impossible*.

This is a beautiful result: if the cost tensor has a simple structure, i.e. constant rank, we can exploit it to solve the problem faster. But once the cost tensor becomes slightly more complex, i.e. super-constant rank, there is a naturally occurring barrier that makes it impossible to solve the problem any faster. This suggests there is some inherent difference between a constant and super-constant rank, a fundamental transition point where the computation goes from fast to slow.

Our main result is thus:
> For $k\text{D}\hspace{1mm}\text{LWS}$ we prove that a polynomial speedup over the standard DP algorithm is possible when the rank of the cost tensor is $O(1)$ but impossible when it is $2^{O(\log^* n)}$ or greater (assuming $\text{SETH}$).

This post explains what the $k\text{D}\hspace{1mm}\text{LWS}$ DP problem is, how we proved these results, and the problems that we can now solve faster.

Let's dive in.

# What is $k\text{D}\hspace{1mm}\text{LWS}$?

When most students learn about DP they think it is all about filling in DP tables. This is not true! The most important part of DP is finding the recurrence relation -- a recursive equation that gives a solution for the problem in terms of simpler sub-problems. This is the heart of dynamic programming and thus a natural way to characterize different DP problems.

$k\text{D}\hspace{1mm}\text{LWS}$ is a class of DP problems with a certain kind of recurrence relation. To develop a working intuition of $k\text{D}\hspace{1mm}\text{LWS}$, we will analyze the recurrence relations of three different DP problems and create a general recurrence relation that captures all of these problems -- this is the $k\text{D}\hspace{1mm}\text{LWS}$ recurrence relation!

## Longest Increasing Subsequence

To begin let's find the recurrence relations for the longest increasing subsequence ($\text{LIS}$) problem:

> Given an integer array $x_1, \dots, x_n$,
> return the length of the longest strictly increasing subsequence in this array.

If our array is $[10,9,2,5,3,7,101,18]$, our $\text{LIS}$ is $[2,3,7,101]$. So we'd return $4$.

The recurrence relation for this problem is
$$
dp[j]
    =
    \begin{cases}
        0 & \text{if $j == 0$} \\\\
        \max_{0 \leq i < j} dp[i] + \mathbb{1}[x_j > x_i] & \text{otherwise.} \\
    \end{cases}
$$
Here, $dp[j]$ is the $\text{LIS}$ of $x_1, \dots, x_j$ and  $\mathbb{1}[x_j > x_i]$ is an indicator variable that returns one when $x_j > x_i$ and zero otherwise. This recurrence relation shows that we can use the already-computed LIS of a shorter sequence, $dp[i]$, to compute the LIS of our sequence, $dp[j]$.

Intuitively, to compute the LIS of $x_1, \dots, x_j$ we take a look at the LIS of all shorter sequences $x_1, \dots, x_i$ where $i < j$ and if the last term of this shorter sequence, $x_i$, is greater than $x_j$.

if $x_j > x_i$, then we can add $x_j$ to the subsequence that ends at $x_i$, thus increasing the length of $dp[i]$ by one. This occurs when $\mathbb{1}[x_j > x_i]$ evaluates to one. But


$dp[j]$ is the maximum over all previous

Let's look at a previous sequence, $dp[i]$ where $i < j$.

If we look at a previous LIS, $dp[i]$, and our current value $x_j$ is greater than the end of that LIS.





we can look at all the numbers that come before $x_j$ and if the sub


If we've already computed $dp[i]$, the LIS of $x_1, \dots, x_i$ where $x_i$ comes before $x_j$, then.

Inteiteivly, if we've already compute the LIS of a shorter subsequence, then we can add

Inutitevily, we can take a look at a LIS that ends at some $x_i$ before $x_j$ and if  $x_j$ is greater than $x_i$ than

We can take a look at all the $x_i$ that come before $x_j$ and their $\text{LIS}$. If


we can look at all the numbers that come before $x_j$ and if the sub

Let's think about what happens if $x_j$ is greater than one of the numbers that comes before it, call it $x_i$.

* If $x_j > x_i$, we can add $x_j$ to the longest increasing subsequence ending at $x_i$, $dp[i]$. Because $x_j$ is greater than $x_i$, the sequence will still be strictly increasing. So $dp[j] = dp[i] + 1$.
* If $x_j \leq x_i$, we cannot add $x_j$ to the longest increasing subsequence ending at $x_i$. So we can just make longest increasing subsequence of $x_1, \dots, x_j$ not include $x_j$ and instead be the longest increasing subsequence of $x_i$, $dp[i]$. So, $dp[j] = dp[i]$.

To compute $dp[j]$, the longest increasing subsequence of $x_1, \dots, x_j$, we should take a look at all $x_i$ that come before $x_j$ and apply the rules above. Formally, we can express this as
$$
  dp[j]
  =
  \max_{1 \leq i < j}
  \begin{cases}
    dp[i] + 1 & \text{if $x_j > x_i$}
    \\
    dp[i] & \text{if $x_j \leq x_i$}
  \end{cases}
$$
This can be written more succinctly as
$$
  dp[j]
  =
  \max_{1 \leq i < j}
  dp[i] + \mathbb{1}[x_j > x_i]
$$
where $\mathbb{1}[b]$ is an indicator variable that equals $1$ if $b$ is True and 0 if $b$ is false.


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


We prove that for $k\text{D}\hspace{1mm}\text{LWS}$ problems a polynomial speedup is possible whenever the cost to transition from one DP state to another is constant. The moment the cost to go from one DP state to another becomes slightly super-constant, it is impossible to achieve a polynomial speedup.
