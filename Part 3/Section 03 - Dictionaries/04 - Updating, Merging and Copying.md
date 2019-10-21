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

### Updating, Merging and Copying


Updating an existing key's value in a dictionary is straightforward:

```python
d = {'a': 1, 'b': 2, 'c': 3}
```

```python
d['b'] = 200
```

```python
d
```

#### The `update` method


Sometimes however, we want to update all the items in one dictionary based on items in another dictionary.


For that we can use the `update` method.


The `update` method has three forms:
1. it can take another dictionary
2. it can take an iterable of iterables of length 2 (key, value)
3. if can take keyword arguments

You'll notice that the arguments we can use with `update` is very similar to the type of arguments we can use with the `dict()` function when we create dictionaries.


Let's look briefly at each of those forms:

```python
d1 = {'a': 1, 'b': 2}
d2 = {'c': 3, 'd': 4}
```

```python
d1.update(d2)
print(d1)
```

Note how the key order is maintained and based on the order in which the dictionaries were create/updated.

```python
d1 = {'a': 1, 'b': 2}
```

```python
d1.update(b=20, c=30)
print(d1)
```

Again notice how the key order reflects the order in which the parameters were specified when calling the `update` method.

```python
d1 = {'a': 1, 'b': 2}
```

```python
d1.update([('c', 2), ('d', 3)])
```

```python
d1
```

Of course we can use more complex iterables. For example we could use a generator expression:

```python
d = {'a': 1, 'b': 2}
d.update((k, ord(k)) for k in 'python')
print(d)
```

So far we have updated dictionaries with other dictionaries or iterables that do not contain the same keys. Sometimes that does happen - in that case, the corresponding key in the dictionary being updated has it's associated value replaced by the new value:

```python
d1 = {'a': 1, 'b': 2, 'c': 3}
d2 = {'b': 200, 'd': 4}
d1.update(d2)
print(d1)
```

#### Unpacking dictionaries


We can also use unpacking to unpack the contents of one dictionary into the elements of another dictionary. This is very similar to how we can unpack iterables. Let's recall that first:

```python
l1 = [1, 2, 3]
l2 = 'abc'
l = (*l1, *l2)
print(l)
```

We can do something similar with dictionaries:

```python
d1 = {'a': 1, 'b': 2}
d2 = {'c': 3, 'd': 4}
d = {**d1, **d2}
print(d)
```

Again note how order is preserved.
What happens when there are conflicting keys in the unpacking?

```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 200, 'c': 3}
d = {**d1, **d2}
print(d)
```

As you can see, the 'last' key/value pair wins.

Now the nice thing about unpacking is that we are not limited to just two dictionaries.


##### Example


In this example we have some dictionaries we use to configure our application.
One dictionary specifies some configuration defaults for every configuration parameter our application will need.
Another dictionary is used to configure some global configuration, and another set of dictionaries is used to define environment specific configurations, maybe dev and prod.

```python
conf_defaults = dict.fromkeys(('host', 'port', 'user', 'pwd', 'database'), None)
print(conf_defaults)
```

```python
conf_global = {
    'port': 5432,
    'database': 'deepdive'}
```

```python
conf_dev = {
    'host': 'localhost',
    'user': 'test',
    'pwd': 'test'
}

conf_prod = {
    'host': 'prodpg.deepdive.com',
    'user': '$prod_user',
    'pwd': '$prod_pwd',
    'database': 'deepdive_prod'
}
```

Now we can generate a full configuration for our dev environment this way:

```python
config_dev = {**conf_defaults, **conf_global, **conf_dev}
```

```python
print(config_dev)
```

and a config for our prod environment:

```python
config_prod = {**conf_defaults, **conf_global, **conf_prod}
```

```python
print(config_prod)
```

##### Example


Another way dictionary unpacking can be really useful, is for passing keyword arguments to a function:

```python
def my_func(*, kw1, kw2, kw3):
    print(kw1, kw2, kw3)
```

```python
d = {'kw2': 20, 'kw3': 30, 'kw1': 10}
```

In this case, we don't really care about the order of the elements, since we'll be unpacking keyword arguments:

```python
my_func(**d)
```

Of course we can even use it this way, but here the dictionary order does matter, as it will be reflected in the order in which those arguments are passed to the function:

```python
def my_func(**kwargs):
    for k, v in kwargs.items():
        print(k, v)
```

```python
my_func(**d)
```

As you can see the function's `kwargs` dictionary received the elements in the same order as the original dictionary we unpacked.


#### Copying Dictionaries


We can make copies of dictionaries. But as with iterables, we have to differentiate between **shallow** and **deep** copies.


The `copy` method that dictionaries implement is a shallow copy mechanism.
This means that a new container is created, but the item references within the collection are maintained.

Let's see a simple example:

```python
d = {'a': [1, 2], 'b': [3, 4]}
```

```python
d1 = d.copy()
```

```python
print(d)
print(d1)
```

```python
id(d), id(d1), d is d1
```

So `d` and `d1` are not the same objects, so we can add and remove keys from one dict without affecting the other. Also, we can completely replace an associated value in one without affecting the other.

```python
del d['a']
```

```python
print(d)
print(d1)
```

```
or even:
```

```python
d['b'] = 100
```

```python
print(d)
print(d1)
```

But let's see what happens if we mutate the value of one dictionary:

```python
d = {'a': [1, 2], 'b': [3, 4]}
d1 = d.copy()
print(d)
print(d1)
```

```python
d['a'].append(100)
```

```python
print(d)
```

```python
print(d1)
```

As you can see the mutation was also "seen" by `d1`. This is because the objects `d['a']` and `d1['a']` are in fact the **same** objects.

```python
d['a'] is d1['a']
```

So if we have nested dictionaries for example, as is often the case with JSON documents, we have to be careful when creating shallow copies.

```python
d = {'id': 123445,
    'person': {
        'name': 'John',
        'age': 78},
     'posts': [100, 105, 200]
    }
```

```python
d1 = d.copy()
```

```python
d1['person']['name'] = 'John Cleese'
d1['posts'].append(300)
```

```python
d1
```

```python
d
```

If we want to avoid this issue, we have to create a **deep** copy.
We can easily do this ourselves using recursion, but the `copy` module implements such a function for us:

```python
from copy import deepcopy
```

```python
d = {'id': 123445,
    'person': {
        'name': 'John',
        'age': 78},
     'posts': [100, 105, 200]
    }
```

```python
d1 = deepcopy(d)
```

```python
d1['person']['name'] = 'John Cleese'
d1['posts'].append(300)
```

```python
d1
```

```python
d
```

We saw earlier that we can also copy a dictionary by essentially unpacking the keys of one, or more dictionaries, into another.
This also creates a **shallow** copy:

```python
d1 = {'a': [1, 2], 'b':[3, 4]}
d = {**d1}
```

```python
d
```

```python
d1['a'].append(100)
```

```python
d1
```

```python
d
```

At this point you're probably asking yourself, whether to use `**` or `.copy()` to create a shallow copy. We can even create a shallow of one dict by passing the dict to the `dict()` constructor.

Firstly, the `**` unpacking is more flexible because you can unpack multiple dictionaries into a single new one - `copy` is restricted to copying a single dictionary.

But what about timings? Is one faster than the other?

What about using a dictionary comprehension to copy a dictionary? Is that faster/slower?

Let's try it out and see:

```python
from random import randint

big_d = {k: randint(1, 100) for k in range(1_000_000)}
```

```python
def copy_unpacking(d):
    d1 = {**d}
    
def copy_copy(d):
    d1 = d.copy()

def copy_create(d):
    d1 = dict(d)
    
def copy_comprehension(d):
    d1 = {k: v for k, v in d.items()}
```

```python
from timeit import timeit
```

```python
timeit('copy_unpacking(big_d)', globals=globals(), number=100)
```

```python
timeit('copy_copy(big_d)', globals=globals(), number=100)
```

```python
timeit('copy_create(big_d)', globals=globals(), number=100)
```

```python
timeit('copy_comprehension(big_d)', globals=globals(), number=100)
```

So, creating, unpacking and `.copy()` are about the same - certainly not significant enough to be concerned. A comprehension on the other hand is substantially slower - so, don't use comprehension syntax to do a simple shallow copy!
