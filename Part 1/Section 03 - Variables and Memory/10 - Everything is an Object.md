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

## Everything is an Object

```python
a = 10
```

**a** is an object of type **int**, i.e. **a** is an instance of the **int** class.

```python
print(type(a))
```

If **int** is a class, we should be able to declare it using standard class instatiation:

```python
b = int(10)
```

```python
print(b)
print(type(b))
```

We can even request the class documentation:

```python
help(int)
```

As we see from the docs, we can even create an **int** using an overloaded constructor:

```python
b = int('10', base=2)
```

```python
print(b)
print(type(b))
```

### Functions are Objects too
---

```python
def square(a):
    return a ** 2
```

```python
type(square)
```

In fact, we can even assign them to a variable:

```python
f = square
```

```python
type(f)
```

```python
f is square
```

```python
f(2)
```

```python
type(f(2))
```

A function can return a function

```python
def cube(a):
    return a ** 3
```

```python
def select_function(fn_id):
    if fn_id == 1:
        return square
    else:
        return cube
```

```python
f = select_function(1)
print(hex(id(f)))
print(hex(id(square)))
print(hex(id(cube)))
print(type(f))
print('f is square: ', f is square)
print('f is cube: ', f is cube)
print(f)
print(f(2))
```

```python
f = select_function(2)
print(hex(id(f)))
print(hex(id(square)))
print(hex(id(cube)))
print(type(f))
print('f is square: ', f is square)
print('f is cube: ', f is cube)
print(f)
print(f(2))
```

We could even call it this way:

```python
select_function(1)(5)
```

A Function can be passed as an argument to another function

(This example is pretty useless, but it illustrates the point effectively)

```python
def exec_function(fn, n):
    return fn(n)
```

```python
result = exec_function(cube, 2)
print(result)
```

We will come back to functions as arguments **many** more times throughout this course!
