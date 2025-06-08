---
layout: post
title: Removing COCOTB_RESOLVE_X
---

`COCOTB_RESOLVE_X` is dead.[^2] Good riddance, it was a terrible idea; now let's discuss why.

### First, an aside for philosophy

`X` values in (System)Verilog are used for numerous, conflicting purposes.
This makes the syntax for the language simpler, condensing multiple idioms into one syntax.
However, as is often the case when doing this, you end up being able to write syntactically-valid nonsense.
Consider the English phase "How blue dogs?"

Language *encodes* ideas.
Syntax is nothing without semantics.
Ideas are real; *language is not*.

This train of thought applies to `X` as well.
`X` is an encoding of the several idioms and intended use cases we are about to discuss.
Other uses are probably nonsense.
The idioms are real; *`X` is not*.

This is important to harp on, as many people who are not exposed to idioms and intended use case *first*,
tend to see only the physical rendering of those ideas and see *that* as real.
Something something "Baudrillard" something "reification".

Without first being presented with the concept of a screw and screwdriver,
a Neanderthal when presented with a screwdriver may use it (as is often done in times of frustration when failing to find the proper tool)
as a digging tool or a box opener.

This is actually commonplace in programming.
Ask your programmer friends what they are doing and see how many people will tell you they are in the process of writing "a class" (there is no such thing).
And this often leads them to abusing the syntax to do incorrect, unidiomatic, fragile, non-composable, etc. etc. all-the-bad-words things.

The material of `X`, as with all material things, has no inherent meaning.
It is up to us to make it meaningful.
So let's do so.

### Language constraints

When ascribing meaning, it's useful to first see the capability of a thing's physical nature.
We can do whatever we want, insofar as physical reality lets us.

There are 3 uses of `X` already encoded into the language we can't ignore.

1. `X` is the value of signals that have not been initialized.
2. `X` is the value of signals that are multiply-driven with different values.
3. `X` is used to describe a "match all" in logic.

When considering the third case, it is important to note the *exact* wording used:

> `X` is used to describe a "match all" ***in logic***.

That is to say, when using things like `casex`, the `X` belongs in the label (the part that will eventually render to gates), *not* on the data.

So the uses of X encoded into the language seem to imply that `X` on data at runtime is indicative of an error.
What about other uses of `X` that aren't necessarily prescribed meaning in the language spec,
but are in common use?

### `X` for synthesis

Synthesizers use `X` as "don't care".
`X` acts as a stand-in for either valid value: `0` or `1`.
This gives the synthesizer the liberty to generate whatever logic it sees fit;
but obviously optimizing for for performance.
Designers often want to take advantage of those optimizations, which leads us to the following uses of `X`.

1. Constant assignment to unused variables so they are optimized out.
2. To provide a value in an expression in unreachable paths.
3. To provide a value in an expression where the value doesn't matter ("don't care").

It's important to note that in case 2 and 3 that a signals' value changes over time and can sometimes be `0` or `1` and other times `X`.
These are all examples of `X`s on data signals that aren't an error in synthesis.
But we are talking about verification, and verification is not synthesis.
In fact, synthesis can make these assumptions *because* verification has been done.

### `X` for verification

So what's the point of verification?

> To ensure the design works correctly and ensure it doesn't work incorrectly.

What that means is we care about errors.
The more errors we can catch, the better.
Synthesis might care about people using `X` to mark unused variables and "don't care" situations for optimization,
but verification has different concerns.
Namely, `X` is useful for catching errors.
Case 1 and 2 of the language spec do that explicitly, and we can't really ignore those cases.
And case 1 and 2 of the synthesis use case are likely also indicative of error if seen in simulation.

The heartburn for using synthesis semantics for `X` in verification comes with case 3.
In those cases, `X` is put on a signal that may be read during verification,
but instead of being an error it means: "Any valid value, I don't care".

This makes `X` completely worthless in simulation when using the same semantics as synthesis.
It's both a valid value and an error and there's no real way to disambiguate.
Better to just ignore it...

### So what's the problem with COCOTB_RESOLVE_X?

Well this should be pretty obvious now.
It was designed to automatically convert `X`s in `BinaryValue`s into `0`, `1`, or a random value.
Clearly this was created in reaction to the above confusion.

Is it a multiple driver error?
A "don't care" value?
Bad reset logic leading to uninitialized values propagating through your design?
Who cares?! Ignore it!
...And that's why it was removed.

### A better solution

The only issue is support for "don't care"s.
What's needed here is a `0` or `1` in simulation and `X` in synthesis.

A quick mock up...

```
#if VERIFICATION
#define DONT_CARE '0
#else if SYNTHESIS
#define DONT_CARE 'x
#endif

// usage
data <= DONT_CARE;
```

We handle both designer intent and the value of `X` properly in both synthesis and verification.
What's not to love?

### Footnotes

[^2]: [I'm not dead yet!](https://github.com/cocotb/cocotb/pull/4253)
