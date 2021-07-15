---
layout: post
title: Tool Automation From First Principles
---

At work we have a piece of EDA tool automation software that is rather complex and fairly capable at its intended purpose; but there are some issue with it.
* the implementation is difficult to read, reason about, and thus maintain
* configuration is done with YAML which makes dynamic or cross-configuration references impossible without extension (see [kuddl](https://github.com/ktbarrett/kuddl))
* configurations for steps are not documented or even apparent by reading the code
* YAML configurations are typically scattered across many files and combined together in complex ways, to the point you are never quite sure what your configuration is at any point
* adding new flows requires *a lot* of work
* we currently have no money to improve it and don't know when that money supply will come back =)

So, I am looking for something new. Something open source.

## Lay of the Land

TODO

## From First Principles
