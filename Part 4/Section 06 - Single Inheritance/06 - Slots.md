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

### Slots


Let's start with an example of how we use slots:

```python
class Location:
    __slots__ = 'name', '_longitude', '_latitude'
    
    def __init__(self, name, longitude, latitude):
        self._longitude = longitude
        self._latitude = latitude
        self.name = name
        
    @property
    def longitude(self):
        return self._longitude
    
    @property
    def latitude(self):
        return self._latitude
```

`Location` still has that mapping proxy, and we can still add and remove **class** attributes from `Location`:

```python
Location.__dict__
```

```python
Location.map_service = 'Google Maps'
```

```python
Location.__dict__
```

But the use of `slots` affects **instances** of the class:

```python
l = Location('Mumbai', 19.0760, 72.8777)
```

```python
l.name, l.longitude, l.latitude
```

The **instance** no longer has a dictionary for maintaining state:

```python
try:
    l.__dict__
except AttributeError as ex:
    print(ex)
```

This means we can no longer add attributes to the instance:

```python
try:
    l.map_link = 'http://maps.google.com/...'
except AttributeError as ex:
    print(ex)
```

Now we can actually delete the attribute from the instance:

```python
del l.name
```

And as we can see the instance now longer has that attribute:

```python
try:
    print(l.name)
except AttributeError as ex:
    print(f'Attribute Error: {ex}')
```

However we can still re-assign a value to that same attribute:

```python
l.name = 'Mumbai'
```

```python
l.name
```

Mainly we use slots when we expect to have many instances of a class and to gain a performance boost (mostly storage, but also attribute lookup speed). 

```python

```
