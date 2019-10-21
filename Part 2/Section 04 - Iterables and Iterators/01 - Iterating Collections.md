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

### Iterating Collections


We saw how sequence types support iteration by being able to access elements by index. We could even write our custom sequence types by implementing the `__getitem__` method.


But there are some limitations:

* items must be numerically indexable, with indexing starting at `0`
* cannot be used with unordered collections, such as sets


If we think about iterating over a collection, what we really need is a way to request the **next** item in the collection.

If we can do that, our collection does not require being indexable, nor does it need to be ordered (i.e. we don't need the notion of relative positions of elements in the container).


This is exactly what iterables are in general - they provide a method that returns the "next" element in the collection. This approach works equally well with sequence type collections, as well as unordered collection types such as sets.

Of course, the order in which **next** returns items from an unordered colllection is not known in advance - and we see that when we iterate over a set for example:

```python
s = {'x', 'y', 'b', 'c', 'a'}
for item in s:
    print(item)
```

As you can see the order in which the elements of the set was returned, did not match the order in which we added elements to the set.


Furthermore, we cannot use indexing to access elements in a set:

```python
s[0]
```

### Rolling our own Next method


Let's go ahead and define a kind of iterable ourselves. 

What we'll want to do is to have a container type of class that implements a `next` method, instead of that `__getitem__` method. 

Every time we call `next`, it should return the next element in the collection - so we'll have to keep track of where we are in the iteration somehow.

Since `next` is a built-in function, which we'll look at in a bit, we'll use `next_` instead.

```python
class Squares:
    def __init__(self):
        self.i = 0
    
    def next_(self):
        result = self.i ** 2
        self.i += 1
        return result
```

```python
sq = Squares()
```

```python
sq.next_()
```

```python
sq.next_()
```

```python
sq.next_()
```

How do we re-start the iteration from the beginning?

We can't - we have to create a new instance of `Squares`:

```python
sq = Squares()
```

```python
for i in range(10):
    print(sq.next_())
```

We even are able to iterate over the squares.

But you'll notice that we essentially have an **infinite** number of items.

We can fix that easily enough - by specifying a length when we create the collection, and raise an exception if `next_()` goes beyond the number of elements in the collection - we'll raise a `StopIteration` exception -- that's a built-in exception Python provides us specifically for this kind of scenario!!

We'll even implement a `__len__` method to support the `len()` function:

```python
class Squares:
    def __init__(self, length):
        self.length = length
        self.i = 0
    
    def next_(self):
        if self.i >= self.length:
            raise StopIteration
        else:
            result = self.i ** 2
            self.i += 1
            return result           
        
    def __len__(self):
        return self.length
```

```python
sq = Squares(3)
```

```python
len(sq)
```

```python
sq.next_()
```

```python
sq.next_()
```

```python
sq.next_()
```

```python
sq.next_()
```

So now, we can essentially loop over the collection in a very similar way to how we did it with sequences and the `__getitem__` method:

```python
sq = Squares(5)
while True:
    try:
        print(sq.next_())
    except StopIteration:
        # reached end of iteration
        # stop looping
        break       
```

There are two issues here.
The first is that the "iterable" `sq` has been exhausted - we can't just "re-start" the iteration:

```python
sq.next_()
```

The second problem is that we can't use a `for` loop - Python does not know about our `next_()` method:

```python
for i in Squares(10):
    print(i)
```

Of course if we had a `__getitem__` method, everything would work again - but remember that `__getitem__` means we have a sequence type. Although our Squares is actually a sequence, we want to look at a more general way of creating containers that are not necessarily sequences.


Much like Python's `len()` function and the `__len__()` method, Python has a built-in `next()` function - it calls the `__next__()` method in our class if there is one.

Let's see this:

```python
class Squares:
    def __init__(self, length):
        self.length = length
        self.i = 0
    
    def __next__(self):
        if self.i >= self.length:
            raise StopIteration
        else:
            result = self.i ** 2
            self.i += 1
            return result   
    
    def __len__(self):
        return self.length
```

```python
sq = Squares(3)
```

```python
next(sq)
```

```python
next(sq)
```

```python
next(sq)
```

```python
next(sq)
```

So that's nice, makes typing a bit easier - our loop we wrote earlier would look something like this now:

```python
sq = Squares(5)
while True:
    try:
        print(next(sq))
    except StopIteration:
        break  
```

Does this mean Python can now iterate over an instance of Squares?

```python
for i in Squares(10):
    print(i)
```

Nope, Python still does not recognize our class as an iterable collection.

We need to do a little bit more work to get there.

We also are going to need to look at how to "reset" the iteration without having to create a whole new object.


You'll notice that technically our `Squares` class could be built as a sequence type - it was just a very simple example.

Instead, let's build another collection that is a container of random numbers, but in no particular order.

```python
import random
```

```python
class RandomNumbers:
    def __init__(self, length, *, range_min=0, range_max=10):
        self.length = length
        self.range_min = range_min
        self.range_max = range_max
        self.num_requested = 0
        
    def __len__(self):
        return self.length
    
    def __next__(self):
        if self.num_requested >= self.length:
            raise StopIteration
        else:
            self.num_requested += 1
            return random.randint(self.range_min, self.range_max)
```

We can now iterate over instances of this object:

```python
numbers = RandomNumbers(10)
```

```python
len(numbers)
```

```python
while True:
    try:
        print(next(numbers))
    except StopIteration:
        break
```

We still cannot use a `for` loop, and if we want to 'restart' the iteration, we have to create a new object every time.

```python
numbers = RandomNumbers(10)
```

```python
for item in numbers:
    print(item)
```

```python

```
