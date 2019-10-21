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

### Yield From


In the last video we saw when we had two nested generators that we had to use a nested loop in order to iterate through both iterators:

```python
def matrix(n):
    gen = ( (i * j for j in range(1, n+1))
            for i in range(1, n+1)
          )
    return gen
```

```python
m = list(matrix(5))
```

```python
m
```

Suppose we want an iterator to iterate over all the values of the matrix, element by element.

We could write it this way:

```python
def matrix_iterator(n):
    for row in matrix(n):
        for item in row:
            yield item
```

All we have done here is create a generator (iterator) that can be used to iterate over the elements of a nested iterator.

We can then use it this way:

```python
for i in matrix_iterator(3):
    print(i)
```

But we can avoid using that nested for loop by using a special form of `yield`: `yield from`

```python
def matrix_iterator(n):
    for row in matrix(n):
        yield from row
```

```python
for i in matrix_iterator(3):
    print(i)
```

<!-- #region -->
As you can see we obtain the same result.

We can think of 
```
yield from <iterator>
```
as a replacement for the code:
```
for i in <iterator>:
    yield i
```
<!-- #endregion -->

We'll come back to `yield from` in more detail, because there's a **lot** more to it than just a simple replacement for that inner loop!


#### Example


Here's an example where using `yield from` can be quite effective.

In this example we need to read car brands from multiple files to get it as a single collection.

We might do it this way:

```python
brands = []

with open('car-brands-1.txt') as f:
    for brand in f:
        brands.append(brand.strip('\n'))
        
with open('car-brands-2.txt') as f:
    for brand in f:
        brands.append(brand.strip('\n'))
        
with open('car-brands-3.txt') as f:
    for brand in f:
        brands.append(brand.strip('\n'))
```

```python
for brand in brands:
    print(brand, end=', ')
```

But notice that we had to load up the entire data set in memory.

As we have discussed before this is not very efficient.

Instead we could use a generator approach as follows:

```python
def brands(*files):
    for f_name in files:
        with open(f_name) as f:
            for line in f:
                yield line.strip('\n')
```

```python
files = 'car-brands-1.txt', 'car-brands-2.txt', 'car-brands-3.txt'
for brand in brands(*files):
    print(brand, end = ', ')
```

We can simplify our function by using `yield from`:

```python
def brands(*files):
    for f_name in files:
        with open(f_name) as f:
            yield from f
```

```python
for brand in brands(*files):
    print(brand, end=', ')
```

Now we still have to clean up that trailing `\n` character...


So, we are going to create generators that can read each line of the file, and yield a clean result, and we'll `yield from` that generator:

```python
def gen_clean_read(file):
    with open(file) as f:
        for line in f:
            yield line.strip('\n')
```

As you can see, this generator function will clean each line of the file before yielding it. Let's try it with a single file and make sure it works:

```python
f1 = gen_clean_read('car-brands-1.txt')
for line in f1:
    print(line, end=', ')
```

Ok, that works. So now, we can proceed with our overarching generator function as before, except we'll `yield from` our generators, instead of directly from the file iterator:

```python
files = 'car-brands-1.txt', 'car-brands-2.txt', 'car-brands-3.txt'
```

```python
def brands(*files):
    for file in files:
        yield from gen_clean_read(file)
```

```python
for brand in brands(*files):
    print(brand, end=', ')
```

I want to point out that in this particular instance, we are using `yield from` as a simple replacement for a `for` loop. We could equally well have written it this way:


Using `yield from`:

```python
def brands(*files):
    for file in files:
        yield from gen_clean_read(file)
```

Without using `yield from`:

```python
def brands(*files):
    for file in files:
        for line in gen_clean_read(file):
            yield line
```

```python
for brand in brands(*files):
    print(brand, end=', ')
```

We'll come back to `yield from` in a lot more detail later when we study coroutines - there's a whole lot more to `yield from` than a replacement for a simple loop!
