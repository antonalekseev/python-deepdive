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

### Iterators


In the last lecture we saw that we could approach iterating over a collection using this concept of `next`.

But there were some downsides that did not resolve (yet!):
* we cannot use a `for` loop
* once we exhaust the iteration (repeatedly calling next), we're essentially done with object. The only way to iterate through it again is to create a new instance of the object.


First we are going to look at making our `next` be usable in a for loop.


This idea of using `__next__` and the `StopIteration` exception is exactly what Python does.

So, somehow we need to tell Python that the object we are dealing with can be used with `next`.

To do so, we create an `iterator` type object.


Iterators are objects that implement:
* a `__next__` method
* an `__iter__` method that simply returns the object itself


That's it - that's all there is to an iterator - two methods, `__iter__` and `__next__`.


Let's go back to our `Squares` example:

```python
class Squares:
    def __init__(self, length):
        self.length = length
        self.i = 0
        
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.i >= self.length:
            raise StopIteration
        else:
            result = self.i ** 2
            self.i += 1
            return result
```

Now we can still call `next`:

```python
sq = Squares(5)
```

```python
print(next(sq))
print(next(sq))
print(next(sq))
```

Of course, our iterator still suffers from not being able to "reset" it - we just have to create a new instance:

```python
sq = Squares(5)
```

But now, we can also use a `for` loop:

```python
for item in sq:
    print(item)
```

Now `sq` is **exhausted**, so if we try to loop through again:

```python
for item in sq:
    print(item)
```

We get nothing...


All we need to do is create a new iterator:

```python
sq = Squares(5)
```

```python
for item in sq:
    print(item)
```

Just like Python's built-in `next` function calls our `__next__` method, Python has a built-in function `iter` which calls the `__iter__` method:

```python
sq = Squares(5)
```

```python
id(sq)
```

```python
id(sq.__iter__())
```

```python
id(iter(sq))
```

And of course we can also use a list comprehension on our iterator object:

```python
sq = Squares(5)
```

```python
[item for item in sq if item%2==0]
```

We can even use any function that requires an iterable as an argument (iterators are iterable):

```python
sq = Squares(5)
list(enumerate(sq))
```

But of course we have to be careful, our iterator was exhausted, so if try that again:

```python
list(enumerate(sq))
```

we get an empty list - instead we have to create a new iterator first:

```python
sq = Squares(5)
list(enumerate(sq))
```

We can even use the `sorted` method on it:

```python
sq = Squares(5)
sorted(sq, reverse=True)
```

#### Python Iterators Summary


Iterators are objects that implement the `__iter__` and `__next__` methods.

The `__iter__` method of an iterator just returns itself.

Once we fully iterate over an iterator, the iterator is **exhausted** and we can no longer use it for iteration purposes.


The way Python applies a `for` loop to an iterator object is basically what we saw with the `while` loop and the `StopIteration` exception.

```python
sq = Squares(5)
while True:
    try:
        print(next(sq))
    except StopIteration:
        break
```

In fact we can easily see this by tweaking our iterator a bit:

```python
class Squares:
    def __init__(self, length):
        self.length = length
        self.i = 0
        
    def __iter__(self):
        print('calling __iter__')
        return self
    
    def __next__(self):
        print('calling __next__')
        if self.i >= self.length:
            raise StopIteration
        else:
            result = self.i ** 2
            self.i += 1
            return result
```

```python
sq = Squares(5)
```

```python
for i in sq:
    print(i)
```

As you can see Python calls `__next__` (and stops once a `StopIteration` exception is raised).

But you'll notice that it also called the `__iter__` method.


In fact we'll see this happening in other places too:

```python
sq = Squares(5)
[item for item in sq if item%2==0]
```

```python
sq = Squares(5)
list(enumerate(sq))
```

```python
sq = Squares(5)
sorted(sq, reverse=True)
```

Why is `__iter__` being called? After all, it just returns itself!

That's the topic of the next lecture!

But let's see how we can mimic what Python is doing:

```python
sq = Squares(5)
sq_iterator = iter(sq)
print(id(sq), id(sq_iterator))
while True:
    try:
        item = next(sq_iterator)
        print(item)
    except StopIteration:
        break
```

As you can see, we first request an iterator from `sq` using the `iter` function, and then we iterate using the returned iterator. In the case of an iterator, the `iter` function just gets the iterator itself back.

```python

```
