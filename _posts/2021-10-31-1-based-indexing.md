---
layout: post
title: Indexing Base-ics 
---

Recently I became aware of [Edsger Dijkstra's opinion on range notation, iteration, and 0-based indexing](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD08xx/EWD831.html).
Compared to most other opinions I've heard on why 0-based indexing is good,
this one actually attempted to make sense.
Unfortunately for Dijkstra, he made a careless assumption,
and as a result he is incorrect.
However, his points are well argued,
and if we correct the mistake it turns into a very good argument for *1-based* indexing.
If you want to read his opinion before I continue, take the chance now.

His arguments starts by talking about conventions of describing ranges.
Ranges could have an inclusive or non-inclusive left bound and an inclusive or non-inclusive right bound.
His first argument is that it is natural for the length of range to be the difference between the first and last element.
This, is an overspecification in my opinion.
It's convenient, but not in anyway necessary.
He does not go into why he holds this opinion.
And this opinion is what leads to his incorrect conclusion that undermines his whole argument.

Next he correctly identifies the problem with non-inclusive left bounds:
you cannot include the lowest natural number in *any* range, since that would require the non-inclusive left bound to be unnatural.
However, he selectively ignores this same problem on the upper bound!
He instead dismisses inclusive rights bounds on the basis of the previous arbitrary requirement `(end - start) == length`.

Using a non-inclusive right bound requires that the value just beyond the end of the range be a representable value.
This is not always possible.
This is a bigger problem than Dijkstra seems to appreciate.
In C, this occurs when you wish to iterate *including* `INT_MAX`.
And would occur in Ada with every enumerated type,
if Ada did not correctly identify this problem and design against it.

With the conclusion that ranges should be described using non-inclusive right bounds,
he then justifies 0-based indexing by stating that iterating over N elements should have the resulting range end in N.
This is to prevent off-by-one errors that might be caused by requiring people to end ranges with N-1 instead.
I agree with this totally.
I work on VHDL codebases (VHDL has inclusive right bounds) where 0-based indexing is used often:
`sig'length - 1` is on every other line of code.
However, sometimes people don't get this indexing logic correct and end up with off-by-one errors.

So if you consider Dijkstra's argument for non-inclusive right bounds faulty,
and his argument for 0-based indexing based on that conclusion legitimate,
then correcting for his faulty conclusion reveals an argument for 1-based indexing.
That is: with inclusive right bounds, where the right bound of an N length sequence should be N, indexing must start at 1.

If you want an example of this, look no further than VHDL's `ieee.std_logic_1164` library.
Functions that return `std_logic_vector`s are indexed `1 to length`.
