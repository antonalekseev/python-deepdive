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

### Custom JSON Decoding


So far we have looked at how to encode (serialize) Python objects to JSON, using the standard as well as custom object serializers.

Now we need to turn our attention to teh reverse process - deserializing (decoding) JSON data.

Once again, the standard simple types such as strings, numbers (ints and floats), arrays, and objects with key/value pairs.
JSON does not differentiate between mutable and immutable lists - so everything that is an array (`[...]`) in JSON will get decoded into a list object.


Let's see a quick example of how to do this:

```python
j = '''
    {
        "name": "Python",
        "age": 27,
        "versions": ["2.x", "3.x"]
    }
'''
```

```python
import json
```

```python
json.loads(j)
```

But what about other data types, such as a date for example. How can we handle that?

```python
p = '''
    {
        "time": "2018-10-21T09:14:00",
        "message": "created this json string"
    }
'''
```

```python
json.loads(p)
```

The deserialization worked just fine, but you'll notice that the dictionary entry for `time` contains a string, not a date. 


This is not a trivial problem, and many 3rd party libraries have been written to deserialize specialized JSON structures into custom Python objects. It basically boils down to having a specific structure (schema) in the JSON and manually loading up some custom (or standard) Python object by specifically looking for certain elements and objects in the JSON object. Remember that JSON only supports a few basic types, so anything beyond that is really a custom **interpretation** of the data in the JSON object.


For example, suppose we have a JSON object where any object that contains the key/value pair `"objecttype": "datetime"` is guaranteed to contain another key called `"value"` containing a date time in the format %Y-%m-%dT%H:%M:%S. 
We could easily do the following:

```python
p = '''
    {
        "time": {
            "objecttype": "datetime",
            "value": "2018-10-21T09:14:15"
            },
        "message": "created this json string"
    }
'''
```

```python
d = json.loads(p)
```

```python
d
```

We could now run through our dictionary (top level only, we'll come back to that), and convert any datetime structures (schema) into actual datetime objects:

```python
from datetime import datetime

for key, value in d.items():
    if (isinstance(value, dict) and 
        'objecttype' in value and 
        value['objecttype'] == 'datetime'):
        d[key] = datetime.strptime(value['value'], '%Y-%m-%dT%H:%M:%S')
```

```python
d
```

<!-- #region -->
As you can see that worked just fine.
We can do this with other "custom" JSON schemas as well.

Let's say we have a JSON schema that will encode fractions using a `fraction` type indicator and associated keys `numerator` and `denominator` with integer values, such as:

```
"pieSlice": {
    "objecttype": "fraction",
    "numerator": 1,
    "denominator": 3
    }
```
<!-- #endregion -->

We can deal with this in the same way as before:

```python
j = '''
    {
        "cake": "yummy chocolate cake",
        "myShare": {
            "objecttype": "fraction",
            "numerator": 1,
            "denominator": 8
        }
    }
'''
```

```python
d = json.loads(j)
```

```python
d
```

```python
from fractions import Fraction

for key, value in d.items():
    if (isinstance(value, dict) and
        'objecttype' in value and
        value['objecttype'] == 'fraction'):
        numerator = value['numerator']
        denominator = value['denominator']
        d[key] = Fraction(numerator, denominator)
```

```python
d
```

We can extend this to even custom objects as long as they follow a specific structure (schema). We could put all this code into a function, even one that can handle multiple types and clean it up quite a bit.
But...


A few things:
1. It's a real pain having to go through the dictionary after the fact and convert the objects
2. Our conversion code only considered top-level objects - what if they are nested deeper in the JSON object - we would need to deal with that possibility.

There has to be a better way!

```
And indeed, there is - but all in all it's still relatively clunky in some respects.

Let's look at the `load`/`loads` functions first. They have an optional argument named `object_hook` that can reference a callable. This is very similar to the `default` argument we saw in the `dump`/`dumps` functions - but works for decoding instead of encoding. That callable, if specified, will be called for every value in the JSON object that is itself an object (including the root object). That dictionary will then be replaced by whatever that decoder returns.

Let's first write a dummy decoder, just to see how and when it gets called:
```

```python
def custom_decoder(arg):
    print('decoding: ', arg)
    return arg
```

```python
j = '''
    {
        "a": 1,
        "b": 2, 
        "c": {
            "c.1": 1,
            "c.2": 2,
            "c.3": {
                "c.3.1": 1,
                "c.3.2": 2
            }
        }
    }
'''
```

```python
d = json.loads(j, object_hook=custom_decoder)
```

As you can see it called our decoder three times, the value for the key `c.3`, the value for the key `c` and the root object itself.


Now, let's write a decoder that will handle the datetime JSON we worked with earlier:

```python
j = '''
    {
        "time": {
            "objecttype": "datetime",
            "value": "2018-10-21T09:14:15"
            },
        "message": "created this json string"
    }
'''
```

```python
def custom_decoder(arg):
    if 'objecttype' in arg and arg['objecttype'] == 'datetime':
        return datetime.strptime(arg['value'], '%Y-%m-%dT%H:%M:%S')
    else:
        return arg  # important, otherwise we lose anything that's not a date!
```

Let's just see how it works as a plain function first:

```python
custom_decoder(dict(objecttype='datetime', value='2018-10-21T09:14:15'))
```

```python
custom_decoder((dict(a=1)))
```

```python
d = json.loads(j, object_hook=custom_decoder)
```

```python
d
```

The nice thing about this approach, is our code is simpler, and this works for nested items too:

```python
j = '''
    {
        "times": {
            "created": {
                "objecttype": "datetime",
                "value": "2018-10-21T09:14:15"
                },
            "updated": {
                "objecttype": "datetime",
                "value": "2018-10-22T10:00:05"
                }
            },
        "message": "log message here..."
    }
'''
```

```python
d = json.loads(j, object_hook=custom_decoder)
```

```python
d
```

We can also extend this custom decoder to include other structures (schemas). Let's add in our fraction decoder:

```python
def custom_decoder(arg):
    ret_value = arg
    if 'objecttype' in arg:
        if arg['objecttype'] == 'datetime':
            ret_value = datetime.strptime(arg['value'], '%Y-%m-%dT%H:%M:%S')
        elif arg['objecttype'] == 'fraction':
            ret_value = Fraction(arg['numerator'], arg['denominator'])
    return ret_value
```

```python
j = '''
    {
        "cake": "yummy chocolate cake",
        "myShare": {
            "objecttype": "fraction",
            "numerator": 1,
            "denominator": 8
        },
        "eaten": {
            "at": {
                "objecttype": "datetime",
                "value": "2018-10-21T21:30:00"
                },
            "time_taken": "30 seconds"
        }
    }
'''
```

```python
d = json.loads(j, object_hook=custom_decoder)
```

```python
print(d)
```

We can't really use a generic single dispatch approach we took with the encoder though - the decoder always receives a dictionary, so we can't build it that way.

We still have the issue of custom objects and classes - how do we handle those?

Well, in pretty much the same way as before - the content of the JSON has to indicate that the object is of a certain "type", and we can then decode it ourselves.

Let's see a simple example:

```python
class Person:
    def __init__(self, name, ssn):
        self.name = name
        self.ssn = ssn
        
    def __repr__(self):
        return f'Person(name={self.name}, ssn={self.ssn})'
```

```python
j = '''
    {
        "accountHolder": {
            "objecttype": "person",
            "name": "Eric Idle",
            "ssn": 100
        },
        "created": {
            "objecttype": "datetime",
            "value": "2018-10-21T03:00:00"
        }
    }
'''
```

```python
def custom_decoder(arg):
    ret_value = arg
    if 'objecttype' in arg:
        if arg['objecttype'] == 'datetime':
            ret_value = datetime.strptime(arg['value'], '%Y-%m-%dT%H:%M:%S')
        elif arg['objecttype'] == 'fraction':
            ret_value = Fraction(arg['numerator'], arg['denominator'])
        elif arg['objecttype'] == 'person':
            ret_value = Person(arg['name'], arg['ssn'])
    return ret_value
```

```python
d = json.loads(j, object_hook=custom_decoder)
```

```python
d
```

We could also provide our custom JSON encoder in the person class to serialize that class in the way we expect when deserializing, as we saw in an earlier video:

```python
class Person:
    def __init__(self, name, ssn):
        self.name = name
        self.ssn = ssn
        
    def __repr__(self):
        return f'Person(name={self.name}, ssn={self.ssn})'
    
    def toJSON(self):
        return dict(objecttype='person', name=self.name, ssn=self.ssn)
```

We can then encode using the techniques we have seen before, and decode using the technique we learned in this video.


There are also a few customized hooks for integers, floats and certain special strings (`-Infinity`, `Infinity` and `NaN`).

For example, we may want to encode floats using a Decimal instead of the standard float.

We could do this by using the `parse_float` argument as follows:

```python
from decimal import Decimal
def make_decimal(arg):
    print('Received:', type(arg), arg)
    return Decimal(arg)
```

```python
j = '''
    {
        "a": 100,
        "b": 0.2,
        "c": 0.5
    }
'''
```

```python
d = json.loads(j, parse_float=make_decimal)
```

```python
d
```

As you can see we have decimals in our dictionary, instead of floats. Note also that the argument we receive is a string - it would make little sense for us to receive a float since our function is the one that wants to specifically handle converting a JSON string to some particular type.

We can also intercept handling of integers and those constant values I mentioned.

```python
j = '''
    {
        "a": 100,
        "b": Infinity
    }
'''
```

```python
json.loads(j)
```

```python
def make_int_binary(arg):
    print('Received:', type(arg), arg)
    return bin(int(arg))
```

```python
def make_const_none(arg):
    print('Received:', type(arg), arg)
    return None
```

```python
json.loads(j, 
           parse_int=make_int_binary, 
           parse_constant=make_const_none)
```

Again note that in all cases, the received argument is the **string** read from the json string.


Finally we have the `object_pairs_hook` argument. It works similarly to the `object_hook` with two differences:
1. the argument is a `list` of 2-tuples - the first value is the key, the second is the value
2. the list is ordered in the same order as the keys in the json document.

Remember that the dictionary is not **guaranteed** to be ordered in the same order as the keys in the json document - given Python 3.6+ has guaranteed dictionary order, this is likely to be true, but the documents do not mention this specifically, so at this point it should be considered an implementation detail and not relied on - if you **must** have gauranteed key order, then you will have to use the `object_pairs_hook`.

Also, you should not specify both `object_hook` and `object_pairs_hook` - if you do, then the `object_pairs_hook` will be used and `object_hook` will be ignored.

```python
j = '''
    {
        "a": [1, 2, 3, 4, 5],
        "b": 100,
        "c": 10.5,
        "d": NaN,
        "e": null,
        "f": "python"
    }
'''
```

```python
def float_handler(arg):
    print('float handler', type(arg), arg)
    return float(arg)
```

```python
def int_handler(arg):
    print('int handler', type(arg), arg)
    return int(arg)
```

```python
def const_handler(arg):
    print('const handler', type(arg), arg)
    return None
```

```python
def obj_hook(arg):
    print('obj hook', type(arg), arg)
    return arg
```

```python
def obj_pairs_hook(arg):
    print('obj pairs hook', type(arg), arg)
    return arg
```

```python
json.loads(j)
```

```python
json.loads(j, 
           object_hook=obj_hook,
           parse_float=float_handler,
           parse_int=int_handler,
           parse_constant=const_handler
          )
```

```python
json.loads(j, 
           object_pairs_hook=obj_pairs_hook,
           parse_float=float_handler,
           parse_int=int_handler,
           parse_constant=const_handler
          )
```

And if we specify both object hooks, then `object_hook` is basically ignored:

```python
json.loads(j, 
           object_hook=obj_hook,
           object_pairs_hook=obj_pairs_hook,
           parse_float=float_handler,
           parse_int=int_handler,
           parse_constant=const_handler
          )
```

As we saw in the decoding videos, we can also subclass the `JSONDecoder` class (just like we subclassed the `JSONEncoder` - we'll look at this next.
