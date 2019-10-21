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

### Python 3.6: Preserved Order of **kwargs


PEP468 was accepted into 3.6, and this means that the order in which keyword arguments are collected into `**kwargs` is now maintained.

Why does that matter you ask?

Well, how about a function that takes in kwargs and needs to build an ordered type based on the *order* of the kwargs. In the past you would have had to use some workaround (often using an ordered dict) instead of the far more convenient `**kwargs` approach.


Let me show you what I mean:


Suppose we want to write a `namedtuple` factory of our own where we can specify default values:
    

```python
from collections import namedtuple

def defaulted_namedtuple(class_name, **fields):
    Struct = namedtuple(class_name, fields.keys())
    Struct.__new__.__defaults__ = tuple(fields.values())
    return Struct    
```

```python
Vector2D = defaulted_namedtuple('Vector2D', 
                                x1=None, y1=None,
                                x2=None, y2=None,
                                origin_x=0, origin_y=0)
```

```python
Vector2D._fields
```

```python
v1 = Vector2D(10, 10, 20, 20)
```

```python
v1
```

This only works if the order of the keyword arguments passed to the function is retained in the `fields` dictionary!
