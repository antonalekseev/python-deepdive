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

### Sending data to Generators


With PEP 342, generators were enhanced to allow not just sending data out (yielding), but also receiving data.

The basic idea is that when a generator is **suspended** after a yield statement, why not allow sending it some data when we resume its execution, exactly at the point where it resumes.

In other words, immediately after the `yield` statement.

And not on the next line of code, but actually in the same line as the `yield` - we should now think of the `yield` keyword, not just as a statement, but as an expression that also *receives* data - and we can assign that received value to a variable using an assignment. We can send data to the suspended generator (and resume running it) by using the `send()` **method** of the generator (instead of just using the `__next__` method (or, same thing, `next()`).

**Note:**
The **same** `yield` keyword is actually used to do both - but make no mistake, these are very different usages of the same keyword.

The key difference is that `yield` is actually an expression, not a simple statement - and of course we can assign expressions to variables.

Let's take a look at a simple example to illustrate how this works:

```python
def echo():
    while True:
        received = yield
        print('You said:', received)
```

```python
e = echo()
```

We now have a generator object, but what is it's state?

```python
from inspect import getgeneratorstate

getgeneratorstate(e)
```

<!-- #region -->
Right, it is created, but not suspended - in order for it to receive data it should be run up to the `yield` expression.

Remember that in assignments, the right hand side is evaluated **first**, and the result of the expression is assigned to the left hand variable.

So, if we call `next` on the generator it will start running.
Once it hits the line
```
received = yield
```
it will first evaluate the right hand side - at which time it will yield and therefore become suspended!
<!-- #endregion -->

```python
next(e)
```

```python
getgeneratorstate(e)
```

Now that it is waiting to resume, we can send it data, and the generator will received that data as if it were the right hand side of the assignment:

```python
e.send('python')
```

And now the generator continued running until it hit a `yield` again - which it does since we have our yield inside an infinite loop:

```python
e.send('I said')
```

So, the `send` method essentially resume the generator just as the `__next__` does - but it also sends in some data that we can capture if we want to, inside the generator.

What happens if we do call `next()` or `__next__` instead of `send()`?

The same as if we had sent the `None` value:

```python
next(e)
```

```python
e.send(None)
```

See, same thing...


You might be asking whether we could have used `send` with all the generators we had written so far - sure!
The `yield` keyword is an expression, and you don't have to assign the result of an expression to a variable:

```python
10 < 100
```

That was an expression, and it was perfectly fine not to assign it to a variable. We can, but we don't have to.

So, in fact the following also works just fine:

```python
def squares(n):
    for i in range(n):
        yield i**2
```

```python
sq = squares(5)
```

```python
next(sq)
```

```python
sq.send(100)
```

```python
sq.send(100)
```

Now, the only thing is that we cannot change a generator from `created` to `suspended` using the `send()` function - we **have** to call `next` first.

In other words this will not work:

```python
e = echo()
```

```python
e.send('hello')
```

We need to **start** or **prime** the generator first, using, `next` which will run the code until the `yield` expression is encountered.

```python
next(e)
```

```python
e.send('hello')
```

At this point we can see that generators can be used to both send and receive data.

You might be asking yourself whether it is possible to do both **at the same time** - i.e. use ` yield` to both yield data and receive data (upon resumption). The answer is yes, but it's kind of mind bending, and unless you actually need to do so, resist the temptation to do it - it can be extremely confusing:

```python
def squares(n):
    for i in range(n):
        received = yield i ** 2
        print('received:', received)
```

```python
sq = squares(5)
```

```python
next(sq)
```

```python
yielded = sq.send('hello')
print('yielded:', yielded)
```

```python
yielded = sq.send('hello')
print('yielded:', yielded)
```

Of course, once the generator no longer `yields`, but `returns` we'll get the same `StopIteration` exception:

```python
def echo(max_times):
    for i in range(max_times):
        received = yield
        print('You said:', received)
    print("that's all, folks!")
```

```python
e = echo(3)
next(e)
```

```python
e.send('python')
e.send('is')
```

The next `send` is going to resume the generator, it will print what we send it, and continue running - but this time the loop is done, so it will print our final `that's all, folks`, and the function will return (`None`) and hence cause a `StopIteration` exception to be raised:

```python
e.send('awesome')
```

<!-- #region -->
So, when we deal with generators and the `yield` expression, we need to distinguish between two different uses:
* one way produces data for iteration
* the other way consumes data

To avoid confusion, try not to mix the two concepts together unless you have to. Try to keep them separate, i.e., either use:
```
yield <expression>
```
or 
```
variable = yield
```
but not both at the same time such as:
```
variable = yield <expression>
```
<!-- #endregion -->

There are cases where using the combination is definitely useful.

Consider this example where we want a generator/coroutine that maintains (and yields) a running average of values we send it.

Let's first see how we would do it without using a coroutine - instead we'll use a closure so we can maintain the state (`total` and `count`):

```python
def averager():
    total = 0
    count = 0
    def inner(value):
        nonlocal total
        nonlocal count
        total += value
        count += 1
        return total / count
    return inner

def running_averages(iterable):
    avg = averager()
    for value in iterable:
        running_average = avg(value)
        print(running_average)
```

```python
running_averages([1, 2, 3, 4])
```

And now the same, but using a coroutine:

```python
def running_averager():
    total = 0
    count = 0
    running_average = None
    while True:
        value = yield running_average
        total += value
        count += 1
        running_average = total / count
```

```python
def running_averages(iterable):
    averager = running_averager()
    next(averager)  # prime generator
    for value in iterable:
        running_average = averager.send(value)
        print(running_average)
```

```python
running_averages([1, 2, 3, 4])
```

As you can see it was useful to use `yield` to both send in the new value, and in the subsequent yield to receive the new average.
