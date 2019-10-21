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

### Coding Exercises - Solution 1


#### Exercise 1


Write a Python function that will create and return a dictionary from another dictionary, but sorted by value. You can assume the values are all comparable and have a natural sort order.


For example, given the following dictionary:

```python
composers = {'Johann': 65, 'Ludwig': 56, 'Frederic': 39, 'Wolfgang': 35}
```

Your function should return a dictionary that looks like the following:

```python
sorted_composers = {'Wolfgang': 35,
                    'Frederic': 39, 
                    'Ludwig': 56,
                    'Johann': 65}
```

Remember if you are using Jupyter notebook to use `print()` to view your dictionary in it's natural ordering (Jupyter will display your dictionary sorted by key).

Also try to keep your code Pythonic - i.e. don't start with an empty dictionary and build it up one key at a time - look for a different, more Pythonic, way of doing it. 

Hint: you'll likely want to use Python's `sorted` function.


##### Solution


My approach here is to sort the `items()` view using Python's `sorted` function and a custom `key` that uses the dictionary values (or second element of each tuple in the `items` view):

```python
composers = {'Johann': 65, 'Ludwig': 56, 'Frederic': 39, 'Wolfgang': 35}

def sort_dict_by_value(d):
    d = {k: v
        for k, v in sorted(d.items(), key=lambda el: el[1])}
    return d
```

```python
print(sort_dict_by_value(composers))
```

Here's a better approach - instead of using a dictionary comprehension, we can simply use the `dict()` function to create a dictionary from the sorted tuples!

```python
def sort_dict_by_value(d):
    return dict(sorted(d.items(), key=lambda el: el[1]))
```

And we end up with the same end result:

```python
sort_dict_by_value(composers)
```

```python

```
