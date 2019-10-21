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

### defaultdict


The `defaultdict` is a specialized dictionary found in the `collections` module. (It is a subclass of the `dict` type).

```python
from collections import defaultdict
```

Standard dictionaries in Python will raise an exception if we try to access a non-existent key:

```python
d = {}
```

```python
d['a']
```

Now, we can certainly use the `.get` method:

```python
result = d.get('a')
type(result)
```

And we can even specify a default value for the key if it is not present:

```python
d.get('a', 0)
```

Often we have dictionaries where we want to return a consistent default value if the requested key does not exist.

Although we can do so using the `.get` method as above, we have to remember to use the same default value every time - plus it gets a little cumbersome.

Let's say we want to keep track of the number of occurrences of individual characters in a string.

We might approach it this way:

```python
counts = {}
sentence = "able was I ere I saw elba"

for c in sentence:
    if c in counts:
        counts[c] += 1
    else:
        counts[c] = 1
```

```python
counts
```

So this works, but we have that `if` statement - it would be nice to simplify our code somewhat:

```python
counts = {}
for c in sentence:
    counts[c] = counts.get(c, 0) + 1
```

```python
counts
```

So, that works well and is much cleaner. But if we have to specify that default value (`0` in this case) many times in our code when working with the same dictionary, we have to remember what the default needs to be each time.

Instead, we could use a `defaultdict`. In a `defaultdict` we specify what the default value is for a missing key - more precisely, we specify a default factory method that is called:

```python
counts = defaultdict(lambda : 0)
```

```python
for c in sentence:
    counts[c] += 1
```

```python
counts
```

As you can see that simplified our code quite a bit, but the result is not quite a dictionary - it is a `defaultdict`. However, it inherits from `dict` so all the dictionary methods we have grown to know and love are still available because ` defaultdict` **is** a `dict`:

```python
isinstance(counts, defaultdict)
```

```python
isinstance(counts, dict)
```

And `counts` behaves like a regular dictionary too:

```python
counts.items()
```

```python
counts['a']
```

The main difference is when we request a non-existent key:

```python
counts['python']
```

We get the default value back - not only that, but it actually created that key as well:

```python
counts
```

So this is a bit different from using `.get`.


And of course we can manipulate our dictionary just like a standard dictionary:

```python
counts['hello'] = 'world'
counts
```

```python
del counts['hello']
counts
```

Very often you will see what looks like a **type** specified as the default factory - but keep in mind that it is in fact the corresponding functions (constructors) that are actually being specified.

For example:

```python
int()
```

```python
bool()
```

```python
str()
```

```python
list()
```

```python
d = defaultdict(int)
d['a']
```

```python
d = defaultdict(bool)
d['a']
```

```python
d = defaultdict(str)
d['a']
```

```python
d = defaultdict(list)
d['a']
```

Note that this no different than writing:

```python
d = defaultdict(lambda: list())
d['a']
```

Let's take a look at another example of where a `defaultdict` can be useful.

Suppose we have a dictionary structure that has people's names as keys, and a dictionary for the value that contains the person's eye color. We want to create a dictionary of eye colors, with a list of the people's names that have that eye color:

```python
persons = {
    'john': {'age': 20, 'eye_color': 'blue'},
    'jack': {'age': 25, 'eye_color': 'brown'},
    'jill': {'age': 22, 'eye_color': 'blue'},
    'eric': {'age': 35},
    'michael': {'age': 27}
}
```

What we want is a dictionary with the eye colors (and `unknown` as the key if the eye color was not specified), and the names of the people with that eye color.

Let's first do this without a `defaultdict`, and also not using `.get`:

```python
eye_colors = {}
for person, details in persons.items():
    if 'eye_color' in details:
        color = details['eye_color']
    else:
        color = 'unknown'
    if color in eye_colors:
        eye_colors[color].append(person)
    else:
        eye_colors[color] = [person]
```

```python
eye_colors
```

Now let's simplify this by leveraging the `.get` method:

```python
eye_colors = {}
for person, details in persons.items():
    color = details.get('eye_color', 'Unknown')
    person_list = eye_colors.get(color, [])
    person_list.append(person)
    eye_colors[color] = person_list
```

```python
eye_colors
```

And finally let's use a `defaultdict`:

```python
eye_colors = defaultdict(list)
for person, details in persons.items():
    color = details.get('eye_color', 'Unknown')
    eye_colors[color].append(person)
```

```python
eye_colors
```

When we create a `defaultdict` we have to specify the factory method as the first argument, but thereafter we can specify key/value pairs just like we would with the `dict` constructor (they are basically just passed along to the underlying `dict`):

```python
d = defaultdict(bool, k1=True, k2=False, k3='python')
```

```python
d
```

So, using this, if we had used a `defaultdict` for the Person values, we could simplify our previous example a bit more:

```python
persons = {
    'john': defaultdict(lambda: 'unknown', 
                        age=20, eye_color='blue'),
    'jack': defaultdict(lambda: 'unknown',
                        age=20, eye_color='brown'),
    'jill': defaultdict(lambda: 'unknown',
                        age=22, eye_color='blue'),
    'eric': defaultdict(lambda: 'unknown', age=35),
    'michael': defaultdict(lambda: 'unknown', age=27)
}
```

```python
eye_colors = defaultdict(list)
for person, details in persons.items():
    eye_colors[details['eye_color']].append(person)
```

```python
eye_colors
```

It was a little tedious defining that `defaultdict` for every instance in our `persons` dictionary.

This is a good example of where a **partial** function would be really useful. (I cover partial functions in Part 1 of this series, or you can review the documentation here: https://docs.python.org/3.7/library/functools.html#functools.partial

(You can also just use a lambda function as well)

```python
from functools import partial
```

```python
eyedict = partial(defaultdict, lambda: 'unknown')
```

Alternatively we could also just define it this way:

```python
eyedict = lambda *args, **kwargs: defaultdict(lambda: 'unknown', *args, **kwargs)
```

```python
persons = {
    'john': eyedict(age=20, eye_color='blue'),
    'jack': eyedict(age=20, eye_color='brown'),
    'jill': eyedict(age=22, eye_color='blue'),
    'eric': eyedict(age=35),
    'michael': eyedict(age=27)
}
```

```python
persons
```

And we can use our previous code just as before:

```python
eye_colors = defaultdict(list)
for person, details in persons.items():
    eye_colors[details['eye_color']].append(person)
```

```python
eye_colors
```

Let's look at another example where we use a non-deterministic factory. We could make a database call, an API call, and so on. To keep this simple I'm going to use the current time as my default.

In this example we want to keep track of how many times certain functions are being called, as well as when they were **first** called. To do this I want to be able to decorate the functions I want to keep track of, and I want to be able to specify the dictionary that should be used so I can keep a reference to it so I can examine the results.


```python
from collections import defaultdict, namedtuple
from datetime import datetime
from functools import wraps

def function_stats():
    d = defaultdict(lambda: {"count": 0, "first_called": datetime.utcnow()})
    Stats = namedtuple('Stats', 'decorator data')
    
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            d[fn.__name__]['count'] += 1
            return fn(*args, **kwargs)
        return wrapper
    
    return Stats(decorator, d)        
```

```python
stats = function_stats()
```

```python
dict(stats.data)
```

```python
@stats.decorator
def func_1():
    pass

@stats.decorator
def func_2(x, y):
    pass
```

```python
dict(stats.data)
```

```python
func_1()
```

```python
dict(stats.data)
```

```python
func_1()
```

```python
dict(stats.data)
```

```python
func_2(10, 20)
```

```python
dict(stats.data)
```
