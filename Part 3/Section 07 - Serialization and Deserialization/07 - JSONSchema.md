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

### JSON Schemas


Often when we work with JSON data, the way the data is formatted is not haphazard - it often conforms to some very precise specification.

For example, REST API's will conform to some specific format for JSON input and output. 

This is called conforming to a **schema**. It is very similar to how relational databases work - we have a schema that precisely defines the columns in tables, the relationships between tables and so on.

One of the main reasons for having these schemas for JSON data is that it allows us to serialize and deserialize the data more easily - we know in advance what the JSON structure will look like, and we can therefore write code that will leverage our understanding of the JSON structure.

There are many ways in which we can define a JSON schema - it could be as simple as creating a Word document that explains how the JSON needs to be structured. Although that works, there are better, standards-based approaches though.

One of these is the JSON Schema standard:
https://json-schema.org/

We don't need Python, or any programming language, to define a schema - the schema definition is completely language-independent.

But given a JSON schema, we can now use a consistent approach to serializing and deserializing the data.

Moreover, we can also write code to serialize and deserialize specific object types - since we know exactly what to expect in the JSON string.


I am not going to cover JSON Schema in any detail here, but I will show you some simple examples of how these schemas can be defined.


Let's say we are creating an API that responds to a POST method to create some resource - let's say a Person. We want our JSON structure to look like the following:


```
{
    "firstName": "...",
    "middleInitial": "...",
    "lastName": "...",
    "age": ...
}
```


We can start with a simple schema as follows:

```python
person_schema = {
    "type": "object",
    "properties": {
        "firstName": {"type": "string"},
        "middleInitial": {"type": "string"},
        "lastName": {"type": "string"},
        "age": {"type": "number"}
    }
}
```

The question now becomes, given a JSON string, does it conform to the schema or not?


For example, this one is OK:

```python
p1 = '''
    {
        "firstName": "John",
        "middleInitial": "M",
        "lastName": "Cleese",
        "age": 79
    }
'''
```

How about this one is does not:

```python
p2 = '''
    {
        "firstName": "John",
        "middleInitial": 100,
        "lastName": "Cleese",
        "age": "Unknown"
    }
'''
```

`p2` does not conform to our schema for two reasons:
1. "middleInitial" should be a string
2. "age" should be a number


How about this one?

```python
p3 = '''
    {
        "firstName": "John",
        "age": -10.5
    }
'''
```

Actually this one **does** conform to our schema - unless we indicate a field as required, it is optional.

The `"age"` field is a number, so it also conforms to our schema. But we really would want it to be an integer, and not allow negative numbers.

Fortunately, JSON Schema does allow us to be more specific with our schema:

```python
person_schema = {
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "minLength": 1
        },
        "middleInitial": {
            "type": "string",
            "minLength": 1,
            "maxLength": 1
        },
        "lastName": {
            "type": "string",
            "minLength": 1
        },
        "age": {
            "type": "integer", 
            "minimum": 0
        }
    },
    "required": ["firstName", "lastName"]
}
```

So in this schema we require that `"firstName"` and `"lastName"` be provided, and have a minimum number of characters (`1`). We do not make `"middleInitial"` required, but if it is provided it must be one, and exactly one, character long.

The `"age"` field is not required, but if it is, it must be a non-negative integer.


The JSON Schema specification is actually quite intricate and can be used to specify schemas with great accuracy and specificity.

For example, we may have a field `"eyeColor"` which must contain (if provided) one of a few specific values: `amber`, `blue`, `brown`, `gray`, `green`, `hazel`, `red`, or `violet`.

We can do this as follows:

```python
person_schema = {
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "minLength": 1
        },
        "middleInitial": {
            "type": "string",
            "minLength": 1,
            "maxLength": 1
        },
        "lastName": {
            "type": "string",
            "minLength": 1
        },
        "age": {
            "type": "integer", 
            "minimum": 0
        },
        "eyeColor": {
            "type": "string",
            "enum": ["amber", "blue", "brown", "gray", 
                     "green", "hazel", "red", "violet"]
        }
    },
    "required": ["firstName", "lastName"]
}
```

We can now go back to our original question - determining if a given JSON string conforms to a given schema. We can easily determine if the JSON is valid (we can just do a `loads` for example), but does it conform to the JSON Schema?

We could write Python code to do this ourselves, but that would be really complicated!!


Instead, I am going to use the excellent Python library linked here: https://github.com/Julian/jsonschema

You will need to install it first (usually `pip install jsonschema` in whatever environment you are using - you are using a virtual environment of some sort, right?!!)

```python
from jsonschema import validate
from jsonschema.exceptions import ValidationError
from json import loads, dumps, JSONDecodeError
```

We can use the `validate` function, but it will not work with a string - it needs to be deserialized into a Python dictionary first (which means it will have to be a valid JSON structure first).

```python
print(p1)

try:
    validate(loads(p1), person_schema)
except JSONDecodeError as ex:
    print(f'Invalid JSON: {ex}')
except ValidationError as ex:
    print(f'Validation error: {ex}')
else:
    print('JSON is valid')
```

```python
print(p2)

try:
    validate(loads(p2), person_schema)
except JSONDecodeError as ex:
    print(f'Invalid JSON: {ex}')
except ValidationError as ex:
    print(f'Validation error: {ex}')
else:
    print('JSON is valid')
```

```python
print(p3)
try:
    validate(loads(p3), person_schema)
except JSONDecodeError as ex:
    print(f'Invalid JSON: {ex}')
except ValidationError as ex:
    print(f'Validation error: {ex}')
else:
    print('JSON is valid')
```

You'll notice that the validator only returns the first validation error it encounters. This can be changed to run the entire validation and return all the validation errors (if any), but utilizes a slightly different way of performing validation:

```python
from jsonschema import Draft4Validator

validator = Draft4Validator(person_schema)
```

```python
for error in validator.iter_errors(loads(p2)):
    print(error, end='\n-----------\n')
```

We can also test out the schema for `eyeColor`:

```python
p4 = '''
    {
        "firstName": "John",
        "middleInitial": null,
        "lastName": "Cleese",
        "eyeColor": "blue-gray"
    }
'''
```

```python
for error in validator.iter_errors(loads(p4)):
    print(error, end='\n-----------\n')    
```

So JSON Schema paired with this library is a great way to ensure a JSON document conforms to some specific schema. It is useful even when you create your own JSON serializer to make sure you are conforming to your own pre-determined schema - especially useful in unit testing to make sure you did not miss something when serializing your objects to JSON.

But all this does not address the other issue we have - serializing and deserializing Python objects to and from JSON strings (marshalling).

Not to worry, there are also quite a few libraries out there that will help with this difficult task too.

In the next video I will look at one of the more popular ones - Mashmallow - but there are others as well.
