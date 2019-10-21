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

### Selecting and Filtering Iterators


#### *filter*  and *filterfalse*


You should already be aware of the Python built-in function `filter`.

Remember that the `filter` function can work with any iterable, including of course iterators and generators.

Let's see a quick example:

```python
def gen_cubes(n):
    for i in range(n):
        print(f'yielding {i}')
        yield i**3
```

Now let's say we only want to use cubes that are odd.

We need a function that will return a True if the number is odd, False otherwise. (This is technically called a **predicate** by the way - any function that given an input returns True or False is called a **predicate**)

```python
def is_odd(x):
    return x % 2 == 1
```

Let's make sure the function works as expected:

```python
is_odd(4), is_odd(81)
```

Now we can use that function (or we could have just used a lambda as well) with the `filter` function.

Note that the `filter` function is also lazy.

```python
filtered = filter(is_odd, gen_cubes(10))
```

Notice that the `gen_cubes(10)` generator was not actually used (no print output).

We can however iterate through it:

```python
list(filtered)
```

As we can see `filtered` will drop any values where the predicate is False.


We could easily reverse this to return not-odd (i.e. even) values:

```python
def is_even(x):
    return x % 2 == 0
```

```python
list(filter(is_even, gen_cubes(10)))
```

But we had to create a new function - instead we could use the `filterfalse` function in the `itertools` module that does the same work as `filter` but retains values where the predicate is False (instead of True as the `filter` function does).

The `filterfalse` function also uses lazy evaluation.

```python
from itertools import filterfalse
```

```python
evens = filterfalse(is_odd, gen_cubes(10))
```

No print output --> lazy evaluation

```python
list(evens)
```


This way we can filter using the same predicate, depending on whether the result is `True` (using `filter`) or `False` (using `filterfalse`).


#### *dropwhile* and *takewhile*


The `takewhile` function in the `itertools` module will yield elements from an iterable, as long as a specific criteria (the predicate) is `True`.

As soon as the predicate is `False`, iteration is stopped - even if subsequent elements would have had a `True` predicate - this is not a filter, this basically iterate over an iterable as long as the predicate remains `True`.

As we might expect, this function also uses lazy evaluation.

```python
from math import sin, pi

def sine_wave(n):
    start = 0
    max_ = 2 * pi
    step = (max_ - start) / (n-1)
    for _ in range(n):
        yield round(sin(start), 2)
        start += step    
```

```python
list(sine_wave(15))
```

```python
from itertools import takewhile

list(takewhile(lambda x: 0 <= x <= 0.9, sine_wave(15)))
```

As you can see iteration stopped at `0.78`, even though we had values later that would have had a `True` predicate. This is different from the `filter` function:

```python
list(filter(lambda x: 0 <= x <= 0.9, sine_wave(15)))
```

The `dropwhile` function on the other hand starts the iteration once the predicate becomes `False`:

```python
from itertools import dropwhile
```

```python
l = [1, 3, 5, 2, 1]
```

```python
list(dropwhile(lambda x: x < 5, l))
```

As you can see the iterable skipped `1` and `3` and started the iteration once the predicate was `False`. Once the iteration begins, it no longer checks the predicate, and so we ended up with `5` and `2` and `1` in the iteration.


#### The *compress* function


The compress function is essentially a filter that takes two iterables as parameters.
The first argument is the iterable (data) that will be filtered, and the second iterable contains elements (selectors), possibly of different length than the iterable being filtered. As always in Python, any object has an associated truth value, and the selectors therefore each have a truth value as well.

The resulting iterator yields elements from the data iterable where the selector at the same "position" is truthy.

A simple analogous way to look at it would be as follows using the `zip` function:


```python
data = ['a', 'b', 'c', 'd', 'e']
selectors = [True, False, 1, 0]
```

```python
list(zip(data, selectors))
```

And only retain the elements where the second value in the tuple is truthy:

```python
[item for item, truth_value in zip(data, selectors) if truth_value]
```

The `compress` function works the same way, except that it is evaluated lazily and returns an iterator:

```python
from itertools import compress
```

```python
list(compress(data, selectors))
```
