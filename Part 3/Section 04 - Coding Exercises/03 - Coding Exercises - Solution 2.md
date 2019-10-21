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

### Coding Exercises - Solution 2


#### Exercise 2


Given two dictionaries, `d1` and `d2`, write a function that creates a dictionary that contains only the keys common to both dictionaries, with values being a tuple containg the values from `d1` and `d2`. (Order of keys is not important).


For example, given two dictionaries as follows:

```python
d1 = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
d2 = {'b': 20, 'c': 30, 'y': 40, 'z': 50}
```

Your function should return a dictionary that looks like this:

```python
d = {'b': (2, 20), 'c': (3, 30)}
```

Hint: Remember that `s1 & s2` will return the intersection of two sets.


Again, try to keep your code Pythonic - don't just start with an empty dictionary and build it up one by one - think of a cleaner approach.


##### Solution


My approach here is to use set intersections to find the keys common to both dictionaries.
Then I use a dictionary comprehension to build up my new dictionary, making each value in the new dictionary a tuple containing the values from the original dictionaries:

```python
d1 = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
d2 = {'b': 20, 'c': 30, 'y': 40, 'z': 50}

def intersect(d1, d2):
    d1_keys = d1.keys()
    d2_keys = d2.keys()
    keys = d1_keys & d2_keys
    d = {k: (d1[k], d2[k]) for k in keys}
    return d
```

```python
intersect(d1, d2)
```
