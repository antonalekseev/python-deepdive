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

### What is a Module?


A module is simply another data type. And the modules we use are instances of that data type.

```python
import math
```

That word `math` is simply a label (think variable name) in our (global) namespace that points to some object in memory that is the `math` module.


Let's see what is in our global namespace:

```python
globals()
```

```python
globals()['math']
```

```python
type(math)
```

```python
math
```

It's just an object of type `module`, and it even has a memory address:

```python
id(math)
```

Take note of this memory address, we'll want to refer to it later!

Let me show you what happens if I set the `math` **label** to `None` (I could even use `del globals()['math']`:

```python
math = None
```

```python
type(math)
```

```python
id(math)
```

As you can see the label `math` now points to something else.

Let me re-import it:

```python
import math
```

And now we can see:

```python
math
```

```python
id(math)
```

You'll notice that the label `math` now is the **same** memory address as the first time we ran the import.


**NOTE**: Please do not do this in your code. You never what side effects you may encounter - I just showed you this to make a point - when I ran the import the second time, I obtained a label that pointed to the **same** object.


What happens is that when you import a module, it is not actually loaded into the module's namespace only. Instead, the module is loaded into an overarching global system dictionary that contains the module name and the reference to the module object. The name we see here is "copied" into our namespace from that system namespace.


If we had a project with multiple modules that each imported `math`, Python will load the `math` module the first time it is requested and put it into memory.

The next time the `math` module is imported (in some different module), Python always looks at the system modules first - if it is there it simply copies that reference into our module's namespace and sets the label accordingly.


Let's take a look at the system modules:

```python
import sys
```

```python
type(sys.modules)
```

The `sys.modules` currently contains a **lot** of entries, so I'm just going to look at the one we're interested in - the `math` module:

```python
sys.modules['math']
```

Aha! The `sys.modules` dictionary contains a key for `math` and as you saw it is the `math` module. In fact we can look at the memory address once more:

```python
id(sys.modules['math'])
```

Compare that to the `id` of the `math` module in our own (main) module - the same!


Now that we have established that a module is just an instance of the `module` type, and where it lives (in memory) with references to it maintained in the `sys.modules` dictionary as well as in any module namespace that imported it, let's see how we could create a module dynamically!


If it's an object, let's inspect it...

```python
math.__name__
```

```python
math.__dict__
```

Notice how all the methods and "constants" (such as pi) are just members of a dictionary with values being functions or values:

```python
math.sqrt is math.__dict__['sqrt']
```

So, when we write `math.sqrt` we are basically just retrieving the function stored in the `math.__dict__` dictionary at that key (`sqrt`).


Now the `math` module is a little special - it is written in C and actually a built-in.

Let's look at another module from the standard library:

```python
import fractions
```

```python
fractions.__dict__
```

Notice a few properties here that look interesting:

```python
fractions.__file__
```

That's where the `fractions` module source code resides. I am using a virtual environment (conda), and the module `fractions.py` resides in that directory.


So a module is an object that is:
- loaded from file (maybe! we'll see that in a second)
- has a namespace
- is a container of global variables (that `__dict__` we saw)
- is an execution environment (we'll see that in an upcoming video)


Of course, modules are just specific data types, and like any other data type in Python (think classes, functions, etc) we can create them dynamically - they do not have to be loaded from file (though that is how we do it most of the time).

```python
import types
```

```python
isinstance(fractions, types.ModuleType)
```

So, modules are instances of the `ModuleType` class.

```python
help(ModuleType)
```

Let's go ahead and create a new module:

```python
mod = types.ModuleType('point', 'A module for handling points.')
```

```python
mod
```

```python
help(mod)
```

OK, so now let's add some functionality to it by simply setting some attributes:

```python
from collections import namedtuple
mod.Point = namedtuple('Point', 'x y')
```

```python
def points_distance(pt1, pt2):
    return math.sqrt((pt1.x - pt2.x) ** 2 + (pt1.y - pt2.y) ** 2)
```

```python
mod.distance = points_distance
```

```python
mod.__dict__
```

```python
p1 = mod.Point(0, 0)
p2 = mod.Point(1, 1)
```

```python
mod.distance(p1, p2)
```

As you can see it behaves just like an ordinary module.

However, one major difference here is that it is not located in the `sys.modules` dictionary - so another module in our program would not know anything about it.

But we can fix that! We'll see this in one of the next videos.

But first we'll need to take a peek at how Python imports a module from file. COming right up!
