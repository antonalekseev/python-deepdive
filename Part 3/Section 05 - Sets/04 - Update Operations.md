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

### Update Operations


We can't really update an element of a set - either we remove one or add one - but replacement would not make sense, much like "replacing" a key in a dictionary (we can replace a value, just not a key, and sets are basically like value-less dictionaries).


Let's first consider how we can create new sets from other sets:


* intersection
* union
* difference
* symetric difference


For each of these cases, we can create new sets as follows:

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
print(s1, id(s1))
s1 = s1 & s2
print(s1, id(s1))
```

As you can see, we calculated the intersection of `s1` and `s2` and set `s1` to the result - but this means we ended up with a new object for `s1`.

We may want to **mutate** `s1` instead.
And the samew goes for the other operations mentioned above.

Python provides us a way to do this using both methods and equivalent operators:


* union updates: `s1.update(s2)` or `s1 |= s2`
* intersection updates: `s1.intersection_update(s2)` or `s1 &= s2`
* difference updates: `s1.difference_update(s2)` or `s1 -= s2`
* symm. diff. updates: `s1.symmetric_difference_update(s2)` or `s1 ^= s2`


All these operations **mutate** the original set.


#### Union Updates

```python
s1 = {1, 2, 3}
s2 = {4, 5, 6}
print(id(s1))
s1 |= s2
print(s1, id(s1))
```

```python
s1 = {1, 2, 3}
s2 = {4, 5, 6}
print(id(s1))
s1.update(s2)
print(s1, id(s1))
```

#### Intersection Updates

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
print(id(s1))
s1 &= s2
print(s1, id(s1))
```

```python
s1 = {1, 2, 3}
s2 = {2, 3, 4}
print(id(s1))
s1.intersection_update(s2)
print(s1, id(s1))
```

#### Difference Updates

```python
s1 = {1, 2, 3, 4}
s2 = {2, 3}
print(id(s1))
s1 -= s2
print(s1, id(s1))
```

```python
s1 = {1, 2, 3, 4}
s2 = {2, 3}
print(id(s1))
s1.difference_update(s2)
print(s1, id(s1))
```

Be careful with this one. These two expressions are **NOT** equivalent (this is because difference operations are not associative):

```python
s1 = {1, 2, 3, 4}
s2 = {2, 3}
s3 = {3, 4}
result = s1 - (s2 - s3)
print(result)
s1 -= s2 - s3
print(s1)
```

```python
s1 = {1, 2, 3, 4}
s2 = {2, 3}
s3 = {3, 4}
result = (s1 - s2) - s3
print(result)
s1.difference_update(s2, s3)
print(s1)
```

#### Symmetric Difference Update

```python
s1 = {1, 2, 3, 4, 5}
s2 = {4, 5, 6, 7}
s1 ^ s2
```

```python
s1 = {1, 2, 3, 4, 5}
s2 = {4, 5, 6, 7}
print(id(s1))
s1 ^= s2
print(s1, id(s1))
```

```python
s1 = {1, 2, 3, 4, 5}
s2 = {4, 5, 6, 7}
print(id(s1))
s1.symmetric_difference_update(s2)
print(s1, id(s1))
```

#### Why the methods as well as the operators?


The methods are actually a bit more flexible than the operators.
What happens when we want to update a set from it's union with multiple other sets?
We can certainly do it this way:

```python
s1 = {1, 2, 3}
s2 = {3, 4, 5}
s3 = {5, 6, 7}
```

```python
print(id(s1))
s1 |= s2 | s3
print(s1, id(s1))
```

So this works quite well, but we **have** to use sets.

Using the method we do not have that restriction, we can actually use iterables (they must contain hashable elements) and Python will implicitly convert them to sets:

```python
s1 = {1, 2, 3}
s1.update([3, 4, 5], (6, 7, 8), 'abc')
print(s1)
```

Of course we can achieve the same thing using the operators, it just requires a little more typing:

```python
s1 = {1, 2, 3}
s1 |= set([3, 4, 5]) | set((6, 7, 8)) | set('abc')
print(s1)
```

#### Where might this be useful?


You're hopefully seeing a parallel between these set mutation operations and list mutation operations such as `append` and `extend`.

So the usefullness of mutating a set is no different than the usefullness of mutating a list.

There might be a reason you want to maintain the same object reference - maybe you are writing a function that needs to mutate some set that was passed as an argument.


##### Example 1


Suppose you are writing a function that needs to return all the words found in multiple strings, but with certain words removed (like `'the'`, `'and'`, etc).

You could take this approach:

```python
def combine(string, target):
    target.update(string.split(' '))
```

```python
def cleanup(combined):
    words = {'the', 'and', 'a', 'or', 'is', 'of'}
    combined -= words
```

```python
result = set()
combine('lumberjacks sleep all night', result)
combine('the mistry of silly walks', result)
combine('this parrot is a late parrot', result)
cleanup(result)
print(result)
```

##### Example 2


You may find the above example a little contrived, so let's see another example which might actually prove more practical.

Suppose we have a program that fetches data from some API, database, whatever - and it retrieves a paged list of city names. We want our program to keep fetching data from the source until the source is exhausted, and filter out any cities we are not interested in from our final result.


To simulate the data source, let's do this:

```python
def gen_read_data():
    yield ['Paris', 'Beijing', 'New York', 'London', 'Madrid', 'Mumbai']
    yield ['Hyderabad', 'New York', 'Milan', 'Phoenix', 'Berlin', 'Cairo']
    yield ['Stockholm', 'Cairo', 'Paris', 'Barcelona', 'San Francisco']
```

And we can use this generator this way:

```python
data = gen_read_data()
```

```python
next(data)
```

```python
next(data)
```

```python
next(data)
```

```python
next(data)
```

Next we're going to create a filter that will look at the data just received, removing any cities that match one we want to ignore:

```python
def filter_incoming(*cities, data_set):
    data_set.difference_update(cities)
```

```python
result = set()
data = gen_read_data()
for page in data:
    result.update(page)
    filter_incoming('Paris', 'London', data_set=result)
print(result)
```
