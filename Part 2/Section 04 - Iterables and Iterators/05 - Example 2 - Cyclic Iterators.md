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

### Cyclic Iterators


Iterables do not have to be finite. In fact we can easily create an infinite cyclical iterator.

<!-- #region -->
Here's an example - suppose we have a loop that iterates over some range of integers. As we loop through those integers we want to create a tuple containing the integer and a string that cycles over a finite set (smaller than the list of integers).

```
1, 2, 3, 4, 5, 6, 7, 8, 9, ...

N, S, W, E
```

and we want to generate

```
1N, 2S, 3W, 4E, 5N, 6S, 7W, 8E, 9N, ...
```

<!-- #endregion -->

We could do it this way by creating a custom iterator for the list `['N', 'S', 'W', 'E']` that will cycle over that list indefinitely:

```python
class CyclicIterator:
    def __init__(self, lst):
        self.lst = lst
        self.i = 0
        
    def __iter__(self):
        return self
    
    def __next__(self):
        result = self.lst[self.i % len(self.lst)]
        self.i += 1
        return result
```

```python
iter_cycl = CyclicIterator('NSWE')
```

```python
for i in range(10):
    print(next(iter_cycl))
```

So, now we can tackle our original problem:

```python
n = 10
iter_cycl = CyclicIterator('NSWE')
for i in range(1, n+1):
    direction = next(iter_cycl)
    print(f'{i}{direction}')
```

And re-working this into a list comprehension:

```python
n = 10
iter_cycl = CyclicIterator('NSWE')
[f'{i}{next(iter_cycl)}' for i in range(1, n+1)]
```

Of course, there's an easy alternative way to do this as well, using:
* repetition
* zip
* a list comprehension


We need to repeat the array ['N', 'S', 'W', 'E'] for as many times as we have elements in our range of integers - we can even create way more than we need - because when we `zip` it up with the range of integers, the smallest length iterable will be used:

```python
n = 10
list(zip(range(1, n+1), 'NSWE' * (n//4 + 1)))
```

```python
[f'{i}{direction}'
 for i, direction in zip(range(1, n+1), 'NSWE' * (n//4 + 1))]
```

There's actually an even easier way yet, and that's to use our `CyclicIterator`, but instead of building it ourselves, we can simply use the one provided by Python in the standard library!!

```python
import itertools
```

```python
n = 10
iter_cycl = CyclicIterator('NSWE')
[f'{i}{next(iter_cycl)}' for i in range(1, n+1)]
```

and using itertools:

```python
n = 10
iter_cycl = itertools.cycle('NSWE')
[f'{i}{next(iter_cycl)}' for i in range(1, n+1)]
```
