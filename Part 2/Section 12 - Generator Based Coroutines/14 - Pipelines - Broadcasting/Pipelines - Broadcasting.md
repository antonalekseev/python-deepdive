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

### Pipelines - Broadcasting


To work along with this notebook you'll need the included data file, `car_data.csv`.

We are going to want to split the data into different files based on some criteria of our choosing.

For example, we may want to create a file that contains all pink cars, another file that contains all Mercedes brands, and another that contains only blue cars of a specific model year, etc.

Let's first write a generator to parse the data for us:

```python
import csv

def data_reader(f_name):
    f = open(f_name)
    try:
        dialect = csv.Sniffer().sniff(f.read(2000))
        f.seek(0)
        reader = csv.reader(f, dialect=dialect)
        yield from reader
    finally:
        f.close()
```

```python
for row in data_reader('car_data.csv'):
    print(row)
```

Let's create our indices, output headers and data converters for this file - basically these are our configuration parameters for this data file.

```python
input_file = 'car_data.csv'

idx_make = 0
idx_model = 1
idx_year = 2
idx_vin = 3
idx_color = 4

headers = ('make', 'model', 'year', 'vin', 'color')

converters = (str, str, int, str, str)
```

Now let's create a generator that will return the parsed data:

```python
def data_parser():
    data = data_reader(input_file)
    next(data)  # skip header row
    for row in data:
        parsed_row = [converter(item)
                      for converter, item in zip(converters, row)]
        yield parsed_row
```

Let's just make sure this is working properly:

```python
data = data_parser()
for _ in range(5):
    print(next(data))
```

Let's also write our coroutine decorator that will auto prime coroutines:

```python
def coroutine(fn):
    def inner(*args, **kwargs):
        g = fn(*args, **kwargs)
        next(g)
        return g
    return inner
```

Next we are going to write a coroutine that will create and write data to a file. We'll need to pass the output file name to the coroutine, and the coroutine will assume that the data is being passed in as a list (basically whatever is coming back from `data_parser`). To make it easier, we'll also pass it the column headers so we can include that in the output file.

```python
@coroutine
def save_data(f_name, headers):
    with open(f_name, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(headers)
        while True:
            data_row = yield
            writer.writerow(data_row)
```

Now we're going to create a filter coroutine that will have the following parameters:
* filter function (predicate)
* next coroutine to send to in the pipeline

That filter coroutine will receive a data row and test if the predicate applied to that data row is True. If it is, it will send the row to the next stage (target) of the pipeline, otherwise it just ignores the data row.

```python
@coroutine
def filter_data(filter_predicate, target):
    while True:
        data_row = yield
        if filter_predicate(data_row):
            target.send(data_row)
```

Next, let's write our broadcaster. It just sends received data to all the generators specified in the `targets` argument:

```python
@coroutine
def broadcast(targets):
    while True:
        data_row = yield
        for target in targets:
            target.send(data_row)
```

<!-- #region -->
OK, we're now ready to put all this together.

```
     data                      
      |                      |--> filter --> save
      v                      |
process_data --> broadcast --|--> filter --> save
                             |
                             |--> filter --> save
```
<!-- #endregion -->

```python
def process_data():
    data = data_parser()
    
    out_pink_cars = save_data('pink_cars.csv', headers)
    out_ford_green = save_data('ford_green.csv', headers)
    out_older = save_data('older.csv', headers)
    
    filter_pink_cars = filter_data(lambda d: d[idx_color].lower() == 'pink',
                                   out_pink_cars)
    
    def pred_ford_green(data_row):
        return (data_row[idx_make].lower() == 'ford' 
                and data_row[idx_color].lower() == 'green')
    
    filter_ford_green = filter_data(pred_ford_green, out_ford_green)
    filter_older = filter_data(lambda d: d[idx_year] <= 2010, out_older)
    filters = (filter_pink_cars, filter_ford_green, filter_older)
    broadcaster = broadcast(filters)
    
    for row in data:
        broadcaster.send(row)
    
    print('Finished processing.')
```

And now let's call it and see what happens!

```python
process_data()
```

Let's see what those files contain:

```python
def print_file_data():
    for file_name in ('pink_cars.csv', 'ford_green.csv', 'older.csv'):
        print(f'***** {file_name} *****')
        for row in data_reader(file_name):
            print(row)
        print('\n\n\n')

print_file_data()
```

There's one more bit of cleanup I want to do though.

I would prefer to have the definition of my pipeline not also be the consumer of the data. Just trying to keep functionality more separated.

So let's rewrite change `process_data` to just be another step in the pipeline.

```python
@coroutine
def pipeline_coro():
    out_pink_cars = save_data('pink_cars.csv', headers)
    out_ford_green = save_data('ford_green.csv', headers)
    out_older = save_data('older.csv', headers)
    
    filter_pink_cars = filter_data(lambda d: d[idx_color].lower() == 'pink',
                                  out_pink_cars)
    
    def pred_ford_green(data_row):
        return (data_row[idx_make].lower() == 'ford'
               and data_row[idx_color].lower() == 'green')
    filter_ford_green = filter_data(pred_ford_green, out_ford_green)
    filter_older = filter_data(lambda d: d[idx_year] <= 2010, out_older)
    
    filters = (filter_pink_cars, filter_ford_green, filter_older)
    
    broadcaster = broadcast(filters)
    
    while True:
        data_row = yield
        broadcaster.send(data_row)    
```

And now we can use the pipeline this way:

```python
pipe = pipeline_coro()
data = data_parser()
for row in data:
    pipe.send(row)
```

OK, so now let's make sure the correct data is in those output files:

```python
print_file_data()
```

Uh-oh, we get an exception. Why did the parser fail to figure out the dialect of the file?

Let's see what's in the file:

```python
with open('pink_cars.csv') as f:
    for row in f:
        print('row', row)
```

The file is empty!!

The issue is that our files have not been closed yet!

The pipeline coroutine is still active, so nothing go released or closed - including the endpoints of our pipeline.

Fortunately this is easy to do - we just need to close the pipeline.

```python
pipe.close()
```

And now we should be able to read those files:

```python
print_file_data()
```

Perfect, so just to recap, here's how we would use our pipeline:

```python
pipe = pipeline_coro()
data = data_parser()
for row in data:
    pipe.send(row)
pipe.close()
```

Hmm... Notice how we open the pipeline, and then close it?
Does this remind you of a context manager?

Let's write a context manager for our pipeline - that way we'll never forget to close it!

```python
from contextlib import contextmanager
```

```python
@contextmanager
def pipeline():
    p = pipeline_coro()
    try:
        yield p
    finally:
        p.close()
```

And now we can use it this way:

```python
with pipeline() as pipe:
    data = data_parser()
    for row in data:
        pipe.send(row)
```

And again, let's just make sure the files are OK:

```python
print_file_data()
```

Perfect!
