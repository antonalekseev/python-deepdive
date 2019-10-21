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

### Project Solution: Goal 2


Here's what we wrote in Goal 1:

```python
from collections import namedtuple
from datetime import datetime
from functools import partial

file_name = 'nyc_parking_tickets_extract.csv'

with open(file_name) as f:
    column_headers = next(f).strip('\n').split(',')
    
column_names = [header.replace(' ', '_').lower() 
                for header in column_headers]

Ticket = namedtuple('Ticket', column_names)

def read_data():
    with open(file_name) as f:
        next(f)
        yield from f
        
def parse_int(value, *, default=None):
    try:
        return int(value)
    except ValueError:
        return default
    
def parse_date(value, *, default=None):
    date_format='%m/%d/%Y'
    try:
        return datetime.strptime(value, date_format).date()
    except ValueError:
        return default
    
def parse_string(value, *, default=None):
    try:
        cleaned = str(value).strip()
        if not cleaned:
            # empty string
            return default
        else:
            return cleaned
    except:
        return default
    
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

def parse_row(row, *, default=None):
    fields = row.strip('\n').split(',')
    # note that I'm using a list comprehension here, 
    # since we'll need to iterate through the entire parsed fields
    # twice - one time to check if nothing is None
    # and another time to create the named tuple
    parsed_data = [func(field) 
                   for func, field in zip(column_parsers, fields)]
    if all(item is not None for item in parsed_data):
        return Ticket(*parsed_data)
    else:
        return default
    
def parsed_data():
    for row in read_data():
        parsed = parse_row(row)
        if parsed:
            yield parsed
```

#### Goal 2: Calculating Number of Violations by Car Make


What we want to do here is iterate through the file and keep a counter, for each make, of how many rows for that make was encountered.

A good approach is to use a dictionary to keep track of the makes (as keys), and the value can be a counter that is initialized to 1 if the key (make) does not already exist, or incremented by 1 if it does.

We could do this using regular dictionaries first:

```python
makes_counts = {}

for data in parsed_data():
    if data.vehicle_make in makes_counts:
        makes_counts[data.vehicle_make] += 1
    else:
        makes_counts[data.vehicle_make] = 1
        
for make, cnt in sorted(makes_counts.items(), 
                        key=lambda t: t[1], 
                        reverse=True):
    print(make, cnt)
    
```

We can also make use of a special type of dictionary called a `defaultdict`. The way a `defaultdict` works, is that if you try to retrieve a non-existent key from the dictionary, it will return a **default** value. It does need to know the data type to use for the default - so we should provide one.

Let's take a look:

```python
from collections import defaultdict
```

```python
d = defaultdict(str)
```

```python
d['a'] = 100
```

```python
d['a']
```

```python
d['b']
```

As you can see it returned an empty string.

In our case, we want to use it to count, so we can make our default be integers:

```python
d = defaultdict(int)
```

```python
d['a'] = 1
```

```python
d['b']
```

So, if we want to either set a key's value to `1` if it does not already exist, or increment it by `1` if it does, it's quite simple. In both cases, we just need to retrieve the key's value, and increment by 1:

```python
d = defaultdict(int)
```

```python
d['make1'] += 1
```

```python
d['make1']
```

```python
d['make1'] += 1
```

```python
d['make1']
```

So, we could simplify our counter algorithm using a default dict:

```python
makes_counts = defaultdict(int)

for data in parsed_data():
    makes_counts[data.vehicle_make] += 1
    
for make, cnt in sorted(makes_counts.items(), 
                        key=lambda t: t[1], 
                        reverse=True):
    print(make, cnt)
    
```

To wrap up this goal, let's make a function that will return that dictionary, and while we're at it we'll sort the dictionary keys based on a descending count. (Remember that in Python 3.6+ dictionaries will now maintain their key order - we don't need to use an `OrderedDict`).

```python
def violation_counts_by_make():
    makes_counts = defaultdict(int)
    for data in parsed_data():
        makes_counts[data.vehicle_make] += 1
        
    return {make: cnt 
            for make, cnt in sorted(makes_counts.items(), 
                                    key=lambda t: t[1], 
                                    reverse=True)
           }
```

```python
print(violation_counts_by_make())
```
