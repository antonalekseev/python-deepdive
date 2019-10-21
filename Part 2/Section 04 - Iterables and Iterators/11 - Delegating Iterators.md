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

### Delegating Iterators


Often we write classes that use some existing iterable for the data contained in our class. By default, that class is not iterable, and we would need to implement an iterator for our class and implement the `__iter__` method in our class to return new instances of that iterator.


But, if our underlying data structure for our class is already an iterable, there's a much quicker way of doing it - delegation.


We'll start with a really simple example first:

```python
from collections import namedtuple

Person = namedtuple('Person', 'first last')
```

```python
class PersonNames:
    def __init__(self, persons):
        try:
            self._persons = [person.first.capitalize()
                             + ' ' + person.last.capitalize()
                            for person in persons]
        except (TypeError, AttributeError):
            self._persons = []
```

```python
persons = [Person('michaeL', 'paLin'), Person('eric', 'idLe'), 
           Person('john', 'cLeese')]
```

```python
person_names = PersonNames(persons)
```

Technically we can see the underlying data by accessing the (pseudo) private variable `_persons`.

```python
person_names._persons
```

But we really would prefer making our `PersonNames` instances iterable.

To do so we need to implement the `__iter__` method that returns an iterator that can be used for iterating over the `_persons` list.

But lists are iterables, so they can provide an iterator, and that's precisely what we'll do - we'll **delegate** our own iterator, to the list's iterator:

```python
class PersonNames:
    def __init__(self, persons):
        try:
            self._persons = [person.first.capitalize()
                             + ' ' + person.last.capitalize()
                            for person in persons]
        except TypeError:
            self._persons = []
    
    def __iter__(self):
        return iter(self._persons)
```

And now, `PersonNames` is iterable!

```python
persons = [Person('michaeL', 'paLin'), Person('eric', 'idLe'), 
           Person('john', 'cLeese')]
person_names = PersonNames(persons)
```

```python
for p in person_names:
    print(p)
```

And of course we can sort, use list comprehensions, and so on - our PersonNames **is** an iterable.


Here we sort the names based on the full name, then split the names (on the space) and return a tuple of first name, last name:

```python
[tuple(person_name.split()) for person_name in sorted(person_names)]
```

Or, if we want to sort based on the last name:

```python
sorted(person_names, key=lambda x: x.split()[1])
```
