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

### Putting it all Together


Positionals Only: no extra positionals, no defaults (all positionals required)

```python
def func(a, b):
    print(a, b)
```

```python
func('hello', 'world')
```

```python
func(b='world', a='hello')
```

Positionals Only: no extra positionals, defaults (some positionals optional)

```python
def func(a, b='world', c=10):
    print(a, b, c)
```

```python
func('hello')
```

```python
func('hello', c='!')
```

Positionals Only: extra positionals, no defaults (all positionals required)

```python
def func(a, b, *args):
    print(a, b, args)
```

```python
func(1, 2, 'x', 'y', 'z')
```

Note that we cannot call the function this way:

```python
func(b=2, a=1, 'x', 'y', 'z')
```

Keywords Only: no positionals, no defaults (all keyword args required)

```python
def func(*, a, b):
    print(a, b)
```

```python
func(a=1, b=2)
```

Keywords Only: no positionals, some defaults (not all keyword args required)

```python
def func(*, a=1, b):
    print(a, b)
```

```python
func(a=10, b=20)
```

```python
func(b=2)
```

Keywords and Positionals: some positionals (no defaults), keywords (no defaults)

```python
def func(a, b, *, c, d):
    print(a, b, c, d)
```

```python
func(1, 2, c=3, d=4)
```

```python
func(1, 2, d=4, c=3)
```

```python
func(1, c=3, d=4, b=2)
```

Keywords and Positionals: some positional defaults

```python
def func(a, b=2, *, c, d=4):
    print(a, b, c, d)
```

```python
func(1, c=3)
```

```python
func(c=3, a=1)
```

```python
func(1, 2, c=3, d=4)
```

```python
func(c=3, a=1, b=2, d=4)
```

Keywords and Positionals: extra positionals

```python
def func(a, b=2, *args, c=3, d):
    print(a, b, args, c, d)
```

```python
func(1, 2, 'x', 'y', 'z', c=3, d=4)
```

Note that if we are going to use the extra arguments, then we cannot actually use a default value for b:

```python
func(1, 'x', 'y', 'z', c=3, d=4)
```

as you can see, **b** was assigned the value **x**


Keywords and Positionals: no extra positionals, extra keywords

```python
def func(a, b, *, c, d=4, **kwargs):
    print(a, b, c, d, kwargs)
```

```python
func(1, 2, c=3, x=100, y=200, z=300)
```

```python
func(x=100, y=200, z=300, c=3, b=2, a=1)
```

Keywords and Positionals: extra positionals, extra keywords

```python
def func(a, b, *args, c, d=4, **kwargs):
    print(a, b, args, c, d, kwargs)
```

```python
func(1, 2, 'x', 'y', 'z', c=3, d=5, x=100, y=200, z=300)
```

Keywords and Positionals: only extra positionals and extra keywords

```python
def func(*args, **kwargs):
    print(args, kwargs)
```

```python
func(1, 2, 3, x=100, y=200, z=300)
```

#### The Print Function

```python
help(print)
```

```python
print(1, 2, 3)
```

```python
print(1, 2, 3, sep='--')
```

```python
print(1, 2, 3, end='***\n')
```

```python
print(1, 2, 3, sep='\t', end='\t***\t')
print(4, 5, 6, sep='\t', end='\t***\n')
```

#### Another Use Case

```python
def calc_hi_lo_avg(*args, log_to_console=False):
    hi = int(bool(args)) and max(args)
    lo = int(bool(args)) and min(args)
    avg = (hi + lo)/2
    if log_to_console:
        print("high={0}, low={1}, avg={2}".format(hi, lo, avg))
    return avg
```

```python
avg = calc_hi_lo_avg(1, 2, 3, 4, 5)
print(avg)
```

```python
avg = calc_hi_lo_avg(1, 2, 3, 4, 5, log_to_console=True)
print(avg)
```
