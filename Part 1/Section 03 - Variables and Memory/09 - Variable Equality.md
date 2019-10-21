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

## Variable Equality


From the previous lecture we know that **a** and **b** will have a **shared** reference:

```python
a = 10
b = 10

print(hex(id(a)))
print(hex(id(b)))
```

When we use the **is** operator, we are comparing the memory address **references**:

```python
print("a is b: ", a is b)
```

But if we use the **==** operator, we are comparing the **contents**:

```python
print("a == b:", a == b)
```

The following however, do not have a shared reference:

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(hex(id(a)))
print(hex(id(b)))
```

Although they are not the same objects, they do contain the same "values":

```python
print("a is b: ", a is b)
print("a == b", a == b)
```

Python will attempt to compare values as best as possible, for example:

```python
a = 10
b = 10.0
```

These are **not** the same reference, since one object is an **int** and the other is a **float**

```python
print(type(a))
print(type(b))
```

```python
print(hex(id(a)))
print(hex(id(b)))
```

```python
print('a is b:', a is b)
print('a == b:', a == b)
```

So, even though *a* is an integer 10, and *b* is a float 10.0, the values will still compare as equal.


In fact, this will also have the same behavior:

```python
c = 10 + 0j
print(type(c))
```

```python
print('a is c:', a is c)
print('a == c:', a == c)
```

### The None Object
----


**None** is a built-in "variable" of type *NoneType*.

Basically the keyword **None** is a reference to an object instance of *NoneType*.

NoneType objects are immutable! Python's memory manager will therefore use shared references to the None object.

```python
print(None)
```

```python
hex(id(None))
```

```python
type(None)
```

```python
a = None
print(type(a))
print(hex(id(a)))
```

```python
a is None
```

```python
a == None
```

```python
b = None
hex(id(b))
```

```python
a is b
```

```python
a == b
```

```python
l = []
```

```python
type(l)
```

```python
l is None
```

```python
l == None
```
