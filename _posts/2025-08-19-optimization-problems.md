---
layout: distill
title: Optimization Problems
date: 2024-01-09 11:59:00-0400
description: Just a boi trying to solve some optimization problems.
tags: comments
categories: algorithms optimization math
giscus_comments: true
related_posts: true
# bibliography: 2018-12-22-distill.bib
---

First let's define our parameters:
* matrices $A \in \mathbb{R}^{m \times n}$; $B \in \mathbb{R}^{m \times n}$; $C \in \mathbb{R}^{p \times n}$; $W \in \mathbb{R}^{m \times m} = \text{diag}\{w_1, ..., w_m \}$ where $w_i > 0$
* vectors $b \in \mathbb{R}^n$; $d \in \mathbb{R}^p$
* scalars $\lambda \in \mathbb{R}+$
* By SVD, $A = U \Sigma V^T$ where $U \in \mathbb{R}^{m \times m}$ is orthonormal; $V \in \mathbb{R}^{n \times n}$ is orthonormal; $\Sigma \in \mathbb{R}^{m \times n}$ is a rectangular diagonal matrix

| Problem | $x^*$ | $f(x^*)$ | Equations | Existence | Uniqueness |
|---------|----|----|-----------|-----------|------------|
|
$\text{argmin} \|\|Ax - b\|\|_2^2 : x \in R^n$ |  $A^\dag b + z : z \in \mathcal{N}(AT)$ | $\| \| A A^\dag b + z - b\|\|_2^2$ |
