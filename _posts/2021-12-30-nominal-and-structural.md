---
layout: post
title: Nominal and Structural Are A False Dichotomy
---

Nominal type systems are type systems where subtypes are defined by explicitly listing the supertype or the subtype in relation to the other in a declaration.
The "nominal" comes from the fact that the "names" of the types are used in the declaration.
This is no longer the reality of the modern languages, as types are often values themselves.
A type-value is bound to a name, or potentially many names (even C has typedef).
So at best, the name "nominal" is a misnomer (heh).
But the important aspect of the term is that types *explicitly* subtype or supertype other types with declarations.

In many structural type systems subtypes occur implicitly, explicit declaration is not needed, meaning subtypedness is checked ad-hoc.
The name "structural" comes from the fact that often only the structural aspect of the type is considered for subtypedness.
Types being both a set of behaviors and a set of values mean that structural subtyping alone is not capable of describing all possible subtype relations.

The real issue is that that term "structural" and the dichotomy of "structural vs nominal" imply that implicit type systems and structural partial type systems are one in the same, *which is not true*.
To prove that point, lets list some contrapositives.

It is entirely possible to have an implicit and more featureful type system, where explicit subtyping declarations are not required and subtyping is tested ad-hoc.
These subtyping test would not just perform structural checks, but *behavioral* checks and even *value* checks.
Behavioral checks are possible with what I call property tests, which are essentially compile-time unit tests that ensure that a subtype behaves the same way a supertype does.
Behavioral checks are already possible in languages like C++ using template metafunctions and `static_assert`.
Value checks are really only possible if you have types where it is possible to enumerate the values of the type.
Implicit value subtyping would require checking that one type's value set is a subset over another type's value set.
This is implied in languages like C++ which allow implicit promotion of integer types to other "wider" integer types without the two types being explicitly defined as subtypes using class inheritance.
Concepts in C++20 are an example of this kind of type system since they are capable of not just structural, but arbitrary subtype checks.

It is also entirely possible to have an *explicit* structural-only type system.
This is essentially what Rust provides with traits and impls.

So, to clarify, the real dichotomy is not "structural" vs "nominal", but *"explicit" vs "implicit"*.
And there is another variable in the equation which is, *"how featureful of a type system do you want?"*
