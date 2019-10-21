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

### Project Solution - Goal 1


First we should look at what's in the file itself. Just a few records should be enough. (You can also "cheat" and look in Excel - but this works because the file is relatively small).

```python
file_name = 'nyc_parking_tickets_extract.csv'
```

```python
with open(file_name) as f:
    for _ in range(10):
        print(next(f))
```

So we should notice that we have these `\n` line terminators in the file - we'll need to strip those out.

Secondly we see that the first row of the file are the column headers - we'll need to skip that line when we want to look at just the data.

We should also not make the assumption that the data is entirely clean - we probably have missing values and will need to deal with that accordingly.

We also will need to determine an appropriatre data type for every column in the data set.


#### Column Definitions and Named Tuple


Let's start with the column definitions, data types and named tuple.

```python
with open(file_name) as f:
    column_headers = next(f).strip('\n').split(',')
    sample_data = next(f).strip('\n').split(',')
```

```python
column_headers
```

```python
sample_data
```

```python
list(zip(column_headers, sample_data))
```

Let's start by creating a tuple that contains the names of the columns:

```python
column_names = [header.replace(' ', '_').lower() 
                for header in column_headers]
```

```python
column_names
```

Next we need to determine the data types for each of these fields:

    0. summons_number: looks like integers
    1. plate_id: string
    2: registration_state: string
    3: plate_type: string
    4: issue_date: looks like valid dates
    5: violation_code: looks like integers
    6: vehicle_body_type: string
    7: vehicle_make: string
    8: violation_description: string



We'll create utility functions to cast the data (which will always be strings) into the appropriate data type for each field.

We have to be careful though, we may have issues with data integrity and our assumptions about the data type.

What we'll do as a first pass is to keep track of the rows where the data was not an integer or date when we expected it (or missing).


Let's create our named tuple data structure:

```python
from collections import namedtuple

Ticket = namedtuple('Ticket', column_names)
```

#### Reading and Cleaning a data row

```python
with open(file_name) as f:
    next(f)
    raw_data_row = next(f)
```

```python
raw_data_row
```

You'll notice that to read the data in the file, we have to skip the first row in the file. Also, I have to use a `with` statement and the file name every time. To make life easier, I'm going to write a small utility function that will yield just the data rows from the file:

```python
def read_data():
    with open(file_name) as f:
        next(f)
        yield from f
```

We can test it out easily:

```python
raw_data = read_data()
for _ in range(5):
    print(next(raw_data))
```

Let's write a function that will try to convert a value to an integer, or return some default if the value is missing or not an integer:

```python
def parse_int(value, *, default=None):
    try:
        return int(value)
    except ValueError:
        return default
```

We need to do the same thing with dates.
It looks like the dates are provided in M/D/YYYY format, so we'll use that to parse the date. 

We'll use the `strptime` function available in the `datetime` package.

```python
from datetime import datetime
def parse_date(value, *, default=None):
    date_format='%m/%d/%Y'
    try:
        return datetime.strptime(value, date_format).date()
    except ValueError:
        return default
```

Let's make sure those functions work as expected:

```python
parse_int('123')
```

```python
parse_int('hello', default='N/A')
```

```python
parse_date('3/28/2018')
```

```python
parse_date('31/31/2000', default='N/A')
```

OK, so these seem to work as expected.

We also need to write a string parser - we want to remove any potential leading and trailing spaces.

```python
def parse_string(value, *, default=None):
    try:
        cleaned = str(value).strip()
        if not cleaned:
            # empty string
            return default
        else:
            return cleaned
    except ValueError:
        return default
```

Let's test this one as well:

```python
parse_string('   hello   ')
```

```python
parse_string('  ', default='N/A')
```

Now that we have our utility functions, we can write our row parser.

To make life easier, I'm going to create a tuple that contains the functions that should be called to clean up each field. The tuple positions will correspond to the fields in the data row.

I'm also going to specify what the default value should be when there is a problem parsing the fields. To do this, I will use `partials`, because I still need a callable for each element of the column parser tuple. (Note that I could just as easily use a lambda as well instead of partials).

```python
from functools import partial
```

```python
column_names
```

```python
column_parsers = (parse_int,  # summons_number, default is None
                  parse_string,  # plate_id, default is None
                  partial(parse_string, default=''),  # state
                  partial(parse_string, default=''),  # plate_type
                  parse_date,  # issue_date, default is None
                  parse_int,  # violation_code
                  partial(parse_string, default=''),  # body type
                  parse_string,  # make, default is None
                  lambda x: parse_string(x, default='')  # description
                 )
```

To parse each field in a row, I'll first separate the data fields into a list of values, then I'll apply the functions in `column_parsers` to the data in that list. 

To do that, I'm going to zip up the parser functions and the data, and use a comprehension to apply each function to its corresponding data field:

```python
def parse_row(row):
    fields = row.strip('\n').split(',')
    parsed_data = (func(field) 
                   for func, field in zip(column_parsers, fields))
    return parsed_data
```

This is not quite what we want yet, but let's test it out and make sure it does what we expect:

```python
rows = read_data()
for _ in range(5):
    row = next(rows)
    parsed_data = parse_row(row)
    print(list(parsed_data))
```

Let's finish up the row parser.

First I want it to return a named tuple instead of a plain iterator.

Also, the way I have set up the parsers, I only want to look at data where none of the fields are `None` - that's why I had some fields default to an empty string instead of `None` - those are the ones I still want to retain, even if they are empty.

To do this efficiently, I'm going to use `all`


Let's just quickly recall how `all` works:

```python
all([10, 'hello'])
```

```python
all([None, 'hello'])
```

But we have to watch out, since we are allowing empty strings in our valid data, we cannot simply use `all`:

```python
all([10, ''])
```

That's because empty strings are falsy. So, we need to tweak this slightly.

I'll use a generator expression for this:

```python
l = [10, '', 0]
all(item is not None for item in l)
```

```python
l = [10, '', 0, None]
all(item is not None for item in l)
```

So, now let's finish up our row parser. We'll return a Ticket named tuple if none of the parsed fields are `None`, and we'll allow the user to specify a default otherwise.

```python
def parse_row(row, *, default=None):
    fields = row.strip('\n').split(',')
    # note that I'm using a list comprehension here, 
    # since we'll need to iterate through the entire parsed fields
    # twice - one time to check if nothing is None
    # and another time to create the named tuple
    parsed_data = [func(field) 
                   for func, field in zip(column_parsers, fields)]
    if all(item is not None for item in parsed_data):
        print(*parsed_data)
        return Ticket(*parsed_data)
    else:
        return default
```

Now let's test it out again:

```python
rows = read_data()
for _ in range(5):
    row = next(rows)
    parsed_data = parse_row(row)
    print(parsed_data)
```

#### Checking What Rows are Missing Required Values


Let's quickly run through the file and see what data issues we might have - maybe our assumptions were incorrect about the various data types.

```python
for row in read_data():
    parsed_row = parse_row(row)
    if parsed_row is None:
        print(list(zip(column_names, row.strip('\n').split(','))), end='\n\n')
```

OK, so mostly the data is clean. Looks like we have a few rows without descriptions. 
Technically there's a whole lot more validation and cleaning we should do. For example, it looks like the states are not always proper state abbreviations (like 99 in some records, etc). But this is good enough for now.


#### Creating an Iterator for the data


Finally, let's create an iterator to easily iterate over the cleaned up and structured data in the file, skipping `None` rows:

```python
def parsed_data():
    for row in read_data():
        parsed = parse_row(row)
        if parsed:
            yield parsed
```

Let's test it out by iterating a few times:

```python
parsed_rows = parsed_data()
for _ in range(5):
    print(next(parsed_rows))
```
