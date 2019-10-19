---
jupyter:
  jupytext:
    formats: ipynb,md
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

## Multi-Line Statements and Strings


Certain physical newlines are ignored in order to form a complete logical line of code.


#### Implicit Examples

```python
a = [1, 
    2, # test 2 
    3]
```

```python
a
```

You may also add comments to the end of each physical line:

```python
a = [1, #first element
    2, #second element
    3, #third element
    ]
```

```python
a
```

Note if you do use comments, you must close off the collection on a new line.

i.e. the following will not work since the closing ] is actually part of the comment:

```python
a = [1, # first element
    2 #second element]
```

This works the same way for tuples, sets, and dictionaries.

```python
a = (1, # first element
    2, #second element
    3, #third element
    )
```

```python
a
```

```python
a = {1, # first element
    2, #second element
    }
```

```python
a
```

```python
a = {'key1': 'value1', #comment,
    'key2': #comment
    'value2' #comment
    }
```

```python
a
```

We can also break up function arguments and parameters:

```python
def my_func(a, #some comment
           b, c):
    print(a, b, c)
```

```python
my_func(10, #comment
       20, #comment
       30)
```

#### Explicit Examples


You can use the ``\`` character to explicitly create multi-line statements.

```python
a = 10
b = 20
c = 30
if a > 5 \
    and b > 10 \
    and c > 20:
    print('yes!!')
```

The identation in continued-lines does not matter:

```python
a = 10
b = 20
c = 30
if a > 5 \
    and b > 10 \
        and c > 20:
    print('yes!!')
```

#### Multi-Line Strings


You can create multi-line strings by using triple delimiters (single or double quotes)

```python
a = '''this is
a multi-line string'''
```

```python
print(a)
```

Note how the newline character we typed in the multi-line string was preserved. Any character you type is preserved. You can also mix in escaped characters line any normal string.

```python
a = """some items:\n
    1. item 1
    2. item 2"""
```

```python
print(a)
```

Be careful if you indent your multi-line strings - the extra spaces are preserved!

```python
def my_func():
    a = '''a multi-line string
    that is actually indented in the second line'''
    return a
```

```python
print(my_func())
```

```python
def my_func():
    a = '''a multi-line string
that is not indented in the second line'''
    return a
```

```python
print(my_func())
```

Note that these multi-line strings are **not** comments - they are real strings and, unlike comments, are part of your compiled code. They are however sometimes used to create comments, such as ``docstrings``, that we will cover later in this course.


In general, use ``#`` to comment your code, and use multi-line strings only when actually needed (like for docstrings).


Also, there are no multi-line comments in Python. You simply have to use a ``#`` on every line.

```python
# this is
#    a multi-line
#    comment
```

The following works, but the above formatting is preferrable.

```python
# this is
    # a multi-line
    # comment
```
