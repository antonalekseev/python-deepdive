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

### Sending Exceptions to Generators


So far we have seen how to send values to a generator using the `send()` method.

We have also seen how we can close a generator using the `close()` method and how that, in essence, raises a `GeneratorExit` exception inside the generator.

In fact we can also raise any exception inside a generator by using the `throw()` method.

Let's first see a simple example:

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    finally:
        print('exception must have happened...')
```

```python
g = gen()
```

```python
next(g)
```

```python
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

As you can see, the exception occurred **inside** the generator, and then propagated up to the caller (we did not intercept and silence the exception). Of course we can do that if we want to:

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    except ValueError:
        print('received the value error...')
    finally:
        print('generator exiting and closing')
```

```python
g = gen()
```

```python
next(g)
g.send('hello')
```

```python
g.throw(ValueError, 'stop it!')
```

We caught the `ValueError` exception, so why did we get a `StopIteration` exception?

Because the generator returned - this raises a `StopIteration` exception.


The behavior of the `throw` is as follows:

* if the generator catches the exception and yields a value, that is the return value of the `throw()` method
* if the generator does not catch the exception, the exception is propagated back to the caller
* if the generator catches the exception, and exits (returns), the `StopIteration` exception is propagated to the caller
* if the generator catches the exception, and raises another exception, that exception is propagated to the caller


Let's see an example of each of those:


##### if the generator catches the exception and yields a value, that is the return value of the throw() method

```python
from inspect import getgeneratorstate
```

```python
def gen():
    while True:
        try:
            received = yield
            print(received)
        except ValueError as ex:
            print('ValueError received...', ex)
```

```python
g = gen()
next(g)
```

```python
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

```python
g.send('hello')
```

And the generator is now in a suspended state, waiting for our next call:

```python
getgeneratorstate(g)
```

##### if the generator does not catch the exception, the exception is propagated back to the caller

```python
def gen():
    while True:
        received = yield
        print(received)
```

```python
g = gen()
next(g)
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

And the generator is now in a closed state:

```python
getgeneratorstate(g)
```

##### if the generator catches the exception, and exits (returns), the StopIteration exception is propagated to the caller

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    except ValueError as ex:
        print('ValueError received', ex)
        return None
```

```python
g = gen()
next(g)
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

And, once again, the generator is in a closed state:

```python
getgeneratorstate(g)
```

##### if the generator catches the exception, and raises another exception, that exception is propagated to the caller

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    except ValueError as ex:
        print('ValueError received...', ex)
        raise ZeroDivisionError('not really...')
```

```python
g = gen()
next(g)
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

And out generator is, once again, in a closed state:

```python
getgeneratorstate(g)
```

As you can see our traceback includes both the `ZeroDivisionError` and the `ValueError` that caused the `ZeroDivisionError` to happen in the first place. If you don't want to have that  traceback you can easily remove it and only display the `ZeroDivisionError` (I will cover this and exceptions in detail in a later part of this series):

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    except ValueError as ex:
        print('ValueError received...', ex)
        raise ZeroDivisionError('not really...') from None
```

```python
g = gen()
next(g)
g.send('hello')
```

```python
g.throw(ValueError, 'custom message')
```

#### Example of where this can be useful


Suppose we have a coroutine that handles writing data to a database.
We have seen in some previous examples where we could use a coroutine to start and either commit or abort a transaction - based on closing the generator or forcing an exception to happen in the body of the generator.

Let's revisit this example, but now we'll want to use exceptions to indicate to our generator whether to commit or abort a transaction, without necessarily exiting the generator:

```python
class CommitException(Exception):
    pass

class RollbackException(Exception):
    pass

def write_to_db():
    print('opening database connection...')
    print('start transaction...')
    try:
        while True:
            try:
                data = yield
                print('writing data to database...', data)
            except CommitException:
                print('committing transaction...')
                print('opening next transaction...')
            except RollbackException:
                print('aborting transaction...')
                print('opening next transaction...')
    finally:
        print('generator closing...')
        print('aborting transaction...')
        print('closing database connection...')
```

```python
sql = write_to_db()
```

```python
next(sql)
```

```python
sql.send(100)
```

```python
sql.throw(CommitException)
```

```python
sql.send(200)
```

```python
sql.throw(RollbackException)
```

```python
sql.send(200)
sql.throw(CommitException)
sql.close()
```

As you can see, we can use exceptions to control the **flow** of our code. Exceptions are not necessarily **errors**! As we have seen with the `StopIteration` exception, or the `GeneratorExit` exception.


#### throw() and close()


The `close()` method does essentially the same thing as `throw(GeneratorExit)` except that when that exception is thrown using `throw()`, Python does not silence the exception for the caller:

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    finally:
        print('closing down...')
```

```python
g = gen()
next(g)
g.send('hello')
g.close()
```

```python
g = gen()
next(g)
g.send('hello')
g.throw(GeneratorExit)
```

Even if we catch the exception, we are still exiting the generator, so using `throw` will result in the caller receiving a `StopIteration` exception.

```python
def gen():
    try:
        while True:
            received = yield
            print(received)
    except GeneratorExit:
        print('received generator exit...')
    finally:
        print('closing down...')
```

```python
g = gen()
next(g)
g.close()
```

```python
g = gen()
next(g)
g.throw(GeneratorExit)
```

So, we can use `throw` to close the generator, but as the caller we now have to handle the exception that propagates up to us:

```python
g = gen()
next(g)
try:
    g.throw(GeneratorExit)
except StopIteration:
    print('silencing GeneratorExit...')
    pass
        
```

Basically this is the exact same scenario as the catch and exit (return) we saw a couple of examples back.
