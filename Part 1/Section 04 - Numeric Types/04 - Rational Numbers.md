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

## Rational Numbers

```python
from fractions import Fraction
```

We can get some info on the Fraction class:

```python
help(Fraction)
```

We can create Fraction objects in a variety of ways:


Using integers:

```python
Fraction(1)
```

```python
Fraction(1, 3)
```

Using rational numbers:

```python
x = Fraction(2, 3)
y = Fraction(3, 4)
# 2/3 / 3/4 --> 2/3 * 4/3 --> 8/9
Fraction(x, y)
```

Using floats:

```python
Fraction(0.125)
```

```python
Fraction(0.5)
```

Using strings:

```python
Fraction('10.5')
```

```python
Fraction('22/7')
```

Fractions are automatically reduced:

```python
Fraction(8, 16)
```

Negative sign is attached to the numerator:

```python
Fraction(1, -4)
```

Standard arithmetic operators are supported:

```python
Fraction(1, 3) + Fraction(1, 3) + Fraction(1, 3)
```

```python
Fraction(1, 2) * Fraction(1, 4)
```

```python
Fraction(1, 2) / Fraction(1, 3)
```

We can recover the numerator and denominator (integers):

```python
x = Fraction(22, 7)
print(x.numerator)
print(x.denominator)
```

Since floats have **finite** precision, any float can be converted to a rational number:

```python
import math
x = Fraction(math.pi)
print(x)
print(float(x))
```

```python
x = Fraction(math.sqrt(2))
print(x)
```

Note that these rational values are approximations to the irrational numbers $\pi$ and $\sqrt{2}$


**Beware!!**


Float number representations (as we will examine in future lessons) do not always have an exact representation.


The number 0.125 (1/8) **has** an exact representation:

```python
Fraction(0.125)
```

and so we see the expected equivalent fraction.

But, 0.3 (3/10) does **not** have an exact representation:

```python
Fraction(3, 10)
```

but

```python
Fraction(0.3)
```

We will study this in upcoming lessons.

But for now, let's just see a quick explanation:

```python
x = 0.3
```

```python
print(x)
```

Everything looks ok here - why am I saying 0.3 (float) is just an approximation?

Python is trying to format the displayed value for readability - so it rounds the number for a better display format!

We can instead choose to display the value using a certain number of digits:

```python
format(x, '.5f')
```

At 5 digits after the decimal, we might still think 0.3 is an exact representation.

But let's display a few more digits:

```python
format(x, '.15f')
```

Hmm... 15 digits and still looking good!

How about 25 digits...

```python
format(x, '.25f')
```

Now we see that **x** is not quite 0.3...


In fact, we can quantify the delta this way:

```python
delta = Fraction(0.3) - Fraction(3, 10)
```

Theoretically, delta should be 0, but it's not:

```python
delta == 0
```

```python
delta
```

**delta** is a very small number, the above fraction...

As a float:

```python
float(delta)
```

#### Constraining the denominator

```python
x = Fraction(math.pi)
print(x)
print(format(float(x), '.25f'))
```

```python
y = x.limit_denominator(10)
print(y)
print(format(float(y), '.25f'))
```

```python
y = x.limit_denominator(100)
print(y)
print(format(float(y), '.25f'))
```

```python
y = x.limit_denominator(500)
print(y)
print(format(float(y), '.25f'))
```
