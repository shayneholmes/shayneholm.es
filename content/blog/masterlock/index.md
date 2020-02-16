---
title: "Making Masterlocks"
date: 2020-02-15T20:15:47-07:00
categories: [puzzles, algorithms]
slug: making-masterlocks
resources:
- name: lock
  src: masterlock-1535dwd.jpg
  title: Master Lock model 1535DWD
markup: mark
---
{{< figure src="lock" class="float-right" >}}

Masterlock makes a few models of four-cylinder combination lock where each cylinder is a ring of ten letters. The combination has to feature one letter from each cylinder in turn, and in order. It's popular with escape room aficionados, since you can theme the combination by setting it to a relevant word. (It comes preset to "LOCK".) After I recently purchased one, I immediately made a list of all the English words I could use on it.

But that's not the problem I'll cover here.

The _real_ problem is in pretending you're the person putting the letters on it: Given the structural constraints, which configuration of letters enables the largest number of valid English words?

# Prior art

Let's first look at what Masterlock actually produced. The lock I bought has this configuration of letters:

```
Ring 1: L,N,B,D,M,J,P,R,S,T
Ring 2: O,U,Y,R,T,L,H,A,E,I
Ring 3: C,D,E,O,R,S,T,L,N,A
Ring 4: K,Y,R,S,T,L,N,E,D,H
```

First, since we'll consider a bunch of different letter configurations in this post, let's come up with some better notation. To make describing them less wordy, I'll write each letter configuration out on one line:[^notation]

[^notation]: This notation is conveniently also a regular expression that matches all the words creatable with that configuration.

```
[lnbdmjprst][ouyrtlhaei][cdeorstlna][kyrstlnedh]
```

To make it easier to compare configurations (and detect duplicates), I'll order each ring's letters lexically, since the order of the letters in the ring doesn't matter to the current problem:

```
[bdjlmnprst][aehilortuy][acdelnorst][dehklnrsty]
```

Okay, we wrote it out. But how many words can we make with it?

This is pretty simple to compute, provided you (A) have a list of all English words and (B) know your regular expressions.

Throughout this doc, I'll use a word list that I've had for years: the [2of12inf list][2of12inf] from the venerable [SCOWL project][SCOWL]. To find all the words that are possible with this lock configuration, one can use `grep`:

```
grep '^[bdjlmnprst][aehilortuy][acdelnorst][dehklnrsty]$' 2of12inf.txt
```

This yields 740 words that I can make with my lock. Pretty neat!

So that's the starting point, and I'll report all the configurations in this doc with their word counts to make it easier to compare:

```
[bdjlmnprst][aehilortuy][acdelnorst][dehklnrsty] 740
```

[2of12inf]: http://wordlist.aspell.net/12dicts-readme/#2of12inf
[SCOWL]: http://wordlist.aspell.net/

# Naiveté

A simple way to try is to count the letter frequencies in each position for all the four-letter words in the list, and then choose the ten most popular letters in each position.

This ignores knowledge of things like digrams ('g' is popular on the fourth ring only if 'n' is on the third) but it already does better than the existing configuration:

```
[bcdfhlmpst][aehilnoruy][aeilmnorst][deklmnpsty] 854
```

We're better than the stock configuration by over a hundred words, including beauties like "FADE". Not bad.

Now that we've trivially shown that we can do better than the existing choices, let's actually try and get the best answer.

# Mapping the territory

Let's first examine the _size_ of the problem: Each ring can have any set of 10 unique letters out of the 26, for a total of 5,311,735 possibilities. That's a pretty big number, but tractable. Unfortunately, since each ring is independent of the others, we raise that number to the fourth power, resulting in about 7.96⨉10<sup>26</sup> configurations. That is a lot gnarlier of a problem: Even if we could evaluate one configuration per nanosecond, it would take 25 billion years to go through them all.

Let's find a way to tame this problem.

There aren't any overlapping subproblems here that we can use to reduce the solution space. Each ring's ideal set of letters is dependent on the letters already chosen for the other rings: If the last ring doesn't include "S", you can't count "MASS" as a candidate for the other rings. That makes dynamic programming of little use here.

One thing that _is_ helpful is the observation that the list of valid words is much smaller than all combinations of letters: My wordlist includes about 2,500 four-letter words, compared to 26<sup>4</sup>=456,976 combinations of four letters. So the wordlist itself can be viewed as a sparse matrix representation of the full four-dimensional word space.

# Local optimization

My personal best here is a variant of the simplex method: Starting at a given configuration, hold three of the rings constant and find the best combination for the remaining ring. It's not too hard to run through the wordlist and count the words we could make with each letter. For example, if the first three rings were "ht-ae-sv", some valid words might be "have, hast, test", making "t" and "e" are our best options for the fourth ring.

In fact, we can evaluate this simultaneously for all four of the rings: For each ring, imagine holding the other three rings constant, and decide which letters on the remaining ring would give us the biggest set of words. Then we can choose to change the single ring that gives us the biggest set. And then we do it again, and again, stopping only when we can't change any single ring to get more words. This point is a local maximum.

This local search would be guaranteed optimal for a convex problem, which this problem isn't. But we can try and "get a feel" for the solution space by starting several local searches at random points throughout the space, seeing if the local searches mostly converge to the same results.

In my tests, the randomly-seeded local searches converge on the same solution about 60% of the time, and there are other local maxima that are not quite as high. The solution is quick enough that I've done tens of thousands of hill climbs. That being said, there could be a better solution that I just haven't happened upon yet -- stochastic methods are tricky that way.

Here is my best result using this method:

```
[bcdfhmpstw][aehilopruw][aceilnorst][deklmnpsty] 876
```

# Closing thoughts

Masterlock could have been optimizing for something other than raw word count. Indeed, they _must_ have been, given how easy it was for us to outperform them. Here are some potential complications that they may have considered:

- Word frequency: Customers probably prefer common words ("LOCK" over "PATE")
- Word type: Customers may prefer combinations that are nouns ("BOOK" over "GOOD")
- Blacklisted words: Masterlock may not want to let customers choose certain words (obscenities, or competitors' names like "ABUS"), and so may intentionally choose a suboptimal configuration to avoid those words

We could include this in our algorithm with custom cost functions, but that's not an area I explored.

If you're just going for raw word count, there are several better options than what comes in the box.
