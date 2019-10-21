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

### Yield From - Two-Way Communications


In the last section on generators, we started looking at `yield from` and how we could delegate iteration to another iterator.

Let's see a simple example again:

```python
def squares(n):
    for i in range(n):
        yield i ** 2
```

```python
def delegator(n):
    for value in squares(n):
        yield value
```

```python
gen = delegator(5)
for _ in range(5):
    print(next(gen))
```

Alternatively we could write the same thing this way:

```python
def delegator(n):
    yield from squares(n)
```

```python
gen = delegator(5)
for _ in range(5):
    print(next(gen))
```

**Terminology:** 
When we use `yield from subgen` we are **delegating** to `subgen`.

The generator that delegates to the other generator is called the **delegator** and the generator that it delegates to is called the **subgenerator**.

So in our example `squares(n)` was the subgenerator, and `delegator()` was the delegator.

The context that contains the code making `next` calls to the delegator, is called the **caller's context**, or simply the **caller**.

<!-- #region -->
What is actually happening when we call
```
next(gen)
```
is that `gen` (the delegator) is passing along the `next` request to the `squares(n)` (the subgenerator).

In return, the subgenerator is yielding values back to the delegator, which in turn yields it back to us (the caller).

There is in fact a **two-way communication channel** established between the caller and the subgenerator - all because of `yield from`.

* caller: next --> delegator --> subgenerator
* caller <-- delegator (yield) <-- subgenerator (yield)
<!-- #endregion -->

So, if `yield from` establishes this 2-way communication channel, and we can send `next` to the subgenerator via the delegator, can we send data using `send` as well?

The answer is yes. We'll take a look at this in some detail over the next few videos.

Let's start by looking at how the delegator works when a subgenerator closes by itself:


We'll want to inspect the delegator and the subgenerator, so let's import what we'll need from the `inspect` module:

```python
from inspect import getgeneratorstate, getgeneratorlocals
```

```python
def song():
    yield "I'm a lumberjack and I'm OK"
    yield "I sleep all night and I work all day"
```

```python
def play_song():
    count = 0
    s = song()
    yield from s
    yield 'song finished'
    print('player is exiting...')
```

Here `play_song` is the delegator, and `song` is the subgenerator. We, the Jupyter notebook, are the caller.

```python
player = play_song()
```

```python
print(getgeneratorstate(player))
print(getgeneratorlocals(player))
```

As you can see, no local variables have been created in `player` yet - that's because it is created, not actually started.

Let's start it:

```python
next(player)
```

Now let's look at the state of things:

```python
print(getgeneratorstate(player))
print(getgeneratorlocals(player))
```

We can now get a handle to the subgenerator `s`:

```python
s = getgeneratorlocals(player)['s']
```

And we can check the state of `s`:

```python
print(getgeneratorstate(s))
```

As we can see the subgenerator is suspended.

Let's iterate a few more times:

```python
print(next(player))
print(getgeneratorstate(player))
print(getgeneratorstate(s))
```

```python
print(next(player))
print(getgeneratorstate(player))
print(getgeneratorstate(s))
```

At this point the subgenerator exited, so its state is `GEN_CLOSED`, but the delegator (`player`) is just suspended, and in fact yielded `song finished`.

We can advance one more time:

```python
print(next(player))
```

We get the `StopIteration` exception because `player` returned, and now both the delegator and the subgenerator are in a closed state:

```python
print(getgeneratorstate(player))
print(getgeneratorstate(s))
```

Important to note here is that when the subgenerator returned, the delegator **continued running normally**.

Let's make a tweak to our `player` generator to make this even more evident:

```python
def player():
    count = 1
    while True:
        print('Run count:', count)
        yield from song()
        count += 1
```

```python
p = player()
```

```python
next(p), next(p)
```

```python
next(p), next(p)
```

```python
next(p), next(p)
```

and so on...
