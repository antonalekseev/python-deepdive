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

### Slicing


Slices can actually be defined using the `slice()` function which creates a `slice` object:

```python
s = slice(0, 2)
```

```python
type(s)
```

```python
s.start
```

```python
s.stop
```

```python
l = [1, 2, 3, 4, 5]
l[s]
```

This can be useful in practice to make code more readable.

Suppose you are parsing fixed-width file. You would need to define the start/end column of each field in the rows of the file.

So you might write something like this:

```python
data = []  # a collection of rows, read from a file maybe
for row in data:
    first_name = row[0:51]
    last_name = row[51:101]
    ssn = row[101:111]
    # etc
```

Instead, you might write:

```python
range_first_name = slice(0, 51)
range_last_name = slice(51, 101)
range_ssn = slice(101, 111)
```

These might even be defined in your global scope, or maybe a config file.

Then in your code you would write this instead:

```python
for row in data:
    first_name = row[range_first_name]
    last_name = row[range_last_name]
    ssn = row[range_ssn]
```

Separating the slice definition from the code that uses the slice makes it now much easier to update your slice definitions in one place, rather than hunt for them all over the place.


#### Slice Fundamentals


Indexing is zero-based in Python, and slices are inclusive of their start-index, and exclusive of their end-index:

```python
l = 'python'
l[0:1], l[0:6]
```

Additionally, extended slicing allows specifying a step value:

```python
l = 'python'
l[0:6:2], l[0:6:3]
```

And extended slices can also be defined using `slice`:

```python
s1 = slice(0, 6, 2)
s2 = slice(0, 6, 3)
l[s1], l[s2]
```

Unlike regular indexing (e.g. `l[n]`), it's OK for slice indexes to be "out of bounds":

```python
l = [1, 2, 3, 4, 5, 6]
l[0:100]
```

```python
l[-10:100]
```

But regular indexing will raise exceptions for out of bound errors:

```python
l = [1, 2, 3, 4, 5, 6]
l[100]
```

In slicing, if we do not specify the start/end index, Python will automatically use the start/end of the sequence we are slicing:

```python
l = [1, 2, 3, 4, 5, 6]
```

```python
l[:4]
```

```python
l[4:]
```

In fact, we can omit both:

```python
l[:]
```

In addition to the start/stop values allowing for negative values, the step value can also be negative. This simply means the sequence will traversed in the opposite direction:

```python
l = [0, 1, 2, 3, 4, 5]
```

```python
l[3:0:-1]
```

Basically we started at `3` (inclusive) and went in steps of `-1`, ending at (but not including) `0`.


If we wanted to include the `0` index element, we could do it by ommitting the end value:

```python
l[3::-1]
```

We could also do the following:

```python
l[3:-100:-1]
```

But this would not work as expected:

```python
l[3:-1:-1]
```

Why?


Remember from the lecture that this range equivalence would be:

`3 --> 3`

`-1 < 0 --> max(-1, 6-1) --> max(-1, 5) --> 5`

so equivalent range would be given by:

```python
list(range(3, 5, -1))
```

which of course is an empty range!


#### Easily Converting a Slice to a Range


We can easily determine the effective range of a slice by using the `indices` method in the `slice` object. The only thing is that in order to do this we must know the length of the sequence we are slicing.


For example, if our list has a length of 10:

```python
slice(1, 5).indices(10)
```

```python
list(range(1, 5, 1))
```

```python
l = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
l[1:5]
```

The `slice` object can also handle extended slicing:

```python
slice(0, 100, 2).indices(10)
```

```python
list(range(0, 10, 2))
```

```python
l[0:100:2]
```

We can easily retrieve a list of indices from a slice by passing the unpacked tuple returned by the `indices` method to the range function's arguments and converting to a list:

```python
list(range(*slice(None, None, -1).indices(10)))
```

As we can see from this example, using a slice such as `[::-1]` returns a sequence that is in reverse order from the original one.

```python
l
```

```python
l[::]
```

```python
l[::-1]
```
