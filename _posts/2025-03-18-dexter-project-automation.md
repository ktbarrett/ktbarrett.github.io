---
layout: post
title: Replacing Makefile-based Project Automation
---

cocotb and coconext have a need for a project automation, as do so many other projects.
Right now they are using [nox](https://github.com/wntrblm/nox) but it's showing it's deficiencies.
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

There are a number of issues with these tools however that cause them to not meet the requirements.

`make` is not cross-platform, there's no native support on Windows without a POSIX environment like msys2.
Next `make` simply runs shell commands, so whatever you do write is tied to the shell environment and shell language features.
This makes even cross-platform support even between Unixes like MacOS and Linux difficult, as the shells and common tools like `grep` have different features.
`just` has the same issues, as does `invoke`.

`make` and `just` also don't necessarily respect ordering requirements.
In the below `make` example, `c` needs `b` to run then `a`, but `b` depends on `a` so `a` must run first.
`make` silent ignores the conflict and runs `a` first.
Perfectly fine for a build system, but not necessarily for a task system where side effects mean the order matters.

```makefile
a:
    @echo "a"

b: a
    @echo "b"

c: b a
    @echo "c"
```

`make` and `just` also have the downside of being bespoke languages for scripting the tasks themselves.
And `tox` is declarative and is simply not capable of being a very flexible project automation tool without leaning on the shell or Python scripts.
why should I ever have to learn what the following line means?

```makefile
 print-%: ; @echo $*=$($*)
```

Python is fine, which is what makes `nox`, `invoke` and `doit` the standouts.
They are distributed via `pip` which runs everywhere;
are implemented in Python which runs everywhere;
the tasks are described in Python which runs everywhere;
and Python is an incredibly capable language.
Being implemented in Python and writing tasks in Python is a goodness.

`tox` and `nox` are Python-focused and run every "environment" in its own virtual environment.
This is what is currently getting in the way in cocotb and coconext.
I need automation to be able to run in the current global environment and not isolated ones to maintain flexibility.
The venv isolation can be great for things like isolated dev tests for CI and isolation package builds,
but I'd rather develop out of a single persistent development environment.

I haven't complained about `doit` yet.
It's task are generators which yield bespoke dicts for describing dependencies without any typing support, so it just feels weird and kinda bad.
Otherwise, its technically good.
I'm allowed to make decisions based on aesthetics!

`nox` can call out other sessions in a recursive way, and you can pass options;
but it does not support declarative and de-duplicated dependencies like Makefiles.
`invoke` does support this, but task arguments must be bound when declaring the dependency relationship
rather than just joining the argument namespace like Makefiles do conceptually by hvaing all options be globals.

Of all the options, `nox`, `invoke`, and `doit` are the closest to the mark, but all have at least one issue...

So I'll make my own.

Dexter
======

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

It will reach a minimal viable product in a couple weeks.
It's housed [here](https://github.com/ktbarrett/dexter/).
Feel free to suggest names in a Github issue.
