---
layout: post
title: Exploring Structured Programming
---

## What is Structured Programming?

Programming with well-defined constructs to control the flow of execution in a program. A paper[^1] long ago proved that there are three structures which can be used to implement any computation.
* sequencing: do this, then do that
* selection: if some condition is true, do this; otherwise, do that
* iteration: while some condition is true, repeat this

These correspond to the ideas of statements, `if`-`else`, and `while` loops in many modern languages. While the paper proved they were "sufficient"[^2] they aren't necessarily ergonomic.

In addition to the immense amount of focus on flow control constructs, there was also a focus on a few other constructs: code "blocks", subroutines, and functions.

### What About Blocks?

Also important to the concept of structured programming is the concept of "blocks". Blocks are a sequence of instructions that are grouped which appear to behave as a single statement. This most importantly implies that any variables declared in the block cease to exist by the time the statement following the block is run.

Blocks also serve another purpose with regards to the limited `goto` statements: `break` and `continue`; supported by many modern programming languages. Many people understand that `break` exits the loop, and `continue` starts the loop back from the top; but I think this is a flawed understanding. Instead `break` and `continue` work as early exits of the blocks in which they reside.

Consider the case where a loop construct iterates over only a single statement. For example, C and C++ support this idiom. `continue` and `break` are statements, and if they were the only statement in the loop, the code would be nonsense. However, as soon as you add a block, their use becomes apparent.

In C, C++ and most other languages that support the limited `goto` statements: `break` and `continue`; these constructs only work in the context of loops. `break` can be located in many nested blocks, but it will break out of the nearest loop's block. `continue` is similar in that it also breaks out of the nearest loop's block, but then continues the loop.

While `continue` makes little sense outside of the context of looping, becuase it is a limited form of goto that jumps lexically *up* the source code; `break` potentially has additional use in jumping out of *any* block in context, lexically down the source code. Another example of this is the use of `break` in `switch` statements in C, C++, and others. It is an early exit out of the current block.

Some languages, e.g. Java, support labeled `break`s to allow for breaking out of multiple nested loops. This is an example of the use of `break` to exit one, or some other block in the current context. And as we will see later, `return` is a form of break. However, `return`, like `break` only works in certain contexts. This is ostensibly by design, to yield the "structured" part of "structure programming".

Are these structuring limitation useful, or simply academic wankery? Are the common selection of constructs too structured, or not structured enough?

### Why are `break` and `continue` safe, but `goto` is harmful?

Unlike the `goto` statement, which allows jumping (depending upon the language) *anywhere* in the program, `break` and `continue` only allow jumping to just after the end of the block, or just before the start of the block, respectively. In both cases, the context they are used in *has ended*, an all block-local variables have been cleaned up. This is very important distinction.

The primary safety issue when using `goto` to jump arbitrarily through a program is jumping over or through variable declarations. The program may accidentally end up in a place where it is operating on variables that were never initialized, or are simply aliasing whatever is in the stack with new variables.

Another problem, is that `goto` allows a program to jump lexically *up* the source code, creating implicit loops without it being obvious which variables are controlling the looping.

Yet another problem has to do with code scalability. Breaking sections of code into multiple subroutines helps code scalability by allowing a programmer to encapsulate sections of their code base into subroutines to reduce the necessary working memory required for them to reason about their program.

Subroutines also greatly improve reuse. Common code can be written once and run many times from many different contexts.
Fortunately, ISA designers clued into this early only, and it never really was debated.

When the programmer only has `goto`, compartmentalization, encapsulation, and reuse is nearly impossible.

### What Are Subprograms?

Subprograms are named reusable blocks that must have pertinent variables passed to them explicitly, because they can assume nothing of their caller's context. Similar to blocks, subprograms can be exited early; however, breaking out of further blocks is not possible due to the lack of knowledge of the caller's context.

***TODO***

### Are Functions Subprograms?

Yes and no... They are like subprograms in that they are named, must have variables passed to them, and can make no assumptions about the caller's context. However, function calls are *not* like blocks... Function calls can yield a value. That yielded value is then used in place of the function call in the *expression* where the function call occured.

***TODO***

## Missing Features

### Exiting Arbitrary Labeled Blocks

### Using Blocks in Expressions

* can sort-of be done with anonymous functions, but syntax isn't great

## Block Operators

* many programming languages constructs can be modeled as AST-level metafunctions on blocks

### Flow Control Constructs

* `break` and `continue` are just "exit block" statements with a boolean for ending the loop or not

### Function Declarations

* functions definitions take blocks and wrap them with a function spec and defer their execution

### Composite Type Declarations

* use executable block final variable declarations for fields

### Structure Type Declaration

* function variables declared in the block could be used to describe the structural type's interface

### Concept Declarations

* concepts are boolean metafunctions that returns true if a type implements the concept

## Syntax For Blocks

There are a few different syntaxes for Block-delination in modern programming languages.
Sometimes languages have equivalent syntax, but the exact spelling is a bit different.
Lets look at a few languages/family of languages:

### Python

Python (and a few other less popular languages) use significant indentation to represent new blocks.
In Python, blocks are only ever accompanied by block operators.

An advantage is that blocks are very clear to visualize and there is less syntactic "noise" by removing the need to end the block explicitly.
A clear disadvantage with the way Python does this is it makes using blocks in expressions impossible.
This is clearly acknowledged by the fact that Python has a separate syntax for anonymous functions and they are single expression only.
This is *necessary* to decide when the end of the function defintions occurs and the rest of the arguments start.

### The C and Pascal Family

This includes C, C++, Java, C#, Javascript, Rust, Pascal, and so many more languages.
In C-like languages, blocks are usually started with the right-facing curly bracket `{` and end with the left-facing curly bracket `}`.
Pascal is very similar, but the block beginning and end are call `begin` and `end`, respectively.
Most of the languages in these family support creating blocks arbitrary, and not just in conjunction with some other syntactic element.

The benefit of this syntax is that it is unambiguous when the block has ended.
This is important if blocks are to be used in greater expressions.
For example, in C++, one can write multi-line lambda functions *inline* with the expression in which they are to be used (such as being passed to a map, filter, or reduce algorithm).
The clear block end is important to distinguish between the end of the lambda body and the rest of the expression.

### The Ada Variation

Ada uses the C/Pascal idiom of clear block beginning and end syntax.
*However*, blocks cannot be created arbitrarily, but must be associated with a block operator.
This is because Ada allows you to optionally specify which block operator and/or the name of the labeled block you are leaving when the `end` keyword is seen.
It is an improvement upon regular C syntax, where the end of a block occurs when a closing bracket is seen "at the same level".

Matching "stupidly" like C does can lead to poor lex error messages when there is a mismatching number of closing brackets.
The lexer cannot decide which block is responsible for the missing bracket.
This leads to the error typically being reported at the end of the file where the final closing brackets were not seen.
In Ada, if the lexer sees `end procedure` when it was expecting `end if`, it can immediately error.

[^1]: C. Böhm and G. Jacopini. Flow diagrams, Turing machines and languages with only two formation rules. Communications of the ACM, pages 366–371, May 1966.

[^2]: Kosaraju and Kozen have separately proven that Böhm-Jacopini is false; that multi-level breaks are necessary as well.
