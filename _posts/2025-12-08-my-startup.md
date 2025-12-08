---
layout: post
title: A good startup idea...
date: 2025-12-08 11:59:00-0400
description: Why I'd like to build a startup that detects LLM generated text
tags: comments
categories: startup, LLM, detection
giscus_comments: true
related_posts: true
---

A friend recently asked me: *if you could start any startup right now, what would it be?*

I'm still in school, so I'm not actually starting anything. But the answer came to me immediately: **a startup that detects LLM-generated text**.

Here's why this would be compelling:

**1. The problem matters.** Distinguishing human writing from AI-generated content is critical for preventing the internet's decay into synthetic slop. We need to preserve spaces where authentic human thought still exists.

**2. It's technically hard.** LLM detection is a rich mathematical problem with no easy solutions. The adversarial dynamics—models getting better at mimicking humans while detectors improve at catching them—create an endless arms race of interesting research questions.

**3. You're competing against nobody.** Big AI labs have zero incentive to solve this problem. In fact, reliable LLM detection would actively hurt their business. If students knew their essays would get flagged, ChatGPT usage drops. If advertisers could detect bot-generated content, ad fraud gets exposed. The companies with the resources to build serious competition are the same ones who benefit from the status quo. Why do you think OpenAI shut down their [AI-detection](https://openai.com/index/new-ai-classifier-for-indicating-ai-written-text/) model instead of fixing it?

![openai classifier shutdown](/assets/img/my-startup/openai-clasifier.png)

It's rare to find a startup idea that checks all three boxes: important problem, hard technology, and a moat built from your competitors' misaligned incentives.

I recently discovered [Pangram](https://www.pangram.com/), a startup doing exactly this.

<aside>
  {% include figure.liquid loading="eager" path="assets/img/my-startup/pangram.png" class="img-fluid rounded z-depth-1" zoomable=true %}
</aside>

They make a bold claim: "Third-party verified results with a near-zero false positive rate." Their [arXiv paper](https://arxiv.org/pdf/2402.14873) from July 2024 lays out the technical details.

**I decided to test it.** First, I fed their detector the actual abstract from their own paper:

<aside>
  {% include figure.liquid loading="eager" path="assets/img/my-startup/human-abstract.png" class="img-fluid rounded z-depth-1" zoomable=true %}
</aside>

Good news—it correctly identified this as human-written. Then I asked ChatGPT to "plz write the abstract for pangram text, a deep learning model that identifies llm generated text":

<aside>
  {% include figure.liquid loading="eager" path="assets/img/my-startup/ai-abstract.png" class="img-fluid rounded z-depth-1" zoomable=true %}
</aside>

The detector caught it. AI-generated, as expected.

**Two examples prove nothing,** but it's a promising start. I don't know who else is tackling this problem seriously, but it's clearly worth pursuing. The technology works, the incentives are aligned, and the market needs it. Someone should build this—and it looks like someone already is.

*Thanks to Yonatan for the great discussion that inspired this post.*
