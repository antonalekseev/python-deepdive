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

### Booleans: Precedence and Short-Circuiting

```python
True or True and False
```

this is equivalent, because of ``and`` having higer precedence than ``or``, to:

```python
True or (True and False)
```

This is not the same as:

```python
(True or True) and False
```

#### Short-Circuiting

```python
a = 10
b = 2

if a/b > 2:
    print('a is at least double b')
```

```python
a = 10
b = 0

if a/b > 2:
    print('a is at least double b')
```

```python
a = 10
b = 0

if b and a/b > 2:
    print('a is at least double b')
```

Can also be useful to deal with null or empty strings in a database:

```python
import string
```

```python
help(string)
```

```python
string.digits
```

```python
string.ascii_letters
```

```python
name = ''
if name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = ''
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = None
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = 'Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = '1Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python

```
