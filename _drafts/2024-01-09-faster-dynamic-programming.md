---
layout: distill
title: Can You Solve Dynamic Programming Problems Faster?
date: 2024-01-09 11:59:00-0400
description: We prove you can solve dynamic programming problems polynomially faster if you have a simple function for the cost of transitioning from one DP state to another.
tags: comments
categories: explain-paper dynamic-programming cs-theory algorithms complexity
giscus_comments: true
related_posts: true
bibliography: 2018-12-22-distill.bib
toc:
    - name: Introduction
    - name: LWS, Dynamic Programming in One Dimension
    - name: Solving LWS Faster
---

# Introduction

Dynamic programming (DP) is one of the most common algorithmic paradigms, used everywhere from DNA sequencing and financial modeling to matrix multiplication and shortest path algorithms. When one solves a problem using DP, a natural question arises: *is this the fastest algorithm to solve the problem?*

Recently, fine-grained complexity has been used to show that for many important problems, the standard DP algorithm *is* optimal. For example, researchers proved that for the [edit distance](https://leetcode.com/problems/edit-distance/description/) problem, there is no algorithm (whether or not it uses DP) that is faster than the standard DP algorithm by a polynomial factor (assuming $\text{SETH}$).

On the other hand, there are some notable examples where a natural DP formulation is *not* optimal. For example, researchers showed that for the [polygon triangulation](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/description/) problem, the standard DP algorithm results in an $O(n^3)$ time solution while geometric techniques lead to an $O(n \log n)$ time solution, a polynomial speedup.

So, we wonder:

> For which problems can we achieve a polynomial speedup over the standard DP algorithm? And when is the standard DP algorithm already optimal?

In our [latest work](https://arxiv.org/abs/2309.04683), we provide an answer to this question for certain kinds of DP problems, specifically for $k$-dimensional Least Weight Subsequence ($k\text{D}\hspace{1mm}\text{LWS}$) DP problems.

We prove that achieving a polynomial speedup for $k\text{D}\hspace{1mm}\text{LWS}$ depends on the cost of transitioning from one DP state to another. This cost is expressed as a matrix in two dimensions or, analogously, a tensor in higher dimensions. If this cost tensor has constant rank, then it *is possible* to achieve a polynomial speedup. But if the rank is slightly greater than constant, a polynomial speedup *is impossible* (assuming $\text{SETH}$).

This is a beautiful result: if the cost tensor has a simple structure, i.e. constant rank, we can exploit it to solve the problem faster. But once the cost tensor becomes slightly more complex, i.e. super-constant rank, there is a naturally occurring barrier that makes it impossible to solve the problem any faster. This suggests there is some inherent difference between a constant and super-constant rank, a fundamental transition point where the computation goes from fast to slow.

Informally, our main result is:

> For $k\text{D}\hspace{1mm}\text{LWS}$ DP problems, we prove that a polynomial speedup over the standard DP algorithm is possible when the rank of the cost tensor is $O(1)$ but impossible when it is $2^{O(\log^* n)}$ or greater (assuming $\text{SETH}$).

This post explains what the $k\text{D}\hspace{1mm}\text{LWS}$ DP problem is, how we proved these results, and the problems that we can now solve faster.

Let's dive in.

# LWS, Dynamic Programming in One Dimension

When most students learn about DP they think it is all about filling in DP tables. This is not true! The most important part of DP is finding the recurrence relation -- a recursive equation that gives a solution for the problem in terms of simpler sub-problems. This is the heart of dynamic programming and thus a natural way to characterize different DP problems.

$k\text{D}\hspace{1mm}\text{LWS}$ is a class of DP problems with a certain kind of recurrence relation. To develop a working intuition of $k\text{D}\hspace{1mm}\text{LWS}$, we will first focus on the one dimensional version of $k\text{D}\hspace{1mm}\text{LWS}$: $\text{LWS}$. In this section we will analyze the recurrence relations of three different DP problems and create a general recurrence relation that captures all of these problems -- this will be the $\text{LWS}$ recurrence relation!

## Longest Increasing Subsequence Problem

### Definition

To begin, let's find the recurrence relation for the [longest increasing subsequence](https://leetcode.com/problems/longest-increasing-subsequence/description/) ($\text{LIS}$) problem:

> Given an integer array $X = [x_1, \dots, x_n]$, return the length of the longest strictly increasing subsequence in this array.

If our array is $[10,9,2,5,3,7,101,18]$, our $\text{LIS}$ is $[2,3,7,101]$. So we'd return $4$.

### Recurrence Relation

The recurrence relation for $\text{LIS}$ is

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \max_{0 \leq i < j} dp[i] + \mathbb{1}[x_j > x_i]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $dp[j]$ is the $\text{LIS}$ of $x_1, \dots, x_j$ and $\mathbb{1}[x_j > x_i]$ is an indicator variable that returns one when $x_j > x_i$ and zero otherwise. The solution to the $\text{LIS}$ problem is given by $dp[n]$. Now let's break down the two cases.

**Base Case $j == 0$:**

Using the above definition of $dp[j]$, $dp[0]$ is the $\text{LIS}$ of $x_1, \dots, x_0$. However, our sequence starts at $x_1$, not $x_0$; our sequence $x_1, \dots, x_0$ is undefined. The $\text{LIS}$ of this sequence is non-existent and so the length of this $\text{LIS}$ must be zero. Thus, $dp[j] == 0$.

**Otherwise:**

Here $j > 0$, meaning we have one or more elements in our sequence $x_1, \dots, x_j$ for which we are computing the $\text{LIS}$. In this case, we use $dp[i]$, the already-computed $\text{LIS}$ values of a shorter sequence $x_1, \dots, x_i$, to compute $dp[j]$, the $\text{LIS}$ of our longer sequence $x_1, \dots, x_j$. We compare the last element of the shorter sequence, $x_i$, to the last element of our longer sequence, $x_j$:

* If $x_j > x_i$, we can add $x_j$ to the $\text{LIS}$ ending at $x_i$, $dp[i]$. Because $x_j$ is greater than $x_i$, the sequence will still be strictly increasing. This will increase the length of the sequence by one, so $dp[j] = dp[i] + 1$.

* If $x_j \leq x_i$, we cannot add $x_j$ to the $\text{LIS}$ ending at $x_i$. So we can just have the $\text{LIS}$ of $x_1, \dots, x_j$ not include $x_j$ and instead be the longest increasing subsequence of $x_i$, $dp[i]$. So we write, $dp[j] = dp[i] + 0$.

These two cases are succinctly captured by the expression $\mathbb{1}[x_j > x_i]$.

### Reformat the Recurrence Relation

Now we will reformat the $\text{LIS}$ recurrence relation so it more closely resembles the $\text{LWS}$ recurrence relation (more details about this later). We previously defined $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \max_{0 \leq i < j} dp[i] + \mathbb{1}[x_j > x_i]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

Now we will reformat $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $w[i, j]$ is an $n \times n$ matrix defined as

$$
w[i, j]
    =
    \begin{cases}
        -1
        &
        \text{if $x_j > x_i$}
        \\
        0
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

Notice that $w[i, j]$ equals negative one when $x_i$ can be added to a subsequence which ends in $x_j$ , thus increasing the length of a strictly increasing subsequence by $1$. Since $\text{LIS}$ is a maximization problem and $\text{LWS}$ is a minimization problem, the weights are $-1$, not $1$, and the solution is given by $−T[n]$.

## Coin Change Problem

### Definition

Next, let's find the recurrence relation for the [coin change](https://leetcode.com/problems/coin-change/description/) ($\text{CC}$) problem:

> Given an integer array of $m$ coins $C = [c_1, \dots, c_m]$ where $c_i$ is a coin worth $c_i$ cents, return the fewest number of coins needed to make $n$ cents. (Assume you have an infinite number of each kind of coin.)

If we have coins $[1, 2, 5]$ and want to make a total of $n=11$ cents, we would return $3$ because $3$ coins -- $5, 5, 1$ -- is the smallest number of coins needed to make $11$ cents.

### Recurrence Relation

The recurrence relation for this problem is

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \infty
        &
        \text{if $j < 0$}
        \\
        \min_{1 \leq i \leq m}
        dp[j-c_i] + 1
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $dp[j]$ is the minimum number of coins needed to make $j$ cents. The solution to the $\text{CC}$ problem is given by $dp[n]$. If $dp[n] == \infty$, then it is not possible to make $n$ cents using the coins in array $C$. Now, let's break down the three cases:

**Base case $j == 0$:**

The minimum number of coins to make $j = 0$ cents is $0$ because you need $0$ coins to make $0$ cents.

**Base case $j < 0$:**

The minimum number of coins needed to make a negative number of cents, i.e. $j < 0$, is undefined. And because this is a minimization problem, we set this to $\infty$ to ensure this choice is not chosen.

**Otherwise:**

Here, we want to find $dp[j]$, the minimum number of coins needed to make $j$ cents. So imagine we select a coin worth $c_i$ cents. Then we need to know the minimum number of coins needed to make the remaining $j-c_i$ cents, i.e. we need to know $dp[j-c_i]$. We add the $+1$ because by using the coin $c_i$, we've increased the number of coins we've used by one.

### Reformat the Recurrence Relation

Now we will reformat the $\text{CC}$ recurrence relation so it more closely resembles the $\text{LWS}$ recurrence relation (more details about this later). Previously, we defined $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \infty
        &
        \text{if $j < 0$}
        \\
        \min_{1 \leq i \leq m}
        dp[j-c_i] + 1
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

Now we will reformat $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $w[i, j]$ is an $n \times n$ matrix defined as

$$
w[i, j]
    =
    \begin{cases}
        1
        &
        \text{if $j-i \in C$}
        \\
        \infty
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

and where solution to $\text{CC}$ is given by $dp[n]$.

The condition $j-i \in C$ means we add a one if there a coin worth $j-i$ cents in our array of coins $C$ because we can only go from having $j$ cents to having $i$ cents if a coin worth $j-i$ cents exist. If no coin worth $j - i$ cents exists, we cannot go from $j$ cents to $i$ cents and thus assign an $\infty$ value to avoid this kind of case.

> In the original recurrence relation, we had a table of size $n$ and to compute each value, looped over all the $m$ coins, giving us a run time of $O(nm)$. In this recurrence relation, we have a table of size $n$ and loop over all values less than $j$, giving us a worst case run time of $O(n^2)$. Does this not change the time complexity from $O(nm)$ to $O(n^2)$? Does this make things worse? Do we always assume $m << n$?

## Airplane Refueling Problem

### Definition

Lastly, let's find the recurrence relation for the [airplane refueling](https://leetcode.com/problems/minimum-number-of-refueling-stops/description/) ($\text{AR}$) problem:

> Suppose an airplane is flying from source $x_0$ to destination $x_n$. Given a list of optional refueling stations at positions $X = [x_1, \dots, x_n]$, return the minimum cost way to fly from $x_0$ to $x_n$ where it costs $([x_j - x_i] - l)^2$ to fly from airport $x_i$ to airport $x_j$ where $l$ is the optimal distance traveled for fuel efficiency reasons.
>
>
> More specifically:
> * Let $0 = x_0 < x_1 < \dots < x_{n-1} < x_n$ so that the source is at position $0$ and the airports are sorted by their distance from the source.
> * Assume the airports are located along a straight line such that distance between airport $x_j$ and airport $x_i$ is $x_i - x_j$.

If the optional refueling airports are located at positions $X=[1, 5, 7, 10]$ and we prefer to travel $l=3$ miles at a time, the minimum cost of traveling from the source $x_0 = 0$ to $x_4 = 10$ is ?.

### Recurrence Relation

The recurrence relation for $\text{AR}$ is

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \max_{1 \leq i < j} dp[i] + ([x_j - x_i] - l)^2
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $dp[j]$ is the minimum cost way to fly from source $x_0$ to airport $x_j$. The solution to the $\text{AR}$ problem is given by $dp[n]$. Let's break down the two cases:

**Base Case $j == 0$:**

When $j == 0$, we are located at airport $x_0$ and it costs $0$ to go from source $x_0$ to destination $x_j = x_0$.

**Otherwise:**

To compute $dp[j]$, the cheapest way of flying from $x_0$ to $x_j$, we take the cheapest combination of 
* the cost of flying from $x_i$ to $x_j$, $([x_j - x_i] - l)^2$
* plus the cheapest possible cost of flying from $x_0$ to $x_i$, $dp[i]$.

### Reformat the Recurrence Relation

Now we will reformat the $\text{CC}$ recurrence relation so it more closely resembles the $\text{LWS}$ recurrence relation (more details about this later). Previously, we defined $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \max_{1 \leq i < j} dp[i] + ([x_j - x_i] - l)^2
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

Now we will reformat $dp[j]$ as

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where $w[i, j]$ is an $n \times n$ matrix defined as

$$
    w[i, j]
    =
    ([x_j - x_i] - l)^2.
$$

The cost $w[i, j]$ is the simply the cost of going from airport $x_i$ to airport $x_j$.

## Putting It All Together

Let's take another look at our three reformatted recurrence relations:

**Longest Increasing Subsequence $\text{LIS}$:**

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
    \ \ \ \ \ \ \ \ 
    \text{ where }
    w[i, j]
    =
    \begin{cases}
        -1
        &
        \text{if $x_j > x_i$}
        \\
        0
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

**Coin Change $\text{CC}$:**

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
    \ \ \ \ \ \ \ \ 
    \text{ where }
    w[i, j]
    =
    \begin{cases}
        1
        &
        \text{if $j-i \in C$}
        \\
        \infty
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

**Airplane Refueling $\text{AR}$:**

$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$} \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
    \ \ \ \ \ \ \ \ 
    \text{ where }
    w[i, j]
    =
    ([x_j - x_i] - l)^2
$$

These three recurrence relations are all the exact same, except for their cost matrices $w$. Interesting -- what does this mean?

In 1985 two researchers Daniel Hirschberg and Lawrence Larmore noticed the very same thing: numerous DP problems have identical recurrence relations and differ only in the cost matrix $w$. Could it be that many famous DP problems are the exact same except for $w[i, j]$, the cost of transitioning from one item $x_i$ to another $x_j$?

Hirschberg and Larmore noticed that this recurrence relation has a fundamental structure: given a sequence of items, this recurrence relation tries to find the *subsequence* of items which have the *minimum weight* when traversing the cost matrix $w$ across our recursive calls. For instance:

* $\text{LIS}$ wants the longest subsequence of integers that is strictly increasing.
* $\text{CC}$ wants the smallest subsequence of coins that equal $n$ cents.
* $\text{AR}$ wants the cheapest subsequence of airports that allow us to fly from our source to our destination.
  
All of these DP problems want to find the minimum weight subsequence of items. Hirschberg and Larmore thus named this DP problem the *least weight subsequence* problem or $\text{LWS}$.

Formally, $\text{LWS}$ is defined as follows:

> Given $n$ items $X = [x_1, \dots, x_n]$ and a $n \times n$ cost matrix $w$ where $w[i, j]$ depends on $x_i, x_j$, compute the value $dp[n]$ given the recurrence relation
>
>$$
dp[j]
    =
    \begin{cases}
        0
        &
        \text{if $j == 0$}
        \\
        \min_{0 \leq i < j} dp[i] + w[i, j]
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

In this recurrence relation, $dp[j]$ is the cheapest way of "getting" to item $x_j$. To compute $dp[j]$ we find the minimum over all previously computed values ($dp[i]$) plus the cost of transitioning from item $x_i$ to item $x_j$ ($w[i, j]$).

Here is where the *subsequence* part of $\text{LWS}$ comes into play: to compute $dp[n]$, we first find the item $x_{i_1}$ with the minimum $dp[i_1] + w[i_1, n]$ value and include it in the our final answer's subsequence. Then to compute $dp[i_1]$ we find the next item $x_{i_2}$ that results in the minimum $dp[i_2] + w[i_2, i_1]$ value and include it in our final answer's subsequence. We repeat the pattern and end up with a subsequence of items $x_{i_1}, x_{i_2}, \dots$ that result in the minimum cost value for $dp[n]$. This subsequence $x_{i_1}, x_{i_2}, \dots$ is our least weight *subsequence*.

Taking a step back, Hirschberg and Larmore noticed that by appropriately setting the cost matrix $w$, they can formulate many famous DP problems as instances of $\text{LWS}$. Indeed, by defining the cost matrix $w$ carefully we can turn the general $\text{LWS}$ recurrence relation into the recurrence relation for $\text{LIS}$, $\text{CC}$, or $\text{AR}$! There are dozens of other important DP problems that can be expressed through the $\text{LWS}$ recurrence relation. Formally, $\text{LWS}$ defines a general class of recurrence relations. Ultimately, Hirschberg and Larmore found that $\text{LWS}$ was a fundamental problem in DP where numerous DP problems have the same underlying structure and are simply $\text{LWS}$ problems in disguise. What a beautiful result...

# Solving LWS Faster

Straightforward DP solves the $\text{LWS}$ problem in $O(n^2)$ time. This is because we have $n$ entries in our $dp$ table and each entry takes $O(n)$ time to compute so we have to do a total of $n \cdot O(n) = O(n^2)$ computations. (We know it takes $O(n)$ time to compute each entry in the dp table because in each entry we iterate over $i$ where $0 \leq i < j$ and in the worst case $j$ equals $n$ and so $0 \leq i < n$ means we loop over $O(n)$ values.) Moreover, since the cost matrix $w$ has $n^2$ entries, it requires quadratic time to read the input, so a faster algorithm isn’t possible in general.

However, in 2017 three researchers from UC San Diego -- Marvin Künnemann, Ramamohan Paturi, and Stefan Schneider -- challenged this assumption. They noticed that if one can input $w$ in a more compact form, perhaps a faster algorithm would be possible.

Künnemann et al. focused on the case where $w$ is a low-rank matrix.

<details>

<summary>Refresher: What is a low-rank matrix? </summary>

I'll tell you what a low rank matrix is!
</details>

If the cost matrix $w$ has rank $r < n^{o(1)}$, then instead of directly inputting $w$ into $\text{LWS}$, we can input the matrices $A, B \in \mathbb{R}^{n \times r}$ into $\text{LWS}$ where $w = A \times B^T$. This reduces the input size of the problem from $n^2$ to $n^{1 + o(1)}$. With a much smaller input size, we now have a hope of coming up with a faster algorithm for $\text{LWS}$. (For the informal reader, $o(1)$ in this context means some constant less than $1$.)


Once Künnemann et al. come up with the clever idea of compressing $w$ by expressing it as a low rank matrix, they take it a step further. They use techniques from fine-grained complexity to show that when $w$ is low-rank, you can actually solve $\text{LWS}$ in $O(n^{2 - 1/r})$ time, a polynomial speedup over the standard $O(n^2)$ runtime of $\text{LWS}$!


this problem reduces to the minimum inner product $\text{Min-IP}$ problem:




If you sort the LWS problem, it becomes low rank!

# What is $k\text{D}\hspace{1mm}\text{LWS}$?

## Grid Airplane Refueling Problem

<!--
We prove that for $k\text{D}\hspace{1mm}\text{LWS}$ problems a polynomial speedup is possible whenever the cost to transition from one DP state to another is constant. The moment the cost to go from one DP state to another becomes slightly super-constant, it is impossible to achieve a polynomial speedup. -->

## APSP

$$
dp[i, j]
    =
    \begin{cases}
        0
        &
        \text{if $j - i == 0$}
        \\
        \min
        \begin{cases}
          d(i, j)
          \\
          \min_{i < k < j}
          dp[i, k] + dp[k, j]
        \end{cases}
        &
        \text{otherwise.}
    \end{cases}
$$

$$
dp[i, j]
    =
    \begin{cases}
        0
        &
        \text{if $j - i == 0$} \\
        \min
        &
        \text{otherwise.}
        \\
    \end{cases}
$$

where

$$
w[i, j, k]
    =
    \begin{cases}
        - \infty
        &
        \text{if $j - i == 0$} \\
        d(i, j)
        &
        \text{otherwise.}
        \\
    \end{cases}
$$


Many of the problems I mention have links -- not to their Wikipedia pages but to their Leetcode pages. This is because many of the problems I discuss are 1) well-known and 2) useful, practical problems. Despite the math and the proofs, this paper does not live in theory alone. It is motivated by real-world problems.

Make note about doing research with awesome collaborators.
