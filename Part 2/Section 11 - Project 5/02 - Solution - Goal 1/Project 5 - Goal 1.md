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

### Project 5 - Goal 1


First, let's take a look at each file and make sure the first row contains the field names:

```python
f_names = 'cars.csv', 'personal_info.csv'

for f_name in f_names:
    with open(f_name) as f:
        print(next(f), end='')
        print(next(f), end='')
    print('\n-----------------')
```

One thing I notice here is that the field names in the `cars.csv` file have uppercase letters - the second does not. I'm going to make those consistent by lowercasing the field names when I create the named tuple.

The second thing I notice is that the delimiter is not the same for both files - one uses a `;`, the other uses a `,`. 

Fortunately, the `csv` module has a `Sniffer` class that we can use to try and deduce the delimiter from sampling some data. Alternatively, we could specify the delimiter to use as part of our conjtext manager's `__init__` method - but it would be nicer if we did not have to do that.


Let's first see how that `Sniffer` class works:

```python
import csv
from itertools import islice

with open('cars.csv') as f:
    dialect = csv.Sniffer().sniff(f.read(1000))
print(dialect.delimiter)
```

And we do this with our other file:

```python
with open('personal_info.csv') as f:
    dialect = csv.Sniffer().sniff(f.read(1000))
print(dialect.delimiter)
```

So, we'll use this to set the dialect for our csv parser.

Let's create a small utility function to handle this for us:

```python
def get_dialect(f_name):
    with open(f_name) as f:
        return csv.Sniffer().sniff(f.read(1000))
```

We want to create a context manager that, given just the file name:
1. reads the header row to get the field names
2. creates the appropriate named tuple
3. uses the `csv.reader` to create an iterator over the data rows of the file
4. returns that iterator from the `__enter__` method
5. closes the file upon `__exit__`

We're actually going to create a class that will be **both** the context manager and the iterator - so we'll implement both of these protocols.

```python
from collections import namedtuple

class FileParser:
    def __init__(self, f_name):
        self.f_name = f_name
        
    def __enter__(self):
        self._f = open(self.f_name, 'r')
        self._reader = csv.reader(self._f, get_dialect(self.f_name))
        headers = map(lambda x: x.lower(), next(self._reader))
        self._nt = namedtuple('Data', headers)
        return self
        
    def __exit__(self, exc_type, exc_value, exc_tb):
        self._f.close()
        return False
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self._f.closed:
            # file has been closed - so we're can't iterate anymore!
            raise StopIteration
        else:
            return self._nt(*next(self._reader))
```

```python
from itertools import islice

with FileParser('cars.csv') as data:
    for row in islice(data, 10):
        print(row)
```

And of course it should work equally well with the other file too:

```python
with FileParser('personal_info.csv') as data:
    for row in islice(data, 10):
        print(row)
```
