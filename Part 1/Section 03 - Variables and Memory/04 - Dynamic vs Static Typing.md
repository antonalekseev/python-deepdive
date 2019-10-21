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

### Dynamic Typing


Python is dunamically typed.

This means that the type of a variable is simply the type of the object the variable name points to (references). The variable itself has no associated type.

```python
a = "hello"
```

```python
type(a)
```

```python
a = 10
```

```python
type(a)
```

```python
a = lambda x: x**2
```

```python
a(2)
```

```python
type(a)
```

As you can see from the above examples, the type of the variable ``a`` changed over time - in fact it was simply the type of the object ``a`` was referencing at that time. No type was ever attached to the variable name itself.

```python

```
