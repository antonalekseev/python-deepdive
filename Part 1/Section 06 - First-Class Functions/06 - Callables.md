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

### Callables


A callable is an object that can be called (using the **()** operator), and always returns a value.

We can check if an object is callable by using the built-in function **callable**


##### Functions and Methods are callable

```python
callable(print)
```

```python
callable(len)
```

```python
l = [1, 2, 3]
callable(l.append)
```

```python
s = 'abc'
callable(s.upper)
```

##### Callables **always** return a value:

```python
result = print('hello')
print(result)
```

```python
l = [1, 2, 3]
result = l.append(4)
print(result)
print(l)
```

```python
s = 'abc'
result = s.upper()
print(result)
```

##### Classes are callable:

```python
from decimal import Decimal
```

```python
callable(Decimal)
```

```python
result = Decimal('10.5')
print(result)
```

##### Class instances may be callable:

```python
class MyClass:
    def __init__(self):
        print('initializing...')
        self.counter = 0
    
    def __call__(self, x=1):
        self.counter += x
        print(self.counter)
```

```python
my_obj = MyClass()
```

```python
callable(my_obj.__init__)
```

```python
callable(my_obj.__call__)
```

```python
my_obj()
```

```python
my_obj()
```

```python
my_obj(10)
```
