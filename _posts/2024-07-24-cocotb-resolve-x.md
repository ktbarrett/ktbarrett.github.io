---
layout: post
title: Removing COCOTB_RESOLVE_X
---

`COCOTB_RESOLVE_X` is dead.[^2] Good riddance. It was a terrible idea. Let's discuss why.

### First, an aside for philosophy

`X` values in (System)Verilog are used for numerous, conflicting purposes.
This makes the syntax for the language simpler, condensing multiple idioms into one syntax.
However, as is often the case when doing this, you end up being able to write syntactically-valid nonsense.
Consider the English phase "How blue dogs?"

Of course, we all grew up communicating ideas in spoken language.
Some of us (not me lol) even know more than one language.
And that really drives home the point that language *encodes* ideas.
Syntax is nothing without semantics.
Ideas are real; language is not.

This same train of thought applies to `X` as well.
`X` is an encoding of the several idioms and intended use cases we are about to discuss.
Other uses are probably nonsense.
The idioms are real; *`X` is not*.

Yeah, that's some Matrix bullshit.[^1]

> "Try to realize the truth... there is no spoon."

This is important to harp on, as many people who are not exposed to the idioms and intended use case *first*,
tend to see only the physical rendering of those ideas and see *that* as real.
Without first being presented with the concept of a screw and screwdriver,
a Neanderthal when presented with a screwdriver may use it (as is often done in times of frustration when failing to find the proper tool)
as a digging tool or a box opener.

This is actually commonplace in programming.
Ask your programmer friends what they are doing and see how many people will tell you they are in the process of writing "a class" (there is no such thing).
And this often leads them to abusing the syntax to do incorrect, unidiomatic, fragile, non-composable, etc. etc. all-the-bad-words things.

The material of `X`, as with all material things, has no inherent meaning. It is up to us to make it meaningful.

### Language constraints

When ascribing meaning, it's useful to first see the capability of a thing's physical nature.
We can do whatever we want, insofar as reality allows us.

There are 3 uses of `X` already encoded into the language we can't ignore.

1. As the value of signals that have not been initialized.
2. As the value of signals that are multiply-driven.
3. As a "match all" in logic.

When considering the third idiom, it is important to note the *exact* wording used:

> As a "match all" ***in logic***.

That is to say, when using things like `casex`, the `X` belongs in the label (the part that will eventually render to gates), *not* on the data.

So the uses of X encoded into the language seem to imply that `X` on data at runtime is indicative of an error.
What about other uses that aren't necessarily part of language, but are a part of how it's used?

### `X` for synthesis

Synthesizers use the presence of `X` values as a liberty to generate whatever logic they see fit.
Often the idea is to generate the most efficient logic in those cases.
Designers often want to take advantage of those optimizations, which leads us to the following uses of `X`.

4. Constant assignment to unused variables so they are optimized out.
5. To provide a value in an expression in unreachable paths.
6. To provide a value in an expression where the value doesn't matter.

It's important to note that in Case 5 and 6 that the value is sometimes `0` or `1` and sometimes `X`.
These are all examples of `X`s on data signals that aren't an error.
But these examples are valid during synthesis.
But, verification is not synthesis.

### `X` for verification

What's the point of verification?

> To ensure the design works correctly and ensure it doesn't work incorrectly.

What that means is we care about errors.
The more errors we can catch, the better.
Synthesis might care about people using `X` to mark unused variables and "don't care" situations for optimization,
but verification has different concerns.
Namely, `X` is useful for catching errors.
Case 1 and 2 explicitly do that and we can't really ignore those cases.
Case 4 can be leveraged to catch people using "unused" variables.
And Case 3 doesn't conflict with that goal, as its only for describing static values.

The heartburn for using synthesis semantics for `X` in verification comes with Case 5 and 6.
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

And that's why it was removed.
It existed to be a work around for a shitty solution.

### A better solution

The only issue with using all the listed semantics in verification are Case 5 and 6.
What's really meant here is: "Any valid value, I don't care".
So why not supply a random valid value, `0` or `1`?

A quick mock up...

```
#if VERIFICATION
#define DONT_CARE randomint(0, 1)
#else if SYNTHESIS
#define DONT_CARE x
#endif
```

We preserve both designer intent and the value of `X` for catching errors.
What's not to love?

### Footnotes

[^1]: Well Post-Structuralist as inherited from Phenomenologist as inherited from Sophic and Pythagorean bullshit.

[^2]: [I'm not dead yet!](https://github.com/cocotb/cocotb/pull/4253)
