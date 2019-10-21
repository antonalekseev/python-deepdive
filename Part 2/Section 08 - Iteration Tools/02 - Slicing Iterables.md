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

### Slicing Iterables


We know that sequence types can be sliced:

```python
l = [1, 2, 3, 4, 5]
```

```python
l[0:2]
```

Equivalently we can use the `slice` object:

```python
s = slice(0, 2)
```

```python
l[s]
```

But this does not work with iterables that are not also sequence types:

```python
import math

def factorials(n):
    for i in range(n):
        yield math.factorial(i)
```

```python
facts = factorials(100)
```

```python
facts[0:2]
```

But we could write a function to mimic this. Let's try a simplistic approach that will only work for a consecutive slice:

```python
def slice_(iterable, start, stop):
    for _ in range(0, start):
        next(iterable)
        
    for _ in range(start, stop):
        yield(next(iterable))
```

```python
list(slice_(factorials(100), 1, 5))
```

This is quite simple, however we don't support a `step` value.


The `itertools` module has a function, `islice` which implements this for us:

```python
list(factorials(10))
```

Now let's use the `islice` function to obtain the first 3 elements:

```python
from itertools import islice
```

```python
islice(factorials(10), 0, 3)
```

`islice` is itself a lazy iterator, so we can iterate through it:

```python
list(islice(factorials(10), 0, 3))
```

We can even use a step value:

```python
list(islice(factorials(10), 0, 10, 2))
```

It does not support negative indices, or step values, but it does support None for all the arguments. The default, as expected would then be the first element, the last element, and a step of 1:

```python
list(islice(factorials(10), None, None, 2))
```

This function can be very useful when dealing with infinite iterators for example.

```python
def factorials():
    index = 0
    while True:
        yield math.factorial(index)
        index += 1
```

Let's say we want to see the first 5 elements. We could do it the way we have up to now:

```python
facts = factorials()
for _ in range(5):
    print(next(facts))
```

Or we could use `islice` as follows:

```python
list(islice(factorials(), 5))
```

One thing to note is that `islice` is a lazy iterator, but when we use a `step` value, there is no magic, Python still has to call `next` on our iterable - it just doesn't always yield it back to us.

To see this, we'll add a print statement to our generator function:

```python
def factorials():
    index = 0
    while True:
        print(f'yielding factorial({index})...')
        yield math.factorial(index)
        index += 1
```

```python
list(islice(factorials(), 9))
```

```python
list(islice(factorials(), None, 10, 2))
```

As you can see, even though 5 elements were yielded from `islice`, it still had to call our generator 10 times!


The same thing happens if we skip elements in the slice, it still has to call next for the skipped elements:

```python
list(islice(factorials(), 5, 10))
```

The other thing to watch out for is that islice is an **iterator** - which means it becomes exhausted, **even if you pass an iterable such as a list to it**!

```python
l = [1, 2, 3, 4, 5]
```

```python
s = islice(l, 0, 3)
```

```python
list(s)
```

```python
list(s)
```

So watch out!


Furthermore, keep in mind that `islice` iterates over our iterable in order to yield the appropriate values. This means that if we use an iterator, that iterator will get consumed, and possibly exhausted:

```python
facts = factorials()
```

```python
next(facts), next(facts), next(facts), next(facts)
```

If we now start slicing `facts` with `islice`, remember that the first four values of `facts` have already been consumed!

```python
list(islice(facts, 0, 3))
```

And of course, `islice` further consumed our iterator:

```python
next(facts)
```

So, just something to keep in mind when we pass iterators to `islice`, and more generally to any of the functions in `itertools`.
