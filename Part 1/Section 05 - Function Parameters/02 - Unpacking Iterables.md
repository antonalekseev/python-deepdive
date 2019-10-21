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

### Unpacking Iterables


#### Side Note on Tuples


This is a tuple:

```python
a = (1, 2, 3)
```

```python
type(a)
```

This is also a tuple:

```python
a = 1, 2, 3
```

```python
type(a)
```

In fact what defines a tuple is not **()**, but the **,** (comma)


To create a tuple with a single element:

```python
a = (1)
```

will not work!!

```python
type(a)
```

Instead, we have to use a comma:

```python
a = (1,)
```

```python
type(a)
```

And in fact, we don't even need the **()**:

```python
a = 1,
```

```python
type(a)
```

The only exception is to create an empty tuple:

```python
a = ()
```

```python
type(a)
```

Or we can use the tuple constructor:

```python
a = tuple()
```

```python
type(a)
```

#### Unpacking


Unpacking is a way to split an iterable object into individual variables contained in a list or tuple: 

```python
l = [1, 2, 3, 4]
```

```python
a, b, c, d = l
```

```python
print(a, b, c, d)
```

Strings are iterables too:

```python
a, b, c = 'XYZ'
print(a, b, c)
```

#### Swapping Two Variables


Here's a quick application of unpacking to swap the values of two variables.


First we look at the "traditional" way you would have to do it in other languages such as Java:

```python
a = 10
b = 20
print("a={0}, b={1}".format(a, b))

tmp = a
a = b
b = tmp
print("a={0}, b={1}".format(a, b))
```

But using unpacking we can simplify this:

```python
a = 10
b = 20
print("a={0}, b={1}".format(a, b))

a, b = b, a
print("a={0}, b={1}".format(a, b))
```

In fact, we can even simplify the initial assignment of values to a and b as follows:

```python
a, b = 10, 20
print("a={0}, b={1}".format(a, b))

a, b = b, a
print("a={0}, b={1}".format(a, b))
```

#### Unpacking Unordered Objects

```python
dict1 = {'p': 1, 'y': 2, 't': 3, 'h': 4, 'o': 5, 'n': 6}
```

```python
dict1
```

```python
for c in dict1:
    print(c)
```

```python
a, b, c, d, e, f = dict1
print(a)
print(b)
print(c)
print(d)
print(e)
print(f)
```

Note that this order is not guaranteed. You can always use an OrderedDict if that is a requirement.


The same applies to sets.

```python
s = {'p', 'y', 't', 'h', 'o', 'n'}
```

```python
type(s)
```

```python
print(s)
```

```python
for c in s:
    print(c)
```

```python
a, b, c, d, e, f = s
```

```python
print(a)
print(b)
print(c)
print(d)
print(e)
print(f)
```
