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

### Sorting Iterables


There's nothing really new here - we have seen the `sorted()` function before when we looked at sorting sequences.

The `sorted()` function will in fact work with any iterable, not just sequences.

Let's try this by creating a custom iterable and then sorting it.


For this example, we'll create an iterable of random numbers, and then sort it.

```python
import random
```

```python
random.seed(0)
```

```python
for i in range(10):
    print(random.randint(1, 10))
```

```python
import random

class RandomInts:
    def __init__(self, length, *, seed=0, lower=0, upper=10):
        self.length = length
        self.seed = seed
        self.lower = lower
        self.upper = upper
        
    def __len__(self):
        return self.length
    
    def __iter__(self):
        return self.RandomIterator(self.length, 
                                   seed = self.seed, 
                                   lower = self.lower,
                                   upper=self.upper)
    
    
    class RandomIterator:
        def __init__(self, length, *, seed, lower, upper):
            self.length = length
            self.lower = lower
            self.upper = upper
            self.num_requests = 0
            random.seed(seed)
            
        def __iter__(self):
            return self
        
        def __next__(self):
            if self.num_requests >= self.length:
                raise StopIteration
            else:
                result = random.randint(self.lower, self.upper)
                self.num_requests += 1
                return result
```

```python
randoms = RandomInts(10)
```

```python
for num in randoms:
    print(num)
```

We can now sort our iterable using the `sorted()` method:

```python
sorted(randoms)
```

```python
sorted(randoms, reverse=True)
```

```python

```
