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

### Booleans


The **bool** class is used to represent boolean values.


The **bool** class inherits from the **int** class.

```python
issubclass(bool, int)
```

Two built-in constants, **True** and **False** are singleton instances of the bool class with underlying int values of 1 and 0 respectively.

```python
type(True), id(True), int(True)
```

```python
type(False), id(False), int(False)
```

These two values are instances of the **bool** class, and by inheritance are also **int** objects.

```python
isinstance(True, bool)
```

```python
isinstance(True, int)
```

Since **True** and **False** are singletons, we can use either the **is** operator, or the **==** operator to compare them to **any** boolean expression.

```python
id(True), id(1 < 2)
```

```python
id(False), id(1 == 3)
```

```python
(1 < 2) is True, (1 < 2) == True
```

```python
(1 == 2) is False, (1 == 2) == False
```

Be careful with that last comparison, the parentheses are necessary!

```python
1 == 2 == False
```

```python
(1 == 2) == False
```

We'll look into this in detail later, but, for now, this happens because a chained comparison such as **a == b == c** is actually evaluated as **a == b and b == c**


So **1 == 2 == False ** is the same as **1 == 2 and 2 == False**

```python
1 == 2, 2 == False, 1==2 and 2==False
```

But, 

```python
(1 == 2)
```

So **(1 == 2) == False** evaluates to True


But since **False** is also **0**, we get the following:

```python
(1 == 2) == 0
```

The underlying integer values of True and False are:

```python
int(True), int(False)
```

So, using an equality comparison:

```python
1 == True, 0 == False
```

But, from an object perspective 1 and True are not the same (similarly with 0 and False)

```python
1 == True, 1 is True
```

```python
0 == False, 0 is False
```

Any integer can be cast to a boolean, and follows the rule:

bool(x) = True for any x except for zero which returns False

```python
bool(0)
```

```python
bool(1), bool(100), bool(-1)
```

Since booleans are subclassed from integers, they can behave like integers, and because of polymorphism all the standard integer operators, properties and methods apply

```python
True > False
```

```python
True + 2
```

```python
False // 2
```

```python
True + True + True
```

```python
(True + True + True) % 2
```

```python
-True
```

```python
100 * False
```

I certainly **do not** recommend you write code like that shown above, but be aware that it does work.
