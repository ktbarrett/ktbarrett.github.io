---
layout: post
title: Removing COCOTB_RESOLVE_X
---

`COCOTB_RESOLVE_X` is dead. Good riddance. It was a terrible idea. Let's discuss why.

### First, an aside for philosophy.

`X` values in (System)Verilog are used for numerous, distinct purposes.
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

Yeah, that's some Matrix bullshit.

> "Try to realize the truth... there is no spoon."

But it's true...

This is important to harp on, as many people who are not exposed to the idioms and intended use case *first*,
tend to see only the physical rendering of those ideas and see *that* as real.
Without first being presented with the concept of a screw and driver,
a Neanderthal when presented with a screwdriver may use it (as is often done in times of frustration when failing to find the proper tool)
as a digging tool or box opener.

This is actually commonplace in programming.
Ask your programmer friends what they are doing and see how many people will tell you they are in the process of writing "a class" (there is no such thing).
And this often leads them to abusing the syntax to do incorrect, unidiomatic, fragile, non-composable, etc. etc. all-the-bad-words things.

### So when should you use X?

X is used...
1. As the value of signals that have no been initialized.
2. As the value of signals that are incorrectly multiply-driven.
3. To encode "don't cares" in logic.

The first two idioms are there so if you see an `X` as the value of a signal during a simulation,
there's a good chance something in your design is *wrong*.
`X` exists for catching errors.
Either your reset logic is incorrect and uninitialized (random) values are flowing through your design,
or you are incorrectly multiply-driving a signal.

When considering the third idiom, it is important to note the *exact* wording used:

> To encode "don't cares" ***in logic***.

That is to say, when using things like `casex`, the `X` belongs in the label (the part that will eventually render to gates), *not* on the data.
If you are using `X` as a "don't care" in the data flowing through your design,
there ends up being no way to distinguish it from an uninitialized or multiply-driven signal value.

### So what's the problem with COCOTB_RESOLVE_X?

Well this should be pretty obvious now.
It was designed to automatically convert `X`s in `BinaryValue`s into `0`, `1`, or a random value.
In doing so it acts to *globally* ignore all errors in your design.
How useful...

I'm glad it's dead and hopefully its death will result in users finding hidden bugs in their design.
