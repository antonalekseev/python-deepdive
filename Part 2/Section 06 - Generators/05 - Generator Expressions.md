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

### Generator Expressions


Recall how list comprehensions worked:

```python
l = [i ** 2 for i in range(5)]
```

```python
l
```

The expression inside the `[]` brackets is called a comprehension expression.

The `[]` brackets resulted in a list being created.

We can easily create a **generator** by using `()` parentheses instead of the `[]` brackets:

```python
g = (i ** 2 for i in range(5))
```

Note that `g` is a generator, and is also lazily evaluated:

```python
type(g)
```

```python
for item in g:
    print(item)
```

And now the generator has been exhausted:

```python
for item in g:
    print(item)
```

Scoping works the same way with generator expressions as with list comprehensions, i.e. generator expressions are created by Python using a function, and therefore have local scopes and can access enclosing nonlocal and global scopes.

```python
import dis
```

Recall for list comprehensions:

```python
exp = compile('[i**2 for i in range(5)]', filename='<string>', mode='eval')
```

```python
dis.dis(exp)
```

```python
exp = compile('(i ** 2 for i in range(5))', filename='<string>', mode='eval')
```

```python
dis.dis(exp)
```

As you can see the internal mechanism for list comprehensions and generator expressions is almost the same - in particular note how a function is created. The main difference is that in one case a list is created (an iterable), while in the other a generator (an iterator) is produced.


We can iterate over the same list comprehension multiple times, since it is an iterable. However, we can only iterate over a comprehension expression once, since it is an iterator.

```python
l = [i * 2 for i in range(5)]
```

```python
type(l)
```

```python
g = (i ** 2 for i in range(5))
```

```python
type(g)
```

#### Nested Comprehensions


Just as with list comprehensions, we can nest generator expressions too:


Let's use some of the same examples we saw with nested list comprehensions.


##### Example 1


A multiplication table:


Using a list comprehension approach first:

```python
start = 1
stop = 10

mult_list = [ [i * j 
               for j in range(start, stop+1)]
             for i in range(start, stop+1)]
```

```python
mult_list
```

The equivalent generator expression would be:

```python
start = 1
stop = 10

mult_list = ( (i * j 
               for j in range(start, stop+1))
             for i in range(start, stop+1))
```

```python
mult_list
```

We can iterate through mult_list:

```python
table = list(mult_list)
```

```python
table
```

But you'll notice that our rows are themselves generators!


To fully materialize the table we need to iterate through the row generators too:

```python
table_rows = [list(gen) for gen in table]
```

```python
table_rows
```

Of course, we can mix list comprehensions and generators. 

In this modification, we'll make the rows list comprehensions, and retain the generator expression in the outer comprehension:

```python
start = 1
stop = 10

mult_list = ( [i * j 
               for j in range(start, stop+1)]
             for i in range(start, stop+1))
```

Notice what is happening here, the table itself is lazy evaluated, i.e. the rows are not yielded until they are requested - but once a row is requested, the list comprehension that defines the row will be entirely evaluated at that point:

```python
for item in mult_list:
    print(item)
```

##### Example 2


Let's try Pascal's triangle again:

<!-- #region -->
```
1
1 1
1 2 1
1 3 3 1
1 4 6 4 1
```

we just need to know how to calculate combinations:
```
C(n, k) = n! / (k! (n-k)!)
```

* row 0, column 0: n=0, k=0: c(0, 0) = 0! / 0! 0! = 1/1 = 1
* row 4, column 2: n=4, k=2: c(4, 2) = 4! / 2! 2! = 4x3x2 / 2x2 = 6

In other words, we need to calculate the following list of lists:
```
c(0,0)
c(1,0) c(1,1)
c(2,0) c(2,1) c(2,2)
c(3,0) c(3,1) c(3,2) c(3,3)
...
```
<!-- #endregion -->

Here's how we did it using a list comprehension:

```python
from math import factorial

def combo(n, k):
    return factorial(n) // (factorial(k) * factorial(n-k))

size = 10  # global variable
pascal = [ [combo(n, k) for k in range(n+1)] for n in range(size+1) ]
```

```python
pascal
```

We can now use generator expressions for either one or both of the nested list comprehensions. In this case I'll use it for both:

```python
size = 10  # global variable
pascal = ( (combo(n, k) for k in range(n+1)) for n in range(size+1) )
```

If we want to materialize the triangle into a list we'll need to do so ourselves:

```python
[list(row) for row in pascal]
```

#### Timings


So we see that the main difference between the two approaches is that in one case we have a fully materialized list (i.e. all the elements have been  created and put into list objects), while in the other we are dealing with lazily evaluated iterators.

One main advantage to using generators is that we do not need the up-front calculations - if we end up not consuming the entire iterator, we have saved some time.

The other advantage, as we saw with lazy iterators is that you do not need to have the entire data set in memory at one time. We saw an example of this when reading files - we can read extremely large files one row at a time, without having to store the entire file in memory.


Let's see the time difference between creating a list comprehension and a generator expression for a large Pascal triangle:

```python
from timeit import timeit
```

```python
size = 600
```

```python
timeit('[[combo(n, k) for k in range(n+1)] for n in range(size+1)]',
      globals=globals(), number=1)
```

```python
timeit('((combo(n, k) for k in range(n+1)) for n in range(size+1))',
      globals=globals(), number=1)
```

As you can see, much faster - but that's because we haven't actually done anything other than set up the nested iterators. Since no iteration took place, no calculations were performed.

In fact, even if we make the inner generator expression a list comprehension, those will not be calculated until the individual rows from the outer generator expression are requested:

```python
timeit('([combo(n, k) for k in range(n+1)] for n in range(size+1))',
      globals=globals(), number=1)
```

In fact, we can quickly create a **huge** Pascal triangle using the generator approach:

```python
size = 100_000

timeit('([combo(n, k) for k in range(n+1)] for n in range(size+1))',
      globals=globals(), number=1)
```

What about timing both creating **and** iterating though all the elements?

Let's do this by creating some functions that will do that:

```python
def pascal_list(size):
    l = [[combo(n, k) for k in range(n+1)] for n in range(size+1)]
    for row in l:
        for item in row:
            pass
```

```python
def pascal_gen(size):
    g = ((combo(n, k) for k in range(n+1)) for n in range(size+1))
    for row in g:
        for item in row:
            pass
```

```python
size = 600
timeit('pascal_list(size)', globals=globals(), number=1)
```

```python
size = 600
timeit('pascal_gen(size)', globals=globals(), number=1)
```

So as you can see, if we actually iterate through each element, we don't end up saving any time - however, creating the iterator is faster, and if we don't use all the elements, then it will be more efficient.


#### Memory Usage


Another thing that is way more efficient is memory usage.

To see this, we'll use a rough technique and the `tracemalloc` standard library module:

```python
import tracemalloc
```

```python
def pascal_list(size):
    l = [[combo(n, k) for k in range(n+1)] for n in range(size+1)]
    for row in l:
        for item in row:
            pass
    stats = tracemalloc.take_snapshot().statistics('lineno')
    print(stats[0].size, 'bytes')
```

```python
def pascal_gen(size):
    g = ((combo(n, k) for k in range(n+1)) for n in range(size+1))
    for row in g:
        for item in row:
            pass
    stats = tracemalloc.take_snapshot().statistics('lineno')
    print(stats[0].size, 'bytes')
```

```python
tracemalloc.stop()
tracemalloc.clear_traces()
tracemalloc.start()
pascal_list(300)
```

```python
tracemalloc.stop()
tracemalloc.clear_traces()
tracemalloc.start()
pascal_gen(300)
```

As you can see, using a generator did not require as much memory. Because we are essentially using a lazy iterator, the memory required by a previous result is released once the next iteration is requested.
