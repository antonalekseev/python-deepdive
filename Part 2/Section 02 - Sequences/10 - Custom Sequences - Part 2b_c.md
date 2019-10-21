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

### Custom Sequences (Part 2b/c)


For this example we'll re-use the Polygon class from a previous lecture on extending sequences.

We are going to consider a polygon as nothing more than a collection of points (and we'll stick to a 2-dimensional space).

So, we'll need a `Point` class, but we're going to use our own custom class instead of just using a named tuple.

We do this because we want to enforce a rule that our Point co-ordinates will be real numbers. We would not be able to use a named tuple to do that and we could end up with points whose `x` and `y` coordinates could be of any type.


First we'll need to see how we can test if a type is a numeric real type.

We can do this by using the numbers module.

```python
import numbers
```

This module contains certain base types for numbers that we can use, such as Number, Real, Complex, etc.

```python
isinstance(10, numbers.Number)
```

```python
isinstance(10.5, numbers.Number)
```

```python
isinstance(1+1j, numbers.Number)
```

We will want our points to be real numbers only, so we can do it this way:

```python
isinstance(1+1j, numbers.Real)
```

```python
isinstance(10, numbers.Real)
```

```python
isinstance(10.5, numbers.Real)
```

So now let's write our Point class. We want it to have these properties:

  1. The `x` and `y` coordinates should be real numbers only
  2. Point instances should be a sequence type so that we can unpack it as needed in the same way we were able to unpack the values of a named tuple.

```python
class Point:
    def __init__(self, x, y):
        if isinstance(x, numbers.Real) and isinstance(y, numbers.Real):
            self._pt = (x, y)
        else:
            raise TypeError('Point co-ordinates must be real numbers.')
            
    def __repr__(self):
        return f'Point(x={self._pt[0]}, y={self._pt[1]})'
    
    def __len__(self):
        return 2
    
    def __getitem__(self, s):
        return self._pt[s]
```

Let's use our point class and make sure it works as intended:

```python
p = Point(1, 2)
```

```python
p
```

```python
len(p)
```

```python
p[0], p[1]
```

```python
x, y = p
```

```python
x, y
```

Now, we can start creatiung our Polygon class, that will essentially be a mutable sequence of points making up the verteces of the polygon.

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        return f'Polygon({self._pts})'
```

Let's try it and see if everything is as we expect:

```python
p = Polygon()
```

```python
p
```

```python
p = Polygon((0,0), [1,1])
```

```python
p
```

```python
p = Polygon(Point(0, 0), [1, 1])
```

```python
p
```

That seems to be working, but only one minor thing - our representation contains those square brackets which technically should not be there as the Polygon class init assumes multiple arguments, not a single iterable.

So we should fix that:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join(self._pts)
        return f'Polygon({pts_str})'
```

But that still won't work, because the `join` method expects an iterable of **strings** - here we are passing it an iterable of `Point` objects:

```python
p = Polygon((0,0), (1,1))
```

```python
p
```

So, let's fix that:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
```

```python
p = Polygon((0,0), (1,1))
```

```python
p
```

Ok, so now we can start making our Polygon into a sequence type, by implementing methods such as `__len__` and `__getitem__`:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
```

Notice how we are simply delegating those methods to the ones supported by lists since we are storing our sequence of points internally using a list!

```python
p = Polygon((0,0), Point(1,1), [2,2])
```

```python
p
```

```python
p[0]
```

```python
p[::-1]
```

Now let's implement concatenation (we'll skip repetition - wouldn't make much sense anyway):

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __add__(self, other):
        if isinstance(other, Polygon):
            new_pts = self._pts + other._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')
```

```python
p1 = Polygon((0,0), (1,1))
p2 = Polygon((2,2), (3,3))
print(id(p1), p1)
print(id(p2), p2)
```

```python
result = p1 + p2
```

```python
print(id(result), result)
```

Now, let's handle in-place concatenation. Let's start by only allowing the RHS of the in-place concatenation to be another Polygon:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __add__(self, other):
        if isinstance(other, Polygon):
            new_pts = self._pts + other._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')
            
    def __iadd__(self, pt):
        if isinstance(pt, Polygon):
            self._pts = self._pts + pt._pts
            return self
        else:
            raise TypeError('can only concatenate with another Polygon')
```

```python
p1 = Polygon((0,0), (1,1))
p2 = Polygon((2,2), (3,3))
print(id(p1), p1)
print(id(p2), p2)
```

```python
p1 += p2
```

```python
print(id(p1), p1)
```

So that worked, but this would not:

```python
p1 = Polygon((0,0), (1,1))
```

```python
p1 += [(2,2), (3,3)]
```

As you can see we get that type error. But we really should be able to handle appending any iterable of Points - and of course Points could also be specified as just iterables of length 2 containing numbers:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')
            
    def __iadd__(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
        return self
```

```python
p1 = Polygon((0,0), (1,1))
```

```python
p1 += [(2,2), (3,3)]
```

```python
p1
```

Now let's implement some methods such as `append`, `extend` and `insert`:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')
            
    def __iadd__(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
        return self
    
    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
            
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
```

Notice how we used almost the same code for `__iadd__` and `extend`?
The only difference is that `__iadd__` returns the object, while `extend` does not - so let's clean that up a bit:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
```

Now let's give all this a try:

```python
p1 = Polygon((0,0), Point(1,1))
p2 = Polygon([2, 2], [3, 3])
print(id(p1), p1)
print(id(p2), p2)
```

```python
p1 += p2
```

```python
print(id(p1), p1)
```

That worked still, now let's see `append`:

```python
p1
```

```python
p1.append((4, 4))
```

```python
p1
```

```python
p1.append(Point(5,5))
```

```python
print(id(p1), p1)
```

`append` seems to be working, now for `extend`:

```python
p3 = Polygon((6,6), (7,7))
```

```python
p1.extend(p3)
```

```python
print(id(p1), p1)
```

```python
p1.extend([(8,8), Point(9,9)])
```

```python
print(id(p1), p1)
```

Now let's see if `insert` works as expected:

```python
p1 = Polygon((0,0), (1,1), (2,2))
```

```python
print(id(p1), p1)
```

```python
p1.insert(1, (100, 100))
```

```python
print(id(p1), p1)
```

```python
p1.insert(1, Point(50, 50))
```

```python
print(id(p1), p1)
```

Now that we have that working, let's turn our attention to the `__setitem__` method so we can support index and slice assignments:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __setitem__(self, s, value):
        # value could be a single Point (or compatible type) for s an int
        # or it could be an iterable of Points if s is a slice
        # let's start by handling slices only first
        self._pts[s] = [Point(*pt) for pt in value]
            
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
```

So, we are only handling slice assignments at this point, not assignments such as `p[0] = Point(0,0)`:

```python
p = Polygon((0,0), (1,1), (2,2))
print(id(p), p)
```

```python
p[0:2] = [(10, 10), (20, 20), (30, 30)]
```

```python
print(id(p), p)
```

So this seems to work fine. But this won't yet:

```python
p[0] = Point(100, 100)
```

If we look at the precise error, we see that our list comprehension is the cause of the error - we fail to correctly handle the case where the value passed in is not an iterable of Points...

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __setitem__(self, s, value):
        # value could be a single Point (or compatible type) for s an int
        # or it could be an iterable of Points if s is a slice
        # we could do this:
        if isinstance(s, int):
            self._pts[s] = Point(*value)
        else:
            self._pts[s] = [Point(*pt) for pt in value]
            
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
```

This will now work as expected:

```python
p = Polygon((0,0), (1,1), (2,2))
print(id(p), p)
```

```python
p[0] = Point(10, 10)
```

```python
print(id(p), p)
```

What happens if we try to assign a single Point to a slice:

```python
p[0:2] = Point(10, 10)
```

As expected this will not work. What about assigning an iterable of points to an index:

```python
p[0] = [Point(10, 10), Point(20, 20)]
```

This works fine, but the error messages are a bit misleading - we probably should do something about that:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __setitem__(self, s, value):
        # we first should see if we have a single Point
        # or an iterable of Points in value
        try:
            rhs = [Point(*pt) for pt in value]
            is_single = False
        except TypeError:
            # not a valid iterable of Points
            # maybe a single Point?
            try:
                rhs = Point(*value)
                is_single = True
            except TypeError:
                # still no go
                raise TypeError('Invalid Point or iterable of Points')
        
        # reached here, so rhs is either an iterable of Points, or a Point
        # we want to make sure we are assigning to a slice only if we 
        # have an iterable of points, and assigning to an index if we 
        # have a single Point only
        if (isinstance(s, int) and is_single) \
            or isinstance(s, slice) and not is_single:
            self._pts[s] = rhs
        else:
            raise TypeError('Incompatible index/slice assignment')
                
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
```

So now let's see if we get better error messages:

```python
p1 = Polygon((0,0), (1,1), (2,2))
```

```python
p1[0:2] = (10,10)
```

```python
p1[0] = [(0,0), (1,1)]
```

And the allowed slice/index assignments work as expected:

```python
p[0] = Point(100, 100)
```

```python
p
```

```python
p[0:2] = [(0,0), (1,1), (2,2)]
```

```python
p
```

And if we try to replace with bad Point data:

```python
p[0] = (0, 2+2j)
```

We also get a better error message.


Lastly let's see how we would implement the `del` keyword and the `pop` method.


Recall how the `del` keyword works for a list:

```python
l = [1, 2, 3, 4, 5]
```

```python
del l[0]
```

```python
l
```

```python
del l[0:2]
```

```python
l
```

```python
del l[-1]
```

```python
l
```

So, `del` works with indices (positive or negative) and slices too. We'll do the same:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __setitem__(self, s, value):
        # we first should see if we have a single Point
        # or an iterable of Points in value
        try:
            rhs = [Point(*pt) for pt in value]
            is_single = False
        except TypeError:
            # not a valid iterable of Points
            # maybe a single Point?
            try:
                rhs = Point(*value)
                is_single = True
            except TypeError:
                # still no go
                raise TypeError('Invalid Point or iterable of Points')
        
        # reached here, so rhs is either an iterable of Points, or a Point
        # we want to make sure we are assigning to a slice only if we 
        # have an iterable of points, and assigning to an index if we 
        # have a single Point only
        if (isinstance(s, int) and is_single) \
            or isinstance(s, slice) and not is_single:
            self._pts[s] = rhs
        else:
            raise TypeError('Incompatible index/slice assignment')
                
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
        
    def __delitem__(self, s):
        del self._pts[s]
```

```python
p = Polygon(*zip(range(6), range(6)))
```

```python
p
```

```python
del p[0]
```

```python
p
```

```python
del p[-1]
```

```python
p
```

```python
del p[0:2]
```

```python
p
```

Now, we just have to implement `pop`:

```python
class Polygon:
    def __init__(self, *pts):
        if pts:
            self._pts = [Point(*pt) for pt in pts]
        else:
            self._pts = []
            
    def __repr__(self):
        pts_str = ', '.join([str(pt) for pt in self._pts])
        return f'Polygon({pts_str})'
    
    def __len__(self):
        return len(self._pts)
    
    def __getitem__(self, s):
        return self._pts[s]
    
    def __setitem__(self, s, value):
        # we first should see if we have a single Point
        # or an iterable of Points in value
        try:
            rhs = [Point(*pt) for pt in value]
            is_single = False
        except TypeError:
            # not a valid iterable of Points
            # maybe a single Point?
            try:
                rhs = Point(*value)
                is_single = True
            except TypeError:
                # still no go
                raise TypeError('Invalid Point or iterable of Points')
        
        # reached here, so rhs is either an iterable of Points, or a Point
        # we want to make sure we are assigning to a slice only if we 
        # have an iterable of points, and assigning to an index if we 
        # have a single Point only
        if (isinstance(s, int) and is_single) \
            or isinstance(s, slice) and not is_single:
            self._pts[s] = rhs
        else:
            raise TypeError('Incompatible index/slice assignment')
                
    def __add__(self, pt):
        if isinstance(pt, Polygon):
            new_pts = self._pts + pt._pts
            return Polygon(*new_pts)
        else:
            raise TypeError('can only concatenate with another Polygon')

    def append(self, pt):
        self._pts.append(Point(*pt))
        
    def extend(self, pts):
        if isinstance(pts, Polygon):
            self._pts = self._pts + pts._pts
        else:
            # assume we are being passed an iterable containing Points
            # or something compatible with Points
            points = [Point(*pt) for pt in pts]
            self._pts = self._pts + points
    
    def __iadd__(self, pts):
        self.extend(pts)
        return self
    
    def insert(self, i, pt):
        self._pts.insert(i, Point(*pt))
        
    def __delitem__(self, s):
        del self._pts[s]
        
    def pop(self, i):
        return self._pts.pop(i)
```

```python
p = Polygon(*zip(range(6), range(6)))
```

```python
p
```

```python
p.pop(1)
```

```python
p
```
