---
layout: distill
title: My Solutions to "How to scale your model"
date: 2025-06-30 11:59:00
description: My Solutions to "How to scale your model"
tags: comments
categories: interview llm inference scale
giscus_comments: true
related_posts: true
bibliography: 2018-12-22-distill.bib
---

# Intro

I thought it would be fun to go through [How to Scale Your Model](https://jax-ml.github.io/scaling-book/), a small book on how to train and perform inference on LLMs at scale. This is not what is a forward or backward pass. This is how do you maximize MFU for an MoE on 1024 GPUs. This is the big leagues and contains a lot of knowledge that is really just kept in large industry labs or the well-connected labs of academia. There is no real textbook on this info because it is so new and "cutting edge" -- that is until now. This material is "solidly untaught," [writes](https://jacobaustin123.substack.com/p/technical-writing-with-unit-tests) Jacob, one of the authors, "I don’t believe another resource exists that tells this technical story end-to-end."

I'm an especially big fan of [How to Scale Your Model](https://jax-ml.github.io/scaling-book/) because it has great practice problems. They are like HW problems, but they really make you think. And they are practical too.

Over the next couple of weeks, I want to work through all 11 chapters of [How to Scale Your Model](https://jax-ml.github.io/scaling-book/). I'm going to record my solutions here.

# Chapter 1 Solutions


## Question 1.1

**Question 1 [int8 matmul]:** Say we want to do $X[B, D] \cdot_D Y[D, F] \rightarrow Z[B, F]$ in int8 precision (1 byte per parameter) instead of bfloat16.<d-footnote>Here and throughout we'll use the notation $A \cdot_D B$ to indicate that the multiplication is performing a contraction over the D dimension. This is an abuse of einsum notation.</d-footnote>

1. How many bytes need to be loaded from memory? How many need to be written back to memory?
2. How many total OPs are performed?
3. What is the arithmetic intensity?
4. What is a roofline estimate for $T_\text{math}$ and $T_\text{comms}$? What are reasonable upper and lower bounds for the runtime of the whole operation?

Assume our HBM bandwidth is `8.1e11` bytes/s and our int8 peak OPs/s is `3.94e14`.

**Answer 1:**

1. Since this is int8, each number takes up a single byte. Therefore, we load $BD$ bytes from $X$, load $DF$ bytes from $Y$, and write $BF$ bytes from $Z$. In total we must communicate $BD + DF + BF$ bytes.
2. Matrix multiplication between $X$ and $Y$ costs $BF(D + (D-1)) = 2BFD$ OPs because for each of the $BF$ entries of the resultant matrix $Z$ we perform a dot product between two $D$ dimensional vectors, with $D$ multiplications and $D-1$ additions.
3. $$\begin{equation} \text{Arithmetic Intensity} = \frac{\text{Computation FLOPs}}{\text{Communication Bytes}} = \frac{2BFD}{BD + DF + BF} \approx \frac{BFD}{DF} = 2B \end{equation}$$. Again, we assume that the batch size $B$ is *much* less than the hidden dimensions $D, F$, i.e. $BD << DF$ and $BF << DF$ and so can ignore these two terms in the denominator.
4. Let's define $$\begin{equation}
T_\text{math} = \frac{\text{Computation FLOPs}}{\text{Accelerator FLOPs/s}} = \frac{2BFD}{8.1e11}
\end{equation}$$
and
$$\begin{equation}
T_\text{comms} = \frac{\text{Communication Bytes}}{\text{Network/Memory Bandwidth Bytes/s}} = \frac{DF}{3.9e14}
\end{equation}$$
Notice that $T_\text{math} > T_\text{comms}$ because
$$\begin{equation}\end{equation}$$
$$\begin{equation}
T_\text{lower}=\max(T_\text{math}, T_\text{comms})
\end{equation}$$

## Question 1.2

**Question 2 [int8 + bf16 matmul]:** In practice we often do different weight vs. activation quantization, so we might store our weights in very low precision but keep activations (and compute) in a higher precision. Say we want to quantize our weights in int8 but keep activations (and compute) in bfloat16. At what batch size do we become compute bound? Assume `1.97e14` bfloat16 FLOPs/s.

*Hint: this means specifically `bfloat16[B, D] * int8[D, F] -> bfloat16[B, F]` where $B$ is the "batch size".*

**Answer 2:**
Each number takes up a single byte in int8 and two bytes in bfloat16. Therefore, we load $2BD$ bytes from $X$, load $DF$ bytes from $Y$, and write $2BF$ bytes from $Z$. In total we communicate $2BD + DF + 2BF$ bytes. Assume that $B<<D$ and $B<<F$ because the batch size $B$ is often much less than the hidden dimensions $D, F$, meaning we approximate this as communicating $DF$ bytes.

Matrix multiplication between $X$ and $Y$ costs $BF(D + (D-1)) = 2BFD$ OPs because for each of the $BF$ entries of the resultant matrix $Z$ we perform a dot product between two $D$ dimensional vectors, with $D$ multiplications and $D-1$ additions.

Therefore
$$\begin{equation} \text{Intensity}(\text{Arithmetic}) = \frac{\text{Computation FLOPs}}{\text{Communication Bytes}} = \frac{2BFD}{BD + DF + BF} \approx \frac{2BFD}{DF} = 2B \end{equation}$$

$$\begin{equation} \text{Intensity(Accelerator)} = \frac{\text{Accelerator FLOPs/sec}}{\text{Accelerator Bytes/sec}} = \frac{1.97e14}{8/1e11} = 243 \end{equation}$$

We are compute bound when
$$\begin{align*}
\text{Intensity}(\text{Arithmetic}) \geq \text{Intensity}(\text{Accelerator})
\\
2B \geq 243
\\
B \geq 121.5
\end{align*}$$


## Question 1.3

**Question 3:** For the problem above, make a roofline plot of peak FLOPs vs. B for several values of D and F.


## Question 1.4

**Question 4:** What if we wanted to perform $\text{int8[B, D]} *_D \text{int8[B, D, F]} \rightarrow \text{int8[B, F]}$ where we imagine having a different matrix for each batch element. What is the arithmetic intensity of this operation?

**Answer 4:**
Here, the second matrix is *three* dimensional, not two dimensional. This will not effect the actual number of operations we perform, but will change the number of bytes communicated. We perform $BF(D + (D-1)) \approx 2BFD$ operations because for each of the $BF$ elements of the resultant matrix $Z$ we perform $D$ multiplications and $D-1$ additions. We load $BD$ bytes from $X$, $BDF$ bytes from $Y$, and write $BF$ bytes from $Z$, totaling $BD + BFD + BF$ bytes. Assuming that $B << D$ and $B << F$, the term $BFD$ dominates and we can effectively ignore the other two terms.

$$\begin{align*}
\text{Intensity}(\text{Arithmetic})
=
\frac{\text{Computation FLOPs}}{\text{Communication Bytes}}
=
\frac{BF(D + (D-1))}{BD + BFD + BF}
\approx
\frac{2BFD}{BFD}
=
2
\end{align*}$$

We have a *constant* arithmetic intensity, meaning that we will always compute 2 operations per byte, no matter what our batch size is. In other words, we will always be communication bottle-necked. This is bad.


## Question 1.5

**Problem 5 [Memory Rooflines for GPUs]:** Using the [spec sheet provided by NVIDIA for the H100](https://www.nvidia.com/en-us/data-center/h100/), calculate the batch size at which a matrix multiplication will become compute-bound. *Note that the Tensor Core FLOPs numbers are twice the true value since they're only achievable with structured sparsity.*

**Answer 5:** For bfloat16 with tensor core, the H100 SXM can compute 1919 teraflops per second, i.e. 1.919e15 FLOPs/sec. Because this is with sparsity, we must divide by two to get the FLOPs/sec for non-sparse operations, 1e15 FLOPs/sec. The H100 SXM has a reported GPU memory bandwidth of 3.35 terabytes per second, i.e. 3.35e12 bytes per second.
$$
\begin{align*}
\text{Intensity}(\text{Accelerator})
=
\frac{\text{Accelerator FLOPs/sec}}{\text{Accelerator Bytes/sec}}
=
\frac{1e15}{3.35e12}
=
298
\end{align*}
$$

Assume $\text{bfloat16[B, D]} *_D \text{bfloat16[D, F]} \rightarrow \text{bfloat16[B, F]}$ is the matrix multiplication we are performing. We load $2BD$ bytes from the first matrix, load $2DF$ bytes from the second matrix, and write $2BF$ bytes from the resultant matrix totaling $2BD + 2DF + 2BF$ bytes. Assume that $B<<F$ and $B<<D$ so that the $2DF$ term dominates and we can ignore the other two terms. We compute $BF(D + (D-1))$ bytes because each of the $BF$ entries in the resultant matrix requires $D$ multiplications and $D-1$ additions. For simplicty, let's approximate this as $2BFD$.
$$
\begin{align*}
\text{Intensity}(\text{Arithmetic})
=
\frac{\text{Computed FLOPs}}{\text{Communicated Bytes}}
=
\frac{BF(D + (D-1))}{2BD + 2DF + 2BF}
\approx
\frac{2BFD}{2DF}
=
2B
\end{align*}
$$

We are compute bound when
$$\begin{align*}
\text{Intensity}(\text{Arithmetic}) \geq \text{Intensity}(\text{Accelerator})
\\
2B \geq 298
\\
B \geq 149
\end{align*}$$
