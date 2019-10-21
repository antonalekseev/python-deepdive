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

### Lazy Iterables


An iterable is an object that can return an iterator (`__iter__`).

In turn an iterator is an object that can return itself (`__iter__`), and return the next value when asked (`__next__`).

Nothing in all this says that the iterable needs to be a finite collection, or that the elements in the iterable need to be materialized (pre-created) at the time the iterable / iterator is created.


Lazy evaluation is when evaluating a value is deferred until it is actually requested.

It is not specific to iterables however.

Simple examples of lazy evaluation are often seen in classes for calculated properties.


Let's look at an example of a lazy class property:

```python
import math

class Circle:
    def __init__(self, r):
        self.radius = r
        
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, r):
        self._radius = r
        self.area = math.pi * r**2
```

As you can see, in this circle class, every time we set the radius, we re-calculate and store the area. When we request the area of the circle, we simply return the stored value.

```python
c = Circle(1)
```

```python
c.area
```

```python
c.radius = 2
```

```python
c.radius, c.area
```

But instead of doing it this way, we could just calculate the area every time it is requested without actually storing the value:

```python
class Circle:
    def __init__(self, r):
        self.radius = r
        
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, r):
        self._radius = r

    @property
    def area(self):
        return math.pi * self.radius ** 2
```

```python
c = Circle(1)
```

```python
c.area
```

```python
c.radius = 2
```

```python
c.area
```

But the area is always recalculated, so we may take a hybrid approach where we want to store the area so we don't need to recalculate it every time (except when the radius is modified), but delay calculating the area until it is requested - that way if it is never requested, we didn't waste the CPU cycles to calculate it, or the memory to store it.

```python
class Circle:
    def __init__(self, r):
        self.radius = r
        
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, r):
        self._radius = r
        self._area = None

    @property
    def area(self):
        if self._area is None:
            print('Calculating area...')
            self._area = math.pi * self.radius ** 2
        return self._area
```

```python
c = Circle(1)
```

```python
c.area
```

```python
c.area
```

```python
c.radius = 2
```

```python
c.area
```

This is an example of lazy evaluation. We don't actually calculate and store an attribute of the class until it is actually needed.


We can sometimes do something similar with iterables - we don't actually have to store every item of the collection - we may be able to just calculate the item as needed.


In the following example we'll create an iterable of factorials of integers starting at `0`, i.e.

`0!, 1!, 2!, 3!, ..., n!`

```python
class Factorials:
    def __init__(self, length):
        self.length = length
    
    def __iter__(self):
        return self.FactIter(self.length)
    
    class FactIter:
        def __init__(self, length):
            self.length = length
            self.i = 0
            
        def __iter__(self):
            return self
        
        def __next__(self):
            if self.i >= self.length:
                raise StopIteration
            else:
                result = math.factorial(self.i)
                self.i += 1
                return result
            
```

```python
facts = Factorials(5)
```

```python
list(facts)
```

So as you can see, we do not store the values of the iterable, instead we just calculate the items as needed.

In fact, now that we have this iterable, we don't even need it to be finite:

```python
class Factorials:
    def __iter__(self):
        return self.FactIter()
    
    class FactIter:
        def __init__(self):
            self.i = 0
            
        def __iter__(self):
            return self
        
        def __next__(self):
            result = math.factorial(self.i)
            self.i += 1
            return result
```

```python
factorials = Factorials()
fact_iter = iter(factorials)

for _ in range(10):
    print(next(fact_iter))
```

You'll notice that the main part of the iterable code is in the iterator, and the iterable itself is nothing more than a thin shell that allows us to create and access the iterator. This is so common, that there is a better way of doing this that we'll see when we deal with generators.
