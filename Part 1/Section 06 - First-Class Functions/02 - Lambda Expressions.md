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

### Lambda Expressions

```python
lambda x: x**2
```

As you can see, the above expression just created a function.


#### Assigning to a Variable

```python
func = lambda x: x**2
```

```python
type(func)
```

```python
func(3)
```

We can specify arguments for lambdas just like we would for any function created using **def**, except for annotations:

```python
func_1 = lambda x, y=10: (x, y)
```

```python
func_1(1, 2)
```

```python
func_1(1)
```

We can even use \* and \*\*:

```python
func_2 = lambda x, *args, y, **kwargs: (x, *args, y, {**kwargs})
```

```python
func_2(1, 'a', 'b', y=100, a=10, b=20)
```

#### Passing as an Argument


Lambdas are functions, and can therefore be passed to any other function as an argument (or returned from another function)

```python
def apply_func(x, fn):
    return fn(x)
```

```python
apply_func(3, lambda x: x**2)
```

```python
apply_func(3, lambda x: x**3)
```

Of course we can make this even more generic:

```python
def apply_func(fn, *args, **kwargs):
    return fn(*args, **kwargs)
```

```python
apply_func(lambda x, y: x+y, 1, 2)
```

```python
apply_func(lambda x, *, y: x+y, 1, y=2)
```

```python
apply_func(lambda *args: sum(args), 1, 2, 3, 4, 5)
```

Of course in the example above, we really did not need to create a lambda!

```python
apply_func(sum, (1, 2, 3, 4, 5))
```

Of course, we don't have to use lambdas when calling **apply_func**, we can also pass in a function defined using a **def** statement:

```python
def multiply(x, y):
    return x * y
```

```python
apply_func(multiply, 'a', 5)
```

```python
apply_func(lambda x, y: x*y, 'a', 5)
```
