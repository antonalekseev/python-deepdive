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


Here is the final `Polygon` class we ended up with in goal 1:

```python
import math

class Polygon:
    def __init__(self, n, R):
        if n < 3:
            raise ValueError('Polygon must have at least 3 vertices.')
        self._n = n
        self._R = R
        
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
        return (self._n - 2) * 180 / self._n

    @property
    def side_length(self):
        return 2 * self._R * math.sin(math.pi / self._n)
    
    @property
    def apothem(self):
        return self._R * math.cos(math.pi / self._n)
    
    @property
    def area(self):
        return self._n / 2 * self.side_length * self.apothem
    
    @property
    def perimeter(self):
        return self._n * self.side_length
    
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

Now we need to create a sequence type that will return these Polygons, starting with 3 vertices, up to (and including) a polygon of `m` sides.

Our sequence type will need to implement:
* a `__len__` method
* a `__getitem__` method
* a method that identifies the polygon with largest area to perimeter ratio: let's call it `max_efficiency_polygon` - note that the Polygon class does not have an `efficiency` method, so we'll have to calculate it outside of the Polygon class.


Let's start with some of the basics:

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
```

Let's make sure this works as intended:

```python
polygons = Polygons(2, 10)
```

That's exactly what we want, so that's good.

```python
polygons = Polygons(3, 1)
```

```python
len(polygons)
```

```python
polygons = Polygons(6, 1)
len(polygons)
```

Let's also test the representation:

```python
polygons
```

 Let's now implement a list that will contain all the polygons.
 
 We'll do that in the `__init__` method as well.

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
```

Before we can test this, we need to implement the `__getitem__` method:

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
```

Notice how easy it was using delegation to the underlying list of polygons!


Let's test this out:

```python
polygons = Polygons(8, 1)
```

```python
for p in polygons:
    print(p)
```

```python
for p in polygons[2:5]:
    print(p)
```

```python
for p in polygons[::-1]:
    print(p)
```

We still need to implement a method that identifies the polygon with highest area:perimeter ratio.

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

```python
polygons = Polygons(10, 1)
```

```python
polygons.max_efficiency_polygon
```

Let's test this to make sure that is correct:

```python
[(p, p.area/p.perimeter) for p in polygons]
```

So, looks like our `max_efficiency_polygon` method is working correctly.


As one last thing, we could look at the surface area of our polygons, as the number of vertices become larger and larger.

As we have more and more sides, the polygon becomes a closer and closer approximation to a circle. So, the area should get closer and closer to $\pi$ if we use a circumradius of `1`.

```python
polygons = Polygons(500, 1)
```

```python
polygons[-1].area
```

Yep, seems to be working!
