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

### Project 1: Transaction Numbers


Next I want something for the transaction ID logic.

Basically, we just need an initialized counter that returns the next value every time it is called.

We have different ways of doing this - since this is an OOP course, you might be tempted to jump straight into a class implementation for this.

We could do it this way:

```python
class TransactionID:
    def __init__(self, start_id):
        self._start_id = start_id
        
    def next(self):
        self._start_id += 1
        return self._start_id
```

We could then use an instance of this class as a class attribute for `Account` thusly:

```python
class Account:
    transaction_counter = TransactionID(100)
    
    def make_transaction(self):
        new_trans_id = Account.transaction_counter.next()
        return new_trans_id
```

```python
a1 = Account()
a2 = Account()

print(a1.make_transaction())
print(a2.make_transaction())
print(a1.make_transaction())
```

So, that works just fine, but if we think about this a bit more, we should see that we really do not need a class to solve this particular problem. We could even implement the iterator protocol for this class (implementing `__iter__` and `__next__`), but a simple infinite generator will work just as well.

So moral of the story here, don't just jump into solving every problem using classes - this isn't Java, we don't need a class for everything!


Instead, I'm going to implement it this way:

```python
def transaction_ids(start_id):
    while True:
        start_id += 1
        yield start_id
```

So we can use it this way:

```python
class Account:
    transaction_counter = transaction_ids(100)
    
    def make_transaction(self):
        new_trans_id = next(Account.transaction_counter)
        return new_trans_id
```

```python
a1 = Account()
a2 = Account()

print(a1.make_transaction())
print(a2.make_transaction())
print(a1.make_transaction())
```

So this works equally well, but if we recall the `counter` method in the `itertools` module, we can simplify this even further:

```python
import itertools

class Account:
    transaction_counter = itertools.count(100)
    
    def make_transaction(self):
        new_trans_id = next(Account.transaction_counter)
        return new_trans_id
```

```python
a1 = Account()
a2 = Account()

print(a1.make_transaction())
print(a2.make_transaction())
print(a1.make_transaction())
```

```python

```
