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

### Decorators Application (Memoization)


Let's go back to our Fibonacci example:

```python
def fib(n):
    print ('Calculating fib({0})'.format(n))
    return 1 if n < 3 else fib(n-1) + fib(n-2)
```

When we run this, we see that it is quite inefficient, as the same Fibonacci numbers get calculated multiple times:

```python
fib(6)
```

It would be better if we could somehow "store" these results, so if we have calculated `fib(4)` and `fib(3)` before, we could simply recall the these values when calculating `fib(5) = fib(4) + fib(3)` instead of recalculating them.

This concept of improving the efficiency of our code by caching pre-calculated values so they do not need to be re-calcualted every time, is called "memoization"


We can approach this using a simple class and a dictionary that stores any Fibonacci number that's already been calculated:

```python
class Fib:
    def __init__(self):
        self.cache = {1: 1, 2: 1}
    
    def fib(self, n):
        if n not in self.cache:
            print('Calculating fib({0})'.format(n))
            self.cache[n] = self.fib(n-1) + self.fib(n-2)
        return self.cache[n]
```

```python
f = Fib()
```

```python
f.fib(1)
```

```python
f.fib(6)
```

```python
f.fib(7)
```

Let's see how we could rewrite this using a closure:

```python
def fib():
    cache = {1: 1, 2: 2}
    
    def calc_fib(n):
        if n not in cache:
            print('Calculating fib({0})'.format(n))
            cache[n] = calc_fib(n-1) + calc_fib(n-2)
        return cache[n]
    
    return calc_fib
```

```python
f = fib()
```

```python
f(10)
```

Now let's see how we would implement this using a decorator:

```python
from functools import wraps

def memoize_fib(fn):
    cache = dict()
    
    @wraps(fn)
    def inner(n):
        if n not in cache:
            cache[n] = fn(n)
        return cache[n]
    
    return inner
```

```python
@memoize_fib
def fib(n):
    print ('Calculating fib({0})'.format(n))
    return 1 if n < 3 else fib(n-1) + fib(n-2)
```

```python
fib(3)
```

```python
fib(10)
```

```python
fib(6)
```

As you can see, we are hitting the cache when the values are available.

Now, we made our memoization decorator "hardcoded" to single argument functions - we could make it more generic.

For example, to handle an arbitrary number of positional arguments and keyword-only arguments we could do the following:

```python
def memoize(fn):
    cache = dict()
    
    @wraps(fn)
    def inner(*args):
        if args not in cache:
            cache[args] = fn(*args)
        return cache[args]
    
    return inner
```

```python
@memoize
def fib(n):
    print ('Calculating fib({0})'.format(n))
    return 1 if n < 3 else fib(n-1) + fib(n-2)
```

```python
fib(6)
```

```python
fib(7)
```

Of course, with this rather generic memoization decorator we can memoize other functions too:

```python
def fact(n):
    print('Calculating {0}!'.format(n))
    return 1 if n < 2 else n * fact(n-1)
```

```python
fact(5)
```

```python
fact(5)
```

And memoizing it:

```python
@memoize
def fact(n):
    print('Calculating {0}!'.format(n))
    return 1 if n < 2 else n * fact(n-1)
```

```python
fact(6)
```

```python
fact(6)
```

Our simple memoizer has a drawback however:
* the cache size is unbounded - probably not a good thing! In general we want to limit the cache to a certain number of entries, balancing computational efficiency vs memory utilization.
* we are not handling **kwargs


Memoization is such a common thing to do that Python actually has a memoization decorator built for us!

It's in the, you guessed it, **functools** module, and is called **lru_cache** and is going to be quite a bit more efficient compared to the rudimentary memoization example we did above.

[LRU Cache = Least Recently Used caching: since the cache is not unlimited, at some point cached entries need to be discarded, and the least recently used entries are discarded first]

```python
from functools import lru_cache
```

```python
@lru_cache()
def fact(n):
    print("Calculating fact({0})".format(n))
    return 1 if n < 2 else n * fact(n-1)
```

```python
fact(5)
```

```python
fact(4)
```

As you can see, `fact(4)` was returned via a cached entry!


Same thing with our Fibonacci function:

```python
@lru_cache()
def fib(n):
    print("Calculating fib({0})".format(n))
    return 1 if n < 3 else fib(n-1) + fib(n-2)
```

```python
fib(6)
```

```python
fib(5)
```

Recall from a few videos back that we timed the calculation for Fibonacci numbers. Calculating fib(35) took several seconds - every time...

```python
from time import perf_counter
```

```python
def fib_no_memo(n):
    return 1 if n < 3 else fib_no_memo(n-1) + fib_no_memo(n-2)
```

```python
start = perf_counter()
result = fib_no_memo(35)
print("result={0}, elapsed: {1}s".format(result, perf_counter() - start))
```

```python
@lru_cache()
def fib_memo(n):
    return 1 if n < 3 else fib_memo(n-1) + fib_memo(n-2)
```

```python
start = perf_counter()
result = fib_memo(35)
print("result={0}, elapsed: {1}s".format(result, perf_counter() - start))
```

And if we make the calls again:

```python
start = perf_counter()
result = fib_no_memo(35)
print("result={0}, elapsed: {1}s".format(result, perf_counter() - start))
```

```python
start = perf_counter()
result = fib_memo(35)
print("result={0}, elapsed: {1}s".format(result, perf_counter() - start))
```

You may have noticed that the `lru_cache` decorator was implemented using `()` - we'll see more on this later, but that's because decorators can themselves have parameters (beyond the function being decorated).


One of the arguments to the `lru_cache` decorator is the size of the cache - it defaults to 128 items, but we can easily change this - for performance reasons use powers of 2 for the cache size (or None for unbounded cache):

```python
@lru_cache(maxsize=8)
def fib(n):
    print("Calculating fib({0})".format(n))
    return 1 if n < 3 else fib(n-1) + fib(n-2)
```

```python
fib(10)
```

```python
fib(20)
```

```python
fib(10)
```

You'll not how Python had to recalculate `fib` for `10, 9,` etc. This is because the cache can only contain 10 items, so when we calculated `fib(20)`, it stored fib for `20, 19, ..., 11` (10 items) and therefore the oldest items fib `10, 9, ..., 1` were removed from the cache to make space.

```python

```
