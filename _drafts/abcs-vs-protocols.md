---
layout: post
title: Abstract Base Classes and Protocols
---

# What are Abstract Base Classes in Python?

Abstract Base Classes -- known as ABCs -- are Python classes which cannot be instantiated, because they have one or more *abstract* attributes or methods.
Abstract attributes or methods are attributes or methods that are declared to exist, but no implementation is given where they are declared.
Users of ABCs are expected to first inherit from the ABC, then provide the implementations for the abstract attributes or methods.
Python provides the [`abc`](https://docs.python.org/3/library/abc.html) module which includes tools for defining ABCs.

## What's the Point?

By using ABCs, you can be sure that subclasses of the ABC will have implemented the methods you listed as abstract;
plus any *non-abstract* methods defined on the ABC.
Typically ABCs are used in type-generic alogirhtms to ensure the given type can support the necessary operations to implement the algorithm.
For example, a generic sort algorithm needs a way to order two different elements of a collection.
So one could define an ABC `Sortable` with an abstract `__le__`, this would be enough to implement a generic sorting algorithm.

```python
from abc import ABC, abstractmethod
import typing

Self = typing.TypeVar("Self")
SortableList = typing.TypeVar("SortableList")


class Sortable(ABC):
    """ """

    @abstractmethod
    def __le__(self: Self, other: Self) -> bool:
        ...


def sort(a: SortableList) -> SortableList:
    ...
```

## ABCs in the Python Standard Library

Python makes extensive use of ABCs in it's standard library.
These ABCs are described quite well in the documentation for the [`collections.abc`](https://docs.python.org/3/library/collections.abc.html) module, where they all reside.

* The numeric tower

## In Other Languages

## Limitations

The most obvious limitation of ABCs is for a type to be used where a particular ABC is expected, it must inherit from that ABC.
This can be a problem if you create a generic algorithm *after* defining a type.
There is no way after the type is defined to make it inherit from a base class.
So what do you do?

### Boxing

One common solution, used extensively in other languages, is to "box" the value in a new type that inherits from the ABC, and proxies useful calls to the underlying value.

```python

```

### Virtual Subclasses

Python works around this limitation by introducing the concept of "virtual subclasses".
One can create a virtual subclass of an ABC, without inheritance, by using the method [`ABCType.register(subclass)`](https://docs.python.org/3/library/abc.html#abc.ABCMeta.register).
The biggest issue with this approach seems to be that it simply trusts the user to have implemented the protocol correctly;
no checking is done to ensure the virtual subclass implements the whole interface of the ABC.
It can also be quite verbose.
But it works in a pinch.

## Best Practices

# What are Protocols in Python?

Protocols are another answer to the limitations of ABCs.
They were first introduced in [PEP 544](https://www.python.org/dev/peps/pep-0544/).

Protocols solve the problem of checking for implementation of a protocol by performing those checks ad-hoc, structurally.
Meaning if a Protocol defines a `index()` method, when checking if a type implements that Protocol, it simply looks to see if the type has an `index()` method.

* convert above example

# What's the Difference?

* subtyping implications
