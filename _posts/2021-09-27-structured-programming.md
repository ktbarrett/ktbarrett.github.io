---
layout: post
title: Exploring Structured Programming
---

Structured programming is programming with well-defined constructs to control the flow of execution in a program.
As opposed to "less-principled" language constructs, like what is seen in assembly languages with arbitrary jumps.

## The Bare Minimum

A paper[^1] long ago proved that there are three structures which can be used to implement any computation.
* sequencing: do this, then do that
* selection: if some condition is true, do this; otherwise, do that
* iteration: while some condition is true, repeat this

These correspond to the concepts of statements, `if`-`else`, and `while` loops in most imperative languages.
While the paper proved they were "sufficient"[^2], they aren't necessarily ergonomic.
In addition to the immense amount of focus on flow control constructs,
structured programming also focused on a few other constructs: code "blocks", subroutines, and functions.

Lets break down these elements of block-structured programming and see what we can learn...

## Blocks

Blocks are a sequence of statements that act as a single arbitrarily-complex statement.
The implication of this is that variables that are defined in a block must be cleaned up after the last statement in the block is evaluated.
Blocks can be arbitrarily nested.
Variables defined in a block are available to nested block definitions.

```C++
int a = 0;
{
    int b = a;  // can use variable a here
}
// b is cleaned up by now
```

## Loops, break, and continue

A loop body, in the common case, is a block.
The loop runs the block, and once the block finishes, the loop restarts.
Because the body is a block, when the loop ends, variables defined in the loop body are cleaned up, to be recreated on the next loop iteration.
You may also want to end the end the loop before the loop condition is checked again:
the `break` and `continue` statements allow you to break out of the current block early.
With labeled breaks (or a similar feature) multiple loop blocks can be exited at once.

```C++
int i = 0;
while (i < 10) {
    int a = i * 2;  // a is created and destroyed on each iteration
    i++;
}

while (1) {
    int c = next(lexer);
    if (c == ' ') {
        continue;   // breaks out of block and restarts loop, c is destroyed and recreated
    }
    else if (c == EOF) {
        break;     // break out of block and ends loop, c is destroyed
    }
    // ...
}
```

## Subroutines

Subroutines are a named block.
Control can enter a subroutine from any context by calling the subroutine.
Once the subroutine ends, control will go to the next statement in the caller's block.
Because they can be called from any context, subroutines cannot make *any* assumptions about the caller's environment.
Variables in the callers scope are not immediately available in the function block,
so they must be explicitly passed as arguments.
You can exit the subroutine early using a `return` statement.
However, since the caller's context isn't known to the subroutine,
it can't cause an early exit past the call point (e.g. can't do a multi-level `return`).

```C++
// caller
int b = 10;
print_stuff(b);

void print_stuff(int a)
{
    // b is not available, must pass explicitly as argument a
    printf("%d\n", a);
    if (cond()) {
        return;  // early return
    }
    printf("cond() is false");
    // implicit return
}
```

## Function

Functions are like subroutines: they are named blocks callable from any context.
However, unlike subroutines or blocks, they aren't acting like a statement, but an expression.
Functions return values that are used in place of the call site in the caller's expression context.
This is extremely useful for describing arbitrarily-complex expressions.
Like in subroutines, `return` can be used to exit the function early.
Unlike in subroutines, `return` is required in all living code paths.

```C++
// caller
int b = 10;
int c = digit_sum(b) + 78;  // used in larger expression

int digit_sum(int c)
{
    int sum = 0;
    while (c > 0) {
        sum += c % 10;
        c /= 10;
    }
    return sum;  // return of a value is required
}
```

## Some Observations

1. Can use `break` to jump out of loops early, or even multiple nested loops.
    Can use `return` to jump out of subroutines early.
    Can't jump out of arbitrary blocks early?
2. Can use functions to create named, reusable, arbitarily-complex expressions.
    Can't directly use a block in an expression, must split it off into a function?

Point 1 has come up many times as a justification for leaving `goto` in a language even though *`goto` is evil*.
And point 2 has come up many times in the form of defining an anonymous function and calling it immediately,
which you see again and again in C++11 and newer code bases. Yet I have never heard anyone complain about it.

## Goto is Evil

The primary safety issue when using `goto` to jump arbitrarily through a program is jumping over or through variable declarations.
The program may accidentally end up in a place where it is operating on variables that were never initialized.
You can certainly work around this issue by disallowing jumps over variable declaration,
but that has a second-order effect of programmers pushing their variable declarations up to the top of the (sub)program to avoid the problem.
Now they are choosing mutable, potentially-uninitialized variables where they otherwise wouldn't.

Another problem is that `goto` allows a program to jump lexically *up* the source code,
creating implicit loops without it being obvious which variables are controlling the looping.
This makes it hard to reason about the current state of the program without considering many, potentially unknown, variables.
This seems to be the primary focus of Djikstra's article *Go To Statement Considered Harmful* [^3].

Yet another problem has to do with code scalability.
Breaking sections of code into multiple subroutines helps code scalability by allowing a programmer to encapsulate sections of their code base into subroutines to reduce the necessary working memory required for them to reason about their program.
Subroutines also greatly improve reuse.
Common code can be written once and run many times from many different contexts.
Fortunately, ISA designers clued into this early only, and it never really was debated.

When the programmer only has `goto`, as in assembly; compartmentalization, encapsulation, and reuse is extremely difficult,
without a principled developers.
As most wise men will tell you,
you shouldn't build systems that only work when people behave perfectly.

## Breaks as a Limited Goto

Expanding on the observation from earlier that you can `break` out of loops, `break` out of nested loops, and `return` early from subroutines, but can't break out of anything else:
we could expand `break` to work on any block, allowing it to act as a form of limited `goto`.
`break` only allows jumping to just after the end of a block;
meaning any block-local variables have been cleaned up.
`break` also can only jump *down* the source code,
making it no more difficult to reason about than loop `break`s or early `return`s in subroutines.
There is no way to create implied loops with `break`.
This makes `break` far safer than `goto`, while being capable enough to cover most of the *valid* use cases of `goto`.

To prove my point, let's dive into examples of *valid* uses of `goto` from a [StackOverflow question on that subject](https://stackoverflow.com/questions/24451/is-it-ever-advantageous-to-use-goto-in-a-language-that-supports-loops-and-func/24476#24476), and rewrite them with labeled `break`s on arbitrary blocks.

### Multi-level Loop Breaks

Sadly, C and C++ support labels, but do not support labeled `break`s, which allow the programmer to break out of multiple levels of loops at once.
In this case, `goto` is used to act as a multi-level `break`.
This is not a good reason to keep `goto` around... C and C++ could at any point add support for labeled `break` and remove the largest reason for `goto` appearing in C and C++ programs.

```C++
// search for first occurence of value in row-major 2D array
int found_row, found_col;
for (int row = 0; row < 10; row++) {
    for (int col = 0; col< 10; col++) {
        if (array[row][col] == value) {
            found_row = row;
            found_col = col;
            goto found;
        }
    }
}
int c = 0;  // WHOOPS... need a variable here, jumped over variable declaration
found:
...  // this only works if there is another statement here...
```

With labeled breaks...

```C++
// search for first occurence of value in row-major 2D array
int found_row, found_col;
found: for (int row = 0; row < 10; row++) {
    for (int col = 0; col< 10; col++) {
        if (array[row][col] == value) {
            found_row = row;
            found_col = col;
            break found;  // multi-level loop break
        }
    }
}
```

### Cleanup or Error Handling at End of a Block

A common idiom in C where RAII, context managers, `defer`, or `exception` and `finally` statements are not available
are `goto` statements which jump to the end of the function where cleanup or error handling occur.
A typical example might look like...

```C
int big_function()
{
    int ret_val = [success];
    /* do some work */
    if([error])
    {
        ret_val = [error];
        goto end;
    }
    /* do some more work */
    if([error])
    {
        ret_val = [error];
        goto end;
    }
    /* do some more work */
    SomeType value;          // WHOOPS... need additional variable here,
    if([error])              // jumped over variable declaration,
    {                        // move the variable decl to top of function
        ret_val = [error];
        goto end;
    }
    /* do some more work */
    if([error])
    {
        ret_val = [error];
        goto end;
    }
end:
    /* clean up*/
    return ret_val;
}
```

Instead, RAII, context managers, `defer`, `exception`/`finally` could be used (I'll leave that as an exercise for the reader),
*or* a labeled `break` on an arbitrary block could as well.

```C
int big_function()
{
    int ret_val = [success];
    work: {
        /* do some work */
        if([error])
        {
            ret_val = [error];
            break work;
        }
        /* do some more work */
        if([error])
        {
            ret_val = [error];
            break work;
        }
        /* do some more work */
        SomeType value;         // this is fine
        if([error])
        {
            ret_val = [error];
            break work;
        }
        /* do some more work */
        if([error])
        {
            ret_val = [error];
            break work;
        }
    }
    /* clean up*/
    return ret_val;
}
```

### Algorithmic Logic

In this example, we are using a `goto` to skip over unnecessary computations by stating "what's next".
Instead, we should state "which sub-computation we are exiting early", as that proposition is far less fragile.

```C++
size_t add_index;

// Overwrite an element with same hash key if it exists
for (add_index=0; add_index < ELEMENTS_PER_BUCKET; add_index++)
  if (slot_p[add_index].hash_key == hash_key)
    goto add;

// Otherwise, find first empty element
for (add_index=0; add_index < ELEMENTS_PER_BUCKET; add_index++)
  if (slot_p[add_index].type == TT_ELEMENT_EMPTY)
    goto add;

// Additional passes go here...

int value = 0;  // WHOOPS... need additional variable here,
                // jumped over variable declaration

add:
// element is written to the hash table here
```

Instead it could look like...

```C++
size_t add_index;

index_logic: {
    // Overwrite an element with same hash key if it exists
    for (add_index=0; add_index < ELEMENTS_PER_BUCKET; add_index++) {
      if (slot_p[add_index].hash_key == hash_key) {
        break index_logic;
      }
    }

    // Otherwise, find first empty element
    for (add_index=0; add_index < ELEMENTS_PER_BUCKET; add_index++) {
      if (slot_p[add_index].type == TT_ELEMENT_EMPTY) {
        break index_logic;
      }
    }

    // Additional passes go here...
    int value = 0;  // this is fine
}

// element is written to the hash table here
```

## Using Blocks in Expressions

As for the other problem of not being able to use blocks in expressions,
I propose a new language construct: the expression block and the result statement.
The expression block can be embedded in an expression, allowing statements to be written.
The expression block is required to have a `result` statement which specifies the value to return in place of the expression block,
similar to requiring `return` in functions.
Like `break`, it could also allow for multi-level `result` statements,
since the body of all lesser-nested expression blocks will be observable (though I doubt its utility).

For example, let's refactor the code that finds the row and column index of a particular value from above to use the new `expr` block and `result` statement.

```C++
const int found_row, found_col = expr {
    for (int row = 0; row < 10; row++) {
        for (int col = 0; col< 10; col++) {
            if (array[row][col] == value) {
                result {row, col};
            }
        }
    }
}
```

As you can see there are immediate benefits: we can make the variables `const`.
This feature is useful in the context of initializing immutable variables,
computing compile-time constants,
or any place where only a simple expression is allowed (like member initialization lists in C++).

This can currently be accomplished in C++11 using lambdas, but there is some syntatic noise associated with it.

```C++
const int found_row, found_col = [&]() {  // capture and argument list for lambda expression
    for (int row = 0; row < 10; row++) {
        for (int col = 0; col< 10; col++) {
            if (array[row][col] == value) {
                return {row, col};
            }
        }
    }
}()  // call it
```

Though I suspect that this is considered sufficient and there no interest in new keywords in C++.
It's something to think about for other languages that don't support anonymous functions,
or if anonymous functions are more syntactically noisy.

[^1]: C. Böhm and G. Jacopini. [Flow diagrams, Turing machines and languages with only two formation rules](https://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.119.9119). Communications of the ACM, pages 366–371, May 1966.

[^2]: Kosaraju and [Kozen](http://www.cs.cornell.edu/~kozen/Papers/BohmJacopini.pdf) have separately proven that Böhm-Jacopini is false; that multi-level breaks are necessary as well.

[^3]: E. Djikstra. [Go To Statement Considered Harmful](https://web.archive.org/web/20070703050443/http://www.acm.org/classics/oct95/). Communications of the ACM, pages 147–148, March 1968.
