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

### The `object` Class


As we discussed earlier, `object` is a built-in Python **class**, and every class in Python inherits from that class.

```python
type(object)
```

As you can see the type of `object` is `type` - this means it is a class, just like `int`, `str`, `dict` are also classes (types):

```python
type(int), type(str), type(dict)
```

When we create a class that does not explicitly inherit from anything, we are implicitly inheriting from `object`:

```python
class Person:
    pass
```

```python
issubclass(Person, object)
```

And it's not just our custom classes that inherit from `object`, every type in Python does too:

```python
issubclass(int, object)
```

Even modules, which are objects and instances of `module` are subclasses of `object`:

```python
import math
```

```python
type(math)
```

So the `math` module is an instance of the `module` type:

```python
ty = type(math)
```

```python
type(ty)
```

```python
issubclass(ty, object)
```

If you're wondering where the `module` type (class) lives, you can get a reference to it the way I did here, or you can look for it in the `types` module where you can it and the other built-in types.

```python
import types
```

```python
dir(types)
```

For example, if we define a function:

```python
def my_func():
    pass
```

```python
type(my_func)
```

```python
types.FunctionType is type(my_func)
```

And `FunctionType` inherits from `object`:

```python
issubclass(types.FunctionType, object)
```

and of course, instances of that type are therefore also instances of `object`:

```python
isinstance(my_func,  object)
```

as well as being instances of `FunctionType`:

```python
isinstance(my_func, types.FunctionType)
```

The `object` class implements a certain amount of base functionality.

We can see some of them here:

```python
dir(object)
```

So as you can see `object` implements methods such as `__eq__`, `__hash__`, `__repr__` and `__str__`.


Let's investigate some of those, starting with `__repr__` and `__str__`:

```python
o1 = object()
```

```python
str(o1)
```

```python
repr(o1)
```

You probably recognize that output! If we define our own class that does not **override** the `__repr__` or `__str__` methods, when we call those methods on instances of that class it will actually call the implementation in the `object` class:

```python
class Person:
    pass
```

```python
p = Person()
str(p)
```

So this actually called the `__str__` method in the `object` class (but it is an instance method, so it applies to our specific instance `p`).


Similarly, the `__eq__` method in the object class is  implemented, and uses the object **id** to determine equality:

```python
o1 = object()
o2 = object()
```

```python
id(o1), id(o2)
```

```python
o1 is o2, o1 == o2, o1 is o1, o1 == o1
```

So we can use the `==` operator with our custom classes even if we did not implement `__eq__` explicitly - because it inherits it from the `object` class. 

And so we have the same functionality - our custom objects will compare equal only if they are the same object (id):

```python
p1 = Person()
p2 = Person()

p1 is p2, p1 == p2, p1 is p1, p1 == p1
```

We can actually see what specific method is being called by looking at the id of the method in our object, and in the object class:

```python
id(Person.__eq__)
```

```python
id(object.__eq__)
```

See? Same method!


In the same way, we can write classes that do not have `__init__` or `__new__` methods - because they just inherit it from `object`:

```python
id(Person.__init__), id(object.__init__)
```

But of course, if we override those methods, then the `object` methods will not be used:

```python
class Person:
    def __init__(self):
        pass
```

```python
id(Person.__init__), id(object.__init__)
```

Different methods...


We'll look at overriding in more detail next.
