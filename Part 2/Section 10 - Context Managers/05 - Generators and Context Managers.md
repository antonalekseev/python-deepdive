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

### Generators and Context Managers


Let's see how we might write something that almost behaves like a context manager, using a generator function:

```python
def my_gen():
    try:
        print('creating context and yielding object')
        lst = [1, 2, 3, 4, 5]
        yield lst
    finally:
        print('exiting context and cleaning up')
```

```python
gen = my_gen()  # create generator
```

```python
lst = next(gen)  # enter context and get "as" object
```

```python
print(lst)
```

```python
next(gen)  # exit context
```

As you can see, the exiting context code ran.
But to make this cleaner, we'll catch the StopIteration exception:

```python
gen = my_gen()
lst = next(gen)
print(lst)
try:
    next(gen)
except StopIteration:
    pass
```

Now let's write a context manager that can use the type of generator we wrote so we can use it using a `with` statement instead:

```python
class GenCtxManager:
    def __init__(self, gen_func):
        self._gen = gen_func()
        
    def __enter__(self):
        return next(self._gen)
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        try:
            next(self._gen)
        except StopIteration:
            pass
        return False
```

```python
with GenCtxManager(my_gen) as lst:
    print(lst)
```

Our `GenCtxManager` class is not very flexible - we cannot pass arguments to the generator function for example. We are also not doing any exception handling...


We could try some of this ourselves. 
For example handling arguments:

```python
class GenCtxManager:
    def __init__(self, gen_func, *args, **kwargs):
        self._gen = gen_func(*args, **kwargs)
        
    def __enter__(self):
        return next(self._gen)
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        try:
            next(self._gen)
        except StopIteration:
            pass
        return False
```

```python
def open_file(fname, mode):
    try:
        print('opening file...')
        f = open(fname, mode)
        yield f
    finally:
        print('closing file...')
        f.close()
```

```python
with GenCtxManager(open_file, 'test.txt', 'w') as f:
    print('writing to file...')
    f.write('testing...')
```

```python
with open('test.txt') as f:
    print(next(f))
```

This works, but is not very elegant, and we still are not doing much exception handling. 
In the next video, we'll look at a decorator in the standard library that does this far more robustly and elegantly...
