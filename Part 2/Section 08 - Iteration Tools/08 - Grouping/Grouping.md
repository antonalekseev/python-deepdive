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

### Grouping


If your familiar with SQL and the `group by` clause, then this will be familiar to you (with the exception that in SQL the order in which rows are selected does not affect the group by - i.e. we have an automatic implicit sort on the group by key - not so here)


If you're not familiar with the `group by` in SQL, let's consider an example to understand what's going on:


Let's look at the file `cars_2014.csv`:

```python
import itertools

with open('cars_2014.csv') as f:
    for row in itertools.islice(f, 0, 20):
        print(row, end = '')
```

This file contains car make and model ordered by make (so all the same makes are together in the file already) and then model.

We may want to know how many models exist for each make.

This is what a group by is used for: we need to make groups of makes, then count the number of items in each group.

Trivial to do with SQL, but a little more work with Python.

We might try doing it this way:

```python
from collections import defaultdict

makes = defaultdict(int)

with open('cars_2014.csv') as f:
    next(f)  # skip header row
    for row in f:
        make, _ = row.strip('\n').split(',')
        makes[make] += 1
        
for key, value in makes.items():
    print(f'{key}: {value}')
```

Instead of doing all this, we could use the `groupby` function in `itertools`.

Again, it is a lazy iterator, so we'll use lists to see what's happening - but let's use a slightly smaller data set as an example first:

```python
data = (1, 1, 2, 2, 3)
```

```python
list(itertools.groupby(data))
```

As you can see, we ended up with an iterable of tuples. The tuple was the groups of numbers in data, so `1`, `2`, and `3`. But what's in the second element of the tuple? Well it's an iterator, but what does it contain?

```python
it = itertools.groupby(data)
for group in it:
    print(group[0], list(group[1]))
```

Basically it just contained the grouped elements themselves.

This might seem a bit confusing at first - so let's look at the second optional argument of group by - it is a key. Basically the idea behind that key is the same as the sort keys, or filter keys we have worked with in the past. It is a **function** that returns a grouping key.

Let's try it out with a simple example:

```python
data = (
    (1, 'abc'),
    (1, 'bcd'),
   
    (2, 'pyt'),
    (2, 'yth'),
    (2, 'tho'),
    
    (3, 'hon')
)
```

So we want to group the data, using the first item of each tuple as the group key:

```python
groups = list(itertools.groupby(data, key=lambda x: x[0]))
```

```python
print(groups)
```

Once again you'll notice that we have the group keys, and some iterable. Let's see what those contain:

```python
groups = itertools.groupby(data, key=lambda x: x[0])
for group in groups:
    print(group[0], list(group[1]))
```

So now let's go back to our car make example.

We want to get all the makes and how many models are in each make.

We could start approaching it this way:

```python
with open('cars_2014.csv') as f:
    make_groups = itertools.groupby(f, key=lambda x: x.split(',')[0])
```

```python
list(itertools.islice(make_groups, 5))
```

What's going on?


Remember that `groupby` is a **lazy** iterator. This means it did not actually do any work when we called it apart from setting up the iterator.

When we called `list()` on that iterator, **then** it went ahead and try to do the iteration.

However, our `with` (context manager) closed the file by then!

So we will need to do our work inside the context manager.

```python
with open('cars_2014.csv') as f:
    next(f)  # skip header row
    make_groups = itertools.groupby(f, key=lambda x: x.split(',')[0])
    print(list(itertools.islice(make_groups, 5)))
```

Next, we need to know how many items are in each `itertools._grouper` iterators.

How about using the `len()` property of the iterator?

```python
with open('cars_2014.csv') as f:
    next(f)  # skip header row
    make_groups = itertools.groupby(f, key=lambda x: x.split(',')[0])
    make_counts = ((key, len(models)) for key, models in make_groups)
    print(list(make_counts))
```

Aww... Iterators don't necessarily implement a `__len__` method - and this one definitely does not.


Well, if we think about this, we could simply "replace" each element in 
the models, with a `1`, and sum that up...

```python
with open('cars_2014.csv') as f:
    next(f)  # skip header row
    make_groups = itertools.groupby(f, key=lambda x: x.split(',')[0])
    make_counts = ((key, sum(1 for model in models)) 
                    for key, models in make_groups)
    print(list(make_counts))
```

#### Caveat


I want to show you something that you may find odd at first. Notice how I iterated through the groups.

Maybe I want to be able to itrerate multiple times through that iterator, so let's make a list out of it first:

```python
groups = list(itertools.groupby(data, key=lambda x: x[0]))
for group in groups:
    print(group[0], group[1])
```

Ok, so this looks fine - we now have a list containing tuples - the first element is the group key, the second is an iterator - we can ceck that easily:

```python
it = groups[0][1]
```

```python
iter(it) is it
```

So yes, this is an iterator - what's in it?

```python
list(it)
```

Empty?? But we did not iterate through it - what happened?


Let's try again, just in case calling the `iter` method did something odd:

```python
groups = list(itertools.groupby(data, key=lambda x: x[0]))
for group in groups:
    print(group[0], list(group[1]))
```

So, the 3rd element is OK, but looks like the first two got exhausted somehow...


Let's make sure they are indeed exhausted:

```python
groups = list(itertools.groupby(data, key=lambda x: x[0]))
```

```python
next(groups[0][1])
```

```python
next(groups[1][1])
```

```python
next(groups[2][1])
```

So, yes, the first two were exhausted when we converted the groups to a list.


The solution here is actually in the Python docs. Let's take a look:


```
The returned group is itself an iterator that shares the underlying iterable with groupby(). Because the source is shared, when the groupby() object is advanced, the previous group is no longer visible. So, if that data is needed later, it should be stored as a list
```


The key thing here is that the elements yielded from the different groups are using the **same** underlying iterable over all the elements. As the documentation states, when we advance to the next group, the previous one's iterator is automatically exhausted - it basically iterates over all the elements until it hits the next group key.

Let's see this by stepping through the iteration manually:

```python
groups = itertools.groupby(data, key=lambda x: x[0])
```

```python
group1 = next(groups)
```

```python
group1
```

And the iterator in the tuple is not exhausted:

```python
next(group1[1])
```

Now, let's try again, but this time we'll advance to group2, and see what is in `group1`'s iterator:

```python
groups = itertools.groupby(data, key=lambda x: x[0])
```

```python
group1 = next(groups)
```

```python
group2 = next(groups)
```

Now `group1`'s iterator has been exhausted (because we moved to `group2`):

```python
next(group1[1])
```

But `group2`'s iterator is still OK:

```python
next(group2[1])
```

We know that there are still two elements in `group2`, so let's advance to `group3` and go back and see what's left in `group2`'s iterator:

```python
group3 = next(groups)
```

```python
next(group2[1])
```

But `group3`'s iterator is just fine:

```python
next(group3[1])
```

So, just be careful here with the `groupby()` - if you want to save all the data into a list you cannot first convert the groups into a list - you **must** step through the groups iterator, and retrieve each individual iterators elements into a list, the way we did it in the first example, or simply using a comprehension:

```python
groups = itertools.groupby(data, key=lambda x: x[0])
```

```python
groups_list = [(key, list(items)) for key, items in groups]
```

```python
groups_list
```
