---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.1'
      jupytext_version: 1.2.4
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

### Objects and Classes


A class is a type of object. In Python we create classes using the `class` keyword.

```python
class Person:
    pass
```

Now this class doesn't do much, but it is an object of type `type` (which is itself an object).

```python
type(Person)
```

```python
type(type)
```

Classes have "built-in" attributes, even though we did not specifically add any to the class ourselves.

For example, they have a name:

```python
Person.__name__
```

They are also callables, and calling a class results in the creation and return of a new **instance** of that class:

```python
p = Person()
```

Now the type of the object is the class used to build that object:

```python
type(p)
```

These instances also have "built_in" properties, which we will cover throughout this course.

For example, they have a `__class__` property that tells us which class was used to create the instance:

```python
p.__class__
```

As you can see that returns the class object used to instantiate `p`.

In fact:

```python
type(p) is p.__class__
```

We can also use `isinstance` to test if an object is an instance of a particular class - now this gets a bit more complicated when we use inheritance, but right now we're not, so it's quite straightforward:

```python
isinstance(p, Person)
```

```python
isinstance(p, str)
```

We can even use `isinstance` with our class, since we know it's type is `type`:

```python
isinstance(Person, type)
```

`type` is like the most generic kind of **class** object - we'll come back to this when discussing meta programming.

We really need inheritance to understand how this works, but every class **is** a `type` object (it inherits all the properties of `type`).

For now let's just see what functionality `type` has:

```python
help(type)
```

As you can see it has a `__call__` method (that's how our class becomes callable), and a bunch of other attributes and methods that we'll see throughout this course.

Our class objects also have these properties, because they inherit from the `type` object.


And in fact, `type` is an instance of itself - that's kind of weird, and not the case for our own classes:

```python
isinstance(type, type)
```

```python
isinstance(Person, Person)
```
