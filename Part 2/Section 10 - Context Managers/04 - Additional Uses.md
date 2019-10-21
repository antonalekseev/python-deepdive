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

### Additional Uses


Remember what I said in the last lecture about some common patterns we can implement with context managers:

* Open - Close
* Change - Reset
* Start - Stop


The open file context manager is an example of the **Open - Close** pattern. But we have other ones as well.


#### Decimal Contexts


Decimals have a context which can be used to define many things, such as precision, rounding mechanism, etc.

By default, Decimals have a "global" context - i.e. one that will apply to any Decimal object by default:

```python
import decimal
```

```python
decimal.getcontext()
```

If we create a decimal object, then it will use those settings.

We can certainly change the properties of that global context:

```python
decimal.getcontext().prec=14
```

```python
decimal.getcontext()
```

And now the default (global) context has a precision set to 14.

Let's reset it back to 28:

```python
decimal.getcontext().prec = 28
```

Suppose now that we just want to temporarily change something in the context - we would have to do something like this:

```python
old_prec = decimal.getcontext().prec
decimal.getcontext().prec = 4
print(decimal.Decimal(1) / decimal.Decimal(3))
decimal.getcontext().prec = old_prec
print(decimal.Decimal(1) / decimal.Decimal(3))
```

Of course, this is kind of a pain to have to store the current value, set it to something new, and then remember to set it back to it's original value.

How about writing a context manager to do that seamlessly for us:

```python
class precision:
    def __init__(self, prec):
        self.prec = prec
        self.current_prec = decimal.getcontext().prec
        
    def __enter__(self):
        decimal.getcontext().prec = self.prec
        
    def __exit__(self, exc_type, exc_value, exc_traceback):
        decimal.getcontext().prec = self.current_prec
        return False      
        
```

Now we can do this:

```python
with precision(3):
    print(decimal.Decimal(1) / decimal.Decimal(3))
print(decimal.Decimal(1) / decimal.Decimal(3))    
```

And as you can see, the precision was set back to it's original value once the context was exited.


In fact, the decimal class already has a context manager, and it's way better than ours, because we can set not only the precision, but anything else we want:

```python
with decimal.localcontext() as ctx:
    ctx.prec = 3
    print(decimal.Decimal(1) / decimal.Decimal(3))
print(decimal.Decimal(1) / decimal.Decimal(3))
```

So this is an example of using a context manager for a **Change - Reset** type of situation.


#### Timing a with block


Here's another example of a **Start - Stop** type of context manager.
We'll create a context manager to time the code inside the `with` block:

```python
from time import perf_counter, sleep

class Timer:
    def __init__(self):
        self.elapsed = 0
        
    def __enter__(self):
        self.start = perf_counter()
        return self
    
    def __exit__(self, exc_type, exc_value, exc_traceback):
        self.stop = perf_counter()
        self.elapsed = self.stop - self.start
        return False
```

You'll note that this time we are returning the context manager itself from the `__enter__` statement. This will allow us to look at the `elapsed` property of the context manager once the `with` statement has finished running.

```python
with Timer() as timer:
    sleep(1)
print(timer.elapsed)
```

#### Redirecting stdout


Here we are going to temporarily redirect `stdout` to a file instead fo the console:

```python
import sys

class OutToFile:
    def __init__(self, fname):
        self._fname = fname
        self._current_stdout = sys.stdout
        
    def __enter__(self):
        self._file = open(self._fname, 'w')
        sys.stdout = self._file
        
    def __exit__(self, exc_type, exc_value, exc_tb):
        sys.stdout = self._current_stdout
        if self._file:
            self._file.close()
        return False
```

```python
with OutToFile('test.txt'):
    print('Line 1')
    print('Line 2')
```

As you can see, no output happened on the console... Instead the output went to the file we specified. And our print statements will now output to the console again:

```python
print('back to console output')
```

```python
with open('test.txt') as f:
    print(f.readlines())
```

#### HTML Tags


In this example, we're going to basically use a context manager to inject opening and closing html tags as we print to the console (of course we could redirect our prints somewhere else as we just saw!):

```python
class Tag:
    def __init__(self, tag):
        self._tag = tag
        
    def __enter__(self):
        print(f'<{self._tag}>', end='')
        
    def __exit__(self, exc_type, exc_value, exc_tb):
        print(f'</{self._tag}>', end='')
        return False
```

```python
with Tag('p'):
    print('some ', end='')
    with Tag('b'):
        print('bold', end='')
    print(' text', end='')
```

#### Re-entrant Context Managers


We can also write context managers that can be re-entered in the sense that we can call `__enter__` and `__exit__` more than once on the **same** context manager. 

These methods are called when a `with` statement is used, so we'll need to be able to get our hands on the context manager object itself - but that's easy, we just return `self` from the `__enter__` method.


Let's write a ListMaker context manager to do see how this works.

```python
class ListMaker:
    def __init__(self, title, prefix='- ', indent=3):
        self._title = title
        self._prefix = prefix
        self._indent = indent
        self._current_indent = 0
        print(title)
        
    def __enter__(self):
        self._current_indent += self._indent
        return self
    
    def __exit__(self, exc_type, exc_value, exc_tb):
        self._current_indent -= self._indent
        return False
        
    def print(self, arg):
        s = ' ' * self._current_indent + self._prefix + str(arg)
        print(s)
```

Because `__enter__` is returning `self`, the instance of the context manager, we can call `with` on that context manager and it will automatically call the `__enter__` and `__exit__` methods. Each time we run `__enter__` we increase the indentation, each time we run `__exit__` we decrease the indentation.

Our `print` method then takes that into account when it prints the requested string argument.

```python
with ListMaker('Items') as lm:
    lm.print('Item 1')
    with lm:
        lm.print('item 1a')
        lm.print('item 1b')
    lm.print('Item 2')
    with lm:
        lm.print('item 2a')
        lm.print('item 2b')
    
```

Of course, we can easily redirect the output to a file instead, using the context manager we wrote earlier:

```python
with OutToFile('my_list.txt'):
    with ListMaker('Items') as lm:
        lm.print('Item 1')
        with lm:
            lm.print('item 1a')
            lm.print('item 1b')
        lm.print('Item 2')
        with lm:
            lm.print('item 2a')
            lm.print('item 2b')
```

```python
with open('my_list.txt') as f:
    for row in f:
        print(row, end='')
```

```python

```
