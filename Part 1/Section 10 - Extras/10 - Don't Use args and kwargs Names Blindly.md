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

### Don't Use *args and **kwargs Names Blindly


In most of the code we have been working with we used `*args` and `**kwargs`. But these were small code snippets where the argument names did not necessarily have meaning, or were used very generically such as with decorators.

In your code, if those variable positional and keyword-only arguments have meaning, then use a meaningful name instead of just `*args` and `**kwargs`.


#### Example 1


Here, using the conventional names `args` and `kwargs` makes sense since we have no idea what those are - we are simply using them as a pass through to call another function in our decorator:

```python
def audit(f):
    def inner(*args, **kwargs):
        print(f'Called {f.__name__}')
        return f(*args, **kwargs)
    return inner
```

But for the following `product` function, it makes more sense to use `*values` than `*args`.

```python
@audit
def say_hello(name):
    return f'Hello {name}'

from operator import mul
from functools import reduce

@audit
def product(*values):
    return reduce(mul, values)
```

```python
say_hello('Polly')
```

```python
product(1, 2, 3, 4)
```

#### Example 2


Same thing here - using `*item_values` makes more sense than `*args`:

```python
def count_multi(lst, *item_values):
    return sum(lst.count(value) for value in item_values)
```

```python
l = 1, 1, 2, 3, 4, 5, 6, 6, 7, 8, 9, 10
```

```python
count_multi(l, 1, 6, 7)
```

#### Example 3


Suppose we want our class init to allow people to send in additional arbitrary (custom) instance attributes:

```python
class Person:
    def __init__(self, name, age, **custom_attributes):
        self.name = name
        self.age = age
        for attr_name, attr_value in custom_attributes.items():
            setattr(self, attr_name, attr_value)        
```

```python
parrot = Person('Polly', 101, status='stiff', vooms=False)
```

```python
print(vars(parrot))
```

```python
michael = Person('Michael', 42, role='shopkeeper', crooked=True)
```

```python
print(vars(michael))
```

```python

```
