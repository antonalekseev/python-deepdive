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

### Chaining and Teeing Iterators


Often we need to chain iterators/iterables together to behave like a single iterable.

We can think of this as analogous to sequence concatenation.

For example, suppose we have some generators producing squares:

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))
```

And we want to essentially iterate through all the values as if they were a single iterator.

We could do it this way:

```python
for gen in (l1, l2, l3):
    for item in gen:
        print(item)
```

In fact, we could even create our own generator function to do this:

```python
def chain_iterables(*iterables):
    for iterable in iterables:
        yield from iterable
```

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

for item in chain_iterables(l1, l2, l3):
    print(item)
```

But, a much simpler way is to use `chain` in the `itertools` module:

```python
from itertools import chain
```

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

for item in chain(l1, l2, l3):
    print(item)
```

Note that `chain` expects a variable number of positional arguments, each of which should be an iterable.

It will not work if we pass it a single iterable:

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(lists):
    print(item)
```

As you can see, it simply took our list and handed it back directly - there was nothing else to chain with:

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(lists):
    for i in item:
        print(i)
```

Instead, we could use unpacking:

```python
l1 = (i**2 for i in range(4))
l2 = (i**2 for i in range(4, 8))
l3 = (i**2 for i in range(8, 12))

lists = [l1, l2, l3]
for item in chain(*lists):
    print(item)
```

Unpacking works with iterables in general, so even the following would work just fine:

```python
def squares():
    yield (i**2 for i in range(4))
    yield (i**2 for i in range(4, 8))
    yield (i**2 for i in range(8, 12))
```

```python
for item in chain(*squares()):
    print(item)
```

But, unpacking is not lazy!! Here's a simple example that shows this, and why we have to be careful using unpacking if we want to preserve lazy evaluation:

```python
def squares():
    print('yielding 1st item')
    yield (i**2 for i in range(4))
    print('yielding 2nd item')
    yield (i**2 for i in range(4, 8))
    print('yielding 3rd item')
    yield (i**2 for i in range(8, 12))
```

```python
def read_values(*args):
    print('finised reading args')
```

```python
read_values(*squares())
```

Instead we can use an alternate "constructor" for chain, that takes a single iterable (of iterables) and lazily iterates through the outer iterable as well:

```python
c = chain.from_iterable(squares())
```

```python
for item in c:
    print(item)
```

Note also, that we can easily reproduce the same behavior of either chain quite easily:

```python
def chain_(*args):
    for item in args:
        yield from item
```

```python
def chain_iter(iterable):
    for item in iterable:
        yield from item
```

And we can use those just as we saw before with `chain` and `chain.from_iterable`:

```python
c = chain_(*squares())
```

```python
c = chain_iter(squares())
for item in c:
    print(item)
```

### "Copying" an Iterator


Sometimes we may have an iterator that we want to use multiple times for some reason.

As we saw, iterators get exhausted, so simply making multiple references to the same iterator will not work - they will just point to the same iterator object.

What we would really like is a way to "copy" an iterator and use these copies independently of each other.


We can use `tee` in `itertools`:

```python
from itertools import tee
```

```python
def squares(n):
    for i in range(n):
        yield i**2
```

```python
gen = squares(10)
gen
```

```python
iters = tee(squares(10), 3)
```

```python
iters
```

```python
type(iters)
```

As you can see `iters` is a **tuple** contains 3 iterators - let's put them into some variables and see what each one is:

```python
iter1, iter2, iter3 = iters
```

```python
next(iter1), next(iter1), next(iter1)
```

```python
next(iter2), next(iter2)
```

```python
next(iter3)
```

As you can see, `iter1`, `iter2`, and `iter3` are essentially three independent "copies" of our original iterator (`squares(10)`)


Note that this works for any iterable, so even sequence types such as lists:

```python
l = [1, 2, 3, 4]
```

```python
lists = tee(l, 2)
```

```python
lists[0]
```

```python
lists[1]
```

But you'll notice that the elements of `lists` are not lists themselves!

```python
list(lists[0])
```

```python
list(lists[0])
```

As you can see, the elements returned by `tee` are actually `iterators` - even if we used an iterable such as a list to start off with!

```python
lists[1] is lists[1].__iter__()
```

```python
'__next__' in dir(lists[1])
```

Yep, the elements of `lists` are indeed iterators!
