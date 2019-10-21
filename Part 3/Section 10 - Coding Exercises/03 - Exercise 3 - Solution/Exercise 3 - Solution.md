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

### Exercise 3 - Solution


You are given three JSON files, representing a default set of settings, and environment specific settings.
The files are included in the downloads, and are named:
* `common.json`
* `dev.json`
* `prod.json`

Your goal is to write a function that has a single argument, the environment name, and returns the "combined" dictionary that merges the two dictionaries together, with the environment specific settings overriding any common settings already defined.

For simplicity, assume that the argument values are going to be the same as the file names, without the `.json` extension. So for example, `dev` or `prod`.

The wrinkle: We don't want to duplicate data for the "merged" dictionary - use `ChainMap` to implement this instead.


The first thing we'll need to do is write a function to load the JSON files:

```python
import json

def load_settings(env):
    # assume file name is <env>.json
    with open(f'{env}.json') as f:
        settings = json.load(f)
    return settings
```

```python
from pprint import pprint
```

```python
pprint(load_settings('common'))
```

```python
pprint(load_settings('dev'))
```

```python
pprint(load_settings('prod'))
```

OK, so our function seems to work fine.
Now time to "combine" our settings - let's try this simple approach first.

Spoiler alert: this won't work as expected!

```python
from collections import ChainMap

def settings(env):
    # combine common.json and <env>.json, with env settings taking precedence
    common_settings = load_settings('common')
    env_settings = load_settings(env)
    return ChainMap(env_settings, common_settings)
```

```python
dev = settings('dev')
```

```python
for k, v in dev.items():
    print(k, ':', v)
```

**What happened to the values that were in `common`??**


For example, we don't see the `database` `port`??


This does not work as intended because of sub-dictionaries - as you can see the dictionary for `database` for example is the one from `dev`, and not a "combined" dictionary.

`ChainMap` is not recursive, so this is not going to work for us as it stands.


We need to use a recursive approach to handle any amount of nesting.

Let's think how we would do this for a single level.

When we chain two dictionaries together, we will have to replace any sub-dictionary with a chain of the sub-dictionaries further down the line - fortunately our line is two, since we only deal with `common` and either `dev` or `prod` (or whatever environment names we want to support).

So if a key in `dev` (for example), has a dictionary value, we need to chain that sub-dictionary too. And if any of the keys in the chained-subdictionary contains nested dictionaries, we need to chain those too.

```python
def chain_recursive(d1, d2):
    chain = ChainMap(d1, d2)
    for k, v in d1.items():
        if isinstance(v, dict) and k in d2:
            chain[k] = chain_recursive(d1[k], d2[k])
    return chain
```

```python
d1 = load_settings('common')
d2 = load_settings('dev')
```

```python
dev = chain_recursive(d2, d1)
```

```python
pprint(dev)
```

This means that we can lookup the log level for example, which we know should be `trace` for our development environment:

```python
dev['logs']['level']
```

If instead we load up our production environment:

```python
d3 = load_settings('prod')
prod = chain_recursive(d3, d1)
```

```python
prod['logs']['level']
```

and the database port, from the common settings:

```python
prod['database']['port']
```

but, we have the override for the user:

```python
prod['database']['user']
```

So now, let's package this up in a neat function for our users:

```python
def settings(env):
    common_settings = load_settings('common')
    env_settings = load_settings(env)
    return chain_recursive(env_settings, common_settings)
```

```python
prod = settings('prod')
```

```python
prod['database']['user']
```

```python
dev = settings('dev')
dev['logs']['level']
```

Let's also check some deeper nested dictionaries:

```python
prod['data']['numerics']['type']
```

```python
dev['data']['numerics']['type']
```

```python
dev['data']['operators']
```

So this seems to work just fine. You may want to further refine this to merge list type values as well - for example, `key1: [1, 2, 3]` in `common` and `key2: [3, 4, 5]` in `dev` might result in `key1: [1, 2, 3, 4, 5]`. This is a rarer requirement, but do note that the solution I present here will simply replace the entire list with what is in the `dev` file, not merge the two lists.
