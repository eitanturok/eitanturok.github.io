---
layout: distill
title: The Ultimate Guide to Dynamic Programming
date: 2024-01-09 11:59:00-0400
description: I explain dynamic programming, but actually explain it well. This is the guide I wish I had when I learned dynamic programming for the first time.
tags: comments
categories: algorithms, dynamic-programming
giscus_comments: true
related_posts: true

---

1. What every DP solution must have
   1. Explain $dp[i]$ in plain English
   2. Recurrence relation for $dp[i]$ (with base cases)
      1. [Optional] Proof of correctness for recurrence relation
   3. Solution is given by which states of $dp$
   4. Pseudo-code
   5. Time and space complexity
2. 7 Steps to solve a DP problem
3. Space and Time complexity
4. Common DP Reccurence Relations (LWS)
5. Implementaiton
   1. Top-down vs bottom up
   2. Evaluation Order (with for loops)
6. DP is DFS on recursion tree
   1. Make visualizations

Differences between top-down and bottom-up:

| # top-down | # bottom-up |
-------------------------
| starts with largest examples first | starts with smallest sub-problems first | 


| Characteristic         | Top-Down     | Bottom-Up |
|--------------|-----------|------------|
| Which kind of problems does it encounter first? | The largest sub-problems      | The smallest sub-problems        |
| Which kind of problems does it solve first? (Trick Q b/c they actually *solve* the problems at the same time.) | The smallest sub-problems      | The smallest sub-problems        |
| How does it store previously solved sub-problems      | A dictionary | An array       |
| Through code, how do you go from one sub-problem to another?  | Via recursion | Via for-loops       |
| Deterministic or stochastic time complexity | Stochastic b/c dictionaries access times are amortized + stochastic | Determintic b/c implemented with an array |
| Easy or hard to determine time/space complexity?  | Hard b/c the code is recursive. You must create a T(n) = ... equation, solve the recurrence relation, and then get the complexity to determine how many times this function is called before hitting the base case | Easy b/c can simply read-off time complexity from the for-loops and space complexity from the amount of pre-allocated space in the array|
| Easy or hard to implement in code?  | Easy b/c literally just read off the recurrence-relation and turn it into code | Harder b/c must translate the recurrence-relation into for loops + determine weird edge cases + evaluation order |
|  Any extra space requirements? | Yes! In addition to normal space to solve DP problem, must have space on recursive-call stack to enable all of these recursive calls. NEED extra space here. | No! This is more space efficient than top-down b/c it does not require space for the recursive-call stack.|
| Can you store already solved sub-problems in a dict?  | Yes | Yes  |
| Can you store already solved sub-problems in an array?  | No -- to determine if a problem is already solved, we look it up in the dictionary in O(1) time  | Yes  |
| Draw a recursion-visualizer diagram | Have full diagram and do DFS on it, not visting states we've already seen. | We enforce a topological sorted ordering on the nodes and then just visit them, one at a time. |


Q: How much time would it add to do top-down DP in a list as oppossed to a dictionary?


1. top-down uses dictionary and bottom-up u