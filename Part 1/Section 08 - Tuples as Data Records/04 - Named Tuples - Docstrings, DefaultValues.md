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

### Named Tuples - DocStrings and Default Values

```python
from collections import namedtuple
```

#### Adding DocStrings to Named Tuples


This is easy to do, both with the generated class, as well as it's properties.

```python
Point2D = namedtuple('Point2D', 'x y')
```

```python
Point2D.__doc__ = 'Represents a 2D Cartesian coordinate'
```

And we can even add docstrings to the properties:

```python
Point2D.x.__doc__ = 'x-coordinate'
Point2D.y.__doc__ = 'y-coordinate'
```

```python
help(Point2D)
```

#### Adding Default Values to Named Tuples


#### Using a Prototype


This technique is in the Python docs, and uses the concept of creating a prototype object that has the default values set:

```python
Vector = namedtuple('Vector', 'x1 y1 x2 y2 origin_x origin_y')
```

```python
vector_zeroorigin = Vector(x1=None, y1=None, x2=None, y2=None, origin_x=0, origin_y=0)
```

```python
vector_zeroorigin
```

The named tuple `vector_zeroorigin` is now a prototype of a vector with zero origin.

To create new vectors using that origin as a default, we no longer use the `Vector` class, but instead use `_replace` as follows:

```python
v1 = vector_zeroorigin._replace(x1=1, y1=1, x2=10, y2=10)
```

```python
v1
```

This certainly works, and can be useful in cases where you may want more than one prototype (e.g. `vector_zeroorigin` and `vector_otherorigin`)


#### Using `__defaults__`


There is an alternative way of doing this. And, in my opinion, a much cleaner alternative.


In Python the default values for a function's parameters are stored as a tuple in the `__defaults__` attribute.



```python
def func(a, b=20, c=30):
    print(a, b, c)
```

```python
func.__defaults__
```

```python
func(10)
```

But the `__defaults__` property is writable:

```python
func.__defaults__ = (200, 300)
```

```python
func(10)
```

In this case, the function we are interested in specifying default values for, is the named tuple class constructor, i.e. `__new__`.

So, we will simply need to set `Vector.__new__.__defaults__` to the desired tuple of default values.

The only thing to note is that if you specify less default values (say `m` values) than the total number of arguments (say `n` values, where `m < n`), then the defaults will apply to the **last** `m` values. Think of it as writing out your field names and default values on two lines, and right-aligning them. (If you specify more, then the values at the beginning are effectively ignored)

```python
Vector.__new__.__defaults__ = (0, 0)
```

Here I am basically setting default values for the last two elements only, i.e. `origin_x` and `origin_y`.

```python
v1 = Vector(0, 0, 10, 10, -10, -10)
```

```python
v1
```

```python
v2 = Vector(5, 5, 20, 20)
```

```python
v2
```

```python
v3 = Vector(x1=1, y1=1, x2=10, y2=10)
```

```python
v3
```

An even simpler way to set default values if you want **all** the defaults to be the same:

```python
Vector.__new__.__defaults__ = (0,) * len(Vector._fields)
```

```python
v5 = Vector()
```

```python
v5
```

Of course, the usual admonishment of not using mutable default values holds here as well.
