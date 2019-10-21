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

### Random Samples


I just want to show you a variant on `random.choices` that we saw in the previous video.


`choices` chooses `k` random elements from some sequence, **with replacement**.

This means we could create a random selection containing more elements than we started off with:

```python
import random
```

```python
random.choices(list('abc'), k=10)
```

Sometimes however, we do not want that replacement - instead we want a population sample (so once an element has been randomly selected, it cannot be selected again).

This is where the `sample` function comes in - it does exactly that. Of course, we can no longer pick more elements than we have in our population. Also, picking a sample equal in size to the population basically returns a "shuffled" population.

```python
l = range(20)
```

```python
random.sample(l, k=10)
```

We can even set the sample size equal to the population size:

```python
random.sample(l, k=20)
```

But no larger than the population size:

```python
random.sample(l, 50)
```

Also worth pointing out is that if you set a specific seed, you will get repeatability of your sample selection:

```python
random.seed(0)
random.sample(l, k=5)
```

```python
random.seed(0)
random.sample(l, k=5)
```

Let's see how we might use this to select some cards from a deck - obviously we don't want replacement here - once a card has ben picked from a deck it's no longer available for a second random pick.

```python
suits = 'C', 'D', 'H', 'A'
ranks = tuple(range(2,11)) + tuple('JQKA')
```

```python
suits
```

```python
ranks
```

Now we have to combine suits and ranks to form a deck.

```python
deck = [str(rank) + suit for suit in suits for rank in ranks]
```

```python
print(deck)
```

Let's import `Counter` from the collections module to make sure we have no repitition when we pull a sample vs when we use `choices`.

```python
from collections import Counter
```

```python
Counter(random.sample(deck, k=20))
```

But if we used `choices` most likely we'll get some repetitions:

```python
Counter(random.choices(deck, k=20))
```
