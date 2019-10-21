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

### Creating Python Dictionaries


There are different mechanisms available to create dictionaries in Python.


#### Literals


We can use a literal to create a dictionary:

```python
a = {'k1': 100, 'k2': 200}
```

```python
a
```

Note that the order in which the items are listed in the literal is maintained when listing out the elements of the dictionary. This does not hold for Python version earlier than 3.6 (practically, version 3.5).


Another thing to note is that dictionary **keys** must be hashable objects. Associated values on the other hand can be any object.


So tuples of hashable objects are themselves hashable, but lists are not, even if they only contain hashable elements. Tuples of non-hashable elements are also not hashable.

```python
hash((1, 2, 3))
```

```python
hash([1, 2, 3])
```

```python
hash(([1, 2], [3, 4]))
```

So we can create dictionaries that look like this:

```python
a = {('a', 100): ['a', 'b', 'c'], 'key2': {'a': 100, 'b': 200}}
```

```python
a
```

Interestingly, functions are hashable:

```python
def my_func(a, b, c):
    print(a, b, c)
```

```python
hash(my_func)
```

Which means we can use functions as keys in dictionaries:

```python
d = {my_func: [10, 20, 30]}
```

A simple application of this might be to store the argument values we want to use to call the function at a later time:

```python
def fn_add(a, b):
    return a + b

def fn_inv(a):
    return 1/a

def fn_mult(a, b):
    return a * b
```

```python
funcs = {fn_add: (10, 20), fn_inv: (2,), fn_mult: (2, 8)}
```

Remember that when we iterate through a dictionary we are actually iterating through the keys:

```python
for f in funcs:
    print(f)
```

We can then call the functions this way:

```python
for f in funcs:
    result = f(*funcs[f])
    print(result)
```

We can also iterate through the items (as tuples) in a dictionary as follows:

```python
for f, args in funcs.items():
    print(f, args)
```

So we could now call each function this way:

```python
for f, args in funcs.items():
    result = f(*args)
    print(result)
```

#### Using the class constructor


We can also use the class constructor `dict()` in different ways:


##### Keyword Arguments

```python
d = dict(a=100, b=200)
```

```python
d
```

The restriction here is that the key names must be valid Python identifiers, since they are being used as argument names.


We can also build a dictionary by passing it an iterable containing the keys and the values:

```python
d = dict([('a', 100), ('b', 200)])
```

```python
d
```

The restriction here is that the elements of the iterable must themselves be iterables with exactly two elements.

```python
d = dict([('a', 100), ['b', 200]])
```

```python
d
```

Of course we can also pass a dictionary as well:

```python
d = {'a': 100, 'b': 200, 'c': {'d': 1, 'e': 2}}
```

Here I am using a dictionary that happens to contain a nested dictionary for the key `c`.


Let's look at the id of `d`:

```python
id(d)
```

And let's create a dictionary:

```python
new_dict = dict(d)
```

```python
new_dict
```

What's the id of `new_dict`?

```python
id(new_dict)
```

As you can see, we have a new object - however, what about the nested dictionary?

```python
id(d['c']), id(new_dict['c'])
```

As you can see they are the same - so be careful, using the `dict` constructor this way essentially creates a **shallow copy**.

We'll come back to copying dicts later.


#### Using Comprehensions


We can also create dictionaries using a dictionary comprehension.
This is very similar to list comprehensions or generator expressions.


Suppose we have two iterables, one containing some keys, and one containing some values we want to associate with each key:

```python
keys = ['a', 'b', 'c']
values = (1, 2, 3)
```

We can then easily create a dictionary this way - the non-Pythonic way!

```python
d = {}  # creates an empty dictionary
for k, v in zip(keys, values):
    d[k] = v
```

```python
d
```

But it is much simpler to use a dictionary comprehension:

```python
d = {k: v for k, v in zip(keys, values)}
```

```python
d
```

Dictionary comprehensions support the same syntax as list comprehensions - you can have nested loops, `if` statements, etc.

```python
keys = ['a', 'b', 'c', 'd']
values = (1, 2, 3, 4)

d = {k: v for k, v in zip(keys, values) if v % 2 == 0}
```

```python
d
```

In the following example we are going to create a grid of 2D coordinate pairs, and calculate their distance from the origin:

```python
x_coords = (-2, -1, 0, 1, 2)
y_coords = (-2, -1, 0, 1, 2)
```

If you remember list comprehensions, we would create all possible `(x,y)` pairs using nested loops (a Cartesian product):

```python
grid = [(x, y) 
         for x in x_coords 
         for y in y_coords]
grid
```

```python
import math
```

We can use the `math` module's `hypot` function to do calculate these distances

```python
math.hypot(1, 1)
```

So to calculate these distances for all our points we would do this:

```python
grid_extended = [(x, y, math.hypot(x, y)) for x, y in grid]
grid_extended
```

We can now easily tweak this to make a dictionary, where the coordinate pairs are the key, and the distance the value:

```python
grid_extended = {(x, y): math.hypot(x, y) for x, y in grid}
```

```python
grid_extended
```

#### Using `fromkeys`


The `dict` class also provides the `fromkeys` method that we can use to create dictionaries.
This class method is used to create a dictionary from an iterable containing the keys, and a **single** value used to assign to each key.

```python
counters = dict.fromkeys(['a', 'b', 'c'], 0)
```

```python
counters
```

If we do not specify a value, then `None` is used:

```python
d = dict.fromkeys('abc')
```

```python
d
```

Notice how I used the fact that strings are iterables to specify the three single character keys for this dictionary!


`fromkeys` method will insert the keys in the order in which they are retrieved from the iterable:

```python
d = dict.fromkeys('python')
```

```python
d
```

Uh-Oh!! Looks like the ordering didn't work!!
I've pointed this out a few times already, but Jupyter (this notebook), uses a printing mechanism that will order the keys alphabetically.

To see the real order of the keys in the dict we should use the print statement ourselves:

```python
print(d)
```

Much better! :-)
