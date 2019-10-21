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

## Function Arguments and  Mutability


Consider a function that receives a *string* argument, and changes the argument in some way:

```python
def process(s):
    print('initial s # = {0}'.format(hex(id(s))))
    s = s + ' world'
    print('s after change # = {0}'.format(hex(id(s))))
```

```python
my_var = 'hello'
print('my_var # = {0}'.format(hex(id(my_var))))
```

Note that when *s* is received, it is referencing the same object as *my_var*.

After we "modify" *s*, *s* is pointing to a new memory address:

```python
process(my_var)
```

And our own variable *my_var* is still pointing to the original memory address:

```python
print('my_var # = {0}'.format(hex(id(my_var))))
```

Let's see how this works with mutable objects:

```python
def modify_list(items):
    print('initial items # = {0}'.format(hex(id(items))))
    if len(items) > 0:
        items[0] = items[0] ** 2
    items.pop()
    items.append(5)
    print('final items # = {0}'.format(hex(id(items))))
```

```python
my_list = [2, 3, 4]
print('my_list # = {0}'.format(hex(id(my_list))))
```

```python
modify_list(my_list)
```

```python
print(my_list)
print('my_list # = {0}'.format(hex(id(my_list))))
```

As you can see, throughout all the code, the memory address referenced by *my_list* and *items* is always the **same** (shared) reference - we are simply modifying the contents (**internal state**) of the object at that memory address.


Now, even with immutable container objects we have to be careful, e.g. a tuple containing a list (the tuple is immutable, but the list element inside the tuple **is** mutable)

```python
def modify_tuple(t):
    print('initial t # = {0}'.format(hex(id(t))))
    t[0].append(100)
    print('final t # = {0}'.format(hex(id(t))))
```

```python
my_tuple = ([1, 2], 'a')
```

```python
hex(id(my_tuple))
```

```python
modify_tuple(my_tuple)
```

```python
my_tuple
```

As you can see, the first element of the tuple was mutated.
