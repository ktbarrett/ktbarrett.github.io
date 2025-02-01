---
layout: post
title: stderr Is A Bad Name
---

I haven't thought about `stdin`, `stdout`, and `stderr` much until recently.
One would think it's pretty straightforward:
`stdin` is program input, `stdout` is program output, and `stderr` is for error messages.
That's what I was told during college...
But let's dig in and see what we find.

## What is stderr for, really?

`stderr` was likely introduced following the creation of the Unix pipeline.
Unix pipelines enable using programs as *functions* which transform input (`stdin`) into output (`stdout`).

Now imagine you are piping two programs together and you get error messages from the first program being fed into the input of the second program.
It becomes the second program's job to separate the *programmatic* output (what it cares about) from the *diagnostic* information of the previous program.
This is impossible.
Instead, output streams are split into *programmatic* output (`stdout`) and *diagnostic* output (`stderr`).
This allows Unix pipes to pipe only the *programmatic* output into the following program while the *diagnostic* information is communicated to the user.

## Are error messages the only kind of control information?

Lets say we have a program that acts as a function, something like `cut`.
Now imagine for a second this version of `cut` has implemented verbosity levels (`-vvvvvvvvvvvv`, `cut` is super complicated) so you can turn on various diagnostic information.
What output does that diagnostic information go out on? `stdout`?
It can't be `stderr` because it's not an error message, just like they said in college, right?
Well obviously that's a little silly, we still don't want *any* debugging prints going to the next program in the pipeline.
Consequently, we should consider that `stderr` is for *all* diagnostic messages, not just for error messages.

If you want an appeal to authority see Section 7.19.3/7 of the C99 standard.
`stderr` exists in many languages,
but arguably C has the greatest claim on the intent of the design seeing as C was the language of the Unix system.
Emphasis mine.

> At program startup, three text streams are predefined and need not be opened explicitly
> — standard input (for reading conventional input), standard output (for writing conventional output),
> **and standard error (for writing diagnostic output)**.

Perhaps `stderr` was a bad name...
