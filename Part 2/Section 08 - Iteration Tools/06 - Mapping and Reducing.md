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

### Mapping and Reducing


#### *map* and *starmap*


You should already know the `map` and `reduce` built-in functions, so let's quickly review them:


The `map` function applies a given function (that takes a single argument) to an iterable of values and yields (lazily) the result of applying the function to each element of the iterable.

Let's see a simple example that calculates the square of values in an iterable:

```python
maps = map(lambda x: x**2, range(5))
```

```python
list(maps)
```

Keep in mind that `map` returns an iterator, so it will become exhausted:

```python
list(maps)
```

Of course, we can supply multiple values to a function by using an iterable of iterables (e.g. tuples) and unpacking the tuple in the function - but we still only use a single argument:

```python
def add(t):
    return t[0] + t[1]
```

```python
list(map(add, [(0,0), [1,1], range(2,4)]))
```

Remember how we can unpack an iterable into separate positional arguments?

```python
def add(x, y):
    return x + y
```

```python
t = (2, 3)
add(*t)
```

It would be nice if we could do that with the `map` function as well.

For example, it would be nice to do the following:

```python
list(map(add, [(0,0), (1,1), (2,2)]))
```

But of course that is not going to work, since `add` expects two arguments, and only a single one (the tuple) was provided.


This is where `starmap` comes in - it will essentially `*` each element of the iterable before passing it to the function defined in the map:

```python
from itertools import starmap
```

```python
list(starmap(add, [(0,0), (1,1), (2,2)]))
```

#### Accumulation


You should already know the `sum` function - it simply calculates the sum of all the elements in an iterable:

```python
sum([10, 20, 30])
```

It simply returns the final sum.

Sometimes we want to perform other operations than just summing up the values. Maybe we want to find the product of all the values in an iterable.

To do so, we would then use the `reduce` function available in the `functools` module. You should already be familiar with that function, but let's review it quickly.

The `reduce` function requires a `binary` function (a function that takes two arguments). It then applies that binary function to the first two elements of the iterable, obtains a result, then continues applying the binary function using the previous result and the next item in the iterable.

Optionally we can specify a seed value that is used as the 'first' element.

For example, to obtain the product of all values in an iterable:

```python
from functools import reduce
```

```python
reduce(lambda x, y: x*y, [1, 2, 3, 4])
```

We can even specify a "start" value:

```python
reduce(lambda x, y: x*y, [1, 2, 3, 4], 10)
```

You'll note that with both `sum` and `reduce`, only the final result is shown - none of the intermediate results are available.

Sometimes we want to see the intermediate results as well.

Let's see how we might try it with the `sum` function:|

```python
def sum_(iterable):
    it = iter(iterable)
    acc = next(it)
    yield acc
    for item in it:
        acc += item
        yield acc
```

And we can use it as follows:

```python
for item in sum_([10, 20, 30]):
    print(item)
```

Of course, this is just going to work for a sum.

We may want the same functionality with arbitrary binary functions, just like `reduce` was more general than `sum`.


We could try doing it ourselves as follows:

```python
def running_reduce(fn, iterable, start=None):
    it = iter(iterable)
    if start is None:
        accumulator = next(it)
    else:
        accumulator = start
    yield accumulator
    
    for item in it:
        accumulator = fn(accumulator, item)
        yield accumulator
    
```

Let's try a running sum first.

We'll use the `operator` module instead of using lambdas.

```python
import operator
```

```python
list(running_reduce(operator.add, [10, 20, 30]))
```

Now we can also use other binary operators, such as multiplication:

```python
list(running_reduce(operator.mul, [1, 2, 3, 4]))
```

And of course, we can even set a "start" value:

```python
list(running_reduce(operator.mul, [1, 2, 3, 4], 10))
```

While this certainly works, we really don't need to code this ourselves - that's exactly what the `accumulate` function in `itertools` does for us.

The order of the arguments however is different, The iterable is defined first - that's because the binary function is optional, and defaults to addition if we don't specify it. Also it does not have a "start" value option. If you really need that feature, you could use the technique I just showed you.

```python
from itertools import accumulate
```

```python
list(accumulate([10, 20, 30]))
```

We can find the running product of an iterable:

```python
list(accumulate([1, 2, 3, 4], operator.mul))
```
