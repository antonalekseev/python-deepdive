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

## Python Optimizations: Interning


Earlier, we saw shared references being created automatically by Python:

```python
a = 10
b = 10
print(id(a))
print(id(b))
```

Note how `a` and `b` reference the same object.

But consider the following example:

```python
a = 500
b = 500
print(id(a))
print(id(b))
```

As you can see, the variables `a` and `b` do **not** point to the same object!

This is because Python pre-caches integer objects in the range [-5, 256]


So for example:

```python
a = 256
b = 256
print(id(a))
print(id(b))
```

and

```python
a = -5
b = -5
print(id(a))
print(id(b))
```

do have the same reference.

This is called **interning**: Python **interns** the integers in the range [-5, 256].


The integers in the range [-5, 256] are essentially **singleton** objects.

```python
a = 10
b = int(10)
c = int('10')
d = int('1010', 2)
```

```python
print(a, b, c, d)
```

```python
a is b
```

```python
a is c
```

```python
a is d
```

As you can see, all these variables were created in different ways, but since the integer object with value 10 behaves like a singleton, they all ended up pointing to the **same** object in memory.
