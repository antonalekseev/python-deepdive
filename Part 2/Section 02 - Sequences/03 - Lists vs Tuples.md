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

### Lists vs Tuples


Remember that both lists and tuples are considered **sequence** types.

Remember also that we should consider tuples as data structures (position has meaning) as we saw in an earlier section on named tuples.

However, in this context we are going to view tuples as "immutable lists".


Generally, tuples are more efficient that lists, so, unless you need mutability of the container, prefer using a tuple over a list.


#### Creating Tuples


We saw some of this already in the first section of this course when we looked at some of the optimizations Python implements, but let's revisit it in this context.


Here is Wikipedia's definition of constant folding:

`
Constant folding is the process of recognizing and evaluating constant expressions at compile time rather than computing them at runtime.
`


To see how this works, we are going to use the `dis` module which allows to see the disassembled Python bytecode - not for the faint of heart, but can be really useful!

```python
from dis import dis
```

We want to understand what Python does when it compiles statements such as:

```python
(1, 2, 3)
[1, 2, 3]
```

```python
dis(compile('(1,2,3, "a")', 'string', 'eval'))
```

```python
dis(compile('[1,2,3, "a"]', 'string', 'eval'))
```

Notice how for a tuple containing constants (such as ints and strings in this case), the values are loaded in one step, a single constant value essentially. 

Lists, on the other hand are built-up one element at a time.


So, that's one reason why tuples can "load" faster than a list.


In fact, we can easily time this:

```python
from timeit import timeit
```

```python
timeit("(1,2,3,4,5,6,7,8,9)", number=10_000_000)
```

```python
timeit("[1,2,3,4,5,6,7,8,9]", number=10_000_000)
```

As you can see creating a tuple was faster.


Now this changes if the tuple elements are not constants, such as lists or functions for example

```python
def fn1():
    pass
```

```python
dis(compile('(fn1, 10, 20)', 'string', 'eval'))
```

```python
dis(compile('[fn1, 10, 20]', 'string', 'eval'))
```

or

```python
dis(compile('([1,2], 10, 20)', 'string', 'eval'))
```

```python
dis(compile('[[1,2], 10, 20]', 'string', 'eval'))
```

And of course this is reflected in the timings too:

```python
timeit("([1, 2], 10, 20)", number=1_000_000)
```

```python
timeit("[[1, 2], 10, 20]", number=1_000_000)
```

#### Copying Lists and Tuples


Let's look at creating a copy of both a list and a tuple:

```python
l1 = [1, 2, 3, 4, 5, 6, 7, 8, 9]
t1 = (1, 2, 3, 4, 5, 6, 7, 8, 9)
```

```python
id(l1), id(t1)
```

```python
l2 = list(l1)
t2 = tuple(t1)
```

Let's time this:

```python
timeit('tuple((1,2,3,4,5,6,7,8,9))', number=1_000_000)
```

```python
timeit('list([1,2,3,4,5,6,7,8,9])', number=1_000_000)
```

That's another win for tuples. But why?


Let's look at the id's of the copies:

```python
id(l1), id(l2), id(t1), id(t2)
```

in other words:

```python
l1 is l2, t1 is t2
```

Notice how the `l1` and `l2` are **not** the same objects, whereas as `t1` and `t2` are!

So for lists, the elements had to be copied (shallow copy, more on this later), but for tuples it did not.


Note that this is the case even if the tuple contains non constant elements:

```python
t1 = ([1,2], fn1, 3)
t2 = tuple(t1)
t1 is t2
```

#### Storage Efficiency


When mutable container objects such as lists, sets, dictionaries, etc are  created, and during their lifetime, the allocated capacity of these containers (the number of items they can contain) is greater than the number of elements in the container. This is done to make adding elements to the collection more efficient, and is called over-allocating.


Immutable containers on the other hand, since their item count is fixed once they have been created, do not need this overallocation - so their storage efficiency is greater.


Let's look at the size (memory) of lists and tuples as they get larger:

```python
import sys
```

```python
prev = 0
for i in range(10):
    c = tuple(range(i+1))
    size_c = sys.getsizeof(c)
    delta, prev = size_c - prev, size_c
    print(f'{i+1} items: {size_c}, delta={delta}')
```

```python
prev = 0
for i in range(10):
    c = list(range(i+1))
    size_c = sys.getsizeof(c)
    delta, prev = size_c - prev, size_c
    print(f'{i+1} items: {size_c}, delta={delta}')
```

As you can see the size delta for tuples as they get larger, remains a constant 8 bytes (the pointer to the element), but not so for lists which will over-allocate space (this is done to achieve better performance when appending elements to a list).

Let's see what happens to the same list when we keep appending elements to it:

```python
c = []
prev = sys.getsizeof(c)
print(f'0 items: {sys.getsizeof(c)}')
for i in range(255):
    c.append(i)
    size_c = sys.getsizeof(c)
    delta, prev = size_c - prev, size_c
    print(f'{i+1} items: {size_c}, delta={delta}')
```

As you can see the size of the list doesn't grow every time we append an element - it only does so occasionally. Resizing a list is expensive, so not resizing every time an item is added helps out, so this method called *overallocation* is used that creates a larger container than required is used - on the other hand you don't want to overallocate too much as this has a memory cost.


If you're interested in learning more about why over-allocating is done and how it works (amortization), Wikipedia also has an excellent article on it: https://en.wikipedia.org/wiki/Dynamic_array

The book "Introduction to Algorithms", by "Cormen, Leiserson, Rivest and Stein" has a thorough discussion on it (under dynamic tables).


#### Retrieving Elements


Let's time retrieving an element from a tuple and a list:

```python
t = tuple(range(100_000))
l = list(t)
```

```python
timeit('t[99_999]', globals=globals(), number=10_000_000)
```

```python
timeit('l[99_999]', globals=globals(), number=10_000_000)
```

As you can see, retrieving elements from a tuple is very slightly faster than from a list. But consideting how small the difference really is, I'm not sure I would worry about it too much.

There is a reason why this should be true, and it has to do with how tuples and lists are implemented in CPython. Tuples have direct access (pointers) to their elements, while lists need to first access another array that contains the pointers to the elements of the list.
