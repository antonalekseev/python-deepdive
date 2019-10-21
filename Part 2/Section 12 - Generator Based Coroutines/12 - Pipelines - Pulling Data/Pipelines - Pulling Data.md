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

### Pipelines - Pulling Data


Included with this notebook, we are going to use the `cars.csv` data file.

Let's start by writing a generator that will produce data from that file:

```python
import csv

def parse_data(f_name):
    f = open(f_name)
    try:
        dialect = csv.Sniffer().sniff(f.read(2000))
        f.seek(0)
        next(f)  # skip header row
        yield from csv.reader(f, dialect=dialect)
    finally:
        f.close()
```

Notice how we are already using delegation to delegate iteration fo the csv reader iterator. Here we are therefore pulling data from the csv reader and yielding that out from the `parse_data` generator.

Let's look at the data:

```python
import itertools

for row in itertools.islice(parse_data('cars.csv'), 5):
    print(row)
```

Now let's filter out rows based on the car make:

```python
def filter_data(rows, contains):
    for row in rows:
        if contains in row[0]:
            yield row
```

<!-- #region -->
We can now start building a (pull) pipeline by pulling data from the data source, through the filter:
```
caller <-- filter <-- data
```
<!-- #endregion -->

```python
data = parse_data('cars.csv')
filtered_data = filter_data(data, 'Chevrolet')

# pipeline: caller <-- filtered_data <-- data

for row in itertools.islice(filtered_data, 5):
    print(row)
```

As you can see, using iteration we are pulling data all the way from the file, through the csv reader, through the filter and back to us (the caller).

But why stop there?
Let's further filter out rows that contain the word 'Carlo' as well:

```python
data = parse_data('cars.csv')
filter_1 = filter_data(data, 'Chevrolet')
filter_2 = filter_data(filter_1, 'Carlo')

# pipeline: caller <-- filter_2 <-- filtered_1 <-- data

for row in itertools.islice(filter_2, 5):
    print(row)
```

We can package all this up into a single delegator generator:

```python
def output(f_name):
    data = parse_data(f_name)
    filter_1 = filter_data(data,'Chevrolet')
    filter_2 = filter_data(filter_1, 'Carlo')
    yield from filter_2
```

And we can use our delegator generator this way:

```python
results = output('cars.csv')
for row in results:
    print(row)
```

We can actually make this a little more generic while we're at it:

```python
def output(f_name, *filter_words):
    data = parse_data(f_name)
    for filter_word in filter_words:
        data = filter_data(data, filter_word)
    yield from data
```

```python
results = output('cars.csv', 'Chevrolet')
for row in itertools.islice(results, 5):
    print(row)
```

```python
results = output('cars.csv', 'Chevrolet', 'Carlo', 'Landau')
for row in itertools.islice(results, 5):
    print(row)
```
