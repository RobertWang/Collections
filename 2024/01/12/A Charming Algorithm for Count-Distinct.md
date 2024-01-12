> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [justinjaffray.com](https://justinjaffray.com/a-charming-algorithm-for-count-distinct/)

> 26 Jan 2023

26 Jan 2023

I recently came across a paper called [Distinct Elements in Streams: An Algorithm for the (Text) Book](https://arxiv.org/abs/2301.10191) by Chakraborty, Vinodchandran, and Meel.

The usage of the phrase “from the book” is of course a reference to Erdős, who often referred to a “book” within which God kept the best proofs of any given theorem. Thus, for something to be “from the book” is for it to be particularly elegant. I have to say, I agree with their assessment. This is an extremely charming little algorithm that I really enjoyed thinking about, so today, I’m going to explain it to you.

The count-distinct problem is to estimate the number of distinct elements appearing in a stream. That is, given some enumeration of “objects,” which you can think of as any data type you like, we want to know approximately how many _unique_ objects there are. For instance, this array:

```
[1,1,2,1,2,3,1,2,1,2,2,2,1,2,3,1,1,1,1]


```

has only three distinct objects: `1`, `2`, and `3`. It’s pretty natural to want to know how many distinct objects appear in such a list. Unfortunately, if you require the actual number, there’s basically only two options:

*   sort the list, or
*   use a hash table.

Both of these options require memory proportional at least to the number of distinct elements, which in some cases could be as big as the entire stream. This might be fine for some smaller data sizes, but if we want to handle millions or billions of elements, that’s a nonstarter for many use cases.

It turns out that if we can tolerate some imprecision, and we often can, there are ways we can vastly reduce the amount of memory we need by using approximate algorithms.

The most well-known approximate count-distinct algorithm is [HyperLogLog](https://en.wikipedia.org/wiki/HyperLogLog), which is widely used in production for all sorts of things. While the idea behind HyperLogLog is simple, the analysis of it is somewhat complex. What this paper provides is an alternative algorithm which:

1.  has simpler analysis, and
2.  doesn’t rely on hashing at all.

The paper provides proofs around the algorithm’s correctness, so I’m just going to explain how it works by way of derivation; one lovely thing about this algorithm is that we can build up to it very naturally.

The most obvious solution to the count-distinct problem is to just maintain a hash table of the objects you’ve seen, and to emit its size at the end:

```
function countDistinct(list) {
    let seen = new Set();
    for (let value of list) {
        seen.add(value);
    }
    return seen.size;
}

console.log(countDistinct([
    "the", "quick", "brown", "fox", "jumps", "jumps", "over",
    "over", "dog", "over", "the", "lazy", "quick", "dog",
]));
// => 8


```

This will have to store every element we’ve seen. If we’re trying to save memory, one obvious trick to try is to just not store everything.

If we attempt to only store half the values, then the expected size of `seen` should be half the actual number of distinct elements, so at the end we can just multiply that size by two to get an approximation of the number of distinct elements.

When we see an element, we flip a coin, and only store it if the flip is heads:

```
function countDistinct(list) {
    let seen = new Set();
    for (let value of list) {
        if (Math.random() < 0.5) {
            seen.add(value);
        }
    }
    return seen.size * 2;
}

console.log(countDistinct([
    "the", "quick", "brown", "fox", "jumps", "jumps", "over",
    "over", "dog", "over", "the", "lazy", "quick", "dog",
]));
// => 10


```

well, this is actually wrong, because if we see the same element multiple times, we’re more likely to have it in our final representative set:

```
console.log(countDistinct([
    "a", "a", "a", "a", "a", "a", "a",
    "a", "a", "a", "a", "a", "a", "a",
    "a", "a", "a", "a", "a", "a", "a",
    "a", "a", "a", "a", "a", "a", "a",
]));
// => 2 (with very high probability)


```

the _number of times_ an element appears shouldn’t impact the output of our algorithm (this is sort of the defining property of count-distinct, I’d say).

There’s an easy fix for this though: when we see an element, we can just remove it from the set before flipping, so the only coin flip that actually matters is the last one (which works out, because every element that appears at least once has exactly one final appearance):

```
function countDistinct(list) {
    let seen = new Set();
    for (let value of list) {
        seen.delete(value);
        if (Math.random() < 0.5) {
            seen.add(value);
        }
    }
    return seen.size * 2;
}


```

In this iteration, every distinct element appears in `seen` with probability 0.5.

We can improve the memory usage even further (at the cost of precision) by requiring each element to win more coin flips to be included in the final set:

```
function countDistinct(list, p) {
    let seen = new Set();
    for (let value of list) {
        seen.delete(value);
        if (Math.random() < p) {
            seen.add(value);
        }
    }
    return seen.size / p;
}

console.log(countDistinct([
    "the", "quick", "brown", "fox", "jumps", "jumps", "over",
    "over", "dog", "over", "the", "lazy", "quick", "dog",
]), 0.125);


```

Now each element is included in the final set with probability `p`, so we divide by `p` to get the actual estimate.

We’ve reduced our memory usage by some constant factor, and that’s maybe good! But it’s not any better asymptotically, and more importantly, it doesn’t let us _bound_ the amount of memory usage: I can’t tell you ahead of time how much memory I’m going to use for this.

The final trick to get us to the actual algorithm is to pick `p` dynamically.

That is, we start with a `p` of 1, and have a threshold for how big is “too big.” If our set grows beyond this size, we “upgrade” `p` so that elements now have to win an additional coin flip to be included in the final set. When we upgrade `p` this way, we have to do two things:

1.  ensure _future_ elements are subject to the new filter, by updating the variable `p`, and
2.  ensure _old_ elements are subject to the new filter, by forcing them to win an additional coin-flip on top of what they won before.

Since elements that have already “won” and are included in `seen` need to be held to this new standard, we have to do a Thanos-snap and have them each win an additional coin-flip in order to stay in the set.

At the end of the day, we still have a set that contains elements with probability `p`, so we can divide its size by `p` to get the true estimate.

The final algorithm looks like this:

```
function countDistinct(list, thresh) {
    let p = 1;
    let seen = new Set();
    for (let value of list) {
        seen.delete(value);
        if (Math.random() < p) {
            seen.add(value);
        }
        if (seen.size === thresh) {
            // Objects now need to win an extra coin flip to be included
            // in the set. Every element in `seen` already won n-1 coin
            // flips, so they now have to win one more.
            seen = new Set([...seen].filter(() => Math.random() < 0.5));
            p *= 1 / 2;
        }
    }
    return seen.size / p;
}


```

That’s the whole algorithm—the paper contains an actual analysis, as well as guidance for picking a value of `thresh` for a desired level of precision.

It’s not really clear to me if this algorithm is appropriate for real-world use. A comparison with HyperLogLog is notably absent from the paper. My immediate suspicion is that it’s not, really, since HyperLogLog has some additional nice properties (for instance, it distributes very well, due to sketches being mergeable) and it’s not clear to me whether they’re preserved here.

Asking that question is sort of missing the point, though, of course, since like the authors emphasize, the appeal of this algorithm is its simplicity, and to me, the surprise of its existence—I actually had no idea it was possible to do efficient count-distinct without hashes, but it turns out it is!