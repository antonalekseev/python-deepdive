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

### Serpy


If you're just looking for deserialization, then `Serpy` might work for you. It is extremely fast, but only provides serialization.

You can read more about Serpy here: https://serpy.readthedocs.io/en/latest/

Here's a simple example first, using our goto Person object.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        
    def __repr__(self):
        return f'Person(name={self.name}, age={self.age})'
```

```python
import serpy
```

Very similarly to `Marshmallow` we need to define a schema for the serialization - Serpy calls those objects serializers:

```python
class PersonSerializer(serpy.Serializer):
    name = serpy.StrField()
    age = serpy.IntField()
```

```python
p1 = Person('Michael Palin', 75)
```

```python
PersonSerializer(p1).data
```

Of course, we can get more complex schemas defined.

Let's implement a schema for our `Movie` example we did in a previous video on Marshmallow.

```python
class Movie:
    def __init__(self, title, year, actors):
        self.title = title
        self.year = year
        self.actors = actors
```

```python
class MovieSerializer(serpy.Serializer):
    title = serpy.StrField()
    year = serpy.IntField()
    actors = PersonSerializer(many=True)
```

```python
p2 = Person('John Cleese', 79)
```

```python
movie = Movie('Parrot Sketch', 1989, [p1, p2])
```

```python
movie.title, movie.year, movie.actors
```

```python
MovieSerializer(movie).data
```

Note that the result of serialization is to a basic Python dictionary, and you can takes this further to JSON or YAML using the standard library `json` module or `PyYaml`.

For example:

```python
import json
import yaml
```

```python
json.dumps(MovieSerializer(movie).data)
```

```python
print(yaml.dump(MovieSerializer(movie).data, 
          default_flow_style=False))
```
