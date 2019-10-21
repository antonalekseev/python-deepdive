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

### Yield From - Closing and Return


Just as we can send `next` and `send` through a delegator, we can also send `close`.

How does this affect the delegator and the subgenerator?

Let's take a look.

```python
def subgen():
    try:
        while True:
            received = yield
            print(received)
    finally:
        print('subgen: closing...')
```

```python
def delegator():
    s = subgen()
    yield from s
    yield 'delegator: subgen closed'
    print('delegator: closing...')
```

```python
d = delegator()
next(d)
```

At this point, both the delegator and the subgenerator are primed and suspended:

```python
from inspect import getgeneratorstate, getgeneratorlocals
```

```python
getgeneratorlocals(d)
```

```python
s = getgeneratorlocals(d)['s']
print(getgeneratorstate(d))
print(getgeneratorstate(s))
```

We can send data to the delegator:

```python
d.send('hello')
```

We can even send data directly to the subgenerator since we now have a handle on it:

```python
s.send('python')
```

In fact, we can close it too:

```python
s.close()
```

So, what is the state of the delegator now?

```python
getgeneratorstate(d)
```

But the subgenerator closed, so let's see what happens when we call `next` on `d`:

```python
next(d)
```

As you can see, the generator code resume right after the `yield from`, and we can do this one more time to close the delegator:

```python
next(d)
```

OK, so this is what happens when the subgenerator closes (directly or indirectly) - the delegator simply resumes running right after the `yield from` when we call `next`.

But what happens if we close the delegator instead of just closing the subgenerator?

```python
d = delegator()
next(d)
s = getgeneratorlocals(d)['s']
print(getgeneratorstate(d))
print(getgeneratorstate(s))
```

```python
d.close()
```

As you can see the subgenerator also closed. Is the delegator closed too?

```python
print(getgeneratorstate(d))
print(getgeneratorstate(s))
```

Yes. So closing the delegator will close not only the delegator itself, but also close the currently active subgenerator (if any).


We should notice that when we closed the subgenerator directly no apparent exception was raised in our context.

What happens if the subgenerator returns something when it closes?

```python
def subgen():
    try:
        while True:
            received = yield
            print(received)
    finally:
        print('subgen: closing...')
        return 'subgen: return value'
```

```python
s = subgen()
next(s)
s.send('hello')
s.close()
```

Hmmm, the `StopIteration` exception was silenced. Let's do this a different way, since we know the `StopIteration` exception should contain the return value:

```python
s = subgen()
next(s)
s.send('hello')
s.throw(GeneratorExit, 'force exit')
```

OK, so now we can see that the `StopIteration` exception contains the return value.

The `yield from` actually captures that value as it's return value - in other words `yield from` is not just a statement, it is in fact, like `yield`, also an expression.

Let's see how that works:

```python
def subgen():
    try:
        yield 1
        yield 2
    finally:
        print('subgen: closing...')
        return 100
```

```python
def delegator():
    s = subgen()
    result = yield from s
    print('subgen returned:', result)
    yield 'delegator suspended'
    print('delegator closing')
```

```python
d = delegator()
```

```python
next(d)
```

```python
next(d)
```

```python
next(d)
```

As you can see the return value of the subgenerator ended up as the result of the `yield from` expression. 
