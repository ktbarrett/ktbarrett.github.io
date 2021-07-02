---
layout: post
title: Subtyping in Python Without Implementation Inheritance
---

Python has your typical class-based OO approach to types:
* There are `class`es
* You can inherit (subclass) from another class (or multiple) when creating a new class
* subclassing implies subtyping

In recent years there has been a shift away from class-based OO, and inheritance especially;
mostly because the coupling of implementation and interface inheritance was (correctly) seen as an anti-pattern.
You really *do* want to consider your type hierarchies separate of your implementation of them.
Not doing so can get *very* grody, *very* quickly.

So how is this typically accomplished?

Conventions related to this grew out of experience with other OO languages like C++ and Java
where non-static fields are mixed into the subtype's fields.
This is a huge wrench in the works.
For example, if you wanted to change an implementation in a subtype to a more performant one,
you are unfortunately stuck accepting the supertype's fields, whether you use them or not.
I won't say this feature is totally bad, it's *great* for implementing mixins;
but if that isn't your goal, you're SOL.
So convention is oriented around avoiding the issue by using the idea of
"only one level of implementation and potentially many levels of abstract interface specification".
You see this convention in abstract classes in C++, Java, and Python;
and it has recently become codified in languages like Go, Rust, and Haskell which provide "interface types" (AKA traits or typeclasses),
and inheritance is not possible.
With an "interface" you can only define an object's interface, that being the methods you can call and attributes you can access,
*maybe* with default implementations of methods.
After the interface is defined, one or more independent implementations of that interface are written (well in Go, implementation is ad-hoc).

I'll concede that this convention does have other good aspects (runtime polymorphism),
but it is responding to, and stuck in the frame of mind of, field inheritance.
Additionally, it does not attempt to provide a feature for implementation (of fields and methods that depend upon those fields) reuse,
so you are stuck repeating yourself.

Now that the background is over, lets go over how inheritance is accomplished in Python and how we can (ab)use that to work around our problem.
In Python class inheritance is split into two parts:
1. Inheritance declaration
2. Constructor delegation

Inheritance declaration is done when the subclass is defined.

```python
class A:
    def __init__(self, a: int) -> None:
        self._a = a

    def get_a(self) -> int:
        return self._a

class B(A):  # Inheritance in the class declaration
    ...
```

In doing so all instances of `B` inherit the *interface* and *implementation of that interface* of `A`.
This operation, by itself, does not cause non-static fields of `A` to be inherted by `B`.
However, by default, instances of `B` will be constructed using `A.__init__`,
leading to non-static field inheritance.

You can specialize construction by overloading `__init__` in `B`.
Convention is to then call the super type's `__init__` to construct its non-static fields in the current object,
resulting in non-static field inheritance.

```python
class A:
    def __init__(self, a: int) -> None:
        self._a = a

    def get_a(self) -> int:
        return self._a

class B(A):
    def __init__(self) -> None:
        super().__init__()  # Delegate construction of super type's fields
        self._b = 5  # Create fields specific to this class
```

You may have picked up my point by just my pointing out of the problem: `super().__init__()`.
Convention, best practices, many college lectures, SO answers, tutorial videos continuously remind you "don't forget `super().__init__()`"!
Some people (even me in the past) have even questioned why not somehow make constructing the supertype compulsory?
Well, *an* answer is that *you can use this feature to create subtypes without inheriting non-static fields*.
Guido, very explicitly, wants subclasses to be subtypes,
and this helps expressivity issues with inheritance hierarchies.
I'm not sure if this is was by design, but it sure is useful.
Taking the example from before...

```python
class A:
    def __init__(self, a: int) -> None:
        self._a = a

    def get_a(self) -> int:
        return self._a

class B(A):
    """B is always 5, why store anything?"""

    def __init__(self):
        # super().__init__()  # NOPE
        pass

    def get_a(self) -> int:
        return 5
```

Of course the caveat is, that for this to work, the subtype `B` must define all of the methods on `A`,
lest the classes become coupled, or certain method calls, left un-overloaded,
simply break because they only work with a particular implementation.

This may seem obvious and a bit silly to point out.
But until it saw it, my mind was stuck in the rut of working around non-static field inheritance.

## Expressing This

Obviously just leaving out `super().__init__()` will raise heads and leaves a lot to chance;
so you want to find a way to explicit show what you are doing,
and protect yourself from self-imposed foot injury.

A simple way would be to create a type that does nothing but add an empty `__init__` to your `__mro__`:

```python
class NoInheritImpl:
    def __init__(self, *args, **kwargs):
        pass

class A:
    a = 8

    def __init__(self):
        pass  # create a bunch of fields in here

    def b(self):
        ...

class B(NoInheritImpl, A):
    """Inherits A's interface, but erases the fields"""

    def __init__(self):
        super().__init__()  # that's fine, no fields are defined by this call
        pass  # create a bunch of fields in here

    def b(self):  # must redefine all public interfaces
        ...
```

The issue being that you still inherit method implementations that might not work...
There *could* be some metafunction that builds a new type with the whole
public interface of a type overloaded with `@abstractmethod`.
This is somewhat tricky since it is not possible to tell what methods on a type were written
by the user or generated by a metafunction (metaclass or decorator).
You would also have to decide if you need to wrap the generated object in `@classmethod`, `@property`, `@staticmethod`, etc.
And it just spirals out of control from there.
An implementation and example usage might look like...

```python
import typing

T = typing.TypeVar("T")


def NoInheritImpl(cls: typing.Type[T]) -> typing.Type[T]:
    abstractattrs = ...  # some perfect impl
    name = f"NoInheritImpl({cls.__qualname__})"
    return type(cls)(name, (cls,), abstractattrs)


class A:
    a = 8

    def __init__(self):
        pass  # create a bunch of fields in here

    def b(self):
        ...

    @classmethod
    def c(cls):
        ...

    @staticmethod
    def d():
        ...


class B(NoInheritImpl(A)):
    """Enforces A's interface, but erases the implementation"""

    a = 12

    def __init__(self):
        super().__init__()  # that's fine, no fields are defined
        pass  # create a bunch of fields in here

    def b(self):  # must redefine all public interfaces
        ...

    @classmethod
    def c(cls):
        ...

    @staticmethod
    def d():
        ...


class BCMethodMixin:
    """Defines methods b and c using some fields"""

    def __init__(self):
        pass  # create a bunch of fields in here

    def b(self):
        ...

    @classmethod
    def c(cls):
        ...


class A2(BCMethodMixin):
    """Implementation of A, but uses BCMethodMixin to implement method b and c again"""

    a = 12

    def __init__(self):
        super().__init__()  # __init__ BCMethodMixing for implement method b and c again
        pass  # create more fields in here

    @staticmethod
    def d():
        ...


class C(BCMethodMixin, NoInheritImpl(A2)):
    """
    Enforces A2's interface, uses BCMetodMixin to implement method b and c again.

    __mro__ handles any diamond-pattern issues.
    """

    def __init__(self):
        super().__init__()  # __init__ BCMethodMixing for implementing methods b and c
        pass  # create more fields in here

    @staticmethod
    def d():
        ...
```
