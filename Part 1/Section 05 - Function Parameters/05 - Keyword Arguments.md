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

### Keyword Arguments


Recall: positional parameters defined in functions can also be passed as named (keyword) arguments.

```python
def func1(a, b, c):
    print(a, b, c)
```

```python
func1(10, 20, 30)
```

```python
func1(b=20, c=30, a=10)
```

```python
func1(10, c=30, b=20)
```

Using a named argument is optional and up to the caller.


What if we wanted to force calls to our function to use named arguments?


We can do so by **exhausting** all the positional arguments, and then adding some additional parameters in teh function definition:

```python
def func1(a, b, *args, d):
    print(a, b, args, d)
```

Now we will need at least two positional arguments, an optional (possibly even zero) number of additional arguments, and this extra argument which is supposed to go into **d**. This argument can **only** be passed to the function using a named (keyword) argument:


So, this will not work:

```python
func1(10, 20, 'a', 'b', 100)
```

But this will:

```python
func1(10, 20, 'a', 'b', d=100)
```

As you can see, **d** took the keyword argument, while the remaining arguments were handled as positional parameters.


We can even define a function that has only optional positional arguments and mandatory keyword arguments:

```python
def func1(*args, d):
    print(args)
    print(d)
```

```python
func1(1, 2, 3, d='hello')
```

We can of course, not pass any positional arguments:

```python
func1(d='hello')
```

but the positional argument is mandatory (since no default was provided in the function definition):

```python
func1()
```

To make the keyword argument optional, we just need to specify a default value in the function definition:

```python
def func1(*args, d='n/a'):
    print(args)
    print(d)
```

```python
func1(1, 2, 3)
```

```python
func1()
```

Sometimes we want **only** keyword arguments, in which case we still have to exhaust the positional arguments first - but we can use the following syntax if we do not want any positional parameters passed in:

```python
def func1(*, d='hello'):
    print(d)
```

```python
func1(10, d='bye')
```

```python
func1(d='bye')
```

Of course, if we do not provide a default value for the keyword argument, then we effectively are forcing the caller to provide the keyword argument:

```python
def func1(*, a, b):
    print(a)
    print(b)
```

```python
func1(a=10, b=20)
```

but, the following would not work:

```python
func1(10, 20)
```

Unlike positional parameters, keyword arguments do not have to be defined with non-defaulted and then defaulted arguments:

```python
def func1(a, *, b='hello', c):
    print(a, b, c)
```

```python
func1(5, c='bye')
```

We can also include positional non-defaulted (first), positional defaulted (after positional non-defaulted) followed lastly (after exhausting positional arguments) by keyword args (defaulted or non-defaulted in any order)

```python
def func1(a, b=20, *args, d=0, e='n/a'):
    print(a, b, args, d, e)
```

```python
func1(5, 4, 3, 2, 1, d=0, e='all engines running')
```

```python
func1(0, 600, d='goooood morning', e='python!')
```

```python
func1(11, 'm/s', 24, 'mph', d='unladen', e='swallow')
```

As you can see, defining parameters and passing arguments is extremely flexible in Python! Even more so, when you account for the fact that the parameters are not statically typed!


In the next video, we'll look at one more thing we can do with function parameters!
