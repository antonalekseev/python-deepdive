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

### Project 5


#### Goal 1


For this goal, you are given a number of CSV files, each of which have their first row with the field name.

You goal is to create a context manager that you can use to produce the data from each file in a named tuple with field names corresponding to the  header row field names.

You should use the `csv` module's `reader` function to help with parsing the data.

Your context manager should be generic in the sense that it should just need the file name, no other configuration or hardcoded functionality is required. You do not need to worry about data types for this goal - just return every field as a string.

In addition, your context manager should produce lazy iterators.

Implement this using a class that implements the context manager protocol


#### Goal 2


The goal is to reproduce the work you did in Goal 1, but using a generator function and the `contextlib` `contextmanager` decorator.


### Notes


The files included with this project are:
* `cars.csv`
* `personal_info.csv`


In addition you might find the following useful.


##### Reading and rewinding data from a File


The file object supports reading data by specifying the amount of data we want to read, and repositioning the "read head" using the `seek` function.

Let's take a look:

```python
with open('cars.csv') as f:
    print('---', f.read(100))
    print('---', f.read(100))
    f.seek(0)
    print('---', f.read(100))
    
```

As you can see, we could read the file calling the `read` method - this reads data and advances the "read head". We can then rewind and start again - either reading directly, or even just iterating through the rows using the iterator:

```python
from itertools import islice

with open('cars.csv') as f:
    print('---', f.read(100))
    print('---', f.read(100))
    print('--------------------')
    print('rewinding to 0...')
    f.seek(0)
    for row in islice(f, 5):
        print(row, end='')
```

##### Sniffing the CSV dialect


The dialect of a CSV file refers to some of the specifics used to define data in a CSV file. The separators can be different (for example some failes use a comma, some use a semi-colon, some use a tab, etc).

Also, as we have seen before, a field is also sometimes delimited using quotes, or double quotes, or maybe some entirely different character.

When we have to deal with files that may be encoded using different dialects it can require quite a bit of work to determine what those specifics are. This is were the `Sniffer` class from the `csv` module can be useful. By providing it a sample fo the CSV file, it can analyze it and determine a best guess as to the specific dialect that was used. We can then use that dialect when we use the `csv.reader` function.

Let's see how to use it with one of our files: `personal_info.csv`:

```python
import csv

with open('personal_info.csv') as f:
    sample = f.read(2000)
    dialect = csv.Sniffer().sniff(sample)
print(vars(dialect))
```

We can now use this dialect to open the csv reader:

```python
with open('personal_info.csv') as f:
    reader = csv.reader(f, dialect)
    for row in islice(reader, 5):
        print(row)
```

```python

```
