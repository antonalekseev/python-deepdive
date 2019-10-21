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

### Infinite Iterators


There are three functions in the `itertools` module that produce infinite iterators: `count`, `cycle` and `repeat`.

```python
from itertools import (
    count,
    cycle,
    repeat, 
    islice)
```

#### count


The `count` function is similar to range, except it does not have a `stop` value. It has both a `start` and a `step`:

```python
g = count(10)
```

```python
list(islice(g, 5))
```

```python
g = count(10, step=2)
```

```python
list(islice(g, 5))
```

And so on. 

Unlike the `range` function, whose arguments must always be integers, `count` works with floats as well:

```python
g = count(10.5, 0.5)
```

```python
list(islice(g, 5))
```

In fact, we can even use other data types as well:

```python
g = count(1+1j, 1+2j)
```

```python
list(islice(g, 5))
```

We can even use Decimal numbers:

```python
from decimal import Decimal
```

```python
g = count(Decimal('0.0'), Decimal('0.1'))
```

```python
list(islice(g, 5))
```

### Cycle


`cycle` is used to repeatedly loop over an iterable:

```python
g = cycle(('red', 'green', 'blue'))
```

```python
list(islice(g, 8))
```

One thing to note is that this works **even** if the argument is an iterator (i.e. gets exhausted after the first complete iteration over it)!


Let's see a simple example of this:

```python
def colors():
    yield 'red'
    yield 'green'
    yield 'blue'
```

```python
cols = colors()
```

```python
list(cols)
```

```python
list(cols)
```

As expected, `cols` was exhausted after the first iteration.

Now let's see how `cycle` behaves:

```python
cols = colors()
g = cycle(cols)
```

```python
list(islice(g, 10))
```

As you can see, `cycle` iterated over the elements of iterator, and continued the iteration even though the first run through the iterator technically exhausted it.


##### Example


A simple application of `cycle` is dealing a deck of cards into separate hands:

```python
from collections import namedtuple
```

```python
Card = namedtuple('Card', 'rank suit')
```

```python
def card_deck():
    ranks = tuple(str(num) for num in range(2, 11)) + tuple('JQKA')
    suits = ('Spades', 'Hearts', 'Diamonds', 'Clubs')
    for suit in suits:
        for rank in ranks:
            yield Card(rank, suit)
```

Assume we want 4 hands, so we can think of the hands as a list containing 4 elements - each of which is itself a list containing cards.

The indices of the hands would be `0, 1, 2, 3` in the hands list:


We could certainly do it this way:

```python
hands = [list() for _ in range(4)]
```

```python
hands
```

```python
index = 0
for card in card_deck():
    index = index % 4
    hands[index].append(card)
    index += 1
```

```python
hands
```

You notice how we had to use the `mod` operator and an `index` to **cycle** through the hands.

So, we can use the `cycle` function instead:

```python
hands = [list() for _ in range(4)]
```

```python
index_cycle = cycle([0, 1, 2, 3])
for card in card_deck():
    hands[next(index_cycle)].append(card)
```

```python
hands
```

But we really can simplify this even further - why are we cycling through the indices? Why not simply cycle through the hand themselves, and append the card to the hands?

```python
hands = [list() for _ in range(4)]
```

```python
hands_cycle = cycle(hands)
for card in card_deck():
    next(hands_cycle).append(card)
```

```python
hands
```

#### Repeat


The `repeat` function is used to create an iterator that just returns the same value again and again. By default it is infinite, but a count can be specified optionally:

```python
g = repeat('Python')
for _ in range(5):
    print(next(g))
```

And we also optionally specify a count to make the iterator finite:

```python
g = repeat('Python', 4)
```

```python
list(g)
```

The important thing to note as well, is that the "value" that is returned is the **same** object every time!


Let's see this:

```python
l = [1, 2, 3]
```

```python
result = list(repeat(l, 3))
```

```python
result
```

```python
l is result[0], l is result[1], l is result[2]
```

So be careful here. If you try to use repeat to create three separate instances of a list, you'll actually end up with shared references:

```python
result[0], result[1], result[2]
```

```python
result[0][0] = 100
```

```python
result[0], result[1], result[2]
```

If you want to end up with three separate copies of your argument, then you'll need to use a copy mechanism (either shallow or deep depending on your needs).

This is easily done using a comprehension expression:

```python
l = [1, 2, 3]
result = [item[:] for item in repeat(l, 3)]
```

```python
result
```

```python
l is result[0], l is result[1], l is result[2]
```

```python
result[0][0] = 100
```

```python
result
```
