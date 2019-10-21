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

### Counter


The `Counter` dictionary is one that specializes for helping with, you guessed it, counters!

Actually we used a `defaultdict` earlier to do something similar:

```python
from collections import defaultdict, Counter
```

Let's say we want to count the frequency of each character in a string:

```python
sentence = 'the quick brown fox jumps over the lazy dog'
```

```python
counter = defaultdict(int)
```

```python
for c in sentence:
    counter[c] += 1
```

```python
counter
```

We can do the same thing using a `Counter` - unlike the `defaultdict` we don't specify a default factory - it's always zero (it's a counter after all):

```python
counter = Counter()
for c in sentence:
    counter[c] += 1
```

```python
counter
```

OK, so if that's all there was to `Counter` it would be pretty odd to have a data structure different than `OrderedDict`.

But `Counter` has a slew of additional methods which make sense in the context of counters:

1. Iterate through all the elements of counters, but repeat the elements as many times as their frequency
2. Find the `n` most common (by frequency) elements
3. Decrement the counters based on another `Counter` (or iterable)
4. Increment the counters based on another `Counter` (or iterable)
5. Specialized constructor for additional flexibility

If you are familiar with multisets, then this is essentially a data structure that can be used for multisets.


#### Constructor


It is so common to create a frequency distribution of elements in an iterable, that this is supported automatically:

```python
c1 = Counter('able was I ere I saw elba')
c1
```

Of course this works for iterables in general, not just strings:

```python
import random
```

```python
random.seed(0)
```

```python
my_list = [random.randint(0, 10) for _ in range(1_000)]
```

```python
c2 = Counter(my_list)
```

```python
c2
```

We can also initialize a `Counter` object by passing in keyword arguments, or even a dictionary:

```python
c2 = Counter(a=1, b=10)
c2
```

```python
c3 = Counter({'a': 1, 'b': 10})
c3
```

Technically we can store values other than integers in a `Counter` object - it's possible but of limited use since the default is still `0` irrespective of what other values are contained in the object.


#### Finding the n most Common Elements


Let's find the `n` most common words (by frequency) in a paragraph of text. Words are considered delimited by white space or punctuation marks such as `.`, `,`, `!`, etc - basically anything except a character or a digit.
This is actually quite difficult to do, so we'll use a close enough approximation that will cover most cases just fine, using a regular expression:

```python
import re
```

```python
sentence = '''
his module implements pseudo-random number generators for various distributions.

For integers, there is uniform selection from a range. For sequences, there is uniform selection of a random element, a function to generate a random permutation of a list in-place, and a function for random sampling without replacement.

On the real line, there are functions to compute uniform, normal (Gaussian), lognormal, negative exponential, gamma, and beta distributions. For generating distributions of angles, the von Mises distribution is available.

Almost all module functions depend on the basic function random(), which generates a random float uniformly in the semi-open range [0.0, 1.0). Python uses the Mersenne Twister as the core generator. It produces 53-bit precision floats and has a period of 2**19937-1. The underlying implementation in C is both fast and threadsafe. The Mersenne Twister is one of the most extensively tested random number generators in existence. However, being completely deterministic, it is not suitable for all purposes, and is completely unsuitable for cryptographic purposes.'''
```

```python
words = re.split('\W', sentence)
```

```python
words
```

But what are the frequencies of each word, and what are the 5 most frequent words?

```python
word_count = Counter(words)
```

```python
word_count
```

```python
word_count.most_common(5)
```

#### Using Repeated Iteration

```python
c1 = Counter('abba')
c1
```

```python
for c in c1:
    print(c)
```

However, we can have an iteration that repeats the counter keys as many times as the indicated frequency:

```python
for c in c1.elements():
    print(c)
```

What's interesting about this functionality is that we can turn this around and use it as a way to create an iterable that has repeating elements.

Suppose we want to to iterate through a list of (integer) numbers that are each repeated as many times as the number itself.

For example 1 should repeat once, 2 should repeat twice, and so on.

This is actually not that easy to do!

Here's one possible way to do it:

```python
l = []
for i in range(1, 11):
    for _ in range(i):
        l.append(i)
print(l)
```

But we could use a `Counter` object as well:

```python
c1 = Counter()
for i in range(1, 11):
    c1[i] = i
```

```python
c1
```

```python
print(c1.elements())
```

So you'll notice that we have a `chain` object here. That's one big advantage to using the `Counter` object - the repeated iterable does not actually exist as list like our previous implementation - this is a lazy iterable, so this is far more memory efficient.

And we can iterate through that `chain` quite easily:

```python
for i in c1.elements():
    print(i, end=', ')
```

Just for fun, how could we reproduce this functionality using a plain dictionary?

```python
class RepeatIterable:
    def __init__(self, **kwargs):
        self.d = kwargs
        
    def __setitem__(self, key, value):
        self.d[key] = value
        
    def __getitem__(self, key):
        self.d[key] = self.d.get(key, 0)
        return self.d[key]
```

```python
r = RepeatIterable(x=10, y=20)
```

```python
r.d
```

```python
r['a'] = 100
```

```python
r['a']
```

```python
r['b']
```

```python
r.d
```

Now we have to implement that `elements` iterator:

```python
class RepeatIterable:
    def __init__(self, **kwargs):
        self.d = kwargs
        
    def __setitem__(self, key, value):
        self.d[key] = value
        
    def __getitem__(self, key):
        self.d[key] = self.d.get(key, 0)
        return self.d[key]
    
    def elements(self):
        for k, frequency in self.d.items():
            for i in range(frequency):
                yield k
```

```python
r = RepeatIterable(a=2, b=3, c=1)
```

```python
for e in r.elements():
    print(e, end=', ')
```

#### Updating from another Iterable or Counter


Lastly let's see how we can update a `Counter` object using another `Counter` object. 

When both objects have the same key, we have a choice - do we add the count of one to the count of the other, or do we subtract them?

We can do either, by using the `update` (additive) or `subtract` methods.

```python
c1 = Counter(a=1, b=2, c=3)
c2 = Counter(b=1, c=2, d=3)

c1.update(c2)
print(c1)
```

On the other hand we can subtract instead of add counters:

```python
c1 = Counter(a=1, b=2, c=3)
c2 = Counter(b=1, c=2, d=3)

c1.subtract(c2)
print(c1)
```

Notice the key `d` - since `Counters` default missing keys to `0`, when `d: 3` in `c2` was subtracted from `c1`, the counter for `d` was defaulted to `0`.


Just as the constructor for a `Counter` can take different arguments, so too can the `update` and `subtract` methods.

```python
c1 = Counter('aabbccddee')
print(c1)
c1.update('abcdef')
print(c1)
```

#### Mathematical Operations


These `Counter` objects also support several other mathematical operations when both operands are `Counter` objects. In all these cases the result is a new `Counter` object.

* `+`: same as `update`, but returns a new `Counter` object instead of an in-place update.
* `-`: subtracts one counter from another, but discards zero and negative values
* `&`: keeps the **minimum** of the key values
* `|`: keeps the **maximum** of the key values

```python
c1 = Counter('aabbcc')
c2 = Counter('abc')
c1 + c2
```

```python
c1 - c2
```

```python
c1 = Counter(a=5, b=1)
c2 = Counter(a=1, b=10)

c1 & c2
```

```python
c1 | c2
```

The **unary** `+` can also be used to remove any non-positive count from the Counter:

```python
c1 = Counter(a=10, b=-10)
+c1
```

The **unary** `-` changes the sign of each counter, and removes any non-positive result:

```python
-c1
```

##### Example


Let's assume you are working for a company that produces different kinds of widgets.
You are asked to identify the top 3 best selling widgets.

You have two separate data sources - one data source can give you a history of all widget orders (widget name, quantity), while another data source can give you a history of widget refunds (widget name, quantity refunded).

From these two data sources, you need to determine the top selling widgets (taking refinds into account of course).


Let's simulate both of these lists:

```python
import random
random.seed(0)

widgets = ['battery', 'charger', 'cable', 'case', 'keyboard', 'mouse']

orders = [(random.choice(widgets), random.randint(1, 5)) for _ in range(100)]
refunds = [(random.choice(widgets), random.randint(1, 3)) for _ in range(20)]
```

```python
orders
```

```python
refunds
```

Let's first load these up into counter objects.

To do this we're going to iterate through the various lists and update our counters:

```python
sold_counter = Counter()
refund_counter = Counter()

for order in orders:
    sold_counter[order[0]] += order[1]

for refund in refunds:
    refund_counter[refund[0]] += refund[1]
```

```python
sold_counter
```

```python
refund_counter
```

```python
net_counter = sold_counter - refund_counter
```

```python
net_counter
```

```python
net_counter.most_common(3)
```

We could actually do this a little differently, not using loops to populate our initial counters.

Recall the `repeat()` function in `itertools`:

```python
from itertools import repeat
```

```python
list(repeat('battery', 5))
```

```python
orders[0]
```

```python
list(repeat(*orders[0]))
```

So we could use the `repeat()` method to essentially repeat each widget for each item of `orders`. We need to chain this up for each element of `orders` - this will give us a single iterable that we can then use in the constructor for a `Counter` object. We can do this using a generator expression for example:

```python
from itertools import chain
```

```python
list(chain.from_iterable(repeat(*order) for order in orders))
```

```python
order_counter = Counter(chain.from_iterable(repeat(*order) for order in orders))
```

```python
order_counter
```

```
#### Alternate Solution not using Counter
```

What if we don't want to use a `Counter` object.
We can still do it (relatively easily) as follows:

```python
net_sales = {}
for order in orders:
    key = order[0]
    cnt = order[1]
    net_sales[key] = net_sales.get(key, 0) + cnt
    
for refund in refunds:
    key = refund[0]
    cnt = refund[1]
    net_sales[key] = net_sales.get(key, 0) - cnt

# eliminate non-positive values (to mimic what - does for Counters)
net_sales = {k: v for k, v in net_sales.items() if v > 0}

# we now have to sort the dictionary
# this means sorting the keys based on the values
sorted_net_sales = sorted(net_sales.items(), key=lambda t: t[1], reverse=True)

# Top three
sorted_net_sales[:3]
```
