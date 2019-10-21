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

### Class Attributes


As we saw, when we create a class Python automatically builds-in properties and behaviors into our class object, like making it callable, and properties like `__name__`.

```python
class Person:
    pass
```

```python
Person.__name__
```

`__name__` is a **class attribute**. We can add our own class attributes easily this way:

```python
class Program:
    language = 'Python'
    version = '3.6'
```

```python
Program.__name__
```

```python
Program.language
```

```python
Program.version
```

Here we used "dotted notation" to access the class attributes. In fact we can also use dotted notation to set the class attribute:

```python
Program.version = '3.7'
```

```python
Program.version
```

But we can also use the functions `getattr` and `setattr` to read and write these attributes:

```python
getattr(Program, 'version')
```

```python
setattr(Program, 'version', '3.6')
```

```python
Program.version, getattr(Program, 'version')
```

Python is a dynamic language, and we can create attributes at run-time, outside of the class definition itself:

```python
Program.x = 100
```

Using dotted notation we added an attribute `x` to the Person class:

```python
Program.x, getattr(Program, 'x')
```

We could also just have used a `setattr` function call:

```python
setattr(Program, 'y', 200)
```

```python
Program.y, getattr(Program, 'y')
```

So where is the state stored? Usually in a dictionary that is attached to the **class** object (often referred to as the **namespace** of the class):

```python
Program.__dict__
```

As you can see that dictionary contains our attributes: `language`, `version`, `x`, `y` with their corresponding current values.


Notice also that `Program.__dict__` does not return a dictionary, but a `mappingproxy` object - this is essentially a read-only dictionary that we cannot modify directly (but we can modify it by using `setattr`, or dotted notation).


For example, if we change the value of an attribute:

```python
setattr(Program, 'x', -10)
```

We'll see this reflected in the underlying dictionary:

```python
Program.__dict__
```

#### Deleting Attributes


So, we can create and mutate class attributes at run-time. Can we delete attributes too?

The answer of course is yes. We can either use the `del` keyword, or the `delattr` function:

```python
del Program.x
```

```python
Program.__dict__
```

```python
delattr(Program, 'y')
```

#### Direct Namespace Access

```python
Program.__dict__
```

Although `__dict__` returns a `mappingproxy` object, it still is a hash map and essentially behaves like a read-only dictionary:

```python
Program.__dict__['language']
```

```python
list(Program.__dict__.items())
```

One word of caution: not every attribute that a class has lives in that dictionary (we'll come back to this later).

For example, you'll notice that the `__name__` attribute is not there:

```python
Program.__name__
```

```python
__name__ in Program.__dict__
```
