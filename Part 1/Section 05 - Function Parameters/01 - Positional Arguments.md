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

### Positional Arguments

```python
def my_func(a, b, c):
    print("a={0}, b={1}, c={2}".format(a, b, c))
```

```python
my_func(1, 2, 3)
```

#### Default Values

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))
```

Note that once a parameter is assigned a default value, **all** parameters thereafter **must** be asigned a default value too!


For example, this will not work:

```python
def fn(a, b=2, c):
    print(a, b, c)
```

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))
```

```python
my_func(10, 20, 30)
```

```python
my_func(10, 20)
```

```python
my_func(10)
```

Since **a** does not have a default value, it **must** be specified:

```python
my_func()
```

#### Keyword Arguments (named arguments)


Positional arguments, can **optionally**, be specified using their corresponding parameter name.

This allows us to pass the arguments without using the positional assignment:

```python
def my_func(a, b=2, c=3):
    print("a={0}, b={1}, c={2}".format(a, b, c))
```

```python
my_func(c=30, b=20, a=10)
```

```python
my_func(10, c=30, b=20)
```

Note that once a keyword argument has been used, **all** arguments thereafter **must** also be named:

```python
my_func(10, b=20, 30)
```

However, if a parameter has a default value, it *can* be omitted from the argument list, named or not:

```python
my_func(10, c=30)
```

```python
my_func(a=30, c=10)
```

```python
my_func(c=10, a=30)
```
