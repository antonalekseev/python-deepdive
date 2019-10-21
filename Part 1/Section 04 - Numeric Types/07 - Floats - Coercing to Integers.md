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

### Coercing Floats to Integers


#### Truncation

```python
from math import trunc
```

```python
trunc(10.3), trunc(10.5), trunc(10.6)
```

```python
trunc(-10.6), trunc(-10.5), trunc(-10.3)
```

The **int** constructor uses truncation when a float is passed in:

```python
int(10.3), int(10.5), int(10.6)
```

```python
int(-10.5), int(-10.5), int(-10.4)
```

#### Floor

```python
from math import floor
```

```python
floor(10.4), floor(10.5), floor(10.6)
```

```python
floor(-10.4), floor(-10.5), floor(-10.6)
```

#### Ceiling

```python
from math import ceil
```

```python
ceil(10.4), ceil(10.5), ceil(10.6)
```

```python
ceil(-10.4), ceil(-10.5), ceil(-10.6)
```

```python

```
