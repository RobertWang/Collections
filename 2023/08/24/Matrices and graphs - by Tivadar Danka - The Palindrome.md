> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [thepalindrome.org](https://thepalindrome.org/p/matrices-and-graphs)

> The single most undervalued fact of linear algebra: matrices are graphs, and graphs are matrices

The single most undervalued fact of linear algebra: matrices are graphs, and graphs are matrices.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F485ec4b4-6869-43bb-b17c-e2151a428dbe_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F485ec4b4-6869-43bb-b17c-e2151a428dbe_1920x1080.png)A (nonnegative) matrix and its graph

Encoding matrices as graphs is a cheat code, making complex behavior simple to study.

Let me show you how!

If you looked at the example above, you probably figured out the rule.

Each row is a node, and each element represents a directed and weighted edge. Edges of zero elements are omitted. The element in the _i_-th row and _j_-th column corresponds to an edge going from _i_ to _j_.

The resulting graph is called _the directed graph_ (or _digraph_) _of the matrix._

To unwrap the definition a bit, let's check the first row, which corresponds to the edges outgoing from the first node.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fca509380-4560-4a3b-af33-32aa5b354caa_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fca509380-4560-4a3b-af33-32aa5b354caa_1920x1080.png)The first row corresponds to the edges outgoing from the first node

Similarly, the first column corresponds to the edges incoming to the first node.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F51590d4d-b767-4744-be6c-488771241c0f_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F51590d4d-b767-4744-be6c-488771241c0f_1920x1080.png)The first column corresponds to the edges incoming to the first node

Here is the full picture, with the nodes explicitly labeled.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F863a3c70-91cd-4ab3-b8bb-a0bb4336876e_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F863a3c70-91cd-4ab3-b8bb-a0bb4336876e_1920x1080.png)

Why is the directed graph representation beneficial for us?

For one, the powers of the matrix correspond to walks in the graph.

Take a look at the elements of the square matrix. All possible 2-step walks are accounted for in the sum defining the elements of A².

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7e2865bf-b66d-4231-a4c3-ac7dad59e259_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7e2865bf-b66d-4231-a4c3-ac7dad59e259_1920x1080.png)Powers of the matrix describe walks on its directed graph

If the directed graph represents the states of a Markov chain, the square of its transition probability matrix essentially shows the probability of the chain having some state after two steps.

There is much more to this connection.

For instance, it gives us a deep insight into the structure of nonnegative matrices.

To see what graphs show about matrices, let's talk about the concept of strongly connected components.

A directed graph is strongly connected if every node can be reached from every other node.

If this is not true, the graph is not strongly connected.

Below, you can see an example of both.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Feb16cf74-eef0-4770-9ae9-c31309e310aa_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Feb16cf74-eef0-4770-9ae9-c31309e310aa_1920x1080.png)Strongly connectedness

Matrices that correspond to strongly connected graphs are called irreducible. All other nonnegative matrices are called reducible. Soon, we'll see why.

(For simplicity, I assumed each edge to have a unit weight, but each weight can be an arbitrary nonnegative number.)

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa505eb53-7e3c-4f5a-b612-68d8e96d7567_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa505eb53-7e3c-4f5a-b612-68d8e96d7567_1920x1080.png)Strongly connected digraphs and

Back to the general case!

Even though not all directed graphs are strongly connected, we can partition the nodes into strongly connected components.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F48d7eea9-9e86-4357-8f2e-4c498629bcf5_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F48d7eea9-9e86-4357-8f2e-4c498629bcf5_1920x1080.png)Strongly connected components

Let's label the nodes of this graph and construct the corresponding matrix!

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Faec83a8c-ad93-4c98-a119-81b82e6265d7_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Faec83a8c-ad93-4c98-a119-81b82e6265d7_1920x1080.png)

(For simplicity, assume that all edges have unit weight.) Do you notice a pattern?

The corresponding matrix of our graph can be reduced to a simpler form!

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1b52f2f5-8a40-4c71-a0f3-253a9513e625_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1b52f2f5-8a40-4c71-a0f3-253a9513e625_1920x1080.png)

Its diagonal comprises blocks whose graphs are strongly connected. (That is, the blocks are irreducible.) Furthermore, the block below the diagonal is zero.

Is this true for all nonnegative matrices?

You bet. Enter the Frobenius normal form.

In general, this block-matrix structure that we have just seen is called the Frobenius normal form.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8ec6efcf-1f79-4fdd-94a3-fcd69b5ed54b_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8ec6efcf-1f79-4fdd-94a3-fcd69b5ed54b_1920x1080.png)The Frobenius normal form

If you have a sharp eye with a hint of OCD, you probably noticed that I switched up the colors a bit. From now on, irreducible blocks (a.k.a. matrices that correspond to strongly connected graphs) will be colored maize, while reducible ones will be light blue.

Let's reverse the question: can we transform an arbitrary nonnegative matrix into the Frobenius normal form?

Yes, and with the help of directed graphs, this is much easier to show than purely using algebra.

Here is the famous theorem in full form.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F497133e9-7231-4869-b7a5-1c6db71ef8f0_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F497133e9-7231-4869-b7a5-1c6db71ef8f0_1920x1080.png)

But what are permutation matrices?

Let’s look at a special case: what happens if we multiply a _2 x 2_ matrix with

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1bbc7da0-c56c-40ec-82f1-c4b69dd8d02d_1920x700.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F1bbc7da0-c56c-40ec-82f1-c4b69dd8d02d_1920x700.png)

a simple zero-one matrix? With a quick calculation, you can verify that

1.  it switches the _rows_ when multiplied from the left,
    
2.  and it switches the _columns_ when multiplied from the right.
    

Just like this:

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fda2e85da-26b6-4758-97c3-fb101ffd3e5f_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fda2e85da-26b6-4758-97c3-fb101ffd3e5f_1920x1080.png)

Multiplying with _P_ from both left and right compounds the effects: it switches rows and columns.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F514daaef-77ad-499e-aa72-c1ed8deebd8b_1920x700.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F514daaef-77ad-499e-aa72-c1ed8deebd8b_1920x700.png)

(By the way, this is a similarity transformation, as our special zero-one matrix is its own inverse. This is not an accident; more about it later.)

Why are we looking at this? Because behind the scenes, this transformation doesn’t change the underlying graph structure, just relabels its nodes!

You can easily verify this by hand, but I visualized it for you below.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8c30ce87-490e-4b74-9b57-624dd9f1bb8f_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8c30ce87-490e-4b74-9b57-624dd9f1bb8f_1920x1080.png)

A similar phenomenon is true in the general _n x n_ case. Here, we define the so-called transposition matrices by switching the _i_-th and _j_-th rows of the identity matrix:

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F35090c32-b7d4-47aa-97c1-dd38ab8c9eef_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F35090c32-b7d4-47aa-97c1-dd38ab8c9eef_1920x1080.png)

Multiplication with a transposition matrix has the same effect: switches rows from the left, and switches columns from the right.

We are not going to spell it out in detail (as the overcomplicated notation makes it especially painful), but you can verify by hand that this is indeed what’s going on.

Here is a brief summary. Take note of these, as they’ll be essential in a bit.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe720646d-9f04-46c5-b55f-ddf5880529d5_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe720646d-9f04-46c5-b55f-ddf5880529d5_1920x1080.png)Properties of transposition matrices

The most important property for us: similarity transformations with transposition matrices just relabel two nodes, but otherwise leave the graph structure invariant.

Now, about the aforementioned permutation matrices. A _permutation matrix_ is simply a product of transposition matrices.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F00dd24c5-fdd0-4955-ad1e-19e1ad14700f_1920x700.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F00dd24c5-fdd0-4955-ad1e-19e1ad14700f_1920x700.png)

Permutation matrices inherit some properties from their building blocks. Most importantly,

1.  their inverse is their transpose,
    
2.  and a similarity transformation with them is just a relabeling of nodes that leave the graph structure invariant.
    

To see this latter one, consider the following argument below.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8a6694b1-79d8-44e2-b7dd-f81e8f4cdaf5_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8a6694b1-79d8-44e2-b7dd-f81e8f4cdaf5_1920x1080.png)

(Recall that transposing a matrix product switches up the order, and transposition matrices are their own transposes.)

Conversely, every node relabeling is equivalent to a similarity transformation with a well-constructed permutation matrix.

Why are we talking about this? Because the proper labeling of nodes is key to the Frobenius normal form.

Now, let’s talk about graphs. We’ll see how _every_ digraph decomposes into strongly connected components. For that, let’s see a concrete one.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd6e15334-a3e9-487e-a3c2-dd46b2f530d2_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd6e15334-a3e9-487e-a3c2-dd46b2f530d2_1920x1080.png)

This’ll be our textbook example.

How many nodes can be reached from a given node? Not necessarily all. Say, for the one at the bottom right, only a portion of the graph is accessible.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F59f09d4e-81f1-4ecf-8b27-b396ddfbc5f3_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F59f09d4e-81f1-4ecf-8b27-b396ddfbc5f3_1920x1080.png)

However, the set of mutually reachable nodes is much smaller.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F27748a78-c498-4387-b3dd-6035fe78f4ea_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F27748a78-c498-4387-b3dd-6035fe78f4ea_1920x1080.png)

Algebraically speaking, the relation _“a and b are mutually reachable from each other“_ is an equivalence relation. In other words, it partitions the set of nodes into disjoint subsets such that

1.  two nodes from the same subset are mutually reachable from each other,
    
2.  and two nodes from different subsets are _not_ mutually reachable.
    

(Equivalence relations are the workhorses of mathematics. If you are not familiar with them, check out [this Wikipedia article about partitions](https://en.wikipedia.org/wiki/Partition_of_a_set#Partitions_and_equivalence_relations) to see how they relate.)

The subsets of this partition are called the _strongly connected components_, and we can always decompose a directed graph in this way.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6fd6e5c-bb65-4180-b359-e1e423ffa82a_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc6fd6e5c-bb65-4180-b359-e1e423ffa82a_1920x1080.png)

Now, let’s connect everything together! (Not in a graph way, but you know, in a wholesome mathematical way.)

We are two steps away from proving that every nonnegative square matrix can be transformed into Frobenius normal form with a permutation matrix.

Here is the plan.

1.  Construct the graph for our nonnegative matrix.
    
2.  Find the strongly connected components.
    
3.  Relabel the nodes in a clever way.
    

And that’s it! Why? Because, as we have seen, relabeling is the same as a similarity transform with a permutation matrix.

There’s just one tiny snag: what is the clever way? I’ll show you.

First, we “skeletonize” the graph: merge the components together, as well as any edges between them. Consider each component as a black box: we don’t care what’s inside, only about their external connections.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd4c514c7-6cae-4933-879d-8f08e4a20a5b_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd4c514c7-6cae-4933-879d-8f08e4a20a5b_1920x1080.png)

In this skeleton, we can find that components that cannot be entered from another components. These will be our starting points, the zeroth-class components. In our example, we only have one.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc5eeb894-124d-4607-82f8-3d6f2f1c75e3_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc5eeb894-124d-4607-82f8-3d6f2f1c75e3_1920x1080.png)

Now, things get a bit tricky. We number each component by the longest path from the farthest zero-class component from which it can be reached.

This is hard to even read, let alone to understand. So, here is what’s happening.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F58c9bc48-a037-4a37-b450-fb31e3194094_1920x700.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F58c9bc48-a037-4a37-b450-fb31e3194094_1920x700.png)

The gist is that if you can reach an _m_-th-class from an _n_-th-class, then _n < m_.

In the end, we have the following.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff9d3ca4b-2851-4de3-88c3-c73e09429a72_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff9d3ca4b-2851-4de3-88c3-c73e09429a72_1920x1080.png)Numbering of the classes

This defines an ordering on the components. (A [partial ordering](https://en.wikipedia.org/wiki/Partially_ordered_set), if you would like to be precise.)

Now, we label the nodes inside such that

*   higher-order classes come first,
    
*   and consecutive indices are labeling nodes from the same component if possible.
    

This is how it goes.

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F92bac5d1-bbe7-48e9-802c-f03b5ea3c2b3_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F92bac5d1-bbe7-48e9-802c-f03b5ea3c2b3_1920x1080.png)That ominous clever node labeling

If the graph itself is coming from an actual matrix, such a relabeling yields the Frobenius normal form!

Here is the matrix in this particular example, with zeros and ones for simplicity:

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff23786bf-7cb8-49dd-988c-5773b1194c33_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff23786bf-7cb8-49dd-988c-5773b1194c33_1920x1080.png)The adjacency matrix of our cleverly labeled graph

Now we have everything, and the “proof” behind the transformation to Frobenius normal form is complete!

To remind you, this is the theorem in its full form:

[![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8490f9d7-5bd1-4fa4-8eda-fa909ffcb1c2_1920x1080.png)](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8490f9d7-5bd1-4fa4-8eda-fa909ffcb1c2_1920x1080.png)

What we have seen is just the tip of the iceberg. For example, with the help of matrices, we can define the eigenvalues of graphs! This idea gave birth to the fascinating topic of spectral graph theory, a beautiful field of mathematics.

Utilizing the relation between matrices and graphs has been extremely profitable for both graph theory and linear algebra.

If you are interested in the details, I have two books to recommend to you. One is the fantastic [A Combinatorial Approach to Matrix Theory and Its Applications](https://www.taylorfrancis.com/books/mono/10.1201/9781420082241/combinatorial-approach-matrix-theory-applications-richard-brualdi-dragos-cvetkovic) by Richard A. Brualdi and Dragos Cvetkovic, my favorite learning resource on the topic.

Second, I recently finished writing my [Linear Algebra for Machine Learning](https://www.tivadardanka.com/books/linear-algebra-for-machine-learning) book. Although the contents of this post are not yet a part of this, the good thing about the ebook format is that I can always go back and add a new chapter here and there.

So, I am planning to expand this post into a textbook-like chapter, possibly with interactive code examples along the way. If you are interested, [check out the first two chapters of my book](https://tivadardanka.com/linear-algebra-for-machine-learning-preview/)!

### Subscribe to The Palindrome

Mathematics for engineers, scientists, and other curious minds. Explaining things like your teachers should have, but probably never did.