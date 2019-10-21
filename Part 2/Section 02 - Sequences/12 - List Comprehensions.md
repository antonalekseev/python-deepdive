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

### List Comprehensions


We've used list comprehensions throughout this course quite a bit, so the concept should not be new, but let's recap quickly what we have seen so far with list comprehensions.

A list comprehension is language construct that allows to easily build a list by transforming, and optionally, filtering, another iterable.

For example, using a more traditional Java style approach we might create a list of squares of the first 100 positive integers in this way:

```python
squares = []  # create an empty list
for i in range(1, 101):
    squares.append(i**2)
```

We now have a list containing the desired numbers:

```python
squares[0:10]
```

Using a list comprehension we can achieve the same results in a far more expressive way:

```python
squares = [i**2 for i in range(1, 101)]
```

```python
squares[0:10]
```

When building a list from another iterable we may sometimes want to skip certain values.

For example, we may want to build a list of squares for even positive integers only, up to 100.

The more traditional way would go like this:

```python
squares = []
for i in range(1, 101):
    if i % 2 == 0:
        squares.append(i**2)
```

```python
squares[0:10]
```

We can also use a list comprehension to achieve the same thing:

```python
squares = [i**2 for i in range(1, 101) if i % 2 == 0]
```

```python
squares[0:10]
```

Although I have been writing the list comprehension on a single line, we can write them over multiple lines if we prefer:

```python
squares = [i**2
          for i in range(1, 101)
          if i % 2 == 0]
```

```python
squares[0:10]
```

Internal Mechanics of List Comprehensions


As we discussed in the lecture, we need to recognize that list comprehensions are essentially temporary functions that Python creates, executes and returns the resulting list from it.

We can see this by compiling a comprehension, and then disassembling the compiled code to see what happened:

```python
import dis
```

```python
compiled_code = compile('[i**2 for i in (1, 2, 3)]', 
                        filename='', mode='eval')
```

```python
dis.dis(compiled_code)
```

As you can see, in step 4, Python created a function (`MAKE_FUNCTION`), called it (`CALL_FUNCTION`), and then returned the result (`RETURN_VALUE`) in the last step.


So, comprehensions will behave like functions in terms of **scope**. They have local scope, and can access global and nonlocal scopes too. And nested comprehensions will also behave like nested functions and closures.


#### Nested Comprehensions


Let's look at a simple example that uses nested comprehensions.

For example, suppose we want to generate a multiplication table:


The traditional way first:

```python
table = []
for i in range(1, 11):
    row = []
    for j in range(1, 11):
        row.append(i*j)
    table.append(row)
```

```python
table
```

We can easily do the same thing using a list comprehension:

```python
table2 = [ [i * j for j in range(1, 11)] 
          for i in range(1, 11)]
```

```python
table2
```

You'll notice here that we nested one list comprehension inside another.

You should also notice that the inner comprehension (the one that has `i*j`) is accessing a local variable `i`, as well as a variable from the enclosing comprehension - the `j` variable. Just like a closure! And in fact, it is exactly that. We'll come back to that in a bit.

<!-- #region -->
Let's do another example - we'll construct Pascal's triangle - which is basically just a triangle of binomial coefficients:

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
c(2,0) c(2,1) c(2,3)
c(3,0) c(3,1) c(3,2) c(3,3)
...
```
<!-- #endregion -->

We can use a nested comprehension for that!

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

Again note how the outer comprehension accessed a global variable (`size`), created a local variable (`n`), and the inner comprehension created its own local variable (`k`) and also accessed the nonlocal variable `n`.


#### Nested Loops


We can also created comprehensions that use nested loops (not nested comprehensions, just nested loops).

Let's start with a simple example.

Suppose we have two lists of characters, and we want to produce a new list consisting of the pairwise concatenated characters.

e.g. 
`l1 = ['a', 'b', 'c']`

`l2 = ['x', 'y', 'z']`

and we want to produce the result:

`['ax', 'ay', 'az', 'bx', 'by', 'bz', 'cx', 'cy', 'cz']`



The traditional way first:

```python
l1 = ['a', 'b', 'c']
l2 = ['x', 'y', 'z']
result = []
for s1 in l1:
    for s2 in l2:
        result.append(s1+s2)

```

```python
result
```

We can do the same nested loop using a comprehension instead:

```python
result = [s1 + s2 for s1 in l1 for s2 in l2]
```

```python
result
```

We could expand this slightly by specifying that pairs resulting in the same letter twice should be ommitted:

```python
l1 = ['a', 'b', 'c']
l2 = ['b', 'c', 'd']
```

```python
result = []
for s1 in l1:
    for s2 in l2:
        if s1 != s2:
            result.append(s1 + s2)
```

```python
result
```

And the comprehension equivalent:

```python
result = [s1 + s2 for s1 in l1 for s2 in l2 if s1 != s2]
```

```python
result
```

Building up the complexity, let's see how we might reproduce the `zip` function.

Remember what the `zip` function does:

```python
l1 = [1, 2, 3, 4, 5, 6, 7, 8, 9]
l2 = ['a', 'b', 'c', 'd']
list(zip(l1, l2))
```

We can do the same thing using a traditional nested loop:

```python
result = []
for index_1, item_1 in enumerate(l1):
    for index_2, item_2 in enumerate(l2):
        if index_1 == index_2:
            result.append((item_1, item_2))
```

```python
result
```

But we can do this using a list comprehension as well:

```python
result = [ (item_1, item_2)
         for index_1, item_1 in enumerate(l1)
         for index_2, item_2 in enumerate(l2)
         if index_1 == index_2]
```

```python
result
```

Of course, using `zip` is way simpler!


List comprehensions can also be quite handy when used in conjunction with functions such as `sum` for example.

Suppose we have two n-dimensional vectors, represented as tuple of numbers, and we want to find the dot product of the two vectors:

`
v1 = (c1, c2, c3, ..., cn)
v2 = (d1, d2, d3, ..., dn)
`

Then, the dot product is:

`
c1 * d1 + c2 * d2 + ... + cn * dn
`


The trick here is that we want to step through each vectors at the same time (a simple nested loop would not work), so a Java-like approach might be:

```python
v1 = (1, 2, 3, 4, 5, 6)
v2 = (10, 20, 30, 40, 50, 60)
```

```python
dot = 0
for i in range(len(v1)):
    dot += (v1[i] * v2[i])
print(dot)
```

But using zip and a list comprehension we can do it this way:

```python
dot = sum([i * j for i, j in zip(v1, v2)])
print(dot)
```

In fact, and we'll cover this later in generator expressions, we don't even need the `[]`:

```python
dot = sum(i * j for i, j in zip(v1, v2))
print(dot)
```

#### Things to watch out for


There are a few things we have to be careful with, and that relates to the scope of variables used inside a comprehension.


Let's first make sure we don't have the `number` symbol in our global scope:

```python
if 'number' in globals():
    del number
```

```python
l = [number**2 for number in range(5)]
print(l)
```

What was the scope of `number`?

```python
'number' in globals()
```

As you can see, `number` was local to the comprehension, not the enclosing (global in this case) scope.


But what if `number` was in our global scope:

```python
number = 100
```

```python
l = [number**2 for number in range(5)]
```

```python
number
```

As you can see, `number` in the comprehension was still local to the comprehension, and our global `number` was not affected. 

This is similar to global and nonlocal variables in functions.

Because `number` is the loop item, it means that it gets *assigned* a value before being referenced, hence it is considered local - even if that symbol exists in a global or nonlocal scope.

On the other hand, consider this example:


```python
number = 100
l = [number * i for i in range(5)]
print(l)
```

As you can see, the scope of the comprehension was able to reach out for `number` in the global scope. Same as functions.


Now let's look at an example we've seen before when we studied closures.

Suppose we want to generate a list of functions that will calculate powers of their argument, i.e. we want to define a bunch of functions

* `fn_1(arg) --> arg ** 1`
* `fn_2(arg) --> arg ** 2`
* `fn_3(arg) --> arg ** 3`
etc...


We could certainly define a bunch of functions one by one:

```python
fn_0 = lambda x: x**0
fn_1 = lambda x: x**1
fn_2 = lambda x: x**2
fn_3 = lambda x: x**3
# etc
```

But this would be very tedious if we had to do it more than just a few times.

Instead, why don't we create those functions as lambdas and put them into a list where the index of the list will correspond to the power we are looking for.

Something like this if we were doing it manually:

```python
funcs = [lambda x: x**0, lambda x: x**1, lambda x: x**2, lambda x: x**3]
```

Now we can call these functions this way:

```python
print(funcs[0](10))
print(funcs[1](10))
print(funcs[2](10))
print(funcs[3](10))
```

Now all we need to do is to create these functions using a loop - the traditional way first:


First let's make sure `i` is not in our global symbol table:

```python
if 'i' in globals():
    del i
```

```python
funcs = []
for i in range(6):
    funcs.append(lambda x: x**i)
```

And let's use them as before:

```python
print(funcs[0](10))
print(funcs[1](10))
print(funcs[2](10))
print(funcs[3](10))
```

What happened?? It looks like every function is actually calculating `10**5`


Let's break down what happened in the loop, but without using a loop.


Firs notice that `i` is now in our global symbol table:

```python
print(i)
```

You'll also note that it has a value of `5` (from the last iteration that ran).

Now let's walk through what happened manually:


In the first iteration, the symbol `i` was created, and assigned a value of `0`:

```python
i = 0
def fn_0(x):
    return x ** i
```

The `i` in `fn_0` is actually the global variable `i`.

For the next 'iteration' we increment `i` by `1`:

```python
i=1
def fn_1(x):
    return x ** i
```

The `i` in `fn_1` is still the global variable `i`.


Now let's set `i` to something else:

```python
i = 5
```

```python
fn_0(10)
```

```python
fn_1(10)
```

and if we change `i` again:

```python
i = 10
```

```python
fn_0(10)
```

And this is **exactly** what happened in our loop based approach:

```python
funcs = []
for i in range(6):
    funcs.append(lambda x: x**i)
```

When the loop ran, `i` was created in our **global** scope.

By the time the loop finished running, `i` was 5

```python
print(i)
```

So when we call the functions, they are referencing the global variable `i` which is now set to `5`.


And the same precise thing will happen if we use a comprehension to do the same thing:


Let's delete the global `i` symbol first:

```python
del i
```

```python
'i' in globals()
```

```python
funcs = [lambda x: x**i for i in range(6)]
```

```python
'i' in globals()
```

As we can see `i` is not in our globals, but `i` was a **local** variable in the list comprehension, and each function created in the comprehension is referencing the same `i` - it is local to the comprehension, and each lambda is therefore a closure with (the same) free variable `i`. And by the time the comprehension has finished running, `i` had a value of 5:

```python
funcs[0](10), funcs[1](10)
```

Can we somehow fix this problem?

Yes, and it relies on default values and when default values are calculated and stored with the function definition. Recall that default values are evaluated and stored with the function's definition **when the function is being created (i.e. compiled)**. Right now we are running into a problem because the free variable `i` is being evauated inside each function's body at **run time**.

So, we can fix this by making each current value of `i` a paramer default of each lambda - this will get evaluated at the functions creation time - i.e. at each loop iteration:

```python
funcs = [lambda x, pow=i: x**pow for i in range(6)]
```

```python
funcs[0](10), funcs[1](10), funcs[2](10)
```

As you can see that solved the problem. But this relies on some pretty detailed understanding of Python's behavior, and it is better not to use such techniques - other people reading your code will find it confusing and will make the code much harder to understand.


We will come back to this comprehension syntax. We used it so far to create lists, but the same syntax will be used to create sets, dictionaries, and generators.
