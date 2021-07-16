---
layout: post
title: EDA Project Automation From First Principles
---

At work we have a piece of EDA project automation software that is rather complex and fairly capable at its intended purpose; but there are some issue with it.
* the implementation is difficult to read, reason about, and thus maintain
* configuration is done with YAML which makes dynamic or cross-configuration references impossible without extension (see [kuddl](https://github.com/ktbarrett/kuddl))
* configurations for steps are not documented or even apparent by reading the code
* YAML configurations are typically scattered across many files and combined together in complex ways, to the point you are never quite sure what your configuration is at any point, or where a particular configuration came from
* adding new flows requires *a lot* of work
* we currently have no money to improve it and don't know when that money supply will come back =)

So, I am looking for something new. Something :sparkles: *open source* :sparkles:.


## What is Task Automation, Really?

Task automation is about *doing stuff*.
Obviously.
Tasks take inputs, *do stuff*, and produce outputs.
Sometimes those inputs are passed by the user.
Sometimes those inputs need to be gathered
Sometimes they are outputs of other tasks.
Sometimes those inputs come from the execution environment.
Usually there is a *bunch of stuff* that needs to get *done* in a project.
Sometimes you only want to *do __some__ stuff*, and other times you want to *do __all__ the stuff*.
Sometimes you want to *do* the same *stuff*, but extend what it's doing slightly.
Sometimes you want to *do* the same *stuff*, but in a totally different order.
Ideally, *stuff* can be reused from project to project, and person to person.
Ideally, you want as much *stuff* to be *done* simultaneously as possible.

Now that that *stuff* is out of the way, we can characterize the problem.
* tasks are functions with an arbitrary number and type of inputs and outputs
* tasks need to be pure functions for parallelization and reuse
* tasks should be extensible
* tasks can depend on other task, and these dependencies form a directed acylic graph
* task interdependency is defined by dependencies between individual outputs and inputs of tasks
* tasks dependencies need to be independent of the task definition so you can reuse and reorder tasks
* the dependency system and execution engine should support many types of inputs and outputs
* the execution engine should support many excution models: serial, parallel, and distributed


## Existing Solutions

### Make

Many people use build tools like `Make`, `CMake` (WHY!?!?!), `SCons`, and others to do task automation.
But they really aren't a good fit for task automation per the prior section's definition.
I don't want to go through all of them, so let's examine the eldest and (likely) most popular of the build tools: `Make`.

Poeple usually treat tasks as `Make` "phony" targets, with the target's recipe being the task body.
There is no way to list inputs and outputs to any particular target.
Passing arguments between tasks is done implicitly using the environment/filesystem;
which prevents isolation, making parallelized and distributed execution difficult.
`Make` allows you to list dependencies only from one task to a another;
which preventing early execution optimization of dependent tasks which can use partially available products.
The dependency accompanies the task definition; which prevents reuse or reordering of tasks.
However, `Make`'s dependencies allow you to run a particular task and all its predecessors;
which is a very limited form of task re-composition.
Finally --- and this is an artifact of it being old --- the scripting language it uses is awful.

Of course I'm not dissing `Make` as a build tool;
a lot of these issues go away when you use it for its intended purpose.
Targets are now the singular output product of a task.
Dependencies are a form of listing inputs.
Code generation and compilation steps are usually pure function, so parallelization is safe.
`Make` supports opportunistic parallelization OOTB.
Another nice feature of `Make` is that it automates some of the task body for you;
deciding if the task body can be skipped if its dependencies have not changed.

### doit

There are also a number of dedicated task-automation frameworks like `doit` and `invoke`.
For this section let's analyze `doit`, which is the basis for my workplace's internal tool automation software.
As with much of software, tools and library that pop-up to replace established solution are usually designed in a purely reactionary way and *not* from first principles.
`doit` is no exception, read the [blog post that started it all](https://schettino72.wordpress.com/2008/04/14/doit-a-build-tool-tale/).

`doit` allows the user to explicitly list dependencies and targets (inputs and outputs, respectively) which is *very* nice.
You do not explicitly specify dependencies between tasks, only intermediate products, and the runtime computes the dependency graph.
This feature could be used as the basis for isolation and parallelization (which is how our automation software uses it);
and could be use for early execution of dependent tasks using partially available products.
The execution engine it ships with does neither of these things OOTB.
Additionally, the types of targets and dependencies is limited to the file, much like `Make`.
It has the same behavior of skipping predecessor if they aren't out of date, much like `Make` does,
which is a convenient ***curse*** (as we have found out).


## From First Principles

Now that we understand the problem at its core, have reflected on prior art, lets design some task automation *from first principles*.
