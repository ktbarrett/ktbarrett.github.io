---
layout: post
title: Replacing Makefile-based Project Automation
---

cocotb and coconext have a need for project automation, as do so many other projects.
Right now those aforementioned projects are using [nox](https://github.com/wntrblm/nox), but it's showing it's deficiencies.
So I decided to try to find a proper replacement.

This new tool needs to...
* Work cross-platform.
* Not do anything but exactly what I tell it. Namely, it shouldn't attempt to build venvs or try to make everything appear like a test when most project automation *isn't* a test; or at least have a way to turn all that off.
* Support passing options.
* Support running commands in an easy way.
* Ideally not require learning anything new.
* Ideally support reuse by allowing tasks to mark other tasks as dependencies, or at least call out tasks in a recursive manner.

There are a bunch of tools that are typically recommended for project automation:
* [just](https://github.com/casey/just)
* [tox](https://github.com/tox-dev/tox)
* [nox](https://github.com/wntrblm/nox)
* [invoke](https://github.com/pyinvoke/invoke)
* make (as in Makefiles)
* [doit](https://github.com/pydoit/doit)

However, there are a number of issues with these tools that prevent them from meeting the requirements.

## Cross-Platform support

`make` is not cross-platform,
there's no native support on Windows without a POSIX environment like msys2.

`make` also simply runs shell commands, so whatever you do write is tied to a particular shell environment and shell language features.
This makes cross-platform support even between Unixes like MacOS and Linux difficult,
as the shells and common tools such as `grep` have different features.

`just` has the same issues, as does `invoke`.

One more nail in the coffin is that each line of a `make` rule is a separate shell invocation.
Super efficient...
In what world is that valuable?

## Dependencies and task reuse

The next bit of the puzzle is task reuse is handled nicely in `make` by supporting both task-to-task dependencies and recursive `make` calls.

`nox` can call out other sessions in a recursive way, and you can pass options;
but it does not support dependencies in the same way as Makefiles.
You run the risk of executing a dependency more than once.

`invoke` does support task dependencies, but task arguments must be bound when declaring the dependency
rather than just joining the argument namespace.
Makefiles do that conceptually by having all options be globals.

`make` and `just` also don't necessarily respect ordering requirements.
In the below `make` example, `c` needs `b` to run then `a`, but `b` depends on `a` so `a` must run first.
`make` silently ignores the conflict and runs `a` first.
Depth-first left-to-right.
Perfectly fine for a build system, but bad news for a task automation system where side effects mean that order matters.

```makefile
a:
    @echo "a"

b: a
    @echo "b"

c: b a
    @echo "c"
```

## Bespoke and useless scripting languages

`make` and `just` also have the downside of using bespoke scripting languages.
why should I ever have to learn what the following line means?

```makefile
 print-%: ; @echo $*=$($*)
```

And `tox` has no language at all.
Its from the bygone (thank god) era of everything being declarative;
so it depends upon bash or Python scripts to do anything interesting.

Python is the way, which is what makes `nox`, `invoke` and `doit` the standouts.
They are distributed via `pip` which runs everywhere;
are implemented in Python which runs everywhere;
the tasks are described in Python which runs everywhere;
and Python is an incredibly capable language.
Being implemented in Python and writing tasks in Python is a goodness.

## Isolated environments

`tox` and `nox` are Python-focused and run every "session" in its own isolated virtual environment.
But this is exactly what is getting in the way in cocotb and coconext.

I need automation to be able to run in the current global environment so that I can run do a dev build and then run a regression against that dev build, but then cd into the dev build and run `cmake` commands.
In isolation builds, scikit-build-core cmake configurations point to temporary dirs and running `cmake` in them after a build just breaks...

Virtual environment isolation can be great for things like isolated release builds,
but I'd rather develop out of a single persistent development environment.

## Final remarks

I haven't complained about `doit` yet.
It's tasks are Python generators which yield dicts with bespoke schemas for describing dependencies
and which lack any meaningful typing support.
It just feels weird and kinda bad.
Otherwise, its technically good.
I'm allowed to make decisions based on aesthetics!

Of all the options, `nox`, `invoke`, and `doit` are the closest to the mark, but all have at least one issue...

So I'll make my own.

## Dexter

I started `dexter` (name subject to change) to fill these needs.

It will...
* Be implemented in Python
* Be distributed via PyPI
* Tasks will be written in Python
* Tasks will be able to list out other Tasks as dependencies
* Task Flows will be checked to ensure a serialization order is possible where all ordering constraints are satisfied, or fail
* All Tasks in a Flow will share arguments
* Tasks will be easy to write, using decorated functions much like `nox` or `invoke`
* Tasks will have access to special "session" object to contain session-wide environment variables, and simplified run functions like `nox` or `invoke`
* Tasks will run commands with `execve`, not shell scripts

The project is housed [here](https://github.com/ktbarrett/dexter/).
Feel free to suggest alternative names in a Github issue.
Currently it's named after my dog that I miss very much.
