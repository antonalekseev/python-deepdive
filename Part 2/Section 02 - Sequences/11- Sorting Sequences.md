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

### Sorting Sequences


Just like with the concatenation and in-place concatenation we saw previously, we have two different ways of sorting a mutable sequence:

* returning a new sorted sequence
* in-place sorting (mutating sequence) - obviously this works for mutable sequence types only!



For any iterable, the built-in `sorted` function will return a **list** containing the sorted elements of the iterable.

So a few things here: 
* any iterable can be sorted (as long as it is finite)
* the elements must be pair-wise comparable (possibly indirectly via a sort key)
* the returned result is always a list
* the original iterable is not mutated

In addition:
* optionally specify a `key` - a function that extracts a comparison key for each element. If that key is not specified, Python will use the natural ordering of the elements (such as __gt__, etc, so that fails if they do not!)
* optional specify the `reverse` argument which will return the reversed sort


Numbers have a natural ordering for example, so sorting an iterable of numbers is easy:

```python
t = 10, 3, 5, 8, 9, 6, 1
sorted(t)
```

As you can see we sorted a `tuple` and got a `list` back.


We can sort non-sequence iterables too:

```python
s = {10, 3, 5, 8, 9, 6, 1}
sorted(s)
```

For things like dictionaries, this works slightly differently. Remember what happens when we iterate a dictionary?

```python
d = {3: 100, 2: 200, 1: 10}
for item in d:
    print(item)
```

We actually are iterating the keys.

Same thing happens with sorting - we'll end up just sorting the keys:

```python
d = {3: 100, 2: 200, 1: 10}
sorted(d)
```

But what if we wanted to sort the dictionary keys based on the values instead?


This is where the `key` argument of `sorted` will come in handy.

We are going to specify to the `sorted` function that it should use the value of each item to use as a sort key:

```python
d = {'a': 100, 'b': 50, 'c': 10}
sorted(d, key=lambda k: d[k])
```

Basically the `key` argument was called on every item being sorted - these items were the keys of the dictionary: `a`, `b`, `c`.
For every key it used the result of the lambda as the sorting key:

dictionary keys --> sorting key:
* `a  --> 100`
* `b --> 50`
* `c --> 10`

Hence the sort order was 10, 20, 100, which means `c, b, a`


Here's a different example, where we want to sort strings, not based on the lexicographic ordering, but based on the length of the string.

We can easily do this as follows:

```python
t = 'this', 'parrot', 'is', 'a', 'late', 'bird'
sorted(t)
```

As you can see the natural ordering for strings was used here, but we can change the behavior by specifying the sort key:


Remember that the `key` is a function that receives the item being sorted, and should return something (else usually!) that we want to use as the sort key. We use lambdas, but you can also use a straight `def` function too:

```python
def sort_key(s):
    return len(s)
```

```python
sorted(t, key=sort_key)
```

or, using a lambda:

```python
sorted(t, key=lambda s: len(s))
```

#### Stable Sorting


You might have noticed that the words `this`,  `late` and `bird` all have four characters - so how did Python determine which one should come first? Randomly? No!

The sort algorithm that Python uses, called the *TimSort* (named after Python core developer Tim Peters - yes, the same Tim Peters that wrote the Zen of Python!!), is what is called a **stable** sort algorithm.

This means that items with equal sort keys maintain their relative position.


but first:

```python
import this
```

If you haven't read this in a while, take a few minutes now to do so again!


Now back to stable sorting:

```python
t = 'aaaa', 'bbbb', 'cccc', 'dddd', 'eeee'
```

```python
sorted(t, key = lambda s: len(s))
```

Now let's change our tuple a bit:

```python
t = 'bbbb', 'cccc', 'aaaa', 'eeee', 'dddd'
```

```python
sorted(t, key = lambda s: len(s))
```

As you can see, when the sort keys are equal (they are all equal to 4), the original ordering of the iterable is preserved.

So in our original example:

```python
t = 'this', 'parrot', 'is', 'a', 'late', 'bird'
```

```python
sorted(t, key = lambda s: len(s))
```

So, `this`, will come before `late` which will come before `bird`.

If we change it up a bit:

```python
t = 'this', 'bird', 'is', 'a', 'late', 'parrot'
sorted(t, key = lambda s: len(s))
```

you'll notice that now `bird` ends up before `late`.


So this `key` argument makes the `sorted` function extremely flexible. We can now even sort objects that are not even comparable!

```python
c1 = 10 + 2j
c2 = 5 - 3j
```

```python
c1 < c2
```

As you can we do not have an ordering defined for complex numbers.

But we may want to sort a sequence of complex numbers based on their distance from the origin:

```python
t = 0, 10+10j, 3-3j, 4+4j, 5-2j
```

We can easily calculate the distace from the origin by using the `abs` function:

```python
abs(3+4j)
```

So now we can use that as a sort key:

```python
sorted(t, key=abs)
```

Of course, you could decide to sort based on the imaginary component instead:

```python
sorted(t, key=lambda c: c.imag)
```

#### Reversed Sort


We also have the `reverse` keyword-only argument that we can use - basically it sorts the iterable, but returns it reversed:

```python
t = 'this', 'bird', 'is', 'a', 'late', 'parrot'
```

```python
sorted(t, key=lambda s: len(s))
```

```python
sorted(t, key=lambda s: len(s), reverse=True)
```

Of course in this case we could have done it this way too:

```python
sorted(t, key=lambda s: -len(s))
```

#### In-Place Sorting


So far we have seen the `sorted` function - it returns a new (list) containing the sorted elements, and the original iterable remains the same.

But mutable sequence types, such as lists, also implement in-place sorting - where the original list is sorted (the memory address does not change, the object is actually mutated).

The syntax for calling the sorted method is identical to the `sorted` function, and is implemented using the same TimSort algorithm.

Of course, this will not work with tuples, which are immutable.

```python
l = ['this', 'bird', 'is', 'a', 'late', 'parrot']
```

```python
id(l)
```

```python
sorted(l, key=lambda s: len(s))
```

```python
l, id(l)
```

As you can see, the list `l` was not mutated and is still the same object.

But this way is different:

```python
result = l.sort(key=lambda s: len(s))
```

First, the `sort` **method** does not return anything:

```python
type(result)
```

and the original list is still the same object:

```python
id(l)
```

but it has mutated:

```python
l
```

That's really the only fundamental difference between the two sorts - one is in-place, while the other is not.


You might be wondering if one is more efficient than the other. 

As far as algorithms go, they are the same, so no difference there (one sort is not more efficient than the other). 

But `list.sort()` will be faster than `sorted()` because it does not have to create a copy of the sequence. 

Of course, for iterables other than lists, you don't have much of a choice, and need to use `sorted` anyways.


Let's try timing this a bit to see if we can see the difference:

```python
from timeit import timeit
import random
```

```python
random.seed(0)
n = 10_000_000
l = [random.randint(0, 100) for n in range(n)]
```

This produces a list of `n` random integers between 0 and 100. 

If you're wondering about what the seed does, look at my video on random seeds in Part 1|Extras of this course - basically it makes sure I will generate the same random sequence every time.

If you're unsure about the `timeit` module, again I have a video on that in Part 1|Extras of this course.


Now, I'm only going to run the tests once, because when using in-place sorting of `l` we'll end up sorting an already sorted list - and that may very well affect the timing...

```python
timeit(stmt='sorted(l)', globals=globals(), number=1)
```

```python
timeit(stmt='l.sort()', globals=globals(), number=1)
```

As you can see, the time difference between the two methods, even for `n=10_000_000` is quite small.


I also just want to point out that sorting a list that is already sorted results in much better performance!

```python
random.seed(0)
n = 10_000_000
l = [random.randint(0, 100) for n in range(n)]
timeit(stmt='l.sort()', globals=globals(), number=1)
```

So now `l` is sorted, and if re-run the sort on it (either method), here's what we get:

```python
timeit(stmt='sorted(l)', globals=globals(), number=1)
```

```python
timeit(stmt='l.sort()', globals=globals(), number=1)
```

Substantially faster!!

Hence why I only timed using a single iteration...


#### Natural Ordering for Custom Classes


I just want to quickly show you that in order to have a "natural ordering" for our custom classes, we just need to implement the `<` or `>` operators. (I discuss these operators in Part 1 of this course)

```python
class MyClass:
    def __init__(self, name, val):
        self.name = name
        self.val = val
        
    def __repr__(self):
        return f'MyClass({self.name}, {self.val})'
    
    def __lt__(self, other):
        return self.val < other.val
```

```python
c1 = MyClass('c1', 20)
c2 = MyClass('c2', 10)
c3 = MyClass('c3', 20)
c4 = MyClass('c4', 10)
```

Now we can sort those objects, without specifying a key, since that class has a natural ordering (`<` in this case). Moreover, notice that the sort is stable.

```python
sorted([c1, c2, c3, c4])
```

In fact, we can modify our class slightly so we can see that `sorted` is calling our `__lt__` method repeatedly to perform the sort:

```python
class MyClass:
    def __init__(self, name, val):
        self.name = name
        self.val = val
        
    def __repr__(self):
        return f'MyClass({self.name}, {self.val})'
    
    def __lt__(self, other):
        print(f'called {self.name} < {other.name}')
        return self.val < other.val
```

```python
c1 = MyClass('c1', 20)
c2 = MyClass('c2', 10)
c3 = MyClass('c3', 20)
c4 = MyClass('c4', 10)
```

```python
sorted([c1, c2, c3, c4])
```
