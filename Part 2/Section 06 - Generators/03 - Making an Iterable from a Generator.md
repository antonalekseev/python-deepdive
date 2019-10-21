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

### Making an Iterable from a Generator


As we now know, generators are iterators.

This means that they become exhausted - so sometimes we want to create an iterable instead.

There's no magic here, we simply have to implement a class that implements the iterable protocol:


Let's write a simple generator that generates the squares of integers:

```python
def squares_gen(n):
    for i in range(n):
        yield i ** 2
```

Now, we can create a new generator:

```python
sq = squares_gen(5)
```

```python
for num in sq:
    print(num)
```

But, `sq` was an iterator - so now it's been exhausted:

```python
next(sq)
```

To restart the iteration we have to create a new instance of the generator (iterator):

```python
sq = squares_gen(5)
```

```python
[num for num in sq]
```

So, let's wrap this in an iterable:

```python
class Squares:
    def __init__(self, n):
        self.n = n
        
    def __iter__(self):
        return squares_gen(self.n)
```

```python
sq = Squares(5)
```

```python
[num for num in sq]
```

And we can do it again:

```python
[num for num in sq]
```

We can put those pieces of code together if we prefer:

```python
class Squares:
    def __init__(self, n):
        self.n = n
        
    @staticmethod
    def squares_gen(n):
        for i in range(n):
            yield i ** 2
        
    def __iter__(self):
        return Squares.squares_gen(self.n)
```

```python
sq = Squares(5)
```

```python
[num for num in sq]
```

#### Generators used with other Generators


I want to point out that you can also easily run into various bugs when you use generators with other generator functions.

Consider this example:

```python
def squares(n):
    for i in range(n):
        yield i ** 2
```

```python
sq = squares(5)
```

```python
enum_sq = enumerate(sq)
```

Now `enumerate` is lazy, so `sq` had not, at this point, been consumed:

```python
next(sq)
```

```python
next(sq)
```

Since we have consumed two elements from `sq`, when we now use `enumerate` it will have two less elements from sq:

```python
next(enum_sq)
```

You'll notice that we don't get the first element of the original `sq` - instead we get the third element (`2 ** 2`).

Moreover, you'll notice that the index returned in the tuple produced by `enumerate` is 0, not 2!
