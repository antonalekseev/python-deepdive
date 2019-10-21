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

### Example: Fibonacci Sequence

<!-- #region -->
Here is the Fibonacci sequence:

```
1 1 2 3 5 8 13 ...
```
<!-- #endregion -->

<!-- #region -->
As you can see there is a recursive definition of the numbers in this sequence:

```
Fib(n) = Fib(n-1) + Fib(n-2)
```
where 

```
Fib(0) = 1
``` 

and

```
Fib(1) = 1
```
<!-- #endregion -->

Although we can certainly use a recursive approach to calculate the *n-th* number in the sequence, it is not a very effective method - we can of course help it by using memoization, but we'll still run into Python's maximum recursion depth. In Python there is a maximum number of times a recursive function can call itself (creating a stack frame at every nested call) before Python gives us an exception that we have exceeded the maximum permitted depth (the number of recursive calls). We can actually change that number if we want to, but if we're running into that limitation, it might be better creating a non-recursive algorithm - recursion can be elegant, but not particularly efficient.

```python
def fib_recursive(n):
    if n <= 1:
        return 1
    else:
        return fib_recursive(n-1) + fib_recursive(n-2)
```

```python
[fib_recursive(i) for i in range(7)]
```

But this quickly becomes an issue as `n` grows larger:

```python
from timeit import timeit
```

```python
timeit('fib_recursive(10)', globals=globals(), number=10)
```

```python
timeit('fib_recursive(28)', globals=globals(), number=10)
```

```python
timeit('fib_recursive(29)', globals=globals(), number=10)
```

We can alleviate this by using memoization:

```python
from functools import lru_cache
```

```python
@lru_cache()
def fib_recursive(n):
    if n <= 1:
        return 1
    else:
        return fib_recursive(n-1) + fib_recursive(n-2)
```

```python
timeit('fib_recursive(10)', globals=globals(), number=10)
```

```python
timeit('fib_recursive(29)', globals=globals(), number=10)
```

As you can see, performance is greatly improved, but we still have a recursion depth limit:

```python
@lru_cache()
def fib_recursive(n):
    if n <= 1:
        return 1
    else:
        return fib_recursive(n-1) + fib_recursive(n-2)
```

```python
fib_recursive(2000)
```

So we can use a non-recursive approach to calculate the `n-th` Fibonacci number:

```python
def fib(n):
    fib_0 = 1
    fib_1 = 1
    for i in range(n-1):
        fib_0, fib_1 = fib_1, fib_0 + fib_1
    return fib_1
```

```python
[fib(i) for i in range(7)]
```

This works well for large `n` values too:

```python
timeit('fib(5000)', globals=globals(), number=10)
```

So now, let's create an iterator approach so we can iterate over the sequence, but without materializing it (i.e. we want to use lazy evaluation, not eager evaluation)


Our first approach is going to be a custom iterator and iterable:

```python
class Fib:
    def __init__(self, n):
        self.n = n
        
    def __iter__(self):
        return self.FibIter(self.n)
        
    class FibIter:
        def __init__(self, n):
            self.n = n
            self.i = 0
            
        def __iter__(self):
            return self
        
        def __next__(self):
            if self.i >= self.n:
                raise StopIteration
            else:
                result = fib(self.i)
                self.i += 1
                return result
```

And we can now iterate the usual way:

```python
fib_iterable = Fib(7)
```

```python
for num in fib_iterable:
    print(num)
```

Of course, we can also use the second form of the `iter` function too, but we have to create a closure first:

```python
def fib_closure():
    i = 0
    def inner():
        nonlocal i
        result = fib(i)
        i += 1
        return result
    return inner
```

```python
fib_numbers = fib_closure()
fib_iter = iter(fib_numbers, fib(7))
for num in fib_iter:
    print(num)
```

But there's two things here:

1. The syntax for either implementation is a little convoluted and not very clear
2. More importantly, notice what happens every time the `next` method is called - it has to calculate every Fibonacci number from scratch (using the `fib` function) - that is wasteful...


Instead, we can use a generator function very effectively here.

Here is our original `fib` function:

```python
def fib(n):
    fib_0 = 1
    fib_1 = 1
    for i in range(n-1):
        fib_0, fib_1 = fib_1, fib_0 + fib_1
    return fib_1    
```

```python
[fib(i) for i in range(7)]
```

Now let's modity it into a generator function:

```python
def fib_gen(n):
    fib_0 = 1
    fib_1 = 1
    for i in range(n-1):
        fib_0, fib_1 = fib_1, fib_0 + fib_1
        yield fib_1    
```

```python
[num for num in fib_gen(7)]
```

We're almost there. We're missing the first two Fibonacci numbers in the sequence - we need to yield those too.

```python
def fib_gen(n):
    fib_0 = 1
    yield fib_0
    fib_1 = 1
    yield fib_1
    for i in range(n-1):
        fib_0, fib_1 = fib_1, fib_0 + fib_1
        yield fib_1    
```

```python
[num for num in fib_gen(7)]
```

And finally we're returning one number too many if `n` is meant to indicate the length of the sequence:

```python
def fib_gen(n):
    fib_0 = 1
    yield fib_0
    fib_1 = 1
    yield fib_1
    for i in range(n-2):
        fib_0, fib_1 = fib_1, fib_0 + fib_1
        yield fib_1    
```

And now everything works fine:

```python
[num for num in fib_gen(7)]
```

Let's time it as well to compare it with the other methods:

```python
timeit('[num for num in Fib(5_000)]', globals=globals(), number=1)
```

```python
fib_numbers = fib_closure()
sentinel = fib(5_001)

timeit('[num for num in iter(fib_numbers, sentinel)]', globals=globals(),
      number=1)
```

```python
timeit('[num for num in fib_gen(5_000)]', globals=globals(), number=1)
```
