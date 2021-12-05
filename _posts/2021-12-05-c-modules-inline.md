---
layout: post
title: C Modules and Inline Functions
---

<!--
- What is C to me
    - historical perspective
        - portable code, but higher performance than interpreters like BASIC
        - high performance compilation due to incremental separate compilation
        - much simpler language than other compiled languages like Fortran or Algol
        - A true systems programming language unlike Fortran and Algol
    - ways in which historical perspective are obsolete
        - analysis is much quicker nowadays
        - prefer cross-compilation over writing or porting a new toolchain
        - actual portability of C code is rather low in comparison to modern expectations
    - current perspective
        - Lingua Franca
        - minimal but sufficient language features to implement any library or program
        - some (very atypical) platforms only have a C compiler
        - mostly overshadowed in utility by C++, but some people don't like C++
-->


## Modules in C

<!--
- C has modules
    - translation units are compiled independently and can have private elements
    - headers list the public elements of the "module"
    - separate incremental compilation is proof of encapsulation
-->

C has modules.
They are implemented as a convention of translation units and headers.
Most C programmers are aware of this technique, but many do not appreciate it as a module system.

With this technique, modules are created by defining interface types and functions in header files.
Modules are "used" by `#include`ing the appropriate header file.
Each module is implemented as one or more translation units (henceforth known as TUs) that contain implementations of the interface types and functions.
TUs also encapsulate implementation details with TU-local types and static functions.

As a testiment to its encapsulation, a C "module" is recompiled when the implementation changes.
However, none of the users of the module need to recompile.
The only additional action needed is to relink the program with the recompiled TU's object file.

## Function Inlining in C

<!--
- what are inline functions
    - to inline functions, the compiler needs the function definitions
    - can't use definitions from other translation units because separate compilation
    - must have duplicate function definitions
    - In ANSI C this is implemented as macro functions or static functions
    - macro functions
        - can't take address of macro function
        - always inline
        - not type safe
        - may result in less informative error messages
    - static functions
        - addresses of static functions will be different
            - does this matter?
        - compiler may or may not inline
        - type safe
        - good error messages
        - duplicates code causing binary size to increase
-->

Function inlining is a performance optimization.
In some cases, such as in hot tight loops, function inlining can considerably increase runtime performance.
This benefit comes from both avoiding the function call overhead associated with dumping registers onto the stack and jumping;
and also comes from being able to optimize the function body in the context of the particular function call.
When the compiler decides to inline a function call, it is weighing the potential performance benefit against increased binary size.

To be able to inline function calls in C, the compiler needs the function's definition.
Because of separate compilation in C, functions defined in one TU are not available when compiling another TU.
So we must have the function definition duplicated in each TU when compiling.
This is typically accomplished by including the function definition in the header where the function is declared.

In ANSI C (AKA ISO C90 or C89) inline functions are defined in headers using either function-like macros or `static` functions.
Both of these approaches have several downsides.

Function-like macros:
* Cannot get a reference to a function-like macro.
* All function calls are *always* inlined.
* Not type-safe.
* May result in less informative error messages (though they are pretty good in the big 3).

`static` functions:
* Can get a reference to a `static` function.
* The address of the `static` functions is different in each TU.
* Compiler can decided on each function call whether to inline or not.
* Type-safe.
* Good error messages.
* Each TU has a duplicate of the function, increasing the final binary size.

Because of these problems, the ISO C99 version of the language added the `inline` specifier for functions.

## What is The Inline Specifier?

<!--
- what is `inline`?
    - started out as a compiler hint
    - very coarse-grained
    - not binding, compilers don't need to inline
    - a form of LTO
    - removes functions duplicated across multiple object files
    - all references will resolve to the same address
-->

The `inline` specifier in C99 started out life as a compiler hint.
Functions declared `inline` were intended to be functions that were good candidates for inlining.
Unfortunately, a function specifier is a very coarse-grained hint.
Thoughtfully, the standard does not force compilers to always inline function calls involving a `inline` function.
Modern compilers even tend to ignore the `inline` specifier as a hint.

What is still important about `inline` functions is the semantics of `extern inline` functions.
If a program has multiple definitions of a function that are all declared `inline`,
and the definitions of these functions are identical,
the multiple definitions are resolved a single definition in the final executable.
This solves the duplication problem seen with `static` functions defined in headers.
It also ensures that all references to that function are the same across the whole program.
This de-duplication is a form of Link-Time Optimization (henceforth known as LTO).

## Inlining Breaks C's Modularity

<!--
- how `inline` and inline function definitions break C's modularity
    - leak implementation details into header
    - cause recompilation when implementation changes
    - adds additional dependencies to header for implementations
    - cause implementations in header to be analyzed many times
        - not really noticable in C
        - VERY noticable in C++
        - exponential when combined with previous points
    - `inline` complicates the linker breaking the "simple to implement a toolchain" of the historical perspective
-->

While inlining functions may lead to performance improvements, inline function definitions breaks C's modularity and spirit of simplicity.

Inline function definitions, no matter how the function is implemented, leaks implementation detail into the header; breaking the encapsulation of TUs.
It means that whenever an implementation changes, users of a module may need to be recompiled;
not just the module definition itself.

It also tends to lead to additional `#include` directives in a header to service the inline function definition.
The inline function definition will also be analyzed multiple times, once for each TU, decreasing whole-program compilation performance.
In combination with the previous points of:
additional `#include` directives in headers,
with inline function definitions to be analyzed in each newly `#include`d header,
recursively,
this multiplicity becomes exponential, and compilation performance drops drastically.
This is *very* apparent in C++.
It is present, but less observable, in C, mostly because C is a simpler language to analyze.

Finally, the `inline` specifier complicates the linker implementation, since it is a form of LTO.
This breaks C's philosophy of simplicity, and the ease of implementation of a C toolchain on a new platform. Easily supporting new platforms was one of the original design goals of C, and part of what made it popular long ago.

<!--
- how that problem is even worse in C++
    - templates must have inline defintion
    - "header-only libraries" exist because people realize how useless the translation unit system has become in the face of inline-only definitions in C++
-->

## What Can We Do?

<!--
- what to do for C?
    - use LTO
        - at link-time all function defintions are known across the whole program
        - LTO will get better
-->

We need a solution to function inlining that does not have a negative effect the semantics of the language.
Ideally, we also have a finer-grain approach to inlining than the `inline` specifier attempts to provide.
Right now, we have LTO.

LTO-capable linkers can inline functions that are defined in different TUs.
This is because all code fragments that are a part of the final binary are available at once while linking.
LTO-capable linkers are also capable of a number of other whole-program optimizations;
like removing unused functions in executables.

That begin said, linkers are not nearly as capable of inlining functions as compilers are.
Part of this is because the technology is relatively young, about 10 years old,
compared to the 40+ years optimizing C compilers have been in developement.
There also doesn't seem to be much in the way of attributes to help guide LTO yet.
But, most *major* issues with LTO in the Big 3 compilers have been solved as of this writing (2021).

So my suggestion to C programmers is:
1. Keep your function definitions in your source files.
2. Turn on LTO.
3. If your functions aren't inlining like they should, move them into the header and use `static`.

Hopefully in the near future, point 3 can be replaced with call-site-specific attributes to help LTO decide to inline function calls.

And my suggestion to the C standards committee is:
1. Make the `inline` specifier optional in C23.

<!--
- what to do for C++?
    - the inline problem in C++ is worse due to templates
    - syntactic modules
        - no more separate compilation
        - more efficient post-analysis representation
        - allows template definitions to be module-private
        - C++20 adds these
-->
