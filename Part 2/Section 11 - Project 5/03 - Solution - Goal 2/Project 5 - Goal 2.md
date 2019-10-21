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

### Project 5 - Goal 2


In the previous goal, we create a class that was both a context manager and an iterator.

Here we want to create our context manager using a generator function instead.

Let's first recall what our context manager looked like in Goal 1:

```python
import csv
from collections import namedtuple

def get_dialect(f_name):
    with open(f_name) as f:
        return csv.Sniffer().sniff(f.read(1000))

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

So, we want to reproduce this using a generator function. The problem is that we implemented two protocols in here, and the context manager decorator relies on a specific structure to the generator function.

Obviously we'll have to separate those two protocols out in order to make this work.

First we'll create a generator function to iterate through the data - it will need to have the data iterator from `csv.reader` as well as the named tuple class we want to use.

Let's write that first:

```python
def parsed_data_iter(data_iter, nt):
    for row in data_iter:
        yield nt(*row)   
```

Next, we have to create our context manager using a generator function:

```python
from contextlib import contextmanager
```

```python
import csv
from collections import namedtuple
    
@contextmanager
def parsed_data(f_name):
    f = open(f_name, 'r')
    try:
        reader = csv.reader(f, get_dialect(f_name))
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield parsed_data_iter(reader, nt)
    finally:
        f.close()    
```

```python
from itertools import islice

with parsed_data('personal_info.csv') as data:
        for row in islice(data, 5):
            print(row)
```

And this should work with our other file too:

```python
with parsed_data('cars.csv') as data:
    for row in islice(data, 5):
        print(row)
```

Now, we can define functions inside other functions in Python, so let's make our context manager more self-contained by integrating both utility functions directly into it:

```python
@contextmanager
def parsed_data(f_name):
    def get_dialect(f_name):
        with open(f_name) as f:
            return csv.Sniffer().sniff(f.read(1000))
    
    def parsed_data_iter(data_iter, nt):
        for row in data_iter:
            yield nt(*row) 
        
    f = open(f_name, 'r')
    try:
        reader = csv.reader(f, get_dialect(f_name))
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield parsed_data_iter(reader, nt)
    finally:
        f.close()  
```

```python
f_names = 'cars.csv', 'personal_info.csv'
for f_name in f_names:
    with parsed_data(f_name) as data:
        for row in islice(data, 5):
            print(row)
    print('-------------------')
```

So that works just fine. But do we really need that parsed_data_iter function?

Maybe we can rewrite it using a generator expression instead:

```python
@contextmanager
def parsed_data(f_name):
    def get_dialect(f_name):
        with open(f_name) as f:
            return csv.Sniffer().sniff(f.read(1000))
    f = open(f_name, 'r')
    try:
        reader = csv.reader(f, get_dialect(f_name))
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield (nt(*row) for row in reader)
    finally:
        f.close()  
```

```python
with parsed_data('cars.csv') as data:
    for row in islice(data, 5):
        print(row)
```

Good, that's one simplification we could make.

How about that `get_dialect` function?
Since that function is inside our main function, we really don't have to pass the file name to it - it already has access to it from the outer scope:

```python
@contextmanager
def parsed_data(f_name):
    def get_dialect():
        with open(f_name) as f:
            return csv.Sniffer().sniff(f.read(1000))
        
    f = open(f_name, 'r')
    try:
        reader = csv.reader(f, get_dialect())
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield (nt(*row) for row in reader)
    finally:
        f.close()  
```

```python
with parsed_data('cars.csv') as data:
    for row in islice(data, 5):
        print(row)
```

But we really don't have to make it into a function:

```python
@contextmanager
def parsed_data(f_name):
    with open(f_name) as f:
        dialect = csv.Sniffer().sniff(f.read(1000))

    f = open(f_name, 'r')
    try:
        reader = csv.reader(f, dialect)
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield (nt(*row) for row in reader)
    finally:
        f.close()  
```

```python
with parsed_data('cars.csv') as data:
    for row in islice(data, 5):
        print(row)
```

Technically, we don't even have to open that file twice - once to sniff the dialect, the other to iterate over the file.

With the file object, we can read some data, and then "rewind" - using the `seek` method. This is not the same as using the `iterator` protocol which the file objects also implement.

Here's a simple example of how that works:

```python
with open('cars.csv') as f:
    print('---', f.read(120))
    print('---', f.read(120))
    f.seek(0)
    print('---', f.read(120))
```

And we can still iterator over the lines after rewinding:

```python
with open('cars.csv') as f:
    print('---', f.read(50))
    f.seek(0)
    for row in islice(f, 5):
        print(row, end='')
```

So let's use that to simplify our code further:

```python
@contextmanager
def parsed_data(f_name):
    f = open(f_name, 'r')
    try:
        dialect = csv.Sniffer().sniff(f.read(1000))
        f.seek(0)
        reader = csv.reader(f, dialect)
        headers = map(lambda x: x.lower(), next(reader))
        nt = namedtuple('Data', headers)
        yield (nt(*row) for row in reader)
    finally:
        f.close()  
```

```python
f_names = 'cars.csv', 'personal_info.csv'
for f_name in f_names:
    with parsed_data(f_name) as data:
        for row in islice(data, 5):
            print(row)
    print('-------------------')
```
