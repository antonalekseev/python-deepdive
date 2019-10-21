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

### Sequence Types


Sequence types have the general concept of a first element, a second element, and so on. Basically an ordering of the sequence items using the natural numbers. In Python (and many other languages) the starting index is set to `0`, not `1`.

So the first item has index `0`, the second item has index `1`, and so on.


Python has built-in mutable and immutable sequence types.


Strings, tuples are immutable - we can access but not modify the **content** of the **sequence**:

```python
t = (1, 2, 3)
```

```python
t[0]
```

```python
t[0] = 100
```

But of course, if the sequence contains mutable objects, then although we cannot modify the sequence of elements (cannot replace, delete or insert elements), we certainly **can** change the contents of the mutable objects:

```python
t = ( [1, 2], 3, 4)
```

`t` is immutable, but its first element is a mutable object:

```python
t[0][0] = 100
```

```python
t
```

#### Iterables


An **iterable** is just something that can be iterated over, for example using a `for` loop:

```python
t = (10, 'a', 1+3j)
```

```python
s = {10, 'a', 1+3j}
```

```python
for c in t:
    print(c)
```

```python
for c in s:
    print(c)
```

Note how we could iterate over both the tuple and the set. Iterating the tuple preserved the **order** of the elements in the tuple, but not for the set. Sets do not have an ordering of elements - they are iterable, but not sequences.


Most sequence types support the `in` and `not in` operations. Ranges do too, but not quite as efficiently as lists, tuples, strings, etc.

```python
'a' in ['a', 'b', 100]
```

```python
100 in range(200)
```

#### Min, Max and Length


Sequences also generally support the `len` method to obtain the number of items in the collection. Some iterables may also support that method.

```python
len('python'), len([1, 2, 3]), len({10, 20, 30}), len({'a': 1, 'b': 2})
```

Sequences (and even some iterables) may support `max` and `min` as long as the data types in the collection can be **ordered** in some sense (`<` or `>`).

```python
a = [100, 300, 200]
min(a), max(a)
```

```python
s = 'python'
min(s), max(s)
```

```python
s = {'p', 'y', 't', 'h', 'o', 'n'}
min(s), max(s)
```

But if the elements do not have an ordering defined:

```python
a = [1+1j, 2+2j, 3+3j]
min(a)
```

`min` and `max` will work for heterogeneous types as long as the elements are pairwise comparable (`<` or `>` is defined). 

For example:

```python
from decimal import Decimal
```

```python
t = 10, 20.5, Decimal('30.5')
```

```python
min(t), max(t)
```

```python
t = ['a', 10, 1000]
min(t)
```

Even `range` objects support `min` and `max`:

```python
r = range(10, 200)
min(r), max(r)
```

#### Concatenation


We can **concatenate** sequences using the `+` operator:

```python
[1, 2, 3] + [4, 5, 6]
```

```python
(1, 2, 3) + (4, 5, 6)
```

Note that the type of the concatenated result is the same as the type of the sequences being concatenated, so concatenating sequences of varying types will not work:

```python
(1, 2, 3) + [4, 5, 6]
```

```python
'abc' + ['d', 'e', 'f']
```

Note: if you really want to concatenate varying types you'll have to transform them to a common type first:

```python
(1, 2, 3) + tuple([4, 5, 6])
```

```python
tuple('abc') + ('d', 'e', 'f')
```

```python
''.join(tuple('abc') + ('d', 'e', 'f'))
```

#### Repetition


Most sequence types also support **repetition**, which is essentially concatenating the same sequence an integer number of times:

```python
'abc' * 5
```

```python
[1, 2, 3] * 5
```

We'll come back to some caveats of concatenation and repetition in a bit.


#### Finding things in Sequences


We can find the index of the occurrence of an element in a sequence:

```python
s = "gnu's not unix"
```

```python
s.index('n')
```

```python
s.index('n', 1), s.index('n', 2), s.index('n', 8)
```

An exception is raised of the element is not found, so you'll want to catch it if you don't want your app to crash:

```python
s.index('n', 13)
```

```python
try:
    idx = s.index('n', 13)
except ValueError:
    print('not found')
```

Note that these methods of finding objects in sequences do not assume that the objects in the sequence are ordered in any way. These are basically searches that iterate over the sequence until they find (or not) the requested element.

If you have a sorted sequence, then other search techniques are available - such as binary searches. I'll cover some of these topics in the extras section of this course.


#### Slicing


We'll come back to slicing in a later lecture, but sequence types generally support slicing, even ranges (as of Python 3.2). Just like concatenation, slices will return the same type as the sequence being sliced:

```python
s = 'python'
l = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

```python
s[0:3], s[4:6]
```

```python
l[0:3], l[4:6]
```

It's ok to extend ranges past the bounds of the sequence:

```python
s[4:1000]
```

If your first argument in the slice is `0`, you can even omit it. Omitting the second argument means it will include all the remaining elements:

```python
s[0:3], s[:3]
```

```python
s[3:1000], s[3:], s[:]
```

We can even have extended slicing, which provides a start, stop and a step:

```python
s, s[0:5], s[0:5:2]
```

```python
s, s[::2]
```

Technically we can also use negative values in slices, including extended slices (more on that later):

```python
s, s[-3:-1], s[::-1]
```

```python
r = range(11)  # numbers from 0 to 10 (inclusive)
```

```python
print(r)
print(list(r))
```

```python
print(r[:5])
```

```python
print(list(r[:5]))
```

As you can see, slicing a range returns a range object as well, as expected.


#### Hashing


Immutable sequences generally support a `hash` method that we'll discuss in detail in the section on mapping types:

```python
l = (1, 2, 3)
hash(l)
```

```python
s = '123'
hash(s)
```

```python
r = range(10)
hash(r)
```

But mutable sequences (and mutable types in general) do not:

```python
l = [1, 2, 3]
```

```python
hash(l)
```

Note also that a hashable sequence, is no longer hashable if one (or more) of it's elements are not hashable:

```python
t = (1, 2, [10, 20])
hash(t)
```

But this would work:

```python
t = ('python', (1, 2, 3))
hash(t)
```

In general, immutable types are likely hashable, while immutable types are not. So numbers, strings, tuples, etc are hashable, but lists and sets are not:

```python
from decimal import Decimal
d = Decimal(10.5)
hash(d)
```

Sets are not hashable:

```python
s = {1, 2, 3}
hash(s)
```

But frozensets, an immutable variant of the set, are:

```python
s = frozenset({1, 2, 3})
```

```python
hash(s)
```

#### Caveats with Concatenation and Repetition


Consider this:

```python
x = [2000]
```

```python
id(x[0])
```

```python
l = x + x
```

```python
l
```

```python
id(l[0]), id(l[1])
```

As expected, the objects in `l[0]` and `l[1]` are the same.

Could also use:

```python
l[0] is l[1]
```

This is not a big deal if the objects being concatenated are immutable. But if they are mutable:

```python
x = [ [0, 0] ]
l = x + x
```

```python
l
```

```python
l[0] is l[1]
```

And then we have the following:

```python
l[0][0] = 100
```

```python
l[0]
```

```python
l
```

Notice how changing the 1st item of the 1st element also changed the 1st item of the second element.


While this seems fairly obvious when concatenating using the `+` operator as we have just done, the same actually happens with repetition and may not seem so obvious:

```python
x = [ [0, 0] ]
```

```python
m = x * 3
```

```python
m
```

```python
m[0][0] = 100
```

```python
m
```

And in fact, even `x` changed:

```python
x
```

If you really want these repeated objects to be different objects, you'll have to copy them somehow. A simple list comprehensions would work well here:

```python
x = [ [0, 0] ]
m = [e.copy() for e in x*3]
```

```python
m
```

```python
m[0][0] = 100
```

```python
m
```

```python
x
```
