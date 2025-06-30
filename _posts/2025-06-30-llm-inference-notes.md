---
layout: distill
title: 30 Interview Questions on LLM Inference
date: 2025-06-30 11:59:00
description: How many FLOPs do you save with a kv-cache? When does the kv-cache hurt?
tags: comments
categories: interview llm inference kv-cache
giscus_comments: true
related_posts: true
bibliography: 2018-12-22-distill.bib
---

# Interview Questions

With Claude's help, I wrote ~30 interview questions on LLM Inference: FLOPs, attention, and the kv-cache. Because this material is not taught in my university courses, I hope these questions can be helpful. My notes on these topics can be found at the bottom.

## Easy

1. **Your model has `n_layers=32`, `n_heads=32`, `d_head=128`, and uses bfloat16 precision. Given an input with sequence length `L=64` and `B=4`, how many FLOPs does it take to compute attention `A(X) = softmax(Q K^T / \sqrt(d)) V`?

2. **Will using a KV cache help you if you are memory bound or compute bound?**

3. **Your model has `n_layers=32`, `n_heads=32`, `d_head=128` and uses bfloat16 precision. How much storage does a KV cache require for a single token? What about if we have `B*L` tokens where `B` is the batch size and `L` is the sequence length?**

3. **How many FLOPs will a KV cache save you? Calculate KV cache FLOPS savings: batch_size=B, context_length=L, generate next_n tokens. What's the speedup?**
    *Without cache: 6×B×(L+next_n)×d_model² total FLOPS. With cache: 6×B×L×d_model² (initial) + 2×B×next_n×d_model² (generation). Speedup = 3×(L+next_n)/(3×L+next_n). For L>>next_n, approaches 3× speedup.*

## Medium

1. **Your model has `n_layers=32`, `n_heads=32`, `d_head=128`, and uses bfloat16 precision and a KV cache. At what sequence length does KV cache exceed 1GB bytes for batch_size=1?**

   The kv cache has shape `(2, B, L, n_layers, n_heads, d_head)` which takes up `M=n_bytes*2*B*L*n_layers*n_heads*d_head` bytes. Here `n_bytes = 16 bits/8 = 2` bytes and `M=1GB=1e+9` bytes. Therefore, 1e+9 = 2x1×L×2×32×32×128 = 524,288×L ≈ 5e+6 L bytes. L ≈ 1e+9/5e+6 ≈ 2,000 tokens. After 2,000 tokens, the kv cache will exceed 1GB. This is why long conversations quickly exhaust GPU memory.

2. **Your GPU has 24GB of memory and like before, your model has `n_layers=32`, `n_heads=32`, `d_head=128` and uses bfloat16 precision and a KV cache. How many users can perform inference simultaneously with 500-token sequences vs 2000-token sequences?**

   The kv cache has shape `(2, B, L, n_layers, n_heads, d_head)` which takes up `M=n_bytes*2*B*L*n_layers*n_heads*d_head` bytes. Notice `24GB = 2.4e+10`.

   `L=500`: `2.4e+10 = 2 x 2 x B x 500 x 32 x 32 x 128`. `B = 24GB/(524KB x 500) ≈ 91`. At most you can have `B=91` different users asking this chatbot questions simultaneously.

    `L=2000`: `2.4e+10 = 2 x 2 x B x 2000 x 32 x 32 x 128`. `B = 24GB/(524KB x 2000) ≈ 22`. At most you can have `B=22` different users asking this chatbot questions simultaneously.


5. **A model spends 60% of time on memory reads during generation. What does this tell you about compute vs memory bandwidth?**

   Memory bandwidth is the bottleneck, not compute. GPU compute units are idle 60% of the time waiting for data. We know it's bandwidth (not a slow computation) because reading cached data should be fast - if it takes 60% of time, the memory system can't supply data fast enough for the compute units.

## Hard

3. **A model has n_layers, n_heads, d_head, and uses n_bytes precision and a KV cache. When generating ALL L tokens sequentially (tokens 1, 2, 3, ..., L) with batch size B, what is the total cumulative memory read from the KV cache and what is the total cumulative memory written to the KV cache?**

    When generating the `l`-th token (`1≤l≤L`), the KV cache stores the `l-1` previous keys and values, has shape `(2, B, l-1, n_layers, n_heads, d_head)`, and takes up `M(l) = n_bytes × 2 × B × (l-1) × n_layers × n_heads × d_head` bytes.

    **Memory Read:**
    When generating the `l`-th token, we must read the entire KV cache containing all `l-1` previous tokens, requiring `M(l)` bytes. The cumulative memory read across ALL L tokens is:
    ```
    R(L) = ∑(l=1 to L) M(l)
        = n_bytes × 2 × B × n_layers × n_heads × d_head × ∑(l=1 to L) (l-1)
        = n_bytes × 2 × B × n_layers × n_heads × d_head × L×(L-1)/2
    ```
    (Here, we used the identity that `\sum_{i=0}^n i = (i+1)*i/2` where `i=l-1`.)

    **Memory Write:**
    When generating the `l`-th token, we write only the new key-value pair for that token: `n_bytes × 2 × B × n_layers × n_heads × d_head` bytes (constant per token). The cumulative memory written across ALL `L` tokens is:
    ```
    W(L) = ∑(l=1 to L) (n_bytes × 2 × B × n_layers × n_heads × d_head)
         = L × (n_bytes × 2 × B × n_layers × n_heads × d_head)
    ```

4. **Which grows more quickly: cumulative memory reads from or cumulative memory writes to the KV cache? If we have a sequence of `L=1000` tokens, how many more/fewer times do we cumulatively read than cumulatively write? What are the implications of this?**

   From the previous question we know that the cumulative reads from the KV cache is `R(L) = n_bytes × 2 × B × n_layers × n_heads × d_head × L×(L-1)/2` and the cumulative writes to the KV cache is `W(L) = L × n_bytes × 2 × B × n_layers × n_heads × d_head`. Notice that memory reads grows quadratically with `L` but memory writes grow linearly with `L`. So the cumulative reads grow faster than the cumulative writes.

   The read-write ratio is `R/W = (L-1)/2` meaning we read `(L-1)/2` times more data than we write. For `L=1000` tokens, we read `(1000-1)/2~500` times more data than we write.

   Implications: If our model becomes memory-bound, this is likely from reading from memory not from writing to memory.


16. **How do you handle KV cache across multiple GPUs? Explain how each parallelism strategy works with the KV-cache: model parallelisim, data parallelsim, sequence apralellim, pipeline paralellsim?**
    *Model parallelism: partition cache by layers across GPUs, but attention requires all-gather communication. Sequence parallelism is challenging due to autoregressive dependencies. Pipeline parallelism works but complicates cache management.*

### System Design & Trade-offs

9. **You're serving a chatbot. User conversations average 50 tokens but 1% go to 10,000 tokens. How do you handle KV cache memory?**

   Implement sliding window attention (keep only last N tokens) or progressive offloading (move old cache entries to CPU/disk). The 1% tail drives 99% of memory costs, so optimize for the outliers.

11. **Production system: 95th percentile latency matters more than throughput. KV cache or large batches?**
    *KV cache for consistent low latency per user, even if total throughput (users/sec) is lower. Large batches increase individual request latency due to queuing effects.*

12. **Two GPUs: 16GB with 2TB/s bandwidth vs 32GB with 1TB/s bandwidth. Which is better for long conversations?**
    *Depends on sequence length. Short sequences (memory bandwidth bound): 16GB GPU wins. Long sequences (memory capacity bound): 32GB GPU wins. The crossover point depends on your specific workload.*

13. **Inference cost optimization: KV cache uses 2× memory but 3× faster generation. At what utilization rate do you break even?**
    *If you can keep GPUs 66%+ utilized with KV cache (⅔ of capacity), the 3× speed improvement compensates for 2× memory cost. Below 66% utilization, large batches without KV cache may be more cost-effective.*

14. **User requests 5 different continuations of same prompt. KV cache strategy?**
    *Cache the common prompt prefix once, then branch KV cache only for the different continuations. This avoids recomputing the shared 90% of work while only duplicating the divergent portions.*

15. **Why might a model perform WORSE with KV cache enabled in some scenarios?**
    *Memory pressure causing GPU memory swapping to CPU, memory bandwidth saturation slowing all operations, or cache management overhead (copying, allocation) exceeding computational savings for short sequences.*


### Advanced

17. **Flash Attention reduces memory usage. How does this interact with KV cache benefits?**
    *Flash Attention optimizes attention computation memory (intermediate activations), while KV cache optimizes recomputation across time steps. They're complementary - Flash Attention reduces per-step memory, KV cache reduces cross-step computation.*

18. **In transformer training vs inference: why is KV cache irrelevant for training?**
    *Training uses teacher forcing - processes entire sequences in parallel with full attention matrices. No autoregressive generation step-by-step, so no opportunity to reuse previous computations.*

19. **Model uses rotary positional embeddings (RoPE). How does this affect what we cache?**
    *Cache unrotated K,V vectors and apply position-dependent rotation during attention computation. This is because the rotation depends on the relative position between query and key, which changes for each new token.*

20. **A sequence has repeated phrases. Could we compress KV cache by deduplicating similar K,V vectors?**
    *Theoretically possible, but positional embeddings make even identical tokens have different representations. The attention mechanism depends on position, not just content, making deduplication complex and potentially harmful to model quality.*

21. **Mixture of Experts (MoE) model: different tokens activate different experts. How does this complicate KV cache?**
    *Each expert may need separate KV caches if they have different dimensions, or the routing decisions affect which cached values are relevant. Cache size scales with number of active experts per token.*

22. **Streaming generation: user types while model generates. How do you update KV cache mid-generation?**
    *Invalidate cache from the interruption point onward, recompute from the user's new input position. This requires careful bookkeeping of which cache entries correspond to which part of the conversation state.*

23. **Speculative decoding: generate multiple tokens in parallel, then verify. How does KV cache work here?**
    *Cache optimistically for all speculated tokens, but maintain checkpoints to rollback if speculation fails. This creates a tree of potential cache states that must be managed efficiently.*

24. **Model pruning removes 50% of attention heads. How does this affect KV cache memory and performance trade-offs?**
    *Memory usage halves (n_heads reduces by 50%), but attention quality may degrade, potentially requiring longer sequences for same performance. Need to rebalance the memory savings vs quality trade-off.*

26. **We store K,V per head in KV cache. Do we also need to store the concatenated output after multi-head attention?**
    *No, only store individual K,V per head. The concatenated output gets recomputed each time because it depends on the new query. KV cache only stores the reusable components (keys and values), not query-dependent results.*

27. **When we say the KV cache memory "grows linearly with sequence length," do we mean input prompt length or total length including generated tokens?**
    *Total length including all generated tokens. During autoregressive generation, sequence length = original_prompt + tokens_generated_so_far. Each new token increases this total length by 1.*

27. **Batch processing sequences of different lengths with padding: do shorter sequences waste KV cache memory?**
    *In naive implementations, yes - all sequences get padded to max_length, wasting memory. Better implementations use attention masks to ignore padding during computation and variable-length caching to avoid storing padding positions in the KV cache.*

29. **What is Grouped Query Attention (GQA): multiple heads share K,V. How does this change the storage formula for a model using a KV cache?**
    GQA is multiple heads share K,V. Formula becomes: 2 × n_bytes × n_layers × n_kv_heads × d_head (where n_kv_heads < n_heads). If 32 query heads share 8 KV heads, you get 4× memory savings while maintaining most of the attention quality.

30. **Beam search: each beam needs its own KV cache. How does this explode memory?**
    *Memory scales as k_beams × sequence_length × cache_size. With 8 beams and 1000 tokens, you need 8× more memory than greedy decoding. When beams split from same parent, cache copying creates temporary memory spikes beyond this 8× baseline.*

32. **What memory and time overheads does KV cache add?**
    *Memory: 2×n_bytes×n_layers×n_heads×d_head×B×L storage overhead. Time: memory allocation/deallocation, cache management, and quadratic read bandwidth growth (reading more cached data for each new token). These can outweigh benefits for short sequences.*

33. **Can we be memory-bound during forward pass but FLOPS-bound during sampling?**
    *Yes! Forward pass reads massive amounts of cached K,V data (memory-bound). Sampling step does text generation, top-k filtering, probabilistic sampling (FLOPS-bound). Different parts of inference have different bottlenecks depending on sequence length and model size.*

---


# Notes

Here are my notes, also written with claude.

## FLOPS
**FLOPS (Floating Point Operations Per Second)** counts arithmetic operations (add, multiply, etc.).

**Matrix-Vector Multiplication:** A(m×k) @ v(k×1) = `2×m×k` FLOPS (k multiply-adds for each of m output elements). The factor of 2 comes from each output element requiring k multiplications + k additions.

**Matrix-Matrix Multiplication:** A(m×k) @ B(k×n) = `2×m×k×n` FLOPS (k multiply-adds for each of m×n output elements). The factor of 2 comes from each output element requiring k multiplications + k additions.

**Compute Bound (FLOPs Bound):** The bottleneck is arithmetic operations. GPU compute units are fully utilized, but memory can supply data faster than it's consumed. Adding more compute power would speed up the process.

**Memory Bound (Bandwidth Bound):** The bottleneck is data movement. GPU compute units are idle waiting for data from memory. Memory bandwidth cannot supply data fast enough for the available compute. Adding more memory bandwidth would speed up the process.

## Attention

**Parameter Definitions:**
- **B:** batch size (number of sequences processed in parallel)
- **L:** sequence length (number of tokens in each sequence)
- **d_model:** model dimension (hidden size, e.g., 4096 for large models)

**Matrix Dimensions & Descriptions:**
- **X (input tokens):** (B, L, d_model) - batch of token sequences
- **W_q (query weights):** (d_model, d_model) - learned projection for queries
- **W_k (key weights):** (d_model, d_model) - learned projection for keys
- **W_v (value weights):** (d_model, d_model) - learned projection for values
- **Q (queries):** (B, L, d_model) - what each token is looking for
- **K (keys):** (B, L, d_model) - what each token represents
- **V (values):** (B, L, d_model) - information content of each token

**Full Attention Equation:**
Attention(Q,K,V) = softmax(QK^T / √d_model) @ V

**FLOPS Breakdown:**
- Q = X @ W_q: `2 × B × L × d_model²` FLOPS
- K = X @ W_k: `2 × B × L × d_model²` FLOPS
- V = X @ W_v: `2 × B × L × d_model²` FLOPS
- QK^T: `2 × B × L² × d_model` FLOPS
- softmax(QK^T / √d_model): `C₁ × B × L²` FLOPS (constant operations per element)
- softmax(scores) @ V: `2 × B × L² × d_model` FLOPS

**Total FLOPS:** `6 × B × L × d_model² + 4 × B × L² × d_model + C₁ × B × L²`

**KV Cache Impact:** Only the first term `6 × B × L × d_model²` changes with KV caching. The terms `4 × B × L² × d_model + C₁ × B × L²` do not change with KV cache. For simplicity, we only focus on this firt term in the rest of this post. But in practice you must add back the two unchanging terms.

## KV Cache

**KV Cache** stores the computed key and value vectors from previous tokens during autoregressive generation, avoiding redundant recomputation. Instead of recalculating K and V for all tokens at each step, we cache them and only compute new entries.

**Example**
```
Prompt: "The weather today"
Target: Generate " is sunny"

Prefill Phase:
- Input: ["The", "weather", "today"] (3 tokens)
- Compute: K₁,V₁, K₂,V₂, K₃,V₃ in parallel
- Cache: Store all 3 K,V pairs
- Time: Fast parallel processing

Generation Phase:
Token 4 " is":
- Read: K₁,V₁, K₂,V₂, K₃,V₃ from cache
- Compute: Q₄, K₄, V₄
- Attention: Q₄ @ [K₁,K₂,K₃,K₄]^T @ [V₁,V₂,V₃,V₄]
- Cache: Append K₄,V₄

Token 5 " sunny":
- Read: K₁,V₁, K₂,V₂, K₃,V₃, K₄,V₄ from cache
- Compute: Q₅, K₅, V₅
- Attention: Q₅ @ [K₁,K₂,K₃,K₄,K₅]^T @ [V₁,V₂,V₃,V₄,V₅]
- Cache: Append K₅,V₅
```

**Implementation**
To store the previous keys and value, the KV cache is defined as a tensor
```
cache = Tensor.empty(2, B, L, n_layers, n_heads, d_head)
```
where:
- 2: separate storage for keys and values
- B, L: the number of tokens (L) across all sequences (B)
- n_layers, n_heads, d_head: the number of keys and value we have across all layers of a model
where the keys are `cache[0]` and the values are `cache[1]`. The kv cache works by reading from and writing to this tensor.

To find the key from the 6th token in the 3rd sequence at the 9th head from 14th layer of the model, we would do
```
cache[0, 3, 6, 14, 9]
```
Naively, this kind of naive KV-cache only works for greedy sampling and it does not work for more complicated sampling schemes like min-p or beam decoding.

**Memory Requirement:** This requires storing `2 × B × L × n_layers × n_heads × d_head × n_bytes` total bytes in memory because the cache is a tensor of shape `2, B, L, n_layers, n_heads, d_head` and each element in that tensor takes up `n_bytes`.

**Prefill Phase (Time to First Token - TTFT):**
- Process entire input prompt in parallel
- Compute K,V for ALL input tokens simultaneously: O(L_input)
- Initialize KV cache with input token representations
- High memory bandwidth and compute utilization
- TTFT dominated by this parallel processing of input

**Generation Phase (Time Between Tokens):**
- Process one new token at a time autoregressively
- Compute K,V only for new token: O(1) per token
- Read entire cached history for attention: O(L_total)
- Memory bandwidth becomes bottleneck as sequence grows


**When FLOPS Bound (KV Cache Helps):**
- Without cache: Spend time computing Q,K,V for all tokens (6 × d_model² FLOPS)
- With cache: Only compute Q for new token (2 × d_model² FLOPS)
- **Result**: 3× speedup because we eliminated computational work

**When Memory Bound (KV Cache May Hurt):**
- Without cache: Read input tokens, compute everything fresh
- With cache: Read massive amounts of cached K,V data plus compute Q
- **Result**: More memory traffic, potentially slower despite less computation

**Example**: Long sequences (1000+ tokens) often become memory bound because reading cached data dominates the time, making the FLOPS savings irrelevant.

### KV Cache Pros and Cons

| **Advantages** | **Disadvantages** |
|----------------|-------------------|
| **3× FLOPS reduction** for Q,K,V computation | **Linear memory growth** with sequence length |
| **Enables real-time generation** for interactive applications | **Reduces maximum batch size** due to memory constraints |
| **Critical for long context** models (would be unusably slow otherwise) | **Memory bandwidth bottleneck** for very long sequences |
| **Powers production chat systems** at scale | **Beam search memory explosion** when caches are copied |
| **Amortizes compute cost** over conversation length | **No benefit for training** (only inference optimization) |
| **Enables streaming responses** for better UX | **Overhead for short sequences** (< 50 tokens) |
| **Facilitates longer conversations** without timeout | **Cache sharing impossible** between different users |

### FLOPS Analysis

| Generation | Without KV Cache | With KV Cache | Explanation |
|----------|------------------|---------------|-------------|
| **Single token (Lth token, batch size B)** | `3 × 2 × d_model² × B + C` | `2 × d_model² × B + C` | Without: compute Q,K,V from scratch (3 matrices × 2 FLOPS factor). With: only compute Q, reuse cached K,V. C = attention matrix operations (same for both) |
| **Cumulative tokens (tokens 1, ..., L, batch size B)** | `3 × 2 × d_model² × B × L + C × L` | `2 × d_model² × B × L + C × L` | Without: recompute Q,K,V for every token. With: compute Q for each new token only. C scales with sequence length |

### KV Cache Memory & I/O

| Generation | Memory Stored | Bandwidth: Read | Bandwidth: Write | Explanation |
|----------|---------------|------|-------|-------------|
| **Single token (Lth token, batch size B)** | `2 × n_bytes × n_layers × n_heads × d_head × B × L` | `2 × n_bytes × n_layers × n_heads × d_head × B × (L-1)` | `2 × n_bytes × n_layers × n_heads × d_head × B` | Memory: total cache size grows with L. Read: access all previous tokens for attention. Write: store current token's K,V |
| **Cumulative tokens (tokens 1, ..., L, batch size B)** | `2 × n_bytes × n_layers × n_heads × d_head × B × L` | `2 × n_bytes × n_layers × n_heads × d_head × B × L × (L-1)/2` | `2 × n_bytes × n_layers × n_heads × d_head × B × L` | Memory: same final size. Read: cumulative bandwidth = 0+1+2+...+(L-1) = L(L-1)/2 per batch. Write: one entry per token generated |

### Beam Search with KV Cache

**Definition:** Beam search maintains the k most promising sequences at each step, exploring multiple paths simultaneously to find higher-quality outputs than greedy decoding.

**Algorithm:**
1. Start with k beams (initially just the prompt)
2. For each beam, generate top-r candidate next tokens
3. Score all k×r candidates
4. Keep top-k sequences as new beams
5. Repeat until completion

**Concrete Example with KV Cache:**
```
Model: 2 layers, 4 heads, d_head=64, n_bytes=2
Beam width k=2, candidates per beam r=3
Starting sequence: "The weather" (2 tokens)

Initial State:
Beam 1: "The weather"
KV Cache 1: K₁,V₁ (for "The"), K₂,V₂ (for "weather")
Cache size: 2×2×2×4×64 = 1,024 bytes

Beam 2: "The weather"
KV Cache 2: K₁,V₁ (for "The"), K₂,V₂ (for "weather")
Cache size: 1,024 bytes (same starting point)

Step 1: Generate third token (each beam explores 3 candidates)
Beam 1 candidates: "is" (0.6), "was" (0.3), "feels" (0.2)
Beam 2 candidates: "today" (0.5), "looks" (0.4), "seems" (0.1)

Step 2: Select top-2 sequences across all 6 candidates
Selected: "The weather is" (0.6), "The weather today" (0.5)

MEMORY EXPLOSION POINT:
New Beam 1: "The weather is"
- Needs: COPY of original Beam 1 cache + new K₃,V₃ for "is"
- Cache: K₁,V₁, K₂,V₂, K₃ⁱˢ,V₃ⁱˢ = 1,536 bytes

New Beam 2: "The weather today"
- Needs: COPY of original Beam 2 cache + new K₃,V₃ for "today"
- Cache: K₁,V₁, K₂,V₂, K₃ᵗᵒᵈᵃʸ,V₃ᵗᵒᵈᵃʸ = 1,536 bytes

Total Memory: 3,072 bytes (3× explosion from beam splitting!)
Note: The beams started identical but diverged due to different candidate selection
```

### Beam Search FLOPS & Memory Analysis

| Generation | FLOPS (vs Greedy) | Memory Stored | Bandwidth: Read | Explanation |
|----------|------------------|---------------|-------------|-------------|
| **Single token (Lth token, k beams)** | `k × (2 × d_model² + C)` | `k × 2 × n_bytes × n_layers × n_heads × d_head × L` | `k × 2 × n_bytes × n_layers × n_heads × d_head × (L-1)` | k separate sequences, each needs own KV cache and computation |
| **Cumulative tokens (tokens 1, ..., L, k beams)** | `k × (2 × d_model² × L + C × L)` | `k × 2 × n_bytes × n_layers × n_heads × d_head × L` | `k × 2 × n_bytes × n_layers × n_heads × d_head × L × (L-1)/2` | Memory can temporarily spike beyond k× during beam splitting when caches must be copied |

**Key Insight:** Beam search with KV cache uses k× more memory than greedy decoding, but beam splitting during search can cause temporary memory explosions when multiple beams inherit from the same parent cache.

## Memory Bound vs. FLOPS Bound

### Definitions
**FLOPS Bound (Compute Bound):** The bottleneck is arithmetic operations. GPU compute units are fully utilized, but memory can supply data faster than it's consumed. Adding more compute power would speed up the process.

**Memory Bound (Bandwidth Bound):** The bottleneck is data movement. GPU compute units are idle waiting for data from memory. Memory bandwidth cannot supply data fast enough for the available compute. Adding more memory bandwidth would speed up the process.

### How to Identify
- **FLOPS Bound**: High GPU utilization (90%+), compute units busy
- **Memory Bound**: Low GPU utilization despite high memory traffic, compute units waiting

### KV Cache Impact
**When FLOPS Bound (KV Cache Helps):**
- Without cache: Spend time computing Q,K,V for all tokens (6 × d_model² FLOPS)
- With cache: Only compute Q for new token (2 × d_model² FLOPS)
- **Result**: 3× speedup because we eliminated computational work

**When Memory Bound (KV Cache May Hurt):**
- Without cache: Read input tokens, compute everything fresh
- With cache: Read massive amounts of cached K,V data plus compute Q
- **Result**: More memory traffic, potentially slower despite less computation

**Example**: Long sequences (1000+ tokens) often become memory bound because reading cached data dominates the time, making the FLOPS savings irrelevant.

## Resources

Some great resources I found:
1. https://kipp.ly/transformer-inference-arithmetic/
2. https://blog.eleuther.ai/transformer-math/
3. https://r4j4n.github.io/blogs/posts/kv/
4. https://medium.com/data-science/deep-dive-into-kv-caching-in-mistral-7e0cea8409a1
