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

### Copying Sets


Just as with other container types, we need to differentiate between shallow copies and deep copies.


Python sets implement a `copy` method that creates a shallow copy of the set. And, just as with lists, tuples, dictionaries, etc, we can also use unpacking to shallow copy sets. We can also just use the `set()` function to shallow copy one set into another.

Deep copies of sets can be done using the `deepcopy` function in the `copy` module.

The concepts and techniques are not new, so I won't spend much time on them.


#### Shallow Copies using the `copy` method


To illustrate the shallow copy vs deepcopy issues, we'll create our own mutable, but hashable type:

```python
class Person:
    def __init__(self, name):
        self.name = name
    
    def __repr__(self):
        return f'Person(name={self.name})'
```

```python
p1 = Person('John')
p2 = Person('Eric')
```

```python
s1 = {p1, p2}
```

```python
s1
```

Now let's make a shallow copy:

```python
s2 = s1.copy()
```

```python
s1 is s2
```

As we can see the sets are not the same, however their contained elements **are**:

```python
p1.name = 'John Cleese'
```

```python
s1
```

```python
s2
```

#### Shallow copies using unpacking


We can use unpacking, similar to iterable unpacking to unpack one set into another:

```python
s3 = {*s2}
```

```python
s3 is s2
```

```python
s3
```

```python
p2.name = 'Eric Idle'
```

```python
print(s1)
print(s2)
print(s3)
```

#### Shallow copies using the `set()` function

```python
s4 = set(s1)
```

```python
s4 is s1
```

```python
s4
```

```python
p1.name = 'Michael Palin'
```

```python
print(s1)
print(s2)
print(s3)
print(s4)
```

#### Deep Copies

```python
from copy import deepcopy
```

```python
s5 = deepcopy(s1)
```

```python
s1 is s5
```

```python
s1
```

```python
s5
```

```python
p1.name = 'Terry Jones'
```

```python
print(s1)
print(s2)
print(s3)
print(s4)
print(s5)
```

As you can see, the deep copy also made (deep) copies of each element in the set being (deep) copied.
