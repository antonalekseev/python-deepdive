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

### Yield From - Throwing Exceptions


We have seen that `yield from` allows us to establish a 2-way communication channel with a subgenerator and we could use `next`, and `send` to send a "request" to a delegated subgenerator via the delegator generator.

In fact, we can also send exceptions by throwing an exception into the delegator, just like a `send`.

```python
class CloseCoroutine(Exception):
    pass

def echo():
    try:
        while True:
            received = yield
            print(received)
    except CloseCoroutine:
        return 'coro was closed'
    except GeneratorExit:
        print('closed method was called')
```

```python
e = echo()
next(e)
```

```python
e.throw(CloseCoroutine, 'just closing')
```

```python
e = echo()
next(e)
e.close()
```

As we can see the difference between `throw` and `close` is that although `close` causes an exception to be raised in the generator, Python essentially silences it.

It works the same way when we delegate to the coroutine in a delegator:

```python
def delegator():
    result = yield from echo()
    yield 'subgen closed and returned:', result
    print('delegator closing...')
```

```python
d = delegator()
next(d)
d.send('hello')
```

```python
d.throw(CloseCoroutine)
```

Now what happens if the `throw` in the subgenerator does not close subgenerator but instead silences the exception and yields a value instead?

```python
class CloseCoroutine(Exception):
    pass

class IgnoreMe(Exception):
    pass

def echo():
    try:
        while True:
            try:
                received = yield
                print(received)
            except IgnoreMe:
                yield "I'm ignoring you..."
    except CloseCoroutine:
        return 'coro was closed'
    except GeneratorExit:
        print('closed method was called')
```

```python
d = delegator()
next(d)
```

```python
d.send('python')
```

```python
result = d.throw(IgnoreMe, 1000)
```

```python
result
```

```python
d.send('rocks!')
```

Why did we not get a yielded value back?

That's because the subgenerator was paused at the yield that yielded "I'm, ignoring you".

If we want to coroutine to continue running normally after ignoring that exception we need to tweak it slightly:


Let's first make sure we close our previous delegator!

```python
d.close()
```

```python
def echo():
    try:
        output = None
        while True:
            try:
                received = yield output
                print(received)
            except IgnoreMe:
                output = "I'm ignoring you..."
            else:
                output = None
    except CloseCoroutine:
        return 'coro was closed'
    except GeneratorExit:
        print('closed method was called')
```

```python
d = delegator()
next(d)
```

```python
d.send('hello')
```

```python
d.throw(IgnoreMe)
```

```python
d.send('python')
```

```python
d.close()
```

What happens if we do not handle the error in the subgenerator and simply let the exception propagate up?
Who gets the exception, the delegator, or the caller?

```python
def echo():
    while True:
        received = yield
        print(received)
```

```python
def delegator():
    yield from echo()
```

```python
d = delegator()
next(d)
```

```python
d.throw(ValueError)
```

OK, so we, the caller see the exception. But did the delegator see it too? i.e. can we catch the exception in the delegator?

```python
def delegator():
    try:
        yield from echo()
    except ValueError:
        print('got the value error')
```

```python
d = delegator()
next(d)
```

```python
d.throw(ValueError)
```

As you can see, we were able to catch the exception in the delegator.
Of course, the way we wrote our code, the delegator still closed, and hence we now see a `StopIteration` exception.


#### Example


Suppose we have a coroutine that creates running averages, and we want to occasionally write the current data to a file:

```python
class WriteAverage(Exception):
    pass

def averager(out_file):
    total = 0
    count = 0
    average = None
    with open(out_file, 'w') as f:
        f.write('count,average\n')
        while True:
            try:
                received = yield average
                total += received
                count += 1
                average = total / count
            except WriteAverage:
                if average is not None:
                    print('saving average to file:', average)
                    f.write(f'{count},{average}\n')
```

```python
avg = averager('sample.csv')
next(avg)
```

```python
avg.send(1)
avg.send(2)
```

```python
avg.throw(WriteAverage)
```

```python
avg.send(3)
```

```python
avg.send(2)
```

```python
avg.throw(WriteAverage)
```

```python
avg.close()
```

Now we can read the data back and make sure it worked as expected:

```python
with open('sample.csv') as f:
    for row in f:
        print(row.strip())
```

Of course we can use a delegator as well.
Maybe the delegator is charged with figuring out the output file name.
Here we'll just hardcode it inside the delegator:

```python
def delegator():
    yield from averager('sample.csv')
```

```python
d = delegator()
next(d)
```

```python
d.send(1)
```

```python
d.send(2)
```

```python
d.send(3)
```

```python
d.send(4)
```

```python
d.throw(WriteAverage)
```

```python
d.send(5)
```

```python
d.throw(WriteAverage)
```

```python
d.close()
```

```python
with open('sample.csv') as f:
    for row in f:
        print(row.strip())
```
