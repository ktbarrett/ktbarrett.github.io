---
layout: post
title: Volatile Isn't A Type Qualifier
subtitle: How C and C++ conflate value type and storage semantics
---

In C and C++ there are the type qualifiers `const` and `volatile`.
C99 and up have `restrict` as well,
but it is only applicable to pointers, so it avoids the problem I'm discussing here today.
The problem is that C conflates the value type and storage semantics,
allowing storage semantics, like `volatile`, to qualify value types, like `int`.
As we will discuss, this is nonsense.
C++ inherits this problem, and it causes much heartburn around the `volatile` type qualifier.

## What is a type qualifier?

It's a keyword which "qualifies" the type it is applied to name an automatically defined related type.
`const int` is a new type which is derived by qualifying the type `int` with `const`.
The new type is a supertype of the base type,
because you can implicitly and safely cast a value of any non-qualified type into its qualified type.
This is the case for both `const` and `volatile`.

Type qualifiers, like `const`, providing an easy way to create the two types from one definition:
an immutable type and mutable subtype.
Seeing as this is needed *very* often, this saves a lot of developer time.
In C++, this is more obvious, as when you write a class you need to apply `const` to member functions so that function can be called on `const` values of that type.
When you write a `class` you are defining the unqualified type, the `const` type, the `volatile` type, and the `const volatile` type all at once.

## How do C and C++ conflate value type and storage semantics?

When you define a `const` variable in C you aren't just defining a variable with the type `const X`,
you are also modifying the storage semantics saying "you cannot assign a new value to the storage".
Sometimes you don't want both...

```c
typedef struct short_string {
    char data[31];
    char length;
} short_string;

char * short_string_begin(short_string const * s)
{
    return s->data;  // warning: returning 'const char[31]' from a function with result type 'char *' discards qualifier
}
```

In the above example, the `const` on the function parameter is stating that the function does not modify the variable,
i.e. it's acting on the storage semantics.
However, it also acts as a type qualifier mixing `const` into `short_string` making the `data` field `const`.

Likewise with `volatile`.
It is most commonly used to act on the storage semantics to prevent the optimizer from performing certain optimizations,
but it also acts as a type qualifier.

```c++
struct Example {
    int ret_1() { return 1; }
};

void volatile_Example()
{
    volatile Example e;
    e.ret_1();  // error: passing 'volatile Example' as 'this' argument discards qualifier
}
```

In the above example, we need the variable `e` to be `volatile`,
however `volatile` also acts as a type qualifier making `this` `volatile`, preventing the method `ret_1` from being called.

## Why is `volatile` a type qualifier?

While it's easy to come up with rationales for why both `const` as a type qualifier and `const` as a storage semantics qualifier are useful,
even if it would be nice if they were separate;
there is almost no rationale for why `volatile` has to be a type qualifier, causing the above example to fail.
The answer lies in C's, and thus C++'s, under-developed type declarations.
