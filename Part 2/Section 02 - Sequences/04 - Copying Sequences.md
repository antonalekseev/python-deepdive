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

### Copying Sequences


#### Shallow Copies


##### Simple Loop


Really not a very Pythonic approach, but it works...

```python
l1 = [1, 2, 3]

l1_copy = []
for item in l1:
    l1_copy.append(item)

print(l1_copy)
```

And we can see that `l1` and `l1_copy` are not the same objects:

```python
l1 is l1_copy
```

##### List Comprehension


We can use a list comprehension to do exactly what we did in the previous example:

```python
l1 = [1, 2, 3]
l1_copy = [item for item in l1]
print(l1_copy)
```

And once again, the objects are not the same:

```python
l1 is l1_copy
```

##### Using the copy() method


Since lists are mutable sequence types, they have the `copy()` method.

```python
l1 = [1, 2, 3]
l1_copy = l1.copy()
print(l1_copy)
```

And once again, the objects are different:

```python
l1 is l1_copy
```

##### Using the built-in list() Function


The built-in `list()` function will make a list out of any iterable. This always ends up with a copy of the iterable:

```python
l1 = [1, 2, 3]
```

```python
l1_copy = list(l1)
print(l1_copy)
```

```python
l1 is l1_copy
```

Note that `list()` will take in any iterable, so you can technically copy any iterable into a list:

```python
t1 = (1, 2, 3)
t1_copy = list(t1)
print(t1_copy)
```

Of course, we get a list, not a tuple - so not exactly a copy.


We've seen this before, but be careful with the `tuple()` built-in function. When we copy tuples, since they are immutable, we just get the original tuple back:

```python
t1 = (1, 2, 3)
t1_copy = tuple(t1)
print(t1_copy)
```

But here, the objects are the **same**:

```python
t1 is t1_copy
```

##### Using Slicing


We can also use slicing to copy sequences.

We'll cover slicing in detail in an upcoming lecture, but with slicing we can also access subsets of the sequence - here we use slicing to select the entire sequence:

```python
l1 = [1, 2, 3]
l1_copy = l1[:]
print(l1_copy)
print(l1 is l1_copy)
```

But again, watch out with tuples!!

```python
t1 = (1, 2, 3)
t1_copy = t1[:]
print(t1_copy)
print(t1 is t1_copy)
```

As you can see, since the slice was the entire tuple, a copy was not made, instead the reference to the original tuple was returned!


Same deal with strings:

```python
s1 = 'python'
s2 = str(s1)
print(s2)
print(s1 is s2)
```

```python
s1 = 'python'
s2 = s1[:]
print(s2)
print(s1 is s2)
```

If you're wondering why Python has that behavior, just think about it.

If you create a copy of a tuple, what are you going to do to that copy? Modify it?? You can't!

Modify the contents of a contained mutable element? Sure you can, but whether you had a copy or not, you would still be modifying the **same** element - having the sequence copied is no safer than not.

Not needed, so Python basically optimizes things for us.


##### The `copy` module

```python
import copy
```

The `copy` module has a generic `copy` function as well:

```python
l1 = [1, 2, 3]
l1_copy = copy.copy(l1)
print(l1_copy)
print(l1 is l1_copy)
```

And for tuples:

```python
t1 = (1, 2, 3)
t1_copy = copy.copy(t1)
print(t1_copy)
print(t1 is t1_copy)
```

As you can see the same thing happens with tuples as we saw before.


#### Shallow vs Deep Copies


What we have been doing so far is creating **shallow** copies.

This means that when a sequence is copied, each element of the new sequence is bound to precisely the same memory address as the corresponding element in the original sequence:

```python
v1 = [0, 0]
v2 = [0, 0]

line1 = [v1, v2]
```

```python
print(line1)
print(id(line1[0]), id(line1[1]))
```

Now let's make a copy of the line using any of the techniques we just looked at:

```python
line2 = line1.copy()
```

```python
line1 is line2
```

So not the same objects. Now let's look at the contained elements themselves:

```python
print(id(line1[0]), id(line1[1]))
print(id(line2[0]), id(line2[1]))
```

As you can see, the element references are the same!


So, if we do this:

```python
line2[0][0] = 100
```

```python
line2
```

```python
line1
```

`line1`'s contents has also changed.


If we want the contained elements **also** to be copied, then we need to explicitly do so as well. This is called creating a **deep** copy.

Let's see how we might do this:

```python
v1 = [0, 0]
v2 = [0, 0]

line1 = [v1, v2]
```

```python
line2 = [item[:] for item in line1]
```

```python
print(id(line1[0]), id(line1[1]))
print(id(line2[0]), id(line2[1]))
```

As you can see, now we have copies of the elements as well:

```python
line1[0][0] = 100
print(line1)
print(line2)
```

and `line2` is unaffacted when we modify `line1`.

So not only did we do a copy of `line1`, but we also made a shallow copy of `v1` and `v2` as well.

But the problem is that we only went two levels deep - what if the variables `v1` and `v2` themselves contained mutable types instead of just integers? We would have to nest deeper and deeper - in general that's what a deep copy needs to do, and usually recursive approaches need to be used.


Fortunately, Python has that functionality built-in for us so we don't have to do that!


The `copy` module has a `deepcopy()` function we can use to create deep copies. It handles all kinds of weird situations where we might have circular references - doing it ourselves is certainly possible, but does take some work.

```python
v1 = [0, 0]
v2 = [0, 0]
line1 = [v1, v2]
```

```python
line2 = copy.deepcopy(line1)
print(id(line1[0]), id(line1[1]))
print(id(line2[0]), id(line2[1]))
```

```python
line2[0][0] = 100
```

```python
print(line1)
print(line2)
```

And of course, it works with any level of nested objects:

```python
v1 = [11, 12]
v2 = [21, 22]
line1 = [v1, v2]

v3 = [31, 32]
v4 = [41, 42]
line2 = [v3, v4]

plane1 = [line1, line2]
print(plane1)
```

```python
plane2 = copy.deepcopy(plane1)
```

```python
print(plane2)
```

```python
print(plane1[0], id(plane1[0]))
print(plane2[0], id(plane2[0]))
```

```python
print(plane1[0][0], id(plane1[0][0]))
print(plane2[0][0], id(plane2[0][0]))
```

#### Even works with custom classes

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return f'Point({self.x}, {self.y})'
    
class Line:
    def __init__(self, p1, p2):
        self.p1 = p1
        self.p2 = p2
        
    def __repr__(self):
        return f'Line({self.p1.__repr__()}, {self.p2.__repr__()})'
```

```python
p1 = Point(0, 0)
p2 = Point(10, 10)
line1 = Line(p1, p2)
line2 = copy.deepcopy(line1)

print(line1.p1, id(line1.p1))
print(line2.p1, id(line2.p1))
```

As you can see, the memory address of the points are different - that was because of the deep copy.


However, if we had done a shallow copy:

```python
p1 = Point(0, 0)
p2 = Point(10, 10)
line1 = Line(p1, p2)
line2 = copy.copy(line1)

print(line1.p1, id(line1.p1))
print(line2.p1, id(line2.p1))
```

As you can see, the memory address of the points are now the **same**.
