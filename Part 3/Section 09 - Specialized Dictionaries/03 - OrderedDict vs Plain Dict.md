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

### OrderedDict vs Python 3.6 Plain Dicts


So, the question, if we are targeting Python 3.6+ is whether we lose anything by not using an `OrderedDict` since plain `dicts` now preserve key order.

As we saw in the previous video there were a few features that `OrderedDicts` offer that `dicts` do not have:

* reverse iteration
* pop first/last item
* move key to beginning/end of dictionary
* equality (`==`) that takes key order into account


We can actually achieve of these things using plain dictionaries, it's just not as straightforward as using the OrdertedDict methods - although I would not be surprised if Python dictionaries eventually get this functionality now that they have a guaranteed key order preservation.

```python
from collections import OrderedDict
```

#### Reverse Iteration

```python
d1 = OrderedDict(a=1, b=2, c=3, d=4)
d2 = dict(a=1, b=2, c=3, d=4)
```

```python
print(d1)
print(d2)
```

```python
for k in reversed(d1):
    print(k)
```

This will not work with a plain dictionary, and neither will it work with the views.

But, it looks like this will get implemented in Python 3.8 - https://bugs.python.org/issue33462

For now, it can be done but it means making a list out of the keys, and then iterating through the reversed list:

```python
for k in reversed(list(d2.keys())):
    print(k)
```

This is of course not iteal since we have to make a copy of all the keys into a list first - not very efficient. So, we should probably wait for Python 3.8 :-)


#### Popping Items


Next let's look at `popitem` - we need to be able to pop either the first or the last element.

To do this, we really need to be able to determine the *first* and *last* key in the dictionary - again, this is not something we currently have natively in plain dictionaries, so we need to calculate them ourselves.


Getting the first key is not difficult - we simply retrieve the first key from the keys() view for example:

```python
first_key = next(iter(d2.keys()))
print(d2)
print(first_key)
```

Fiding the last key is a bit more challenging, but fortunately, we can just use the `popitem` method on plain dictionaries that is guaranteed to pop the last insert item - again, this is a guarantee only in Python 3.7 and above:

```python
d1 = OrderedDict(a=1, b=2, c=3, d=4)
d2 = dict(a=1, b=2, c=3, d=4)

print(d2)
print(d2.popitem())
print(d2)
```

So we could combine these into a custom function as follows:

```python
def popitem(d, last=True):
    if last:
        return d.popitem()
    else:
        first_key = next(iter(d.keys()))
        return first_key, d.pop(first_key)
```

```python
d2 = dict(a=1, b=2, c=3, d=4)
print(d2)
print(popitem(d2))
print(d2)
```

```python
d2 = dict(a=1, b=2, c=3, d=4)
print(d2)
print(popitem(d2, last=False))
print(d2)
```

#### Move to End


Next let's look at the `move_to_end` method, which can move any key to either the beginning or the end of the dictionary.

Moving a key to the end of the dictionary is easy - we simply pop the item, and insert it again - because of the gauranteed insertion order, this means the key will now be placed at the end of the dictionary:

```python
d2 = dict(a=1, b=2, c=3, d=4)
print(d2)
key = 'b'
d2[key] = d2.pop(key)
print(d2)
```

<!-- #region -->
Moving to the beginning however is not as easy - the only way I could think of was to take the desired key and moving it to the end first. Then, take every key preceding it, and pop them off and add them back to the dictionary one by one, until, but not including the target key we wanted to move to the beginning of the dictionary.

In other words something like this:

```a b c d e f```

To move `c` to the front, first pop it and add it to the dictionary:
``` a b d e f c```

Now we do the same thing to every key preceding `c`, essentially moving each key one by one to the end of the dictionary:

```b d e f c a```

```d e f c a b```

```e f c a b d```

```f c a b d e```

```c a b d e f```
<!-- #endregion -->

We can code it this way:

```python
d = dict(a=1, b=2, c=3, d=4, e=5, f=6)
key = 'c'

print(d.keys())

# first move desired key to end
d[key] = d.pop(key)  
print(d.keys())

keys = list(d.keys())[:-1]
for key in keys:
    d[key] = d.pop(key)
    print(d.keys())
    
print(d)
```

We can combine both into a single function:

```python
def move_to_end(d, key, *, last=True):
    d[key] = d.pop(key)
    
    if not last:
        for key in list(d.keys())[:-1]:
            d[key] = d.pop(key)       
```

```python
d = dict(a=1, b=2, c=3, d=4, e=5, f=6)
```

```python
move_to_end(d, 'c')
print(d)
```

```python
move_to_end(d, 'c', last=False)
print(d)
```

#### Equality Comparison


Lastly let's look at equality (`==`) comparisons.
Even though Python 3.6+ guarantees key ordering based on the insertion order, two dictionaries with the same key/values but in different order will compare equal, but not so with `OrderedDict`.

To achieve the same type of "key-order-sensitive" comparison we therefore need to make sure of two things:
1. the dictionaries are equal - i.e. have the same key/value pairs
2. the order of the keys is the same in both dictionaries

We can easily achieve this comparing the dictionaries and the `keys()` views to make sure they are equal:

```python
d1 = {'a': 10, 'b': 20, 'c': 30}
d2 = {'b': 20, 'c': 30, 'a': 10}
```

```python
d1 == d2
```

Now just comparing the `keys()` views will not work:

```python
d1.keys() == d2.keys()
```

Remember that the `keys()` view behaves like a `set`, so comparisons will be `True` as long as the same elements (keys) are present in both sets - but ordering does not matter.


Instead, we can materialize these views as lists, and then compare the lists:

```python
list(d1.keys()) == list(d2.keys())
```

So to test for "key-order-sensitive" equality, we can simply do this:

```python
d1 == d2 and list(d1.keys()) == list(d2.keys())
```

Of course, materializing the lists incurs some overhead, so instead we could use iteration through both key views and make sure each corresponding key is equal.

There are a number of ways to do this, here I'm going to use `zip` to do it:

```python
def dict_equal_sensitive(d1, d2):
    if d1 == d2:
        for k1, k2 in zip(d1.keys(), d2.keys()):
            if k1 != k2:
                return False
        return True
    else:
        return False
```

```python
dict_equal_sensitive(d1, d2)
```

```python
dict_equal_sensitive(d1, d1)
```

If you want a pure functional programming approach that does not use a loop, we can do it this way too, using `all` and `map`:

```python
def dict_equal_sensitive(d1, d2):
    if d1 == d2:
        return all(map(lambda el: el[0] == el[1], 
                       zip(d1.keys(), d2.keys())
                      )
                  )
    else:
        return False
```

```python
dict_equal_sensitive(d1, d2)
```

```python
dict_equal_sensitive(d1, d1)
```

So, we can perform all these operations on a standard dictionary, but it is a lot more work to do so - for now I would stick to using an `OrderedDict` when I need those specific methods beyond just a guaranteed key order. If the guaranteed key order is all I need, then a plain `dict` will work just fine.


#### Timings


What about timings?

Let's look at a few timings to see the performance difference between plain `dicts` and `OrderedDicts`.

```python
from timeit import timeit
```

```python
def create_dict(n=100):
    d = dict()
    for i in range(n):
        d[i] = i
    return d
```

```python
def create_ordered_dict(n=100):
    d = OrderedDict()
    for i in range(n):
        d[i] = i
    return d
```

```python
timeit('create_dict(10_000)', globals=globals(), number=1_000)
```

```python
timeit('create_ordered_dict(10_000)', globals=globals(), number=1_000)
```

As you can see, creating an OrderedDict has slightly more overhead.

Let's see if recovering a key from an `OrderedDict` is slower than a plain `dict`:

```python
d1 = create_dict(10_000)
d2 = create_ordered_dict(10_000)

timeit('d1[9_999]', globals=globals(), number=100_000)
```

```python
timeit('d2[9_999]', globals=globals(), number=100_000)
```

So no significant difference between these two.



Let's see how pop (first and last) differs:

```python
n = 1_000_000
d1 = create_dict(n)
timeit('d1.popitem()', globals = globals(), number=n)
```

```python
n = 1_000_000
d2 = create_ordered_dict(n)
timeit('d2.popitem(last=True)', globals = globals(), number=n)
```

Perhaps not surprisingly, the built-in `dict` is substantially faster at popping the last item of the dictionary.

What about popping the first item?

```python
n = 100_000
d1 = create_dict(n)
timeit('popitem(d1, last=False)', globals = globals(), number=n)
```

```python
n = 100_000
d2 = create_ordered_dict(n)
timeit('d2.popitem(last=False)', globals = globals(), number=n)
```

As you can see, substantially faster in an `OrderedDict`.


You can try the other methods (`move_to_end` and equality testing) yourself - if you do, please post your results in the **Q&A** section!
Or maybe you can come up with more efficient alternatives to what we have here for pop, move, etc.

```python

```
