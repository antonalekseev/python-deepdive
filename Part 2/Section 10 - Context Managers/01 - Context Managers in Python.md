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

### Context Managers in Python


You should be familiar with `try` and `finally`.

We use the `finally` block to make sure a piece of code is executed, whether an exception has happened or not:

```python
try:
    10 / 2
except ZeroDivisionError:
    print('Zero division exception occurred')
finally:
    print('finally ran!')    
```

```python
try:
    1 / 0
except ZeroDivisionError:
    print('Zero division exception occurred')
finally:
    print('finally ran!')
```

You'll see that in both instances, the `finally` block was executed. Even if an exception is raised in the `except` block, the `finally` block will **still** execute!

Even if the finally is in a function and there is a return statement in the `try` or `except` blocks:

```python
def my_func():
    try:
        1/0
    except:
        return
    finally:
        print('finally running...')
```

```python
my_func()
```

This is very handy to release resources even in cases where an exception occurs. For example making sure a file is closed after being opened:

```python
try:
    f = open('test.txt', 'w')
    a = 1 / 0
except:
    print('an exception occurred...')
finally:
    print('Closing file...')
    f.close()
```

We should **always** do that when dealing with files.

But that can get cumbersome...

So, there is a better way.


Let's talk about context managers, and the pattern we are trying to solve:

1. Run some code to create some object(s)
2. Work with object(s)
3. Run some code when done to clean up object(s)


Context managers do precisely that.

We use a context manager to create and clean up some objects. The key point is that the cleanup needs to happens automatically - we should not have to write code such as the `try...except...finally` code we saw above.


When we use context managers in conjunction with the `with` statement, we end up with the "cleanup" phase happening as soon as the `with` statement finishes:

```python
with open('test.txt', 'w') as file:
    print('inside with: file closed?', file.closed)
print('after with: file closed?', file.closed)
```

This works even in this case:

```python
def test():
    with open('test.txt', 'w') as file:
        print('inside with: file closed?', file.closed)
        return file
```

As you can see, we return directly out of the `with` block...

```python
file = test()
```

```python
file.closed
```

And yet, the file was still closed.

It also works even if we have an exception in the middle of the block:

```python
with open('test.txt', 'w') as f:
    print('inside with: file closed?', f.closed)
    raise ValueError()
```

```python
print('after with: file closed?', f.closed)
```

Context managers can be used for more than just opening and closing files.

If we think about it there are two phases to a context manager:
1. when the `with` statement is executing: we **enter** the context
2. when the `with` block is done: we **exit** the context

We can create our own context manager using a class that implements an `__enter__` method which is executed when we enter the context, and an `__exit__` method that is executed when we exit the context.

There is a general pattern that context managers can help us deal with:
* Open - Close
* Lock - Release
* Change - Reset
* Enter - Exit
* Start - Stop


The `__enter__` method is quite straightforward. It can (but does not have to) return one or more objects we then use inside the `with` block.

The `__exit__` method however is slightly more complicated.

1. It needs to return a boolean True/False. This indicates to Python whether to suppress any errors that occurred in the with block. As we saw with files, that was not the case - i.e. it returns a False
2. If an error does occur in the with block, the error information is passed to the `__exit__` method - so it needs three things: the exception type, the exception value and the traceback. If no error occured, then those values will simply be None.

We haven't covered exceptions in detail yet, so let's quickly see what those three things are:

```python
def my_func():
    return 1.0 / 0.0

my_func()
```

The exception type here is `ZeroDivisionError`.

The exception value is `float division by zero`.

The traceback is an object of type `traceback` (that itself points to other `traceback` objects forming the trace stack) used to generate that text shown in the output.

I am not going to cover `traceback` objects at this point - we'll do this in a future part (OOP) of this series.


Let's go ahead and create a context manager:

```python
class MyContext:
    def __init__(self):
        self.obj = None
        
    def __enter__(self):
        print('entering context...')
        self.obj = 'the Return Object'
        return self.obj

    def __exit__(self, exc_type, exc_value, exc_traceback):
        print('exiting context...')
        if exc_type:
            print(f'*** Error occurred: {exc_type}, {exc_value}')
        return False  # do not suppress exceptions
```

We can even cause an exception inside the `with` block:

```python
with MyContext() as obj:
    raise ValueError
```

As you can see, the `__exit__` method was still called - which is exactly what we wanted in the first place. Also, the exception that was raise inside the `with` block is seen.


We can change that by returning `True` from the `__exit__` method:

```python
class MyContext:
    def __init__(self):
        self.obj = None
        
    def __enter__(self):
        print('entering context...')
        self.obj = 'the Return Object'
        return self.obj

    def __exit__(self, exc_type, exc_value, exc_traceback):
        print('exiting context...')
        if exc_type:
            print(f'*** Error occurred: {exc_type}, {exc_value}')
        return True  # suppress exceptions
```

```python
with MyContext() as obj:
    raise ValueError
print('reached here without an exception...')
```

Look at the output of this code:

```python
with MyContext() as obj:
    print('running inside with block...')
    print(obj)
print(obj)
```

Notice that the `obj` we obtained from the context manager, still exists in our scope after the `with` statement.

The `with` statement does **not** have its own local scope - it's not a function!

However, the context manager could manipulate the object returned by the context manager:

```python
class Resource:
    def __init__(self, name):
        self.name = name
        self.state = None
```

```python
class ResourceManager:
    def __init__(self, name):
        self.name = name
        self.resource = None
        
    def __enter__(self):
        print('entering context')
        self.resource = Resource(self.name)
        self.resource.state = 'created'
        return self.resource
    
    def __exit__(self, exc_type, exc_value, exc_traceback):
        print('exiting context')
        self.resource.state = 'destroyed'
        if exc_type:
            print('error occurred')
        return False
```

```python
with ResourceManager('spam') as res:
    print(f'{res.name} = {res.state}')
print(f'{res.name} = {res.state}')
```

We still have access to `res`, but it's internal state was changed by the resource manager's `__exit__` method.


Although we already have a context manager for files built-in to Python, let's go ahead and write our own anyways - good practice.

```python
class File:
    def __init__(self, name, mode):
        self.name = name
        self.mode = mode
        
    def __enter__(self):
        print('opening file...')
        self.file = open(self.name, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_value, exc_traceback):
        print('closing file...')
        self.file.close()
        return False
```

```python
with File('test.txt', 'w') as f:
    f.write('This is a late parrot!')
```

Even if we have an exception inside the `with` statement, our file will still get closed.

Same applies if we return out of the `with` block if we're inside a function:

```python
def test():
    with File('test.txt', 'w') as f:
        f.write('This is a late parrot')
        if True:
            return f
        print(f.closed)
    print(f.closed)
```

```python
f = test()
```

Note that the `__enter__` method can return anything, including the context manager itself.

If we wanted to, we could re-write our file context manager this way:

```python
class File():
    def __init__(self, name, mode):
        self.name = name
        self.mode = mode
        
    def __enter__(self):
        print('opening file...')
        self.file = open(self.name, self.mode)
        return self
    
    def __exit__(self, exc_type, exc_value, exc_traceback):
        print('closing file...')
        self.file.close()
        return False
```

Of course, now we would have to use the context manager object's `file` property to get a handle to the file:

```python
with File('test.txt', 'r') as file_ctx:
    print(next(file_ctx.file))
    print(file_ctx.name)
    print(file_ctx.mode)
```

```python

```
