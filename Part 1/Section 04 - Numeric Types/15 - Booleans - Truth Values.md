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

### Booleans: Truth Values


All objects in Python have an associated **truth value**, or **truthyness**


We saw in a previous lecture that integers have an inherent truth value:

```python
bool(0)
```

```python
bool(1), bool(-1), bool(100)
```

This truthyness has nothing to do with the fact that **bool** is a subclass of **int**.

Instead, it has to do with the fact that the **int** class implements a `__bool__()` method:

```python
help(bool)
```

If you scroll down in the documentation you shoudl reach a section that looks like this:


`` 
|  __bool__(self, /)
|      self != 0
``


So, when we write:

```python
bool(100)
```

Python is actually calling 100.__bool__() and returning that:

```python
(100).__bool__()
```

```python
(0).__bool__()
```

Most objects will implement either the `__bool__()` or `__len__()` methods. If they don't, then their associated value will be **True** always.


#### Numeric Types


Any non-zero numeric value is truthy. Any zero numeric value is falsy:

```python
from fractions import Fraction
from decimal import Decimal
bool(10), bool(1.5), bool(Fraction(3, 4)), bool(Decimal('10.5'))
```

```python
bool(0), bool(0.0), bool(Fraction(0,1)), bool(Decimal('0')), bool(0j)
```

#### Sequence Types


An empty sequence type object is Falsy, a non-empty one is truthy:

```python
bool([1, 2, 3]), bool((1, 2, 3)), bool('abc'), bool(1j)
```

```python
bool([]), bool(()), bool('')
```

#### Mapping Types


Similarly, an empty mapping type will be falsy, a non-empty one truthy:

```python
bool({'a': 1}), bool({1, 2, 3})
```

```python
bool({}), bool(set())
```

#### The None Object


The singleton **None** object is always falsy:

```python
bool(None)
```

#### One Application of Truth Values


Any conditional expression which involves objects other than **bool** types, will use the associated truth value as the result of the conditional expression.

```python
a = [1, 2, 3]
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

```python
a = []
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

```python
a = 'abc'
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

```python
a = ''
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

We could write this using a more lengthy expression:

```python
a = 'abc'
if a is not None and len(a) > 0:
    print(a[0])
else:
    print('a is None, or a is empty')
```

Doing the following would break our code in some instances:

```python
a = 'abc'
if a is not None:
    print(a[0])
```

works, but:

```python
a = ''
if a is not None:
    print(a[0])
```

```
or even:
```

```python
a = None
if len(a) > 0:
    print(a[0])
```

To be torough we would need to write:

```python
a = None
if a is not None and len(a) > 0:
    print(a[0])
```

Also, the order of the boolean expressions matter here!

We'll discuss this and short-circuit evaluations in an upcoming video.

For example:

```python
a = None
if len(a) > 0 and a is not None:
    print(a[0])
```
