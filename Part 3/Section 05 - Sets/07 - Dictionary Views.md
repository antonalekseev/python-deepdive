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

### Views: Keys, Values and Items


#### Views are not Static


These view objects are not static - so it's not like Python makes a copy of the keys, values or items, and uses these static copies. They are like windows (views) into the **current** state of the dictionary. If the dictionary changes, then these views reflect those changes immediately.

Basically these views provide methods that access the underlying dictionary. They do not "own" any data.

```python
d = {'a': 1, 'b': 2}
```

```python
keys = d.keys()
values = d.values()
items = d.items()
```

```python
print(id(keys), id(values), id(items))
```

```python
print(keys)
print(values)
print(items)
```

```python
d['z'] = 100
```

```python
print(id(keys), id(values), id(items))
```

As you can see the memory address of these view objects has not changed:

```python
print(keys)
print(values)
print(items)
```

but the view 'contents' have changed. These views are **dynamic**.


#### Mutating a dictionary while iterating over these views


Because these views instantly reflect any modifications made to the underlying dictionary, we have to be careful changing the dictionary while we iterate over a view! 

```python
d = {'a': 1, 'b': 2, 'c': 3}
```

```python
for k, v in d.items():
    print(k, v)
    del d[k]
```

As you can see Python complains about this. But the interesting thing is that Python does not complain about the deletion itself - notice where the exception occurs - at the loop, not the delete statement.

In fact, the dictionary **has** changed:

```python
d
```

As you can see the key `a` is gone.

So the deletion happens just fine, but when Python continues the loop, at that point it detects that the dictionary has changed - and an exception is raised at that point. But notice the exception message - Python is complaining about the **size** of the dictionary changing... 
We'll come back to that point in a minute.


What about insertions, will Python complain about it?

```python
d = {'a': 1, 'b': 2, 'c': 3}
for k, v in d.items():
    print(k, v)
    d['z'] = 100
```

No, that's not allowed either.


It is perfectly fine to modify the values though:

```python
d = {'a': 1, 'b': 2, 'c': 3}
for k, v in d.items():
    print(k, v)
    d[k] = 1000
```

and of course our dictionary values have changed:

```python
d
```

What about the other views, are they more tolerant of underlying mutations? We would not expect the key view to allow this, but what about the values view? After all it is not referencing the keys at all...

```python
d = {'a': 1, 'b': 2, 'c': 3}
for v in d.values():
    print(v)
    del d['a']
```

No, not allowed either! We just cannot change the size of the dictionary (and hence the size of the view too) while iterating over it.


So, if that's the limitation, then we should be able to modify the values of elements as we iterate over the keys:

```python
d = {'a': 1, 'b': 2, 'c': 3}
```

```python
for key in d.keys():
    d[key] = 100
```

```python
d
```

Even this will work fine:

```python
for k, v in d.items():
    d[k] = v * 2
```

```python
d
```

The moral here is that you should not manipulate the keys of a dictionary as you iterate over it - either directly, or using the views.
Mutating associated values is perfectly fine.


#### Iterating over a dictionary vs iterating over the keys view


As we just mentioned, dictionaries implement the iterable protocol which iterates over the keys of the dictionary:

```python
d = dict.fromkeys('python', 0)
```

```python
for k in d:
    print(k)
```

This would be the same as requesting the iterator from the dictionary and using the iterator:

```python
d_iter = iter(d)
for k in d_iter:
    print(k)
```

So, you may very well be asking yourself whether we should iterate keys using the dictionaries iterator, or using the key view? After all it seems to do the same thing...


And yes, either one would be just fine for iterating over the keys of a dictionary. Let's just make sure the performance is about the same:

```python
from timeit import timeit
from random import randint

d = {k: randint(0, 100) for k in range(10_000)}
keys = d.keys()

def iter_direct(d):
    for k in d:
        pass
    
def iter_view(d):
    for k in d.keys():
        pass

def iter_view_direct(view):
    for k in view:
        pass
    
print(timeit('iter_direct(d)', globals=globals(), number=20_000))
print(timeit('iter_view(d)', globals=globals(), number=20_000))
print(timeit('iter_view_direct(keys)', globals=globals(), number=20_000))
```

As you can see, unless you are re-creating a new view object every time, the performance difference between iterating via the dictionary's iterator and the view's iterator is about the same. [In fact, it's the same iterator in both cases!] 
But since there is really no need to re-create a view once it's been created (since is is dynamic), the overhead of creating the `keys` view is a one-time hit.
And a `keys` view provides far more functionality than just iteration - as we know it behaves like a set - so if you need to perform set operations on the keys you'll need to use the `keys` view.


#### Iterating over keys and values


As we saw, we can use the `.items()` view to iterate over both the keys and values of a dictionary.

```python
d = {'a': 1, 'b': 2, 'c': 3}
```

```python
for k, v in d.items():
    print(k, v)
```

You might be tempted to do it this way as well:

```python
for k in d:
    print(k, d[k])
```

But this is quite inefficient!
Let's try some timings.

```python
d = {k: randint(0, 100) for k in range(10_000)}
items = d.items()

def iterate_view(view):
    for k, v in view:
        pass
    
def iterate_clunky(d):
    for k in d:
        d[k]
        
print(timeit('iterate_view(items)', globals=globals(), number=5_000))
print(timeit('iterate_clunky(d)', globals=globals(), number=5_000))
```

As you can see, it is substantially slower to iterate over both the keys and the values of the dictionary using the second approach. This is because in the second approach, we have to perform a lot of dictionary lookups - while lookups are particularlay efficient in Python, they are slower than not doing a lookup at all!


#### Iterating a dictionary while mutating keys


As we mentioned earlier, we cannot mutate a dictionary's keys while iterating over it:


Let's see an example of this:

```python
d = {'a': 1, 'b': 2, 'c': 3}
for k, v in d.items():
    print(k, v ** 2)
    del d[k]
```

One way to solve this is to create a static list of all the keys, and iterate over that instead:

```python
d = {'a': 1, 'b': 2, 'c': 3}
keys = list(d.keys())
print(keys)
```

```python
for k in keys:
    value = d.pop(k)
    print(f'{value} ** 2 = {value ** 2}')
```

```python
d
```

Another way would be to use the `popitem` method. We just need to know how many times we can call `popitem`, or catch the `KeyError` exception when it occurs:

```python
d = {'a':1, 'b':2, 'c':3}
for _ in range(len(d)):
    key, value = d.popitem()
    print(key, value, value**2)
```

Or we can use a `while` loop:

```python
d = {'a':1, 'b':2, 'c':3}
while len(d) > 0:
    key, value = d.popitem()
    print(key, value, value**2)
```

Or we can simply keep iterating indefinitely until a `KeyError` exception occurs:

```python
d = {'a':1, 'b':2, 'c':3}
while True:
    try:
        key, value = d.popitem()
    except KeyError:
        break
    else:
        print(key, value, value**2)
```
