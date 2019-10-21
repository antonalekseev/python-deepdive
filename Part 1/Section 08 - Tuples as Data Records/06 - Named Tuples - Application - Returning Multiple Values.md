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

### Named Tuples - Application - Returning Multiple Values


We already know that we can easily return multiple values from a function by using a tuple:

```python
from random import randint, random

def random_color():
    red = randint(0, 255)
    green = randint(0,255)
    blue = randint(0, 255)
    alpha = round(random(), 2)
    return red, green, blue, alpha
```

```python
random_color()
```

So of course, we could call the function this and unpack the results at the same time:

```python
red, green, blue, alpha = random_color()
```

```python
print(f'red={red}, green={green}, blue={blue}, alpha={alpha}')
```

But it might be nicer to use a named tuple:

```python
from collections import namedtuple
```

```python
Color = namedtuple('Color', 'red green blue alpha')

def random_color():
    red = randint(0, 255)
    green = randint(0,255)
    blue = randint(0, 255)
    alpha = round(random(), 2)
    return Color(red, green, blue, alpha)
```

```python
color = random_color()
```

```python
color.red
```

```python
color
```
