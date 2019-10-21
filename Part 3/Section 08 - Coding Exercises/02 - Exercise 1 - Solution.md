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

### Exercise 1 - Solution


The first thing I am going to do is add an `as_dict` method to both my classes to make serialization a bit easier:

```python
class Stock:
    def __init__(self, symbol, date_, open_, high, low, close, volume):
        self.symbol = symbol
        self.date = date_
        self.open = open_
        self.high = high
        self.low = low
        self.close = close
        self.volume = volume
        
    def as_dict(self):
        return dict(symbol=self.symbol, 
                    date=self.date,
                    open=self.open,
                    high=self.high,
                    low=self.low,
                    close=self.close,
                    volume=self.volume)
        
class Trade:
    def __init__(self, symbol, timestamp, order, price, volume, commission):
        self.symbol = symbol
        self.timestamp = timestamp
        self.order = order
        self.price = price
        self.commission = commission
        self.volume = volume
        
    def as_dict(self):
        return dict(
            symbol=self.symbol,
            timestamp=self.timestamp,
            order=self.order,
            price=self.price,
            volume=self.volume,
            commission=self.commission)
```

```python
from datetime import date, datetime
from decimal import Decimal

activity = {
    "quotes": [
        Stock('TSLA', date(2018, 11, 22), 
              Decimal('338.19'), Decimal('338.64'), Decimal('337.60'), Decimal('338.19'), 365_607),
        Stock('AAPL', date(2018, 11, 22), 
              Decimal('176.66'), Decimal('177.25'), Decimal('176.64'), Decimal('176.78'), 3_699_184),
        Stock('MSFT', date(2018, 11, 22), 
              Decimal('103.25'), Decimal('103.48'), Decimal('103.07'), Decimal('103.11'), 4_493_689)
    ],
    
    "trades": [
        Trade('TSLA', datetime(2018, 11, 22, 10, 5, 12), 'buy', Decimal('338.25'), 100, Decimal('9.99')),
        Trade('AAPL', datetime(2018, 11, 22, 10, 30, 5), 'sell', Decimal('177.01'), 20, Decimal('9.99'))
    ]
}
```

<!-- #region -->
My approach is going to be to serialize these classes using a special class name identifier.

For example to serialize `Stock` objects I will use this format:

```
{
    "object": "Stock",
    "symbol": "...",
    ...
}
```

Similarly for a `Trade` objects.
<!-- #endregion -->

Furthermore, I need to pay special attention to dates, timestamps and prices.

For dates and timestamps I will use the standard ISO format (`YYYY-MM-DD` and `YYYY-MM-DDTHH:MM:SS`).

Prices are stored in `Decimal` objects - so we'll have to handle serialization for those objects too.

```python
from json import JSONEncoder, dumps

class CustomEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Stock):
            return obj.as_dict()
        elif isinstance(obj, Trade):
            return obj_as_dict()
        else:
            super().default(obj)
```

This will not work quite yet - we are not handling decimal, date and datetime serialization:

```python
dumps(activity, cls=CustomEncoder)
```

There's a few ways we can fix that - we could serialize by coding the date formatting directly in the `Trade` or `Stock` serializers:

```python
from json import JSONEncoder, dumps

class CustomEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Stock):
            result = obj.as_dict()
            result['date'] = result['date'].strftime('%Y-%m-%d')
            return result
        elif isinstance(obj, Trade):
            result = obj.as_dict()
            result['timestamp'] = result['timestamp'].strftime('%Y-%m-%dT%H:%M:%S')
            return result
        else:
            super().default(obj)
```

This will still not quite work because we are not handling serizliation of `Decimal` objects. But I would rather not have to handle them the way we are handling `date` and `datetime` objects - that would be very tedious.

In fact, I am going to write handlers for the `Decimal` as well as `date` and `datetime` classes this way:

```python
from json import JSONEncoder, dumps

class CustomEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Stock) or isinstance(obj, Trade):
            return obj.as_dict()
        elif isinstance(obj, datetime):
            # check for datetime first, because a datetime is also a date
            return obj.strftime('%Y-%m-%dT%H:%M:%S')
        elif isinstance(obj, date):
            return obj.strftime('%Y-%m-%d')
        elif isinstance(obj, Decimal):
            return str(obj)
        else:
            super().default(obj)
```

```python
encoded = dumps(activity, cls=CustomEncoder, indent=2)
```

```python
print(encoded)
```

We're almost there - the serialization works just fine, but if I'm going to deserialize the objects later, I will need to know what the object type is for the `Trade` and `Stock` objects. We could add it to the `as_dict` methods of each class, but I don't necessarily want it all the time - so instead I am going to inject the class name during the serialization:

```python
class CustomEncoder(JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Stock) or isinstance(obj, Trade):
            result =  obj.as_dict()
            result['object'] = obj.__class__.__name__
            return result
        elif isinstance(obj, datetime):
            return obj.strftime('%Y-%m-%dT%H:%M:%S')
        elif isinstance(obj, date):
            return obj.strftime('%Y-%m-%d')
        elif isinstance(obj, Decimal):
            return str(obj)
        else:
            super().default(obj)
```

```python
result = dumps(activity, cls=CustomEncoder, indent=2)
print(result)
```
