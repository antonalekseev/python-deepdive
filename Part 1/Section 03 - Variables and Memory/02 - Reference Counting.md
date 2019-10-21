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

## Reference Counting


Method that returns the reference count for a given variable's memory address:

```python
import ctypes

def ref_count(address):
    return ctypes.c_long.from_address(address).value
```

Let's make a variable, and check it's reference count:

```python
my_var = [1, 2, 3, 4]
ref_count(id(my_var))
```

There is another built-in function we can use to obtain the reference count:

```python
import sys
sys.getrefcount(my_var)
```

But why is this returning 2, instead of the expected 1 we obtained with the previous function?

Answer: The *sys.getrefcount()* function takes **my_var** as an argument, this means it receives (and stores) a reference to **my_var**'s memory address **also** - hence the count is off by 1. So we will use *from_address()* instead.


We make another reference to the **same** reference as `my_var`:

```python
other_var = my_var
```

Let's look at the memory address of those two variables and the reference counts:

```python
print(hex(id(my_var)), hex(id(other_var)))
print(ref_count(id(my_var)))
```

Force one reference to go away:

```python
other_var = None
```

And we look at the reference count again:

```python
print(ref_count(id(my_var)))
print(my_var)
```

We see that the reference count has gone back to 1.


You'll probably never need to do anything like this in Python. Memory management is completely transparent - this is just to illustrate some of what is going behind the scenes as it helps to understand upcoming concepts.


**My note:** In order to find actual references one should probably use `gc.get_referrers(my_var)`, see also https://stackoverflow.com/questions/11891755/find-all-references-to-an-object-in-python, https://docs.python.org/2/library/gc.html
