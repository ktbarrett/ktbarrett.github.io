---
layout: post
title: What's Your Function?
---

How many kinds of functions are there?
Well, there's only one isn't there?
Most programming languages only have the one concept of a function.
They may have methods, but that is mostly just special syntax for a regular function.
They may have callable objects, but ultimately the call is calling some function, it's not actually the object being called.
So it seems likere there is only one...

Let's presume for a second there is more than one kind of function.
What are the kinds of functions?
What are the differing characteristics of these types of functions?
Can we reduce a function to its elements and compose them again to find new kinds of functions?
Let's find out.

## What Kind of Functions Are There?

So let's widen the net on our definition and see what we fish up.
Let's say a function is...
* parameterizable with arguments
* flow control is given to it
* it has code
* may modify global state
* then flow control is returned to the caller
* and potentially values as well

So what does that encompass?
It certainly encompasses functions, but also asymmetric coroutines.
Exceptions are also a way of returning flow control and values back to a caller.
It encompasses what some languages differentiate as procedures vs functions (like Ada and Fortran);
and what some languages differentiate as generators and coroutines (like Python).

## What Are The Elements of Functions?

Looking at the things we caught in our definition first and foremost is that functions are code containers.
This is absolutely their primary role in life.
There is an emphasis on flow control and there are several kinds of flow control over a function boundary.
And the point of functions seems to be passing value values to the function and either modifying global state based on those value,
or returning values derived from the inputs back to the caller.

So our elements are
* code
* flow control
* input and output values
* how the function interacts with its execution environment

## Flow Control

There are a couple different kinds of flow of control we discussed that a function can support.
And a function can support all of these kinds of flow control simultaneously.

We mentioned:
* function call and return
	* This involves suspending the execution of the current function and letting another run to completion.
* exception multiple function return statements
	* This involves returning from multiple nested function calls at once back to some special handler statement, or not at all, resulting in a panic.
* asymmetric coroutines
	* This involves suspending the execution of the current function and letting another run *partially*.
	* When the called coroutine finishes part of it's execution it can yield a value *and control* back to the caller.
	* But it is expected to have control handled back to it to continue executing.
	* You can think of asymmetric coroutines as functions with multiple partial return statements.
	* And asymetric coroutines can be either stack-less like Python's generators and coroutines, or stack-ful like Go's or Lua's. So a yield may be a partial return from multiple nested function calls.

There may be even more useful flow control mechanism that pertain to functions, but these are very common in popular programming languages.

## Returning vs. Modifying Global State

This seems to be a well-understood dichotomy because several programming language (Ada, Fortran, Pascal, and probably others) separate "procedures" from "functions".
In Ada, procedures have no return value and functions are required to have a return value.
Presumably the point of procedures were to document that the code modifies either it's inputs or it's environment, while functions document that the code returns a value based purely on it's inputs or it's environment, and doesn't return values in some other manner.
Or maybe they just didn't think of C's `void` type that makes functions into procedures (`void` can't be used in an expression).
VHDL takes this dichotomy even further towards that presumed intent by introducing function purity and making function pure by default!

But there are three distinct features here we need to think about:
* How functions capture their global execution environment.
	* Do they capture by mutable reference, allowing the user to modify global state in the function?
	* Do they capture by immutable reference or value, preventing the user from causing side effects, but allowing them to use global state in the synthesis of return values.
		* Some might argue this is just as bad as mutable capture, as other functions can mutate the environment and cause side-effect-ful results in other functions.
	* Do they not capture the global state at all, like pure VHDL functions?
	* How much do they capture? Everything or nothing? Just imports? A set of whitelist listed variables?
	* Very fine granularity of environment capture is possible already in some languages like with C++ lamabda's, but I am not aware of this feature in any other language or on "regular" function definitions.
* How function capture their inputs.
	* Are functions pass by value or immutable reference?
		* This does tend to go hand in hand with not capturing environment to create *pure* functions.
		* Some might argue "inout" and "out" function arguments are fine in terms of purity as there is a clear transformation of value before and after the function call.
	* Are functions pass by mutable reference?
* What does the function call return.
	* Nothing?
	* A Value?

You might consider the first and second bullet points one in the same considering that global variables can be considered implicit arguments to every function.
The main difference is the lack of need for the user to actually pass the values, and even be totally unaware of the passing of these arguments (a form of encapsulation both to the user and the ABI).
So I think it's very valuable to distinguish them.

## Syntax To Fully Describe Functions

So let's write a syntax that describes any function and then reflect and iterate.

So we need syntax for:
* parameter names and in what way they are captured by the function
* global variable capture and in what way the listed variables are captured
	* plus shorthands for "all variables" or "no variables"
* return type
* exception type
* coroutine yield type

Here's what I came up with:

```
callable_def = capture_list? "func" parameter_list returns_def? throws_def? yields_def?
direction = "in" | "inout" | "out"
capture_list = "[" (capture_def ",")* (capture_def ","?)? "]"
capture_def = direction (var_def | "*")
parameter_list = "(" (param_def, ",")* (param_def ","?)? ")"
param_def = direction var_def ":" type_def
returns_def = "returns" (type_def | "_")
throws_def = "throws" (type_def | "_")
yields_def = "yields" (type_def | "_")
```

We keep the definition simple by requiring the "func" keyword to identify quickly that the code is a function definition.
I'm really not a fan of how function definitions are done in C and several other langauges that don't use a keyword.
A cluster of parens, stars, and typenames can get pretty unreadable pretty quickly, as most examples you can Google will show you.

We have parts of the declaration for the environment capture, 
