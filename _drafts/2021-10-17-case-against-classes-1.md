---
layout: post
title: A Case Against Classes, Part 1
---

I was first inspired to think more about the topic of object-oriented programming,
and the language feature central to the concept: classes,
after watching some of Kevlin Henney's ***excellent*** talks:
[*The Forgotten Art Of Structured Programming*](https://www.youtube.com/watch?v=SFv8Wm2HdNM)
and [*Old Is The New New*](https://www.youtube.com/watch?v=AbgsfeGvg3E),
late last year.
I'll be drawing some ideas from both of these talks over the series.

This is the first in a series of articles on the problems with the common language features of "classes" and "inheritance" as they exist in modern "object-oriented" programming languages.
The series will analysis what classes and inheritance bring us,
judge how well they accomplish that,
find what is missing,
and suggest alternatives.
I don't think any of these ideas are novel, but they deserve some extra recognition, and I'm bored...

## What The Hell is Object-Oriented Programming?

I have heard many definitions of object-oriented programming.
Some definitions mention specifically the language features of classes and inheritance.
Other definitions try to emphasize the patterns and functionality brought about by classes and inheritance, and de-emphasize the mechanism.
However, the result of doing that is the OOP becomes a collection of good ideas that is no more than the sum of its parts.

Often, I hear the following as aspects of OOP that are mechanism-agnostic:
* polymorphism
* implementation reuse
* encapsulation

These are aspects of any good language.
Polymorphism enables writing generic code, decreasing repeated code, and making projects more maintainable.
Implementation reuse is another great way to decrease repeated code.
Encapsulation is abstraction,
allowing users to write their code against a well-defined public interface and unload implementation details from their working memory,
making programs easier to reason about,
leading to less programming errors on the initial implementation,
and increasing the complexity of tasks that are workable by a programmer.

*However*, all of those aspects are achievable without classes and inheritance.
There are languages that support all these aspects that many would *not* called "object-oriented".
This seems, to me, to re-enforce the idea that OOP really is just classes and inheritance.
So, let's be honest with each other.
I think we can, and should, throw the term "object-oriented programming" in the garbage bin.
It's an obfuscation on "classes and inheritance" that brings with it mysticism and a pretention of higher value.
Once we eliminate the mysticism and pretention, we can analyze the language features of classes and inheritance on their own.

## Classes Are Overloaded

Kevlin talks about classes and objects in both talks in slightly different contexts:
objects as captured block structures in *The Forgotten Art Of Structure Programming*,
where classes are just factories;
and objects as a generalization of purely reactive processes which pass messages to one another in *Old Is The New New*,
based on the history of early OOP starting with Simula running through the Smalltalk family of languages.

Immediately, we can note that, between the two talks, classes and objects are discussed in *multiple contexts*.
This is because the class is a language feature which can be used to encode multiple patterns.
That sounds "neat", and does make the language "simpler" by providing less language feature the programmer must learn to be effective;
however, it is actually a negative.
Programmers who cannot separate patterns from implementation,
or who are simply ignorant to the many patterns classes are used to encode,
will conflate the patterns and produce a muddled design that will be hard to use and/or modify.  

They aren't making a reactive process with explicitly-held process state.  
They aren't making a datatype with encapsulated implementation.  
They aren't making a protocol and implementation for runtime generic programming.  
They aren't using one of the many design patterns described in [GoF](https://en.wikipedia.org/wiki/Design_Patterns) (and many of the listed patterns could be seen to be way to (ab)use classes to work around expressivity issues in the host language).  
They are "writing a class", whatever that means...

One could argue that it's just a matter of education and vigilance.
That same argument could be applied to a number of programming language features or idioms that are considered faux pas, see `goto`;
so I'm not sure how much merit that argument has.
Unlike reality, we can fix the problem by ripping out the transgressor and not giving people the option of doing it the wrong way.
And we have done this multiple times to good effect, see "structured programming".
So let's do that with classes and inheritance.

The next few articles will go over alternatives to classes for many patterns and compare them to alternatives.
Ideally, it will show the alternatives are as expressive or more than the class-based implementation.
We will cover:
* Reactive Processes and Messaging
* Encapsulation
* Subtyping
* Implementation Reuse
* Subtype Polymorphism
* Structural Subtyping
