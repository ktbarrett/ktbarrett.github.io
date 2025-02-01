---
layout: post
title: Protected is Public
---

For context, I mean `protected` with respect to the keyword in C++.

I've heard a number of people say things along the lines of "`protected` is bad" or "`protected` breaks encapsulation and leads to coupling"
and I'm here to tell you that these people don't know what they are talking about.
Don't get me wrong, I think the whole class-based encapsulation thing that C++ does is super cringe,
but there's nothing wrong with `protected` if you buy into the rest of it.
The real problem is that people don't understand it and use it wrong.

`protected` *is* public.

That should demystify my position, but assuming it doesn't, let me explain.

### Scrutiny

`protected` marks a class member as being visible to classes which inherit from it, but not to "user" code.
This is opposed to `private` where the member **is not** visible to both users and heirs and `public` where the member **is** visible to both users and heirs.

The whole system seems to be designed to create built-in support for two contextual interfaces for a class;
one being "user" and the other "heir".
You could complain this is too simplistic, there are *many* possible contextual interfaces for a class,
but these two are extremely common so they get official recognition[^1].

### Synthesis

So it's best to think of `protected` as `public`, but in the context of the heir.
And this understanding is what dispels most of the bad arguments against `protected`.

> It breaks encapsulation!

Only if you let it...
Exposing data members publicly?
Yeah, of course that breaks encapsulation; so don't.

> It creates coupling!

Users are supposed to use public API?
I don't get how this is an argument.
Unless you want to complain that all use of public API creates coupling.
You'd be correct, it does, but that's an inevitability.

### Consequence

Most programmers understand that carefully designing a public API is important.
You are going to have to live with it for a very long time.
You can't just break it willy-nilly.
However, most people don't seem to apply the same caution to `protected`.

Start.

### Further Thoughts

Of course, that's easy to say and ignores the obvious fact that `protected` is *rarely* used (in my experience) to encode a well-crafted public API for heirs;
but instead exists to share implementation with subclasses written by the same author as the parent class,
often in the same file.
This underscores the larger conceptual issue with class-based encapsulation as a way to encode encapsulation that I alluded to earlier.
There is no easy way for implementors to simply share all the private details of a class with other classes they have written in the same library without either using `protected`, or by making everything `private` and making all the classes `friend`s[^2].

What many other languages use is module/package-based encapsulation.
For example, in Java, the default privacy mode is "package private".
This means members appear `private` to users of the package,
but appears public to all code within the package.
Translating to C++... this is like making all classes and functions in a package `friend`s.
The recommendation in Java is to *only* use `public` and package private[^3].

A package is usually implemented and maintained by a single entity (often just one person),
so there's no need to hide details about the implementation from itself.
If a codebase gets so big one needs to employ encapsulation to make it manageable,
maybe the codebase should be multiple packages?

I bring this up because package/module-based encapsulation is probably a more coder-friendly solution to encapsulation than class-based.
With C++20's modules, "package private" class members could become possible in the future, though I don't see any interest in it right now.

#### Footnotes and Asides

[^1]: They are also the only zero-cost contextual interfaces. If you want more, you'll need to write access adapters or build interface classes. Both may cost CPU cycles. Bad C++! Not providing "zero-cost abstractions" like you promised.

[^2]: Actually... this might be a better recommendation for C++ *right now*.

[^3]: And perhaps `protected` as well if you are intending to provide a base class for users that needs a well-crafted API.
