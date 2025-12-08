---
layout: distill
title: Can You Solve Dynamic Programming Problems Faster? Part 1, One-Dimensional LWS
date: 2024-01-09 11:59:00-0400
description: We prove you can solve dynamic programming problems polynomially faster if you have a simple cost function for kD LWS problems.
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

A friend recently asked me: if you could start any startup right now, what would it be?

I'm still in school, so I'm not actually starting anything. But the answer came to me immediately: **a startup that detects LLM-generated text**.

Here's why this would be compelling:

**1. The problem matters.** Distinguishing human writing from AI-generated content is critical for preventing the internet's decay into synthetic slop. We need to preserve spaces where authentic human thought still exists.

**2. It's technically hard.** LLM detection is a rich mathematical problem with no easy solutions. The adversarial dynamics—models getting better at mimicking humans while detectors improve at catching them—create an endless arms race of interesting research questions.

**3. You're competing against nobody.** Big AI labs have zero incentive to solve this problem. In fact, reliable LLM detection would actively hurt their business. If students knew their essays would get flagged, ChatGPT usage drops. If advertisers could detect bot-generated content, ad fraud gets exposed. The companies with the resources to build serious competition are the same ones who benefit from the status quo.

It's rare to find a startup idea that checks all three boxes: important problem, hard technology, and a moat built from your competitors' misaligned incentives.

I recently discovered [Pangram](https://www.pangram.com/), a startup doing exactly this.

![pangram-website](assets/img/my-startup/pangram.png)

They make a bold claim: "Third-party verified results with a near-zero false positive rate." Their [arXiv paper](https://arxiv.org/pdf/2402.14873) from July 2024 lays out the technical details.

**I decided to test it.** First, I fed their detector the actual abstract from their own paper:

![](assets/img/my-startup/real-abstract)

Good news—it correctly identified this as human-written. Then I asked ChatGPT to "plz write the abstract for pangram text, a deep learning model that identifies llm generated text":

![](assets/img/my-startup/ai-abstract)

The detector caught it. AI-generated, as expected.

**Two examples prove nothing,** but it's a promising start. I don't know who else is tackling this problem seriously, but it's clearly worth pursuing. The technology works, the incentives are aligned, and the market needs it. Someone should build this—and it looks like someone already is.
