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

### Coroutines


What is a coroutine?

The word co actually comes from **cooperative**.

A coroutine is a generalization of subroutines (think functions), focused on **cooperation** between routines.

If you have some concepts of multi-threading, this is similar in some ways. But whereas in multi-threaded applications the **operating system** usually decides when to suspend one thread and resume another, without asking permission, so-called **preemptive** multitasking, here we have routines that voluntarily yield control to something else - hence the term **cooperative**.

We actually have all the tools we need to start looking at this.

It is the `yield` statement we studied in the last section on generator functions.

Let's dig a little further to truly understand what coroutines are and how they can be used.


We'll need to first define quickly what a queue is.

It is a collection where items are added to the back of the queue, and items are removed from the end of the queue. So, very similar to a queue in a supermarket - you join the queue at the back of the queue, and the person in the front of the queue is the first one to leave the queue and go to the checkout counter.

This is also called a First-In First-Out data structure.

(For comparison, you also have a **stack** which is like a stack of pancakes - the last cooked pancake is placed on top of the stack of pancakes (called a **push**), and it's the first one you take fomr the stack and eat (called a **pop**) - so that is called First-In Last-Out)

We can just use a simple list to act as queue, but lists are not particularly effecient when adding elements to the beginning of the list - they are fine for adding element to the end, but less so at inserting elements, including at the front.

So, instead of using a list, let's just use a more efficient data structure for our queue.

The `queue` module has some queue implementations, including some very specialized ones. In Python 3.7, it also has the `SimpleQueue` class that is more lightweight.

In this case though, I'm going to use the `deque` class (double-ended queue) from the `collections` module - it is very efficient adding and removing elements from both the start and the end of the queue - so, it's very general purpose and widely used. The `queue` implementations are more specialized and have several features useful for multi-tasking that we won't actually need.

```python
from collections import deque
```

We can specify a maximum size for the queue when create it - this allows us to limit the number of items in the queue. 

We can then add and remove items by using the methods:
* `append`: appends an element to the right of the queue
* `appendleft`: appends an element to the left of the queue
* `pop`: remove and return the element at the very right of the queue
* `popleft`: remove and return the element at the very left of the queue

(Note that I'm avoiding calling it the start and end of the queue, because what you consider the start/end of the queue might depend on how you are using it)


Let's just try it out to make sure we're comfortable with it:

```python
dq = deque([1, 2, 3, 4, 5])
dq
```

```python
dq.append(100)
dq
```

```python
dq
```

```python
dq.appendleft(-10)
dq
```

```python
dq.pop()
```

```python
dq
```

```python
dq.popleft()
```

```python
dq
```

We can create a capped queue:

```python
dq = deque([1, 2, 3, 4], maxlen=5)
```

```python
dq.append(100)
dq
```

```python
dq.append(200)
dq
```

```python
dq.append(300)
dq
```

As you can see the first item (`2`) was automatically discarded from the left of the queue when we added `300` to the right.


We can also find the number of elements in the queue by using the `len()` function:

```python
len(dq)
```

as well as query the `maxlen`:

```python
dq.maxlen
```

There are more methods, but these will do for now.


Now let's create an empty queue, and write two functions - one that will add elements to the queue, and one that will consume elements from the queue:

```python
def produce_elements(dq):
    for i in range(1, 36):
        dq.appendleft(i)
```

```python
def consume_elements(dq):
    while len(dq) > 0:
        item = dq.pop()
        print('processing item', item)
```

Now we can use them as follows:

```python
def coordinator():
    dq = deque()
    producer = produce_elements(dq)
    consume_elements(dq)
```

```python
coordinator()
```

But suppose now that the `produce_elements` function is reading a ton of data from somewhere (maybe an API call that returns course ratings on some Python course :-) ).

The goal is to process these after some time, and not wait until all the items have been added to the queue - maybe the incoming stream is infinite even.

In that case, we want to "pause" adding elements to the queue, process (consume) those items, then once they've all been processed we want to resume adding elements, and rinse and repeat.


We'll use a capped `deque`, and change our producer and consumers slightly, so that each one does it's work, the yields control back to the caller once it's done with its work - the producer adding elements to the queue, and the consumer removing and processing elements from the queue:

```python
def produce_elements(dq, n):
    for i in range(1, n):
        dq.appendleft(i)
        if len(dq) == dq.maxlen:
            print('queue full - yielding control')
            yield
        
def consume_elements(dq):
    while True:
        while len(dq) > 0:
            print('processing ', dq.pop())
        print('queue empty - yielding control')
        yield
    
def coordinator():
    dq = deque(maxlen=10)
    producer = produce_elements(dq, 36)
    consumer = consume_elements(dq)
    while True:
        try:
            print('producing...')
            next(producer)
        except StopIteration:
            # producer finished
            break
        finally:
            print('consuming...')
            next(consumer)
```

```python
coordinator()
```

Notice a **really important** point here - the producer and consumer generator functions do not use `yield` for iteration purposes - they are simply using `yield` to suspend themselves and cooperatively hand control back to the caller - our coordinator function in this case.

The generators used `yield` to cooperatively suspend themselves and yield control back to the caller.

Similarly, we are not using `next` for iteration purposes, but more for starting and resuming the generators.

This is a fundamentally different idea than using `yield` to implement iterators, and forms the basis for the idea of using generators as coroutines.


### Timings using Lists and Deques for Queues


Let's see some timing differences between `lists` and `deques` when inserting and popping elements. We'll compare this with appending elements to a `list` as well.

```python
from timeit import timeit
```

```python
list_size = 10_000

def append_to_list(n=list_size):
    lst = []
    for i in range(n):
        lst.append(i)

def insert_front_of_list(n=list_size):
    lst = []
    for i in range(n):
        lst.insert(0, i)
        
lst = [i for i in range(list_size)]
def pop_from_list(lst=lst):
    for _ in range(len(lst)):
        lst.pop()
        
lst = [i for i in range(list_size)]
def pop_from_front_of_list(lst=lst):
    for _ in range(len(lst)):
        lst.pop(0)
```

Let's time those out:

```python
timeit('append_to_list()', globals=globals(), number=1_000)
```

```python
timeit('insert_front_of_list()', globals=globals(), number=1_000)
```

```python
timeit('pop_from_list()', globals=globals(), number=1_000)
```

```python
timeit('pop_from_front_of_list()', globals=globals(), number=1_000)
```

As you can see, insert elements at the front of the list is not very efficient compared to the end of the list. So lists are OK to use as stacks, but not as queues.

The standard library's `deque` is efficient at adding/removing items from both the start and end of the collection:

```python
from collections import deque
```

```python
list_size = 10_000

def append_to_deque(n=list_size):
    dq = deque()
    for i in range(n):
        dq.append(i)

def insert_front_of_deque(n=list_size):
    dq = deque()
    for i in range(n):
        dq.appendleft(i)
        
dq = deque(i for i in range(list_size))
def pop_from_deque(dq=dq):
    for _ in range(len(lst)):
        dq.pop()
        
dq = deque(i for i in range(list_size))
def pop_from_front_of_deque(dq=dq):
    for _ in range(len(lst)):
        dq.popleft()
```

```python
timeit('append_to_deque()', globals=globals(), number=1_000)
```

```python
timeit('insert_front_of_deque()', globals=globals(), number=1_000)
```

```python
timeit('pop_from_deque()', globals=globals(), number=1_000)
```

```python
timeit('pop_from_front_of_deque()', globals=globals(), number=1_000)
```

```python

```
