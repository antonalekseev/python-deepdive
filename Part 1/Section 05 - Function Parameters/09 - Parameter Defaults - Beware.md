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

### Default Values - Beware!

```python
from datetime import datetime
```

```python
print(datetime.utcnow())
```

```python
def log(msg, *, dt=datetime.utcnow()):
    print('{0}: {1}'.format(dt, msg))
```

```python
log('message 1')
```

```python
log('message 2', dt='2001-01-01 00:00:00')
```

```python
log('message 3')
```

```python
log('message 4')
```

As you can see, the default for **dt** is calculated when the function is **defined** and is **NOT** re-evaluated when the function is called.


#### Solution Pattern


Here is one pattern we can use to achieve the desired result:

We actually set the default to None - this makes the argument optional, and we can then test for None **inside** the function and default to the current time if it is None.

```python
def log(msg, *, dt=None):
    dt = dt or datetime.utcnow()
    # above is equivalent to:
    #if not dt:
    #    dt = datetime.utcnow()
    print('{0}: {1}'.format(dt, msg))    
```

```python
log('message 1')
```

```python
log('message 2')
```

```python
log('message 3', dt='2001-01-01 00:00:00')
```

```python
log('message 4')
```
