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

### Class Body Scope


The class body is a scope and therefore has it's own namespace. Inside that scope we can reference symbols like we would within any other scope:

```python
class Language:
    MAJOR = 3
    MINOR = 7
    REVISION = 4
    FULL = '{}.{}.{}'.format(MAJOR, MINOR, REVISION)
```

```python
Language.FULL
```

However, functions defined inside the class are not nested in the body scope - instead they are nested in whatever scope the class itself is in.


This means that we cannot reference the class symbols inside a function without also telling Python where to look for it:

```python
class Language:
    MAJOR = 3
    MINOR = 7
    REVISION = 4
    
    @property
    def version(self):
        return '{}.{}.{}'.format(self.MAJOR, self.MINOR, self.REVISION)
    
    @classmethod
    def cls_version(cls):
        return '{}.{}.{}'.format(cls.MAJOR, cls.MINOR, cls.REVISION)
    
    @staticmethod
    def static_version():
        return '{}.{}.{}'.format(Language.MAJOR, Language.MINOR, Language.REVISION)
```

```python
l = Language()
l.version
```

```python
Language.cls_version()
```

```python
Language.static_version()
```

Basically think that the function symbols are in the class body namespace, but the functions themselves are defined externally to the class - just as if we had written it this way:

```python
def full_version():
 return '{}.{}.{}'.format(Language.MAJOR, Language.MINOR, Language.REVISION)
```

```python
full_version()
```

So writing something like this will not work:

```python
class Language:
    MAJOR = 3
    MINOR = 7
    REVISION = 4
    
    @classmethod
    def cls_version(cls):
        return '{}.{}.{}'.format(MAJOR, MINOR, REVISION)
```

```python
Language.cls_version()
```

This behavior can lead to subtle bugs if we aren't careful. 


What happens if the names `MAJOR`, `MINOR` and `REVISION` **are** defined in the enclosing scope?

```python
MAJOR = 0
MINOR = 0
REVISION = 1
```

```python
Language.cls_version()
```

See what happened?!!


Now of course, the nested scopes follow the same usual rules, so we could technically have something like this:

```python
MAJOR = 0
MINOR = 0
REVISION = 1

def gen_class():
    MAJOR = 0
    MINOR = 4
    REVISION = 2
    
    class Language:
        MAJOR = 3
        MINOR = 7
        REVISION = 4

        @classmethod
        def version(cls):
            return '{}.{}.{}'.format(MAJOR, MINOR, REVISION)
        
    return Language
```

```python
cls = gen_class()
```

```python
cls.version()
```

Notice how the scope of `version` was nested inside `gen_class` which itself is nested in the `global` scope.

When we called the `version` method, it found the `MAJOR`, `MINOR` and `REVISION` in the closest enclosing scope - which turned out to be the `gen_class` scope.

This means by the way, that `version` is not only a method, but actually a closure.

```python
import inspect
```

```python
inspect.getclosurevars(cls.version)
```

This last example of "unexpected" behavior I want to show you was show to me by a friend who was puzzled by it:

```python
name = 'Guido'

class MyClass:
    name = 'Raymond'
    list_1 = [name] * 3
    list_2 = [name.upper() for i in range(3)]
    
    @classmethod
    def hello(cls):
        return '{} says hello'.format(name)
```

```python
MyClass.list_1
```

Since the expression `[name] * 3` lives in the class body, it uses `name` that it finds in the class namespace.

```python
MyClass.hello()
```

Here, `name` is used inside a function, so the closest `name` symbol is the one in the module/global scope. Hence we see that `Guido` was used.

```python
MyClass.list_2
```

That one is more puzzling... Why is the expression `[name.upper() for i in range(3)]` using `name` from the enclosing (module/global) scope, and not the one from the class namespace like `[name] * 3` did?


Remember what we discussed about comprehensions?


They are essentially thinly veiled **functions**!!!


So they behave like a function would, and therefore are not nested in the class body scope, but, in this case, in the module/global scope!
