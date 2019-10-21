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

### Decimals: Constructors and Contexts


The **Decimal** constructor can handle a variety of data types

```python
import decimal
from decimal import Decimal
```

#### Integers

```python
Decimal(10)
```

```python
Decimal(-10)
```

#### Strings

```python
Decimal('0.1')
```

```python
Decimal('-3.1415')
```

#### Tuples

```python
Decimal ((0, (3,1,4,1,5), -4))
```

```python
Decimal((1, (1,2,3,4), -3))
```

```python
Decimal((0, (1,2,3), 3))
```

#### But don't use Floats

```python
format(0.1, '.25f')
```

```python
Decimal(0.1)
```

As you can see, since we passed an approximate binary float to the Decimal constructor it did it's best to represent that binary float **exactly**!!


So, instead, use strings or tuples in the Decimal constructor.


#### Context Precision and the Constructor


The context precision does nto affect the precision used when creating a Decimal object - those are independent of each other.


Let's set our global (default) context to a precision of 2

```python
decimal.getcontext().prec = 2
```

Now we can create decimal numbers of higher precision than that:

```python
a = Decimal('0.12345')
b = Decimal('0.12345')
```

```python
a
```

```python
b
```

But when we add those two numbers up, the context precision will matter:

```python
a+b
```

As you can see, we ended up with a sum that was rounded to 2 digits after the decimal point (precision = 2)


#### Local and Global Contexts are Independent

```python
decimal.getcontext().prec = 6
```

```python
decimal.getcontext().rounding
```

```python
a = Decimal('0.12345')
b = Decimal('0.12345')
print(a + b)
with decimal.localcontext() as ctx:
    ctx.prec = 2
    c = a + b
    print('c within local context: {0}'.format(c))
print('c within global context: {0}'.format(c))
```

Since **c** was created within the local context by adding **a** and **b**, and the local context had a precision of 2, **c** was rounded to 2 digits after the decimal point.

Once the local context is destroyed (after the **with** block), the variable **c** still exists, and its precision is **still** just 2 - it doesn't magically suddenly get the global context's precision of 6.
