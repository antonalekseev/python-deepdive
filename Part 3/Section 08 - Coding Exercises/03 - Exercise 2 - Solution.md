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

### Exercise 2 - Solution


Here's where we ended up after completing Exercise 1:

```python
from json import JSONEncoder, dumps
from datetime import date, datetime
from decimal import Decimal

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

And we could serialize our objects:

```python
encoded = dumps(activity, cls=CustomEncoder, indent=2)
print(encoded)
```

Now we want to reverse the process and deserialize this JSON object. I am not going to assume any specific schema other than `Stock` and `Trade` objects will contain the `"class": "Stock"` or `"object": "Trade"` entries and the required additional fields to define those objects.


What I want to do is examine each dictionary, and if it contains those entries, I will want to deserialize as the corresponding objects.

We'll need to pay attention also to `date`, `datetime`, and `Decimal` type objects.


Let's start by writing a utility function that will convert a JSON dictionary of each specific type to the corresponding object type:

```python
def decode_stock(d):
    # assumes "class": "Stock" is in the dictionary
    # and contains all the required serialized fields needed to re-create the object
    # if working in Python 3.7, we could use date.fromisoformat(d['date']) instead
    s = Stock(d['symbol'], 
              datetime.strptime(d['date'], '%Y-%m-%d').date(), 
              Decimal(d['open']), 
              Decimal(d['high']), 
              Decimal(d['low']), 
              Decimal(d['close']),
              int(d['volume']))
    return s
```

Let's make sure this works:

```python
s = decode_stock({
      "symbol": "AAPL",
      "date": "2018-11-22",
      "open": "176.66",
      "high": "177.25",
      "low": "176.64",
      "close": "176.78",
      "volume": 3699184,
      "object": "Stock"
    })
```

```python
type(s), vars(s)
```

Now let's do the same thing with a `Trade`:

```python
def decode_trade(d):
    # assumes "class": "Trade" is in the dictionary
    # and contains all the required serialized fields needed to re-create the object
    s = Trade(d['symbol'], 
              datetime.strptime(d['timestamp'], '%Y-%m-%dT%H:%M:%S'), 
              d['order'], 
              Decimal(d['price']), 
              int(d['volume']), 
              Decimal(d['commission']))
    return s
```

```python
t = decode_trade({
      "symbol": "TSLA",
      "timestamp": "2018-11-22T10:05:12",
      "order": "buy",
      "price": "338.25",
      "volume": 100,
      "commission": "9.99",
      "object": "Trade"
    })
```

```python
type(t), vars(t)
```

OK, these look good to go, so one last utility function that can take in **either** a `Stock` or `Trade` type JSON object, and decode accordingly:

```python
def decode_financials(d):
    object_type = d.get('object', None)
    if object_type == 'Stock':
        return decode_stock(d)
    elif object_type == 'Trade':
        return decode_trade(d)
    return d  
```

```python
decode_financials({
      "symbol": "TSLA",
      "timestamp": "2018-11-22T10:05:12",
      "order": "buy",
      "price": "338.25",
      "volume": 100,
      "commission": "9.99",
      "object": "Trade"
    })
```

```python
decode_financials({
      "symbol": "AAPL",
      "date": "2018-11-22",
      "open": "176.66",
      "high": "177.25",
      "low": "176.64",
      "close": "176.78",
      "volume": 3699184,
      "object": "Stock"
    })
```

So now let's write our custom JSON decoding class:

```python
from json import JSONDecoder, loads
```

```python
class CustomDecoder(JSONDecoder):
    def decode(self, arg):
        data = loads(arg)
        # now we have to recursively look for `Trade` and `Stock` objects
        return self.parse_financials(data)
 
    def parse_financials(self, obj):
        if isinstance(obj, dict):
            obj = decode_financials(obj)
            if isinstance(obj, dict):
                for key, value in obj.items():
                    obj[key] = self.parse_financials(value)
        elif isinstance(obj, list):
            for index, item in enumerate(obj):
                obj[index] = self.parse_financials(item)
        return obj
```

Let's recall our serialized data first:

```python
print(encoded)
```

```python
decoded = loads(encoded, cls=CustomDecoder)
```

```python
decoded
```

How can we check of the two objects are "equal"? The problem is that we did not define equality for `Stock` and `Trade` objects, so we cannot compare two instances of the same class and expect equality even if they have the same data. We need to define that first!
Let's do that:

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
    
    def __eq__(self, other):
        return isinstance(other, Stock) and self.as_dict() == other.as_dict()
        
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
    
    def __eq__(self, other):
        return isinstance(other, Trade) and self.as_dict() == other.as_dict()
```

```python
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

```python
encoded = dumps(activity, cls=CustomEncoder)
```

```python
decoded = loads(encoded, cls=CustomDecoder)
```

```python
decoded == activity
```
