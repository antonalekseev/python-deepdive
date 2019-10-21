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

### The `__del__` Method


The `__del__` method as we discussed in the lecture is called right before the object is about to be garbage collected. This is sometimes called the **finalizer**. It is sometimes referred to as the **destructor**, but that's not really accurate since that method does not destroy the object - that's the GC's responsibility - `__del__` just gets called prior to the GC destroying the object.


Although this method can be useful in some circumstances we need to be aware of some pitfalls:


1. Using the `del` keyword does not call `__del__` directly - it just removes the symbol for wehatever namespace it is being deleted from and reduces the reference count by 1.
2. The `__del__` method is not called until the object is about to be destroyed - so using `del obj` decreases the ref count by 1, but if something else is referencing that object then `__del__` is **not** called.
3. Unhandled exceptions that occur in the `__del__` method are essentially ignored, and the exceptions are written to `sys.stderr`.


It's actually pretty easy to have unwitting references to an object.


Let's first write a small helper function to calculate the reference count for an object using it's memory address (which only works correctly if the object actually exists):

```python
import ctypes

def ref_count(address):
    return ctypes.c_long.from_address(address).value
```

Now let's write a class that implements the `__del__` method:

```python
class Person:
    def __init__(self, name):
        self.name = name
        
    def __repr__(self):
        return f'Person({self.name})'
    
    def __del__(self):
        print(f'__del__ called for {self}...')
```

Let's first see how the `__del__` gets called when we create then remove a reference to an instance in our global scope:

```python
p = Person('Alex')
```

We can now remove that reference from the symbol `p` to the instance either by using `del p` or even just setting `p` to `None`:

```python
p = None
```

As you can see the `__del__` was called.

It works the same way with the `del` statement:

```python
p = Person('Alex')
```

```python
del p
```

Now let's see how we might create an unwitting extra reference to the object.

Let's implement a method that is going to create an exception:

```python
class Person:
    def __init__(self, name):
        self.name = name
    
    def gen_ex(self):
        raise ValueError('Something went bump...')
        
    def __repr__(self):
        return f'Person({self.name})'
    
    def __del__(self):
        print(f'__del__ called for {self}...')
```

```python
p = Person('Alex')
```

At this point we have one reference to the object, the reference held by `p`:

```python
p_id = id(p)
ref_count(p_id)
```

Now let's make that exception happen and store the exception in a variable:

```python
try:
    p.gen_ex()
except ValueError as ex:
    error = ex
    print(ex)
```

```python
ref_count(p_id)
```

As you can see our reference count is now `2`. Why?


Let's look at the `error` variable:

```python
dir(error)
```

```python
dir(error.__traceback__)
```

```python
dir(error.__traceback__.tb_frame)
```

```python
for key, value in error.__traceback__.tb_frame.f_locals.copy().items():
    if isinstance(value, Person):
        print(key, value, id(value), id(key))
```

As you can see the traceback contains a refererence to our object in it's dictionary - so we have a second reference to our object.


Let's check our reference count now, to make sure we did not inadvertently create even more references:

```python
ref_count(p_id)
```

Now, even if we remove our reference to the object, we will still have something handing on to it, and the `__del__` method will not get called:

```python
del p
```

See! `__del__` was not called!


But now let's get rid of that exception we stored:

```python
del error
```

And now, as you can see, we finally had the `__del__` method called. (Note that depending on what you were doing in your notebook, you may not even see this call at all - which just means that something else is holding on to our object somewhere!)


For this reason it is rare for devs to use the `__del__` method for critical things like closing a file, or closing committing a transaction in a database, etc - instead use a context manager, and avoid using the `__del__` method.


Because you do not know when the `__del__` method is going to get called (unless you know exactly how your code might be creating references to the object), you could also get into a situation where other objects (like global objects) referenced in the `__del__` method will even still be around by the time `__del__` is called (it would get called when the module is destroyed, such as at program shutdown).


The last point to make about `__del__` is that any unhandled exceptions in the `__del__` method are essentially ignored by Python (although their output is sent to `sys.stderr`).


Let's see this:

```python
class Person:
    def __del__(self):
        raise ValueError('Something went bump...')
```

```python
p = Person()
```

```python
del p
```

What we are seeing here is actually the `stderr` output, which Jupyter redirects into our notebook.

```python
import sys
```

```python
sys.stderr, sys.stdout
```

What I'm going to do here is redirect `stderr` to a file instead, using a context manager:

```python
class ErrToFile:
    def __init__(self, fname):
        self._fname = fname
        self._current_stderr = sys.stderr
        
    def __enter__(self):
        self._file = open(self._fname, 'w')
        sys.stderr = self._file
        
    def __exit__(self, exc_type, exc_value, exc_tb):
        sys.stderr = self._current_stderr
        if self._file:
            self._file.close()
        return False
```

```python
p = Person()
```

```python
with ErrToFile('err.txt'):
    del p
```

As you can see, no exception was generated and our code continues to run happily along.

But let's examine the contents of that file:

```python
with open('err.txt') as f:
    print(f.readlines())
```

So, as you can see the exception was silenced and the exception data was just sent to `stderr`.


What this means is that you cannot trap exceptions that occur in the `__del__` method (from outside the `__del__` method to be exact):

```python
p = Person()

try:
    del p
    print('p was deleted (succesfully)')
except ValueError as ex:
    print('Exception caught!')
else:
    print('No exception seen!')
```

Now all this does not mean you should just altogether avoid using the `__del__` method - you just need to be aware of its limitations, and be extra careful in your code with circular references or unintentional extra references to your objects.
Things get even dicier when using multi-threading, but that's beyond the scope of this course!


Personally I never use `__del__`. Instead I use context managers to manage releasing resources such as files, sockets, database connections, etc.
