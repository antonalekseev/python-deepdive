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

### Coding Exercises - Solution 3


#### Exercise 3

```
You have text data spread across multiple servers.
Each server is able to analyze this data and return a dictionary that contains words and their frequency.

Your job is to combine this data to create a single dictionary that contains all the words and their combined frequencies from all these data sources. Bonus points if you can make your dictionary sorted by frequency (highest to lowest).

For example, you may have three servers that each return these dictionaries:
```

```python
d1 = {'python': 10, 'java': 3, 'c#': 8, 'javascript': 15}
d2 = {'java': 10, 'c++': 10, 'c#': 4, 'go': 9, 'python': 6}
d3 = {'erlang': 5, 'haskell': 2, 'python': 1, 'pascal': 1}
```

Your resulting dictionary should look like this:

```python
d = {'python': 17,
     'javascript': 15,
     'java': 13,
     'c#': 12,
     'c++': 10,
     'go': 9,
     'erlang': 5,
     'haskell': 2,
     'pascal': 1}
```

If only servers 1 and 2 return data (so d1 and d2), your results would look like:

```python
d = {'python': 16,
     'javascript': 15,
     'java': 13,
     'c#': 12,
     'c++': 10, 
     'go': 9}
```

##### Solution


My approach here is to first create a combined dictionary that contains all the keys from all the dictionaries, and adds the values together if the key exists in more than one dictionary.
I do this by looping through all the dictionaries and all the items in each of those dictionaries.
You could do this instead by first getting all the keys and unioning them to create a dictionary with just the keys, but then you would still have to lookup each key in each dictionary to see if it is present - that's three lookups for each key (or as many lookups as we have input dictionaries) - I think it's probably more efficient to just take the first approach I mention.

Then in a second phase, I create a new dictionary based on the one I just created to have it sorted by the value.

```python
d1 = {'python': 10, 'java': 3, 'c#': 8, 'javascript': 15}
d2 = {'java': 10, 'c++': 10, 'c#': 4, 'go': 9, 'python': 6}
d3 = {'erlang': 5, 'haskell': 2, 'python': 1, 'pascal': 1}

def merge(*dicts):
    unsorted = {}
    for d in dicts:
        for k, v in d.items():
            unsorted[k] = unsorted.get(k, 0) + v
            
    # create a dictionary sorted by value
    return dict(sorted(unsorted.items(), key=lambda e: e[1], reverse=True))
```

```python
merged = merge(d1, d2, d3)
for k, v in merged.items():
    print(k, v)
```

```python
merged = merge(d1, d2)
for k, v in merged.items():
    print(k, v)
```

```python

```
