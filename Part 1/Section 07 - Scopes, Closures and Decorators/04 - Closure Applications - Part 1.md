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

### Closure Applications (Part 1)


In this example we are going to build an averager function that can average multiple values.

The twist is that we want to simply be able to feed numbers to that function and get a running average over time, not average a list which requires performing the same calculations (sum and count) over and over again.

```python
class Averager:
    def __init__(self):
        self.numbers = []
    
    def add(self, number):
        self.numbers.append(number)
        total = sum(self.numbers)
        count = len(self.numbers)
        return total / count
```

```python
a = Averager()
```

```python
a.add(10)
```

```python
a.add(20)
```

```python
a.add(30)
```

We can do this using a closure as follows:

```python
def averager():
    numbers = []
    def add(number):
        numbers.append(number)
        total = sum(numbers)
        count = len(numbers)
        return total / count
    return add
```

```python
a = averager()
```

```python
a(10)
```

```python
a(20)
```

```python
a(30)
```

Now, instead of storing a list and reclaculating `total` and `count` every time wer need the new average, we are going to store the running total and count and update each value each time a new value is added to the running average, and then return `total / count`.

Let's start with a class approach first, where we will use instance variables to store the running total and count and provide an instance method to add a new number and return the current average.

```python
class Averager:
    def __init__(self):
        self._count = 0
        self._total = 0
    
    def add(self, value):
        self._total += value
        self._count += 1
        return self._total / self._count
```

```python
a = Averager()
```

```python
a.add(10)
```

```python
a.add(20)
```

```python
a.add(30)
```

Now, let's see how we might use a closure to achieve the same thing.

```python
def averager():
    total = 0
    count = 0
    
    def add(value):
        nonlocal total, count
        total += value
        count += 1
        return 0 if count == 0 else total / count
    
    return add
        
```

```python
a = averager()
```

```python
a(10)
```

```python
a(20)
```

```python
a(30)
```

#### Generalizing this example


We saw that we were essentially able to convert a class to an equivalent functionality using closures. This is actually true in a much more general sense - very often, classes that define a single method (other than initializers) can be implemented using a closure instead.

Let's look at another example of this.


Suppose we want something that can keep track of the running elapsed time in seconds.

```python
from time import perf_counter
```

```python
class Timer:
    def __init__(self):
        self._start = perf_counter()
    
    def __call__(self):
        return (perf_counter() - self._start)
```

```python
a = Timer()
```

Now wait a bit before running the next line of code:

```python
a()
```

Let's start another "timer":

```python
b = Timer()
```

```python
print(a())
print(b())
```

Now let's rewrite this using a closure instead:

```python
def timer():
    start = perf_counter()
    
    def elapsed():
        # we don't even need to make start nonlocal 
        # since we are only reading it
        return perf_counter() - start
    
    return elapsed
```

```python
x = timer()
```

```python
x()
```

```python
y = timer()
```

```python
print(x())
print(y())
```

```python
print(a())
print(b())
print(x())
print(y())
```

```python

```
