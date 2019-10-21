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

### Project Solution: Goal 2


For this goal we need to rewrite our sequence type `Polygons` into something that implements the iterable protocol.

Furthermore, all iterators should be lazy.


We'll need our `Polygon` class:

```python
import math

class Polygon:
    def __init__(self, n, R):
        if n < 3:
            raise ValueError('Polygon must have at least 3 vertices.')
        self._n = n
        self._R = R
        
        self._interior_angle = None
        self._side_length = None
        self._apothem = None
        self._area = None
        self._perimeter = None
        
    def __repr__(self):
        return f'Polygon(n={self._n}, R={self._R})'
    
    @property
    def count_vertices(self):
        return self._n
    
    @property
    def count_edges(self):
        return self._n
    
    @property
    def circumradius(self):
        return self._R
    
    @property
    def interior_angle(self):
        if self._interior_angle is None:
            self._interior_angle = (self._n - 2) * 180 / self._n
        return self._interior_angle

    @property
    def side_length(self):
        if self._side_length is None:
            self._side_length = 2 * self._R * math.sin(math.pi / self._n)
        return self._side_length
    
    @property
    def apothem(self):
        if self._apothem is None:
            self._apothem = self._R * math.cos(math.pi / self._n)
        return self._apothem
    
    @property
    def area(self):
        if self._area is None:
            self._area = self._n / 2 * self.side_length * self.apothem
        return self._area
    
    @property
    def perimeter(self):
        if self._perimeter is None:
            self._perimeter = self._n * self.side_length
        return self._perimeter
    
    def __eq__(self, other):
        if isinstance(other, self.__class__):
            return (self.count_edges == other.count_edges 
                    and self.circumradius == other.circumradius)
        else:
            return NotImplemented
        
    def __gt__(self, other):
        if isinstance(other, self.__class__):
            return self.count_vertices > other.count_vertices
        else:
            return NotImplemented
```

And here's our original implementation of `Polygons` as a sequence type:

```python
class Polygons:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        self._polygons = [Polygon(i, R) for i in range(3, m+1)]
        
    def __len__(self):
        return self._m - 2
    
    def __repr__(self):
        return f'Polygons(m={self._m}, R={self._R})'
    
    def __getitem__(self, s):
        return self._polygons[s]
    
    @property
    def max_efficiency_polygon(self):
        sorted_polygons = sorted(self._polygons, 
                                 key=lambda p: p.area/p.perimeter,
                                reverse=True)
        return sorted_polygons[0]
```

We now need to implement the iterable protocol - which means we'll need to implement an iterator first.

```python
class PolygonsIterator:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        
    def __iter__(self):
        return self
    
    def __next__(self):
        pass
```

This is the basic skeleton we need to implement the iterator protocol.

So now we need to implement the __next__ method.

This method should simply return the `next` instance of a Polygon - we start with polygons with `3` sides, and work our way up.

To do this, we'll use a private variable `_i` to trqack the number-of-side Polygon to hand out next. So, we'll start `_i` at `3` and work our way up.

```python
class PolygonsIterator:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        self._i = 3
        
    def __iter__(self):
        return self
    
    def __next__(self):
        if self._i > self._m:
            raise StopIteration
        else:
            result = Polygon(self._i, self._R)
            self._i += 1
            return result
```

Let's make sure the iterator works as expected:

```python
p_iter = PolygonsIterator(5, 1)
for p in p_iter:
    print(p)
```

Of course, this is an iterator, so it should be exhausted now:

```python
list(p_iter)
```

Looks good, so next we have to replace the sequence protocol with the iterable protocol in our `Polygons` class.

```python
class Polygons:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        
    def __len__(self):
        return self._m - 2
    
    def __repr__(self):
        return f'Polygons(m={self._m}, R={self._R})'
    
    def __iter__(self):
        return PolygonsIterator(self._m, self._R)
    
    @property
    def max_efficiency_polygon(self):
        sorted_polygons = sorted(self._polygons, 
                                 key=lambda p: p.area/p.perimeter,
                                reverse=True)
        return sorted_polygons[0]
```

And now we should have an iterable:

```python
polygons = Polygons(5, 1)
```

```python
for p in polygons:
    print(p)
```

```python
for p in polygons:
    print(p)
```

Finally, we also need to make our `max_efficiency_polygon` a lazy property:

```python
class Polygons:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        self._max_efficiency_polygon = None
        
    def __len__(self):
        return self._m - 2
    
    def __repr__(self):
        return f'Polygons(m={self._m}, R={self._R})'
    
    def __iter__(self):
        return PolygonsIterator(self._m, self._R)
    
    @property
    def max_efficiency_polygon(self):
        if self._max_efficiency_polygon is None:
            sorted_polygons = sorted(self._polygons, 
                                     key=lambda p: p.area/p.perimeter,
                                    reverse=True)
            self._max_efficiency_polygon = sorted_polygons[0]
        return self._max_efficiency_polygon
```

Let's test that to make sure it still calculates correctly (should always return the largest (in terms of edges/vertices) Polygon in the iterable.

```python
polygons = Polygons(10, 1)
print(polygons.max_efficiency_polygon)
```

As you can see, we have a slight problem. We also need to change the iterable passed to the `sorted` method - we no longer have a list of Polygons.

But that's easily fixed since `sorted` can work with iterables and iterators in general!

```python
class Polygons:
    def __init__(self, m, R):
        if m < 3:
            raise ValueError('m must be greater than 3')
        self._m = m
        self._R = R
        self._max_efficiency_polygon = None
        
    def __len__(self):
        return self._m - 2
    
    def __repr__(self):
        return f'Polygons(m={self._m}, R={self._R})'
    
    def __iter__(self):
        return PolygonsIterator(self._m, self._R)
    
    @property
    def max_efficiency_polygon(self):
        if self._max_efficiency_polygon is None:
            sorted_polygons = sorted(PolygonsIterator(self._m, self._R), 
                                     key=lambda p: p.area/p.perimeter,
                                    reverse=True)
            self._max_efficiency_polygon = sorted_polygons[0]
        return self._max_efficiency_polygon
```

And let's test that again:

```python
polygons = Polygons(10, 1)
print(polygons.max_efficiency_polygon)
```

OK, that seems to work!
