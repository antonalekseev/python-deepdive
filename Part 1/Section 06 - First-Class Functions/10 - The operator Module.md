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

### The **operator** Module

```python
import operator
```

```python
dir(operator)
```

#### Arithmetic Operators


A variety of arithmetic operators are implemented.

```python
operator.add(1, 2)
```

```python
operator.mul(2, 3)
```

```python
operator.pow(2, 3)
```

```python
operator.mod(13, 2)
```

```python
operator.floordiv(13, 2)
```

```python
operator.truediv(3, 2)
```

These would have been very handy in our previous section:

```python
from functools import reduce
```

```python
reduce(lambda x, y: x*y, [1, 2, 3, 4])
```

Instead of defining a lambda, we could simply use **operator.mul**:

```python
reduce(operator.mul, [1, 2, 3, 4])
```

#### Comparison and Boolean Operators


Comparison and Boolean operators are also implemented as functions:

```python
operator.lt(10, 100)
```

```python
operator.le(10, 10)
```

```python
operator.is_('abc', 'def')
```

We can even get the truthyness of an object:

```python
operator.truth([1,2])
```

```python
operator.truth([])
```

```python
operator.and_(True, False)
```

```python
operator.or_(True, False)
```

#### Element and Attribute Getters and Setters


We generally select an item by index from a sequence by using **[n]**:

```python
my_list = [1, 2, 3, 4]
my_list[1]
```

We can do the same thing using:

```python
operator.getitem(my_list, 1)
```

If the sequence is mutable, we can also set or remove items:

```python
my_list = [1, 2, 3, 4]
my_list[1] = 100
del my_list[3]
print(my_list)
```

```python
my_list = [1, 2, 3, 4]
operator.setitem(my_list, 1, 100)
operator.delitem(my_list, 3)
print(my_list)
```

We can also do the same thing using the **operator** module's **itemgetter** function.

The difference is that this returns a callable:

```python
f = operator.itemgetter(2)
```

Now, **f(my_list)** will return **my_list[2]**

```python
f(my_list)
```

```python
x = 'python'
f(x)
```

Furthermore, we can pass more than one index to **itemgetter**:

```python
f = operator.itemgetter(2, 3)
```

```python
my_list = [1, 2, 3, 4]
f(my_list)
```

```python
x = 'pytyhon'
f(x)
```

Similarly, **operator.attrgetter** does the same thing, but with object attributes.

```python
class MyClass:
    def __init__(self):
        self.a = 10
        self.b = 20
        self.c = 30
        
    def test(self):
        print('test method running...')
```

```python
obj = MyClass()
```

```python
obj.a, obj.b, obj.c
```

```python
f = operator.attrgetter('a')
```

```python
f(obj)
```

```python
my_var = 'b'
operator.attrgetter(my_var)(obj)
```

```python
my_var = 'c'
operator.attrgetter(my_var)(obj)
```

```python
f = operator.attrgetter('a', 'b', 'c')
```

```python
f(obj)
```

Of course, attributes can also be methods.

In this case, **attrgetter** will return the object's **test** method - a callable that can then be called using **()**:

```python
f = operator.attrgetter('test')
```

```python
obj_test_method = f(obj)
```

```python
obj_test_method()
```

Just like lambdas, we do not need to assign them to a variable name in order to use them:

```python
operator.attrgetter('a', 'b')(obj)
```

```python
operator.itemgetter(2, 3)('python')
```

Of course, we can achieve the same thing using functions or lambdas:

```python
f = lambda x: (x.a, x.b, x.c)
```

```python
f(obj)
```

```python
f = lambda x: (x[2], x[3])
```

```python
f([1, 2, 3, 4])
```

```python
f('python')
```

##### Use Case Example: Sorting


Suppose we want to sort a list of complex numbers based on the real part of the numbers:

```python
a = 2 + 5j
a.real
```

```python
l = [10+1j, 8+2j, 5+3j]
sorted(l, key=operator.attrgetter('real'))
```

Or if we want to sort a list of string based on the last character of the strings:

```python
l = ['aaz', 'aad', 'aaa', 'aac']
sorted(l, key=operator.itemgetter(-1))
```

Or maybe we want to sort a list of tuples based on the first item of each tuple:

```python
l = [(2, 3, 4), (1, 2, 3), (4, ), (3, 4)]
sorted(l, key=operator.itemgetter(0))
```

#### Slicing

```python
l = [1, 2, 3, 4]
```

```python
l[0:2]
```

```python
l[0:2] = ['a', 'b', 'c']
print(l)
```

```python
del l[3:5]
print(l)
```

We can do the same thing this way:

```python
l = [1, 2, 3, 4]
```

```python
operator.getitem(l, slice(0,2))
```

```python
operator.setitem(l, slice(0,2), ['a', 'b', 'c'])
print(l)
```

```python
operator.delitem(l, slice(3, 5))
print(l)
```

#### Calling another Callable

```python
x = 'python'
x.upper()
```

```python
operator.methodcaller('upper')('python')
```

Of course, since **upper** is just an attribute of the string object **x**, we could also have used:

```python
operator.attrgetter('upper')(x)()
```

If the callable takes in more than one parameter, they can be specified as additional arguments in **methodcaller**:

```python
class MyClass:
    def __init__(self):
        self.a = 10
        self.b = 20
    
    def do_something(self, c):
        print(self.a, self.b, c)
```

```python
obj = MyClass()
```

```python
obj.do_something(100)
```

```python
operator.methodcaller('do_something', 100)(obj)
```

```python
class MyClass:
    def __init__(self):
        self.a = 10
        self.b = 20
    
    def do_something(self, *, c):
        print(self.a, self.b, c)
```

```python
obj.do_something(c=100)
```

```python
operator.methodcaller('do_something', c=100)(obj)
```

More information on the **operator** module can be found here:

https://docs.python.org/3/library/operator.html
