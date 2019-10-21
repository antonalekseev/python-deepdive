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

### Callable Class Attributes


Class attributes can be any object type, including callables such as functions:

```python
class Program:
    language = 'Python'
    
    def say_hello():
        print(f'Hello from {Program.language}!')
```

```python
Program.__dict__
```

As we can see, the `say_hello` symbol is in the class dictionary.

We can also retrieve it using either `getattr` or dotted notation:

```python
Program.say_hello, getattr(Program, 'say_hello')
```

And of course we can call it, since it is a callable:

```python
Program.say_hello()
```

```python
getattr(Program, 'say_hello')()
```

We can even access it via the namespace dictionary as well:

```python
Program.__dict__['say_hello']()
```

```python

```
