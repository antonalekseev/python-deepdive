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

### Frozen Sets


`frozenset` is the **immutable** equivalent of the plain `set`.

Apart from the fact that you cannot mutate the collection (i.e. add or remove elements), the interesting thing is that frozen sets are hashable (as long as each contained element is also hashable).

This means that whereas we cannot create a set of sets, we can create a set of frozen sets (or a frozen set of frozen sets). It also means that we can use frozen sets as dictionary keys.

There is no literal for frozen sets - we have to use the `frozenset()` callable. It is used the same way to create frozensets that `set()` would be used to create sets.

```python
s1 = {'a', 'b', 'c'}
```

```python
hash(s1)
```

```python
s2 = frozenset(['a', 'b', 'c'])
```

```python
hash(s2)
```

And we can create a set of frozen sets:

```python
s3 = {frozenset({'a', 'b'}), frozenset([1, 2, 3])}
```

```python
s3
```

#### Copying Frozen Sets


Remember what happens when we create a shallow copy of a tuple using the `tuple()` callable?

```python
t1 = (1, 2, [3, 4])
```

```python
t2 = tuple(t1)
```

```python
t1 is t2
```

This is quite different from what happens with a list:

```python
l1 = [1, 2, [3, 4]]
l2 = list(l1)
```

```python
l1 is l2
```

Remember that there's really no point in making a shallow copy of an immutable container - so, Python optimizes this for us and just returns the original tuple. Of course, lists are mutable, and that optimization cannot happen.

The same thing happens with sets and frozen sets:

```python
s1 = {1, 2, 3}
s2 = set(s1)
s1 is s2
```

```python
s1 = frozenset([1, 2, 3])
s2 = frozenset(s1)
print(type(s1), type(s2), s1 is s2)
```

Same goes with the `copy()` method:

```python
s2 = s1.copy()
print(type(s1), type(s2), s1 is s2)
```

Of course, this will not happen with a deep copy in general:

```python
from copy import deepcopy
```

```python
s2 = deepcopy(s1)
print(type(s1), type(s2), s1 is s2)
```

#### Set Operations


All the non-mutating set operations we studied with sets also apply to frozen sets.

But, in addition, we can mix sets and frozen sets when performing these operations.

For example:

```python
s1 = frozenset({'a', 'b'})
s2 = {1, 2}
s3 = s1 | s2
```

```python
s3
```

What's important to note here is the data type of the result - it is a frozen set.
Let's do this operation again, but switch around `s1` and `s2`:

```python
s3 = s2 | s1
```

```python
s3
```

As you can see, the result is now a standard set.

Basically the data type of the first operand determines the data type of the result.

```python
s1 = frozenset({'a', 'b', 'c'})
s2 = {'c', 'd', 'e'}
```

```python
s1 & s2
```

```python
s2 & s1
```

Same goes with differences and symmetric differences:

```python
s1 - s2
```

```python
s2 - s1
```

```python
s1 ^ s2
```

```python
s2 ^ s1
```

What about equality?

```python
s1 = {1, 2}
s2 = frozenset(s1)
```

```python
s1 is s2
```

```python
s1 == s2
```

As you can see, this is very similar behavior to numerical values:

```python
1 == 1.0
```

```python
1 == 1 + 0j
```

Even though they are not the same data type (and hence cannot possibly be the same object), equality still works "as expected".


##### Application 1


One application of frozen sets, assuming they are hashable, is as keys for a dictionary.

Recall an example we worked on in the past where we wanted a `Person` object to be used as a key in a dictionary.

We had to define the class, equality and the hash - that was quite a bit of work for what amounted to, in the end just checking that the name and age were the same.

Of course, we may have more complex instances of this, but for a simple case like that, especially if we consider our `Person` class to be immutable, it would have been easier to just use a frozen set containing the name and age:

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
        
    def __repr__(self):
        return f'Person(name={self._name}, age={self._age})'
    
    @property
    def name(self):
        return self._name
        
    @property
    def age(self):
        return self._age
    
    def key(self):
        return frozenset({self.name, self.age})
```

```python
p1 = Person('John', 78)
p2 = Person('Eric', 75)
```

```python
d = {p1.key(): p1, p2.key(): p2}
```

```python
d
```

And we can easily lookup using those keys now:

```python
d[frozenset({'John', 78})]
```

```python
d[frozenset({78, 'John'})]
```

Of course this is kind of a limited use case, but in the event you have the need to use sets as dictionary keys, then you technically can using a frozen set (as long as the elements are all hashable).


##### Application 2


A slightly more interesting application of this is memoization. I cover memoization in detail in Part 1 of this series in the section on decorators.


Recall that memoization is basically a technique to cache the results of a (deterministic) function call based on the provided arguments. A cache is created that contains the results of calling the function with a particular set of arguments, the next time the function is called, the arguments are checked against the cache - if the arguments exist in the cache, then the cached value is returned instead of re-executing the function.

Although Python's `functools` has the `lru_cache` decorator available, there is one drawback - the order of the keyword arguments matters.

Let's see this:

```python
from functools import lru_cache
```

```python
@lru_cache()
def my_func(*, a, b):
    print('calculating a+b...')
    return a + b
```

```python
my_func(a=1, b=2)
```

```python
my_func(a=1, b=2)
```

Notice how the second time around, we did not see `calculating a+b...` printed out - that's because the value was pulled from cache.

But now look at this:

```python
my_func(b=2, a=1)
```

Even though the values are technically the same, the order in which we specified them as different, and the cache considered the arguments to be different. Now of course, both "styles" are cached:

```python
my_func(a=1, b=2)
my_func(b=2, a=1)
```

An interesting side note, now that we know all about hashability!
You'll notice that the way `my_func` works we can actually pass in other data types than just numbers. We could use strings, tuples, even lists or sets:

```python
my_func(a='abc', b='def')
```

```python
my_func(a='abc', b='def')
```

As you can see caching works just fine.
But what is being used to back the cache for `lru_cache`? A dictionary...
And what do we know about dictionary keys? They must be hashable!

So this will actually fail, and not because the function can't handle it, but because the `lru_cache` mechanism cannot:

```python
my_func(a=[1, 2, 3], b=[4, 5, 6])
```

Let's write our own version of this.
We'll use a dictionary to cache the arguments - so we'll need to come up with a key representing the arguments - and one in which the order of the keyword-only arguments does not matter. We'll have the same limitation in terms of hashable keys as `lru_cache`, but at least we won't have the argument ordering issue:

```python
def memoizer(fn):
    cache = {}
    def inner(*args, **kwargs):
        key = (*args, frozenset(kwargs.items()))
        if key in cache:
            return cache[key]
        else:
            result = fn(*args, **kwargs)
            cache[key] = result
            return result
    return inner
```

```python
@memoizer
def my_func(*, a, b):
    print('calculating a+b...')
    return a + b
```

```python
my_func(a=1, b=2)
```

```python
my_func(a=1, b=2)
```

So far so good... Now let's swap the arguments around:

```python
my_func(b=2, a=1)
```

Yay!! It used the cache!


We can even tweak this to effectively provide more efficient caching when the order of positional arguments is not important either:

```python
def memoizer(fn):
    cache = {}
    def inner(*args, **kwargs):
        key = frozenset(args) | frozenset(kwargs.items())
        if key in cache:
            return cache[key]
        else:
            result = fn(*args, **kwargs)
            cache[key] = result
            return result
    return inner
```

```python
@memoizer
def adder(*args):
    print('calculating...')
    return sum(args)
```

```python
adder(1, 2, 3)
```

```python
adder(3, 2, 1)
```

```python
adder(2, 1, 3)
```

```python
adder(1, 2, 3, 4)
```

```python
adder(4, 2, 1, 3)
```

Isn't Python fun!!
