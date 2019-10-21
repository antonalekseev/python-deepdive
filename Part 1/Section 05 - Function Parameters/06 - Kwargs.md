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

### **kwargs

```python
def func(**kwargs):
    print(kwargs)
```

```python
func(x=100, y=200)
```

We can also use it in conjunction with **\*args**: 

```python
def func(*args, **kwargs):
    print(args)
    print(kwargs)
```

```python
func(1, 2, a=100, b=200)
```

Note: You cannot do the following:

```python
def func(*, **kwargs):
    print(kwargs)
```

There is no need to even do this, since **\*\*kwargs** essentially indicates no more positional arguments.

```python
def func(a, b, **kwargs):
    print(a)
    print(b)
    print(kwargs)
```

```python
func(1, 2, x=100, y=200)
```

Also, you cannot specify parameters **after** **\*\*kwargs** has been used:

```python
def func(a, b, **kwargs, c):
    pass
```

If you want to specify both specific keyword-only arguments and **\*\*kwargs** you will need to first get to a point where you can define a keyword-only argument (i.e. exhaust the positional arguments, using either **\*args** or just **\***)

```python
def func(*, d, **kwargs):
    print(d)
    print(kwargs)
```

```python
func(d=1, x=100, y=200)
```
