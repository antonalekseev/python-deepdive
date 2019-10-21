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

### UserDict


Suppose we want to create our own dictionary type that only allows real numbers for the values, and always returns the values as truncated integers.

We can do this simplistically, without using inheritance, by simply using a "backing" dictionary and implementing our getter and setter methods:

```python
from numbers import Real

class IntDict:
    def __init__(self):
        self._d = {}
        
    def __setitem__(self, key, value):
        if not isinstance(value, Real):
            raise ValueError('Value must be a real number.')
        self._d[key] = value
        
    def __getitem__(self, key):
        return int(self._d[key])
```

```python
d = IntDict()
```

```python
d['a'] = 10.5
```

```python
d['a']
```

```python
d['a'] = 3 + 2j
```

The problem with this approach is that we have lost all the other functionality associated with dictionaries - for example, we cannot use the `get` method, or the `update` method, view objects, etc.

The solution here is to use inheritance. (I will cover OOP and inheritance in detail in Part 4 of this series, but wanted to point a few things out now).

When we inherit from a parent class, we get the functionality of the parent class, and override what we need to override.

In this case, we're going to inherit from the `dict` class, and override the `__setitem__` and `__getitem__` methods.

```python
class IntDict(dict):
    def __setitem__(self, key, value):
        if not isinstance(value, Real):
            raise ValueError('Value must be a real number.')
        super().__setitem__(key, value)
        
    def __getitem__(self, key):
        return int(super().__getitem__(key))        
```

```python
d = IntDict()
d['a'] = 10.5
```

```python
d['a']
```

```python
d['b'] = 'python'
```

So this works, and we also have all the functionality of dictionaries available to us as well - the only things that are different is that we have created overrides for `__setitem__` and `__getitem__`.

```python
d['b'] = 100.5
```

```python
d.keys()
```

We even get the `get` method:

```python
d.get('x', 'N/A')
```

```python
d.get('a')
```

Hmmm... Why did we not get `10` back? We did override the `__getitem__` method after all...


Same problem with the `update` method:

```python
d1 = {}
d1.update(d)
```

```python
d1
```

OK, so that does not work either.
What about merging another dictionary into our custom dictionary. Will that at least honor the override we put in place for the `__setitem__` method?

```python
d.update({'x': 'python'})
```

```python
d
```

Nope... So using the getter and setter directly seems to work, but it looks like many other methods in the dictionary class that get and set values are not actually calling our `__getitem__` and `__setitem__` methods.


The problem is inheriting from these **built-in** types. They do not necessarily use the `__xxx__` methods that we use in our user defined types. For example, when we call `len('abc')`, it does not actually call the `___len__` method that exists in the string class. These special methods are used in our custom classes, but there's absolutely no guarantee that they get used by the built-ins.

And in fact that's exactly what's happening here - the `update` and `get` methods are not using the `__getitem__` method - if they were, our overrides would be called instead - but obviously they are not.

So, inheriting from `dict` works just fine, except when it doesn't!!!

Fortunately, this is where the `UserDict` can help us.

Provided as part of the standard library (in the `collections` module) it allows us to create custom dictionary objects and enjoy the normal inheritance behavior we would expect from non built-in types.

Let's try it out with our example:

```python
from collections import UserDict
```

```python
help(UserDict)
```

As you can see, the methods we would expect from regular `dicts` seem to be present in the `UserDict` class. 
Let's build a custom dictionary type using it:

```python
class IntDict(UserDict):
    def __setitem__(self, key, value):
        if not isinstance(value, Real):
            raise ValueError('Value must be a real number.')
        super().__setitem__(key, value)
        
    def __getitem__(self, key):
        return int(super().__getitem__(key))        
```

```python
d = IntDict()
```

```python
d['a'] = 10.5
d['b'] = 100.5
```

```python
d['c'] = 'python'
```

```python
d.get('a')
```

Nice! The `get` method called our override method.
What about the `update` method?

```python
d1 = {}
d1.update(d)
```

```python
d1
```

Yes! That worked too.


Moreover, we can recover the underlying `dict` object from the `UserDict` objects:

```python
d.data
```

```python
isinstance(d.data, dict)
```

In fact, we can also use the initializer that `UserDict` provides us:

```python
d2 = IntDict(a=10)
d2
```

```python
d1 = IntDict({'a': 1.1, 'b': 2.2, 'c': 3.3})
```

```python
d1
```

You'll notice that the representation here lists the original values - that is correct, since to recreate the exact object we would need to use these values, not the truncated integers returned by `__getitem__`.


However, if we retrieve the items:

```python
d1['a'], d1['b'], d1['c']
```

What if we try to create an instance with an incorrect value type:

```python
d2 = IntDict({'a': 'python'})
```

That works too - so even the initializer is using our overridden `__setitem__` method.


In fact, this even works if we try merging another dictionary into our custom integer dictionary:

```python
d1
```

```python
d1.update({'a': 'python'})
```

So as you can see, subclassing `UserDict` is preferrable to subclassing `dict` - the inheritance behaves more like we would expect with inheritance of user defined classes. The bottom line is that the built-ins are written in C, and make no guarantee as to whether they use these special methods at all.


#### Example


Let's suppose we want to write a custom dictionary where keys can only be from a limited specified set of keys, and the values must be integers from 0-255.

We can attempt to do this in a more general form as follows:

```python
class LimitedDict(UserDict):
    def __init__(self, keyset, min_value, max_value, *args, **kwargs):
        self._keyset = keyset
        self._min_value = min_value
        self._max_value = max_value
        super().__init__(*args, **kwargs)
        
    def __setitem__(self, key, value):
        if key not in self._keyset:
            raise KeyError('Invalid key name.')
        if not isinstance(value, int):
            raise ValueError('Value must be an integer type.')
        if value < self._min_value or value > self._max_value:
            raise ValueError(f'Value must be between {self._min_value} and {self._max_value}')
        super().__setitem__(key, value)
```

```python
d = LimitedDict({'red', 'green', 'blue'}, 0, 255, red=10, green=10, blue=10)
```

```python
d
```

```python
d['red'] = 200
```

```python
d
```

```python
d['purple'] = 100
```

and, similarly we also have bounded key values:

```python
d['red'] = 300
```
