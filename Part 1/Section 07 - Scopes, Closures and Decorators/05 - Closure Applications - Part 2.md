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

### Closure Applications (Part 2)


#### Example 1


Let's write a small function that can increment a counter for us - we don't have an incrementor in Python (the ++ operator in Java or C++ for example):

```python
def counter(initial_value):
    # initial_value is a local variable here
    
    def inc(increment=1):
        nonlocal initial_value
        # initial_value is a nonlocal (captured) variable here
        initial_value += increment
        return initial_value
    
    return inc
```

```python
counter1 = counter(0)
```

```python
print(counter1(0))
```

```python
print(counter1())
```

```python
print(counter1())
```

```python
print(counter1(8))
```

```python
counter2 = counter(1000)
```

```python
print(counter2(0))
```

```python
print(counter2(1))
```

```python
print(counter2())
```

```python
print(counter2(220))
```

As you can see, each closure maintains a reference to the **initial_value** variable that was created when the **counter** function was **called** - each time that function was called, a new local variable **initial_value** was created (with a value assigned from the argument), and it became a nonlocal (captured) variable in the inner scope.


#### Example 2


Let's modify this example to now build something that can run, and maintain a count of how many times we have run some function.

```python
def counter(fn):
    cnt = 0  # initially fn has been run zero times
    
    def inner(*args, **kwargs):
        nonlocal cnt
        cnt = cnt + 1
        print('{0} has been called {1} times'.format(fn.__name__, cnt))
        return fn(*args, **kwargs)
    
    return inner
```

```python
def add(a, b):
    return a + b
```

```python
counted_add = counter(add)
```

And the free variables are:

```python
counted_add.__code__.co_freevars
```

We can now call the `counted_add` function:

```python
counted_add(1, 2)
```

```python
counted_add(2, 3)
```

```python
def mult(a, b, c):
    return a * b * c
```

```python
counted_mult = counter(mult)
```

```python
counted_mult(1, 2, 3)
```

```python
counted_mult(2, 3, 4)
```

#### Example 3


Let's take this one step further, and actually store the function name and the number of calls in a global dictionary instead of just printing it out all the time.

```python
counters = dict()

def counter(fn):
    cnt = 0  # initially fn has been run zero times
    
    def inner(*args, **kwargs):
        nonlocal cnt
        cnt = cnt + 1
        counters[fn.__name__] = cnt  # counters is global
        return fn(*args, **kwargs)
    
    return inner
```

```python
counted_add = counter(add)
counted_mult = counter(mult)
```

Note that `counters` is a **global** variable, and therefore **not** a free variable:

```python
counted_add.__code__.co_freevars
```

```python
counted_mult.__code__.co_freevars
```

We can now call out functions:

```python
counted_add(1, 2)
```

```python
counted_add(2, 3)
```

```python
counted_mult(1, 2, 'a')
```

```python
counted_mult(2, 3, 'b')
```

```python
counted_mult(1, 1, 'abc')
```

```python
print(counters)
```

Of course this relies on us creating the **counters** global variable first and making sure we are naming it that way, so instead, we're going to pass it as an argument to the **counter** function:

```python
def counter(fn, counters):
    cnt = 0  # initially fn has been run zero times
    
    def inner(*args, **kwargs):
        nonlocal cnt
        cnt = cnt + 1
        counters[fn.__name__] = cnt  # counters is nonlocal
        return fn(*args, **kwargs)
    
    return inner
```

```python
func_counters = dict()
counted_add = counter(add, func_counters)
counted_mult = counter(mult, func_counters)
```

```python
counted_add.__code__.co_freevars
```

As you can see, `counters` is now a free variable.


We can now call our functions:

```python
for i in range(5):
    counted_add(i, i)

for i in range(10):
    counted_mult(i, i, i)
```

```python
print(func_counters)
```

Of course, we don't have to assign the "counted" version of our functions a new name - we can simply assign it to the same name!

```python
def fact(n):
    product = 1
    for i in range(2, n+1):
        product *= i
    return product
```

```python
fact = counter(fact, func_counters)
```

```python
fact(0)
```

```python
fact(3)
```

```python
fact(4)
```

```python
print(func_counters)
```

Notice, how we essentially **added** some functionality to our `fact` function, without modifying what the `fact` function actually returns.

This leads us straight into our next topic: decorators!

```python

```
