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

### Hashing and Equality


By default, when we create a custom class, we inherit `__eq__` and `__hash__` from the `object` class.

```python
dir(object)
```

This means that by default our custom classes produce hashable objects that can be used in mapping types such as dictionaries and sets.

```python
class Person:
    pass
```

```python
p1 = Person()
p2 = Person()
```

```python
hash(p1), hash(p2)
```

```python
p1 == p2
```

By default `__hash__` uses the object's identity, and `__eq__` will only evaluate to `True` if the two objects are the same objects (identity).


We can override those default implementations ourselves. 


If we override the `__eq__` method, Python will automatically make our class unhashable:

```python
class Person:
    def __init__(self, name):
        self.name = name
        
    def __eq__(self, other):
        return isinstance(other, Person) and self.name == other.name
            
```

```python
p1 = Person('John')
p2 = Person('John')
p3 = Person('Eric')
```

```python
p1 == p2, p1 == p3
```

But now we have lost hashing:

```python
try:
    hash(p1)
except TypeError as ex:
    print(ex)
```

This is because two objects that compare equal should also have the same hash. However, Python's default is to use the object's identity. So if that were the case then `p1` and `p2` would be equal, but would not have the same hash.

So Python sets the `__hash__` property to `None`:

```python
type(p1.__hash__)
```

The downside to this is that we can no longer use instances of this class as keys in a dictionary or elements of a set:

```python
try:
    d = {p1: 'person 1'}
except TypeError as ex:
    print(ex)
```

We can however provide our own override for `__hash__`:

```python
class Person:
    def __init__(self, name):
        self.name = name
        
    def __eq__(self, other):
        return isinstance(other, Person) and self.name == other.name
            
    def __hash__(self):
        return hash(self.name)
```

We now have a `Person` class that supports equality based on the state of the class (the `name` in this instance) and is hashable too.

We should also keep in mind that for this to work well in data structurfes such as dictionaries, what we use to create a hash of the class should remain immutable.

So, a better approach would be to make the `name` property a read-only property:

```python
class Person:
    def __init__(self, name):
        self._name = name
        
    @property
    def name(self):
        return self._name
    
    def __eq__(self, other):
        return isinstance(other, Person) and self.name == other.name
            
    def __hash__(self):
        return hash(self.name)
```

And now our Person instances can be used in sets and dictionaries (keys)

```python
p1 = Person('Eric')
```

```python
d = {p1: 'Eric'}
```

```python
d
```

```python
s = {p1}
```

And of course since we now have equality defined in terms of the object state (and not the default of, essentially, the memory address), we can recover an element from a dictionary using different objects (identity wise) that have the same state (equality wise).
