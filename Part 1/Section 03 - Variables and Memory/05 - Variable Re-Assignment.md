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

## Variable Re-Assignment


Notice how the memory address of **a** is different every time.

```python
a = 10
hex(id(a))
```

```python
a = 15
hex(id(a))
```

```python
a = 5
hex(id(a))
```

```python
a = a + 1
hex(id(a))
```

However, look at this:

```python
a = 10
b = 10
print(hex(id(a)))
print(hex(id(b)))
```

The memory addresses of both **a** and **b** are the same!! 

We'll revisit this in a bit to explain what is going on.
