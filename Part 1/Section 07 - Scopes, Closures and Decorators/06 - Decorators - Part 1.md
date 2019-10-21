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

### Decorators (Part 1)


Recall the example in the last section where we wrote a simple closure to count how many times a function had been run:

```python
def counter(fn):
    count = 0
    
    def inner(*args, **kwargs):
        nonlocal count
        count += 1
        print('Function {0} was called {1} times'.format(fn.__name__, count))
        return fn(*args, **kwargs)
    return inner
```

```python
def add(a, b=0):
    """
    returns the sum of a and b
    """
    return a + b
```

```python
help(add)
```

Here's the memory address that `add` points to:

```python
id(add)
```

Now we create a closure using the `add` function as an argument to the `counter` function:

```python
add = counter(add)
```

And you'll note that `add` is no longer the same function as before. Indeed the memory address `add` points to is no longer the same:

```python
id(add)
```

```python
add(1, 2)
```

```python
add(2, 2)
```

What happened is that we put our **add** function 'through' the **counter** function - we usually say that we **decorated** our function **add**.

And we call that **counter** function a **decorator**.

There is a shorthand way of decorating our function without having to type:

``func = counter(func)``

```python
@counter
def mult(a: float, b: float=1, c: float=1) -> float:
    """
    returns the product of a, b, and c
    """
    return a * b * c
```

```python
mult(1, 2, 3)
```

```python
mult(2, 2, 2)
```

Let's do a little bit of introspection on our two decorated functions:

```python
add.__name__
```

```python
mult.__name__
```

As you can see, the name of the function is no longer **add** or **mult**, but instead it is the name of that **inner** function in our decorator.

```python
help(add)
```

```python
help(mult)
```

As you can see, we've also lost our docstring and parameter annotations!


What about introspecting the parameters of **add** and **mult**:

```python
import inspect
```

```python
inspect.getsource(add)
```

```python
inspect.getsource(mult)
```

Even the signature is gone:

```python
inspect.signature(add)
```

```python
inspect.signature(mult)
```

Even the parameter defaults documentation is are gone:

```python
inspect.signature(add).parameters
```

In general, when we create decorated functions, we end up "losing" a lot of the metadata of our original function!


However, we **can** put that information back in - it can get quite complicated.

Let's see how we might be able to do that for some simple things, like the docstring and the function name.

```python
def counter(fn):
    count = 0
    
    def inner(*args, **kwargs):
        nonlocal count
        count += 1
        print("{0} was called {1} times".format(fn.__name__, count))
    inner.__name__ = fn.__name__
    inner.__doc__ = fn.__doc__
    return inner
```

```python
@counter
def add(a: int, b: int=10) -> int:
    """
    returns sum of two integers
    """
    return a + b
```

```python
help(add)
```

```python
add.__name__
```

At least we have the docstring and function name back... But what about the parameters? Our real **add** function takes two positional parameters, but because the closure used a generic way of accepting **\*args** and **\*\*kwargs**, we lose this information


We can use a special function in the **functools** module, called **wraps**. In fact, that function is a decorator itself!

```python
from functools import wraps
```

```python
def counter(fn):
    count = 0
    
    @wraps(fn)
    def inner(*args, **kwargs):
        nonlocal count
        count += 1
        print("{0} was called {1} times".format(fn.__name__, count))

    return inner
```

```python
@counter
def add(a: int, b: int=10) -> int:
    """
    returns sum of two integers
    """
    return a + b
```

```python
help(add)
```

Yay!!! Everything is back to normal.

```python
inspect.getsource(add)
```

```python
inspect.signature(add)
```

```python
inspect.signature(add).parameters
```
