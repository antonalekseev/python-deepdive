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

### The `iter()` Function


As we have seen before, the `iter()` function is used to request an iterator object from an iterable.


For example:

```python
l = [1, 2, 3, 4]
```

```python
l_iter = iter(l)
```

```python
type(l_iter)
```

And we can use that iterator to iterate the collection by calling `next()` until a `StopIteration` exception is raised.

```python
next(l_iter)
```

```python
next(l_iter)
```

We also saw how sequence types are also iterable even though they are not actual iterables - they do not have an `__iter__` method, but instead they have a `__getitem__` method.


Python had no problem iterating a sequence object - in fact behind the scenes an iterator is built by Python in order to iterate using the `__getitem__` method:

```python
class Squares:
    def __init__(self, n):
        self._n = n
    
    def __len__(self):
        return self._n
    
    def __getitem__(self, i):
        if i >= self._n:
            raise IndexError
        else:
            return i ** 2
```

```python
sq = Squares(5)
```

```python
for i in sq:
    print(i)
```

But, we can also do this:

```python
sq_iter = iter(sq)
```

And we now have an iterator for `sq`!

```python
type(sq_iter)
```

```python
'__next__' in dir(sq_iter)
```

What happens is that Python will first try to get the iterator by invoking the `__iter__` method on our object.

If it does not have that method, it will look for `__getitem__` next - if it's there it will create an iterator for us that will leverage `__getitem__` and the fact that sequence indices should start at 0.

If neither `__iter__` nor `__getitem__` are found, then we'll get an exception such as this one:

```python
for i in 10:
    print(i)
```

Here's how we might build an iterator using the `__getitem__` method ourselves - not that we have to do that since Python does it for us.

```python
class Squares:
    def __init__(self, n):
        self._n = n
    
    def __len__(self):
        return self._n
    
    def __getitem__(self, i):
        if i >= self._n:
            raise IndexError
        else:
            return i ** 2
```

```python
class SquaresIterator:
    def __init__(self, squares):
        self._squares = squares
        self._i = 0
        
    def __iter__(self):
        return self
    
    def __next__(self):
        if self._i >= len(self._squares):
            raise StopIteration
        else:
            result = self._squares[self._i]
            self._i += 1
            return result
```

```python
sq = Squares(5)
sq_iterator = SquaresIterator(sq)
```

```python
type(sq_iterator)
```

```python
print(next(sq_iterator))
print(next(sq_iterator))
print(next(sq_iterator))
print(next(sq_iterator))
print(next(sq_iterator))
```

The iterator is now exhausted, so:

```python
print(next(sq_iterator))
```

Technically, we don't actually need to implement the `__len__` method in our sequence type, but since we are using it in our iterator, we'll have to think of something else - we can leverage the fact that the sequence will raise an IndexError if the index is out of bounds:

```python
class SquaresIterator:
    def __init__(self, squares):
        self._squares = squares
        self._i = 0
        
    def __iter__(self):
        return self
    
    def __next__(self):
        try:
            result = self._squares[self._i]
            self._i += 1
            return result
        except IndexError:
            raise StopIteration()
```

And things will work as before:

```python
sq_iterator = SquaresIterator(sq)
```

```python
for i in sq_iterator:
    print(i)
```

#### How to test if an object is iterable


Basically an object is iterable if it:
* implements the **iterable** protocol (`__iter__` that returns an iterator)
* implements the **sequence** protocol (`__getitem__`, and `__len__`) - although `__len__` is not required for iteration



Given some object, how can we test to see if it is iterable or not?


The problem is that we would need to test for both `__iter__` (making sure it returns an iterator), and `__getitem__`. Far easier to do a try/except.


For example, just testing that `__iter__` is defined is not sufficient:

```python
class SimpleIter:
    def __init__(self):
        pass
    
    def __iter__(self):
        return 'Nope'
```

```python
s = SimpleIter()
```

```python
'__iter__' in dir(s)
```

However, if we call `iter()` on `SimpleIter`, look at what happens:

```python
iter(s)
```

So the best way, if you have some need to detect if something is iterable or not, is the following:

```python
def is_iterable(obj):
    try:
        iter(obj)
        return True
    except TypeError:
        return False
```

```python
is_iterable(SimpleIter())
```

```python
is_iterable(Squares(5))
```

That said, we'll cover exception handling in Python later in this course, but there is rarely a need to test if something is iterable, only to then go ahead and iterate over it right after that if it is.

Consider the following two alternatives:

```python
obj = 100
if is_iterable(obj):
    for i in obj:
        print(i)
else:
    print('Error: obj is not iterable')
```

vs

```python
obj = 100
for i in obj:
    print(i)
```

As you can see, the error Python itself raises tells us the same thing, and provides even more information!!

Instead of guarding for potential errors as we did in the first example, try doing the action you really want to do, and let Python raise the exception for you.

If you want to handle the exception, wrap your action inside a try/except:


So instead of writing it this way (*ask before you leap*):

```python
obj = 100
if is_iterable(obj):
    for i in obj:
        print(i)
else:
    print('Error: obj is not iterable')
    print('Taking some action as a consequence of this error')
```

prefer writing it this way (*ask for forgiveness later*):

```python
obj = 100
try:
    for i in obj:
        print(i)
except TypeError:
    print('Error: obj is not iterable')
    print('Taking some action as a consequence of this error')
```

This approach to exception handling we'll cover in a lot more detail later, but boils down to the simple idea:

*"It's easier to ask forgiveness than it is to get permission"*

(commonly attributed to Grace Hopper)
