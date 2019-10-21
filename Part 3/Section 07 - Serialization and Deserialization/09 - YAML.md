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

### YAML Format


YAML, like JSON, is another data serialization standard. It is actually easier to read than JSON, and although it has been around for a long time (since 2001), it has gained a lot of popularity, especially in the Dev Ops world for configuration files (Docker, Kubernetes, etc).

Like JSON it is able to represent simple data types (strings, numbers, boolean, etc) as well as collections and associative arrays (dictionaries).

YAML focuses on human readability, and is a little more complex to parse.

Here is a sample YAML file:


```
title: Parrot Sketch
year: 1989
actors:
    - first_name: John
      last_name: Cleese
      dob: 1939-10-27
    - first_name: Michael
      last_name: Palin
      dob: 1943-05-05
```


As you can see this is much easier to read than JSON or XML.


To parse YAML into a Python dictionary would take a fair amount of work - especially since YAML is quite flexible.


Fortunately, we can use the 3rd party library, `pyyaml` to do this for us.

Again, I'm only going to show you a tiny bit of this library, and you can read more about it here:
https://pyyaml.org/wiki/PyYAMLDocumentation

(It's definitely less of a learning curve than Marshmallow!!)


#### Caution
When you load a yaml file using pyyaml, be careful - like pickling it can actually call out to Python functions - so do not load untrusted YAML files using `pyyaml`!

```python
import yaml
```

```python
data = '''
---
title: Parrot Sketch
year: 1989
actors:
    - first_name: John
      last_name: Cleese
      dob: 1939-10-27
    - first_name: Michael
      last_name: Palin
      dob: 1943-05-05
'''
```

```python
d = yaml.load(data)
```

```python
type(d)
```

```python
from pprint import pprint

pprint(d)
```

You'll notice that unlike the built-in JSON parser, PyYAML was able to automatically deduce the `date` type in our YAML, as well of course as strings and integers.

Of course, serialization works the same way:

```python
d = {'a': 100, 'b': False, 'c': 10.5, 'd': [1, 2, 3]}
```

```python
print(yaml.dump(d))
```

You'll notice in the above example that the list was represented using `[1, 2, 3]` - this is valid YAML as well, and is equivalent to this notation:


```
d:
    - 1
    - 2
    - 3
```


If you prefer this block style, you can force it this way:

```python
print(yaml.dump(d, default_flow_style=False))
```

What's interesting about PyYAML is that it can also automatically serialize and deserialize complex objects:

```python
class Person:
    def __init__(self, name, dob):
        self.name = name
        self.dob = dob
        
    def __repr__(self):
        return f'Person(name={self.name}, dob={self.dob})'
```

```python
from datetime import date

p1 = Person('John Cleese', date(1939, 10, 27))
p2 = Person('Michael Palin', date(1934, 5, 5))
```

```python
print(yaml.dump({'john': p1, 'michael': p2}))
```

Notice that weird looking syntax? It's actually useful when we deserialize the YAML string - of course it means we must have a `Person` class defined with the appropriate init method.

```python
yaml_data = '''
john: !!python/object:__main__.Person 
    dob: 1939-10-27
    name: John Cleese
michael: !!python/object:__main__.Person 
    dob: 1934-05-05
    name: Michael Palin
'''
```

```python
d = yaml.load(yaml_data)
```

```python
d
```

As you can see, `john` and `michael` were deserialized into `Person` type objects.

This is why you have to be quite careful with the source of any YAML you deserialize.

Here's an evil example:

```python
yaml_data = '''
exec_paths: 
    !!python/object/apply:os.get_exec_path []
exec_command:
    !!python/object/apply:subprocess.check_output [['ls', '/']]
'''
```

```python
yaml.load(yaml_data)
```

So, be very careful with `load`. In general it is safer practice to use the `safe_load` method instead, but you will lose the ability to deserialize into custom Python objects, unless you override that behavior. You can always use Marshmallow to do that secondary step in a safer way.

```python
yaml.safe_load(yaml_data)
```

To override and allow certain Python objects to be deserialized in `safe_load` we can proceed this way.

Firstly we are going to simplify the object tag notation by customizing it in our `Person` class, and we are also going to make our object as safe to be deserialized. Our `Person` class will now have to inherit from the `yaml.YAMLObject`:

```python
from yaml import YAMLObject, SafeLoader

class Person(YAMLObject):
    yaml_tag = '!Person'
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
        
    def __repr__(self):
        return f'Person(name={self.name}, age={self.age})'
```

First let's see how objects are now serialized:

```python
yaml.dump(dict(john=Person('John Cleese', 79),
               michael=Person('Michael Palin', 74)))
```

As you can see we have a slightly cleaner syntax.

Now let's try to load the serialized version:

```python
yaml_data = '''
john: !Person
    name: John Cleese
    age: 79
michael: !Person
    name: Michael Palin
    age: 74
'''
```

```python
yaml.load(yaml_data)
```

And `safe_load`:

```python
yaml.safe_load(yaml_data)
```

So now let's mark our `Person` object as safe:

```python
class Person(YAMLObject):
    yaml_tag = '!Person'
    yaml_loader = SafeLoader
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
        
    def __repr__(self):
        return f'Person(name={self.name}, age={self.age})'
```

```python
yaml.safe_load(yaml_data)
```

And as you can see, the deserializtion now works for the `Person` class.


There's a lot more this library can do, so look at the reference if you want to use YAML. 

Also, as I mentionmed before, you can combine this with `Marshmallow` for example to get to a full marshalling solution to complex (custom) Python types.
