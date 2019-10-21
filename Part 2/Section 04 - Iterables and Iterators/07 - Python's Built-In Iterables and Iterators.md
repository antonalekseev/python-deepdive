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

### Python's Built-In Iterables and Iterators


Python has a lot of built-in functions that return iterators or iterables.


Let's look at the simple `range` function first:

```python
r_10 = range(10)
```

Now, `r_10` is an **iterable**:

```python
'__iter__' in dir(r_10)
```

But it is not an **iterator**:

```python
'__next__' in dir(r_10)
```

However, we can request an iterator by calling the `__iter__` method, or simply using the `iter()` function:

```python
r_10_iter = iter(r_10)
```

And of course this is now an iterator:

```python
'__iter__' in dir(r_10_iter)
```

```python
'__next__' in dir(r_10_iter)
```

Most built-in iterables in Python use lazy evaluation (including the `range`) function - i.e. when we execute `range(10)` Python does no pre-compute a "list" of all the elements in the range. Instead it uses lazy evluation and the iterator computes and returns elements one at a time.

This is why when we print a range object we do not actually see the contents of the range - they don't exist yet!

Instead, we need to iterate through the iterator and put it into something like a list:

```python
[num for num in range(10)]
```

The `zip` function on the other hand returns an iterator:

```python
z = zip([1, 2, 3], 'abc')
```

```python
z
```

It is an **iterator**:

```python
print('__iter__' in dir(z))
print('__next__' in dir(z))
```

Just like `range()` though, it also uses lazy evaluation, so we need to iterate through the iterator and make a list in order to see the contents:

```python
list(z)
```

Even reading a file line by line is done using lazy evaluation:

```python
with open('cars.csv') as f:
    print(type(f))
    print('__iter__' in dir(f))
    print('__next__' in dir(f))
```

As you can see, the `open()` function returns an **iterator** (of type `TextIOWrapper`), and we can read lines from the file one by one using the `next()` function, or calling the `__next__()` method. The class also implements a `readline()` method we can use to get the next row:

```python
with open('cars.csv') as f:
    print(next(f))
    print(f.__next__())
    print(f.readline())
```

Of course we can just iterate over all the lines using a `for` loop as well:

```python
with open('cars.csv') as f:
    for row in f:
        print(row, end='')
```

The `TextIOWrapper` class also provides a method `readlines()` that will read the entire file and return a list containing all the rows:

```python
with open('cars.csv') as f:
    l = f.readlines()
```

```python
l
```

So you might be wondering which method to use? Use the `readlines()` method, or use the iterator methods?

Especially if you ending up reading the entire file - would one method be better than the other?


Consider this example, where we want to find out all the different origins in the file (last column of each row) - let's do this using both approaches.

```python
origins = set()
with open('cars.csv') as f:
    rows = f.readlines()
for row in rows[2:]:
    origin = row.strip('\n').split(';')[-1]
    origins.add(origin)
print(origins)
```

```python
origins = set()
with open('cars.csv') as f:
    next(f), next(f)
    for row in f:
        origin = row.strip('\n').split(';')[-1]
        origins.add(origin)
print(origins)
```

Now consider the first approach: we loaded the **entire** file into memory (a list), and then iterated through all the rows.

But in the second approach, we still iterated through all the rows, but we only need to store **one row** at a time - the overhead was therefore far smaller.

Often we can process files one row at a time and loading the entire file first, especially for huge files, is not always desirable.


The `enumerate` function is another lazy iterator:

```python
e = enumerate('Python rocks!')
```

```python
print('__iter__' in dir(e))
print('__next__' in dir(e))
```

```python
iter(e)
```

```python
e
```

As we can see, the object and its iterator are the same object.


But `enumerate` is also lazy, so we need to iterate through it in order to recover all the elements:

```python
list(e)
```

Of course, once we have exhausted the iterator, we cannot use it again:

```python
list(e)
```

The dictionary object provides methods that return iterables for the keys, values or tuples of key/value pairs:

```python
d = {'a': 1, 'b': 2}
```

```python
keys = d.keys()
```

```python
'__iter__' in dir(keys), '__next__' in dir(keys)
```

More simply, we can just test to see if `iter(keys)` **is** the same object as `keys` - if not then we are dealing with an iterable.

```python
iter(keys) is keys
```

So we have an iterable.

Similarly for `.values()` and `.items()`:

```python
values = d.values()
iter(values) is values
```

```python
items = d.items()
iter(items) is items
```

There are many other such functions and methods in Python, and we'll cover more of them in some upcoming videos

Just be careful and know whether you are dealing with an iterable or an iterator. You can iterate an iterable over and over again, but can only do so once with an iterator.

```python

```
