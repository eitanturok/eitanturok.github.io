Review #12A
===========================================================================

Overall merit
-------------
5. Accept

Reviewer expertise
------------------
4. Expert

Paper summary
-------------
The paper presents a nice set of results about a general class of "dynamic programming problems" (in the sense that they can be defined naturally with a recurrence relation that corresponds to what we typically call a DP).

The results are along the same lines of a previous work by KPS from ICALP 2017. The main novelty is that the results are extended to higher dimensions (meaning that the "DP table" is now more of a "DP tensor").

The results have the following form: if a problem is a DP problem of a certain form, and its "rank" is small (large) then it is easy (hard). Where:
-  "hard" means it cannot be solved faster than the trivial running time, assuming some conjecture like SETH or APSP, and "easy" means that it can be solved faster by some n^0.1 factor.
- small/large means something different depending on the type of the DP. In one case it is constant vs. super-constant and in another it is 1 vs. at-least-3.

The results are interesting because they make us feel like we understand the complexity of a naturally defined class of problems. They also give non-trivial statements about some specific problems.

The weakness of the paper is that all reductions and algorithms are standard after one sets up the framework. The latter should be considered the main contribution of the paper and is indeed laudable: defining the high dimensional generalizations and the notions of rank, and observing that natural problems have small rank.

Another weakness is that the DP model is not the most general. For example, there are lots of 2-dimensional DP problems, but, for example, I think longest common subsequence isn't captured by 2D LWS, because this only considers DP updates along “rook moves”.

Comments for authors
--------------------
- The introduction is missing a bit of clarity regarding the tightness of the results. You should mention that the lower bounds you get match the upper bounds (even though it's implicit from the arrows in the figure). Moreover, you should discuss why there isn't an equivalence in the high dimensional cases, when there is one for one dimension.

- In the second paragraph of the intro: some of the examples/references are inaccurate:
-- The reference [ABBK17] seems to be confused with the paper:
Amir Abboud, Arturs Backurs, Virginia Vassilevska Williams:
If the Current Clique Algorithms Are Optimal, so Is Valiant's Parser.
-- The examples of parsing and RNA folding are not so good because the n^3 bound from DP can indeed be improved by using fast matrix multiplication (as you do in your paper).


* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *


Review #12B
===========================================================================

Overall merit
-------------
3. Weak reject

Reviewer expertise
------------------
2. Some familiarity

Paper summary
-------------
In previous work, Künnemann, Paturi and Schneider (ICALP 2017)
considered fine-grained complexity aspects of a general 1-dimensional
dynamic programming problem, "LWS". In that work, they showed
subquadratic equivalence of many variants of the problem to apparently
quite different computational problems, and showed how both known and
new upper and lower bounds could be captured by considering variants
of LWS using these equivalences.

In the present work, the authors attempt something similar for a
"k-dimensional version" of LWS. They focus on a parameterization by
rank and show:

If an instance of kD-LWS has weights with constant tensor rank, then
some speedup over O(n^{k+1}) is possible.

If the tensor rank is slightly superconstant, then this is not
possible under SETH.

They also show some results for special cases or variants:

2D-LWS with slice rank 1 has a speedup (but with slice rank 3 it does
not).

A variant, "polygon triangulation 2D-LWS", has no speedup if the rank
is slightly superconstant or if the slice rank is 3.

The weakness, unfortunately, is that unlike LWS the kD-LWS problem
does not appear to have any rich collection of natural applications.

Furthermore, the "static kD-LWS" problem is not, unlike in the 1D-case,
shown to be "speedup equivalent" to kD-LWS.

The "polygon triangulation 2D-LWS" problem has a number of genuine
applications, but the speedup for these cases are known from the
literature and no generalization of these results is given. (For
example, unless I'm confused, the positive special case from the
literature should be a special case of weights with tensor rank 1,
and the general case of weights with tensor rank 1 is not demonstrated.)

Also, almost all results are lower bounds.

I find this particularly concerning considering that a number of
decisions have needed to be made in the move to k-dimensional
problems:

What is "the k-dimensional version" of the problem? They have chosen
the variant where the "recursive step" has to be taken along the
k-dimensional grid graph (with arbitrary step length, i.e., perhaps I
should call it the "k-dimensional rook's graph").

Why this particular definition of the "static version" of the problem?
Given that you are not able to show equivalence of the versions, and
(as far as I can tell) don't really draw any consequences out of
hardness for the static versions?

Why study the rank versions of the problem (tensor rank and slice
rank)? They are surely natural choices, especially if positive results
are possible, but consequences (positive or negative) for problems
previously considered would be nice.

In summary, I'm just not convinced that this paper asks the right
questions or achieves the right results.


* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *


Review #12C
===========================================================================

Overall merit
-------------
5. Accept

Reviewer expertise
------------------
2. Some familiarity

Paper summary
-------------
This paper aims to study the complexity of general dynamic programming. In the past few years, the hardness of dynamic programming problems has been significantly advanced within the context of fine-grained complexity theory. Many dynamic programming problems, such as edit distance and longest common subspace, have their fine-grained complexity settled. However, these results are based on case studies of the individual problems. It was unknown how these results are generalized to other dynamic programming problems.

Understanding general dynamic programming hardness has been studied by Kunnemann, Paturi, and Schneider [KPS17]. They give the fine-grained complexity for the least weight subsequence problem, a one-dimensional dynamic programming problem in which the transition function is a low-rank matrix. This result covers the fine-grained complexity of a wide range of one-dimensional dynamic programming problems.

This paper generalizes the work by Kunnemann, Paturi, and Schneider to the high-dimensional case. This paper defines a multi-dimensional least weight subsequence problem, in which the transition function is defined by a high-dimensional tensor. The new problem proposed is general and captures dynamic programming problems such as polygon triangulation. This paper further shows that the fine-grained complexity of this new problem is related to the rank of the high-dimensional tensor. Technically, this paper establishes the fine-grained complexity for low-rank or low-slice-rank high-dimensional tensors.

Overall, the result presented in this paper represents a nice attempt to study the fine-grained complexity of general high-dimensional dynamic programming. The results presented in this paper do not solve the entire problem but give a very good start for the fine-grained complexity for general high-dimensional dynamic programming problems.

Comments for authors
--------------------
The conference name is missed in [KPS17] in the reference section.