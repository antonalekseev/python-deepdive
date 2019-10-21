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

### Reversed Iteration


Sometimes we may want to iterate through an iterable but in **reverse** order.

Of course, this means the collection being iterated must be finite.


Python has a built-in function called `reversed()` to do this that will work with any type that implement the sequence protocol. But for iterables in general it's a little more complicated.

Let's first build a custom iterable.


For this example we are going to build a custom iterable that returns cards from a 52-card deck.

The deck will be in order of suits (Spades, Hearts, Diamonds and Clubs) and card values (from 2 (lowest) to Ace (highest)).

We are going to use lazy loading - i.e. we are not going to pre-build our card deck.


We just need to recognize that each suit contains `13` cards, so an integer division of the index of the card in the deck will tell us which suit it is. But of course we start indexing at 0.

**Example**

If the requested card is the `6`th in the deck (i.e. index = `5`):

`5 // 13 = 0` ==> first suit (Spades)

If the requested card is the `13`th in the deck (i.e. index = `12`):

`12 // 13 = 0` ==> first suit (Spades)

If the requested card is the `14`th in the deck (i.e. index = `13`):

`13 // 13 = 1` ==> second suit (Hearts)


To determine which card in the suit we are interested in, we simply need to use the `%` operator, again recognizing that there are `13` cards in each suit:

**Example**

If the requested card is the `6`th in the deck (i.e. index = `5`):

`5 % 13 = 5` ==> `5`th card in the suit

If the requested card is the `13`th in the deck (i.e. index = `12`):

`12 % 13 = 12` ==> `12`th card in the suit

If the requested card is the `14`th in the deck (i.e. index = `13`):

`13 % 13 = 0` ==> `1`st card in the suit

```python
_SUITS = ('Spades', 'Hearts', 'Diamonds', 'Clubs')
_RANKS = tuple(range(2, 11) ) + tuple('JQKA')
from collections import namedtuple

Card = namedtuple('Card', 'rank suit')

class CardDeck:
    def __init__(self):
        self.length = len(_SUITS) * len(_RANKS)

    def __len__(self):
        return self.length
    
    def __iter__(self):
        return self.CardDeckIterator(self.length)
        
    class CardDeckIterator:
        def __init__(self, length):
            self.length = length
            self.i = 0
            
        def __iter__(self):
            return self
        
        def __next__(self):
            if self.i >= self.length:
                raise StopIteration
            else:
                suit = _SUITS[self.i // len(_RANKS)]
                rank = _RANKS[self.i % len(_RANKS)]
                self.i += 1
                return Card(rank, suit)
```

We can now iterate over a deck of cards as follows:

```python
deck = CardDeck()
```

```python
for card in deck:
    print(card)
```

Now that we have our deck, how would we obtain the last `7` cards in reverse order from the deck?


One option is to generate a list of all the cards in the deck, then use a slice.


What about iterating in reverse? Using the same technique we generate a list that contains all the cards, reverse the list, and then iterate over the reversed list.

```python
deck = list(CardDeck())
```

```python
deck[:-8:-1]
```

And to iterate backwards:

```python
deck = list(CardDeck())
deck = deck[::-1]
for card in deck:
    print(card)
```

This is kind of inefficient since we had to generate the entire list of cards, to then reverse it, and then only pick the first 7 cards from that reversed list.


Maybe we can try Python's built-in `reversed` function instead:

```python
deck = CardDeck()
```

```python
deck = reversed(deck)
```

As we can see, Python's `reversed` function will not work with out iterator. (It would work automatically with a sequence type, but in this case we don't have a sequence type)

What to do?


We need to somehow define a "reverse" iteration option for our iterator!

We do so by defining the __reversed__ special method in our iterable and instructing out iterator to return elements in reverse order.

If the `__reversed__` method is in our iterable, Python will use that to get the iterator when we call the `reverse()` function:

Let's try that out:

```python
_SUITS = ('Spades', 'Hearts', 'Diamonds', 'Clubs')
_RANKS = tuple(range(2, 11) ) + ('J', 'Q', 'K', 'A')
from collections import namedtuple

Card = namedtuple('Card', 'rank suit')

class CardDeck:
    def __init__(self):
        self.length = len(_SUITS) * len(_RANKS)

    def __len__(self):
        return self.length
    
    def __iter__(self):
        return self.CardDeckIterator(self.length)
        
    def __reversed__(self):
        return self.CardDeckIterator(self.length, reverse=True)
    
    class CardDeckIterator:
        def __init__(self, length, *, reverse=False):
            self.length = length
            self.reverse = reverse
            self.i = 0
            
        def __iter__(self):
            return self
        
        def __next__(self):
            if self.i >= self.length:
                raise StopIteration
            else:
                if self.reverse:
                    index = self.length -1 - self.i
                else:
                    index = self.i
                suit = _SUITS[index // len(_RANKS)]
                rank = _RANKS[index % len(_RANKS)]
                self.i += 1
                return Card(rank, suit)
            

```

```python
deck = CardDeck()
```

```python
for card in deck:
    print(card)
```

```python
deck = reversed(CardDeck())
for card in deck:
    print(card)
```

#### Reversing Sequences


I just want to point out that if we have a custom **sequence** type we don't need to worry about this.

Let's see a quick example:

```python
class Squares:
    def __init__(self, length):
        self.squares = [i **2 for i in range(length)]
        
    def __len__(self):
        return len(self.squares)
    
    def __getitem__(self, s):
        return self.squares[s]
```

```python
sq = Squares(10)
```

```python
for num in Squares(5):
    print(num)
```

```python
for num in reversed(Squares(5)):
    print(num)
```

As you can see Python was able to automatically reverse the sequence for us.


Also worth noting is that the `__len__` method **must** be implemented for `reversed()` to work:

```python
class Squares:
    def __init__(self, length):
        self.squares = [i **2 for i in range(length)]
        
#     def __len__(self):
#         return len(self.squares)
    
    def __getitem__(self, s):
        return self.squares[s]
```

```python
for num in reversed(Squares(5)):
    print(num)
```

In addition, we can override what is returned when the `reversed()` function is called on our custom sequence type. Here, I'll return the list of the integers themselves instead of squares just to make this really stand out:

```python
class Squares:
    def __init__(self, length):
        self.length = length
        self.squares = [i **2 for i in range(length)]
        
    def __len__(self):
        return len(self.squares)
    
    def __getitem__(self, s):
        return self.squares[s]
    
    def __reversed__(self):
        print('__reversed__ called')
        return [i for i in range(self.length-1, -1, -1)]
```

```python
for num in Squares(5):
    print(num)
```

```python
for num in reversed(Squares(5)):
    print(num)
```

```python

```
