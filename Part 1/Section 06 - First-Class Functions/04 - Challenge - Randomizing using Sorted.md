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

### Challenge: Randomizing an Iterable using Sorted

```python
import random
```

```python
help(random.random)
```

```python
random.random()
```

```python
l = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

```python
sorted(l, key=lambda x: random.random())
```

Of course, this works for any iterable:

```python
sorted('abcdefg', key = lambda x: random.random())
```

And to get a string back instead of just a list:

```python
''.join(sorted('abcdefg', key = lambda x: random.random()))
```
