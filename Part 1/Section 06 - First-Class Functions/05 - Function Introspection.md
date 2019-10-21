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

### Function Introspection

```python
def fact(n: "some non-negative integer") -> "n! or 0 if n < 0":
    """Calculates the factorial of a non-negative integer n
    
    If n is negative, returns 0.
    """
    if n < 0:
        return 0
    elif n <= 1:
        return 1
    else:
        return n * fact(n-1)
```

Since functions are objects, we can add attributes to a function:

```python
fact.short_description = "factorial function"
```

```python
print(fact.short_description)
```

We can see all the attributes that belong to a function using the **dir** function:

```python
dir(fact)
```

We can see our **short_description** attribute, as well as some attributes we have seen before: **__annotations__** and **__doc__**:

```python
fact.__doc__
```

```python
fact.__annotations__
```

We'll revisit some of these attributes later in this course, but let's take a look at a few here:

```python
def my_func(a, b=2, c=3, *, kw1, kw2=2, **kwargs):
    pass
```

Let's assign my_func to another variable:

```python
f = my_func
```

The **__name__** attribute holds the function's name:

```python
my_func.__name__
```

```python
f.__name__
```

The **__defaults__** attribute is a tuple containing any positional parameter defaults:

```python
my_func.__defaults__
```

```python
my_func.__kwdefaults__
```

Let's create a function with some local variables:

```python
def my_func(a, b=1, *args, **kwargs):
    i = 10
    b = min(i, b)
    return a * b
```

```python
my_func('a', 100)
```

The **__code__** attribute contains a **code** object:

```python
my_func.__code__
```

This **code** object itself has various properties:

```python
dir(my_func.__code__)
```

Attribute **__co_varnames__** is a tuple containing the parameter names and local variables:

```python
my_func.__code__.co_varnames
```

Attribute **co_argcount** returns the number of arguments (minus any \* and \*\* args)

```python
my_func.__code__.co_argcount
```

#### The **inspect** module


It is much easier to use the **inspect** module!

```python
import inspect
```

```python
inspect.isfunction(my_func)
```

By the way, there is a difference between a function and a method! A method is a function that is bound to some object:

```python
inspect.ismethod(my_func)
```

```python
class MyClass:
    def f_instance(self):
        pass
    
    @classmethod
    def f_class(cls):
        pass
    
    @staticmethod
    def f_static():
        pass
```

**Instance methods** are bound to the **instance** of a class (not the class itself)


**Class methods** are bound to the **class**, not instances


**Static methods** are no bound either to the class or its instances

```python
inspect.isfunction(MyClass.f_instance), inspect.ismethod(MyClass.f_instance)
```

```python
inspect.isfunction(MyClass.f_class), inspect.ismethod(MyClass.f_class)
```

```python
inspect.isfunction(MyClass.f_static), inspect.ismethod(MyClass.f_static)
```

```python
my_obj = MyClass()
```

```python
inspect.isfunction(my_obj.f_instance), inspect.ismethod(my_obj.f_instance)
```

```python
inspect.isfunction(my_obj.f_class), inspect.ismethod(my_obj.f_class)
```

```python
inspect.isfunction(my_obj.f_static), inspect.ismethod(my_obj.f_static)
```

If you just want to know if something is a function or method:

```python
inspect.isroutine(my_func)
```

```python
inspect.isroutine(MyClass.f_instance)
```

```python
inspect.isroutine(my_obj.f_class)
```

```python
inspect.isroutine(my_obj.f_static)
```

We'll revisit this in more detail in section on OOP.


#### Introspecting Callable Code


We can get back the source code of our function using the **getsource()** method:

```python
inspect.getsource(fact)
```

```python
print(inspect.getsource(fact))
```

```python
inspect.getsource(MyClass.f_instance)
```

```python
inspect.getsource(my_obj.f_instance)
```

We can also find out where the function was defined:

```python
inspect.getmodule(fact)
```

```python
inspect.getmodule(print)
```

```python
import math
```

```python
inspect.getmodule(math.sin)
```

```python
# setting up variable
i = 10

# comment line 1
# comment line 2
def my_func(a, b=1):
    # comment inside my_func
    pass
```

```python
inspect.getcomments(my_func)
```

```python
print(inspect.getcomments(my_func))
```

#### Introspecting Callable Signatures

```python
# TODO: Provide implementation
def my_func(a: 'a string', 
            b: int = 1, 
            *args: 'additional positional args', 
            kw1: 'first keyword-only arg', 
            kw2: 'second keyword-only arg' = 10,
            **kwargs: 'additional keyword-only args') -> str:
    """does something
       or other"""
    pass
```

```python
inspect.signature(my_func)
```

```python
type(inspect.signature(my_func))
```

```python
sig = inspect.signature(my_func)
```

```python
dir(sig)
```

```python
for param_name, param in sig.parameters.items():
    print(param_name, param)
```

```python
def print_info(f: "callable") -> None:
    print(f.__name__)
    print('=' * len(f.__name__), end='\n\n')
    
    print('{0}\n{1}\n'.format(inspect.getcomments(f), 
                              inspect.cleandoc(f.__doc__)))
    
    print('{0}\n{1}'.format('Inputs', '-'*len('Inputs')))
    
    sig = inspect.signature(f)
    for param in sig.parameters.values():
        print('Name:', param.name)
        print('Default:', param.default)
        print('Annotation:', param.annotation)
        print('Kind:', param.kind)
        print('--------------------------\n')
        
    print('{0}\n{1}'.format('\n\nOutput', '-'*len('Output')))
    print(sig.return_annotation)
```

```python
print_info(my_func)
```

#### A Side Note on Positional Only Arguments


Some built-in callables have arguments that are positional only (i.e. cannot be specified using a keyword).

However, Python does not currently have any syntax that allows us to define callables with positional only arguments.

In general, the documentation uses a **/** character to indicate that all preceding arguments are positional-only. But not always :-(

```python
help(divmod)
```

Here we see that the **divmod** function takes two positional-only parameters:

```python
divmod(10, 3)
```

```python
divmod(x=10, y=3)
```

Similarly, the string **replace** function also takes positional-only arguments, however, the documentation does not indicate this!

```python
help(str.replace)
```

```python
'abcdefg'.replace('abc', 'xyz')
```

```python
'abcdefg'.replace(old='abc', new='xyz')
```
