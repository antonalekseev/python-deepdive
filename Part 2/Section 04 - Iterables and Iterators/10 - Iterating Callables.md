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

### Iterating Callables


We can easily create iterators that are based on callables in general.

Let's look at an example:


##### Example 1


In this example we are going to create a counter function (using a closure) - it's a pretty simplistic function - `counter()` will return a closure that we can then call to increment an internal counter by `1` every time it is called:

```python
def counter():
    i = 0
    
    def inc():
        nonlocal i
        i += 1
        return i
    return inc
```

This function allows us to create a simple counter, which we can use as follows:

```python
cnt = counter()
```

```python
cnt()
```

```python
cnt()
```

Technically we can make an iterator to iterate over this counter:

```python
class CounterIterator:
    def __init__(self, counter_callable):
        self.counter_callable = counter_callable
        
    def __iter__(self):
        return self
    
    def __next__(self):
        return self.counter_callable()
```

Do note that this is an **infinite** iterable!

```python
cnt = counter()
cnt_iter = CounterIterator(cnt)
for _ in range(5):
    print(next(cnt_iter))
```

So basically we were able to create an **iterator** from some arbitrary callable.

But one issue is that we have an **inifinite** iterable.

One way around this issue, would be to specify a "stop" value when the iterator should decide to end the iteration.

Let's see how we would do this:

```python
class CounterIterator:
    def __init__(self, counter_callable, sentinel):
        self.counter_callable = counter_callable
        self.sentinel = sentinel
        
    def __iter__(self):
        return self
    
    def __next__(self):
        result = self.counter_callable()
        if result == self.sentinel:
            raise StopIteration
        else:
            return result
```

Now we can essentially provide a value that if returned from the callable will result in a `StopIteration` exception, essentially terminating the iteration:

```python
cnt = counter()
cnt_iter = CounterIterator(cnt, 5)
for c in cnt_iter:
    print(c)
```

Now there is technically an issue here: the cnt_iter is still "alive" - our iterator raised a `StopIteration` exception, but if we call it again, it will happily resume from where it left off!

```python
next(cnt_iter)
```

We really should make sure the iterator has been consumed, so let's fix that:

```python
class CounterIterator:
    def __init__(self, counter_callable, sentinel):
        self.counter_callable = counter_callable
        self.sentinel = sentinel
        self.is_consumed = False
        
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.is_consumed:
            raise StopIteration
        else:
            result = self.counter_callable()
            if result == self.sentinel:
                self.is_consumed = True
                raise StopIteration
            else:
                return result
```

Now it should behave as a normal iterator that cannot continue iterating once the first `StopIteration` exception has been raised:

```python
cnt = counter()
cnt_iter = CounterIterator(cnt, 5)
for c in cnt_iter:
    print(c)
```

```python
next(cnt_iter)
```

As we just saw, we can essentially make an iterator based on any callable, and our `CounterIterator` was actually quite generic, it only needed a callable and a sentinel value to work.

In fact, that's exactly what the second form of the `iter()` function allows us to do!


Let's see the help on `iter`:

```python
help(iter)
```

As we can see `iter` has a second form, that takes in a callable and a sentinel value.

And it will result in exactly what we have been doing, but without having to create the iterator class ourselves!

```python
cnt = counter()
cnt_iter = iter(cnt, 5)
for c in cnt_iter:
    print(c)
```

```python
next(cnt_iter)
```

##### Example 2


Both of these approaches can be made to work with any callable.

For example, you may want to iterate through random numbers until a specific random number is generated:

```python
import random
```

```python
random.seed(0)
for i in range(10):
    print(i, random.randint(0, 10))
```

As you can see in this example (I set my seed to 0 to have repeatable results), the number `8` is reached at the `5`th iteration.

(I am just doing this to find an easy sentinel value so we can easily verify that our code is working properly)

```python
random_iterator = iter(lambda : random.randint(0, 10), 8)
```

```python
random.seed(0)

for num in random_iterator:
    print(num)
```

Neat!


##### Example 3


Let's try a countdown example like the one we discussed in the lecture.

We'll use a closure to get our countdown working:

```python
def countdown(start=10):
    def run():
        nonlocal start
        start -= 1
        return start
    return run
```

```python
takeoff = countdown(10)
for _ in range(15):
    print(takeoff())
```

So the countdown function works, but we would like to be able to iterate over it and stop the iteration once we reach 0.

```python
takeoff  = countdown(10)
takeoff_iter = iter(takeoff, -1)
```

```python
for val in takeoff_iter:
    print(val)
```
