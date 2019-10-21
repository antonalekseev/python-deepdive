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

### Complex Numbers


Python's built-in class provides support for complex numbers.

Complex numbers are defined in rectangular coordinates (real and imaginary parts) using either the constructor or a literal expression.


The complex number 1 + 2j can be defined in either of these ways:

```python
a = complex(1, 2)
b = 1 + 2j
```

```python
a == b
```

Note that the real and imaginary parts are defined as floats, and can be retrieved as follows:

```python
a.real, type(a.real)
```

```python
a.imag, type(a.imag)
```

The complex conjugate can be calculated as follows:

```python
a.conjugate()
```

The standard arithmetic operatots are polymorphic and defined for complex numbers

```python
a = 1 + 2j
b = 3 - 4j
c = 5j
d = 10
```

```python
a + b
```

```python
b * c
```

```python
c / d
```

```python
d - a
```

The // and % operators, although also polymorphic, are not defined for complex numbers:

```python
a // b
```

```python
a % b
```

The == and != operators support complex numbers - but since the real and imaginary parts of complex numbers are floats, the same problems comparing floats using == and != also apply to complex numbers.

```python
a = 0.1j
a + a + a == 0.3j
```

In addition, the standard comparison operators (<, <=, >, >=) are not defined for complex numbers.

```python
a = 1 + 1j
b = 100 + 100j
a < b
```

#### Math Functions


The **cmath** module provides complex alternatives to the standard **math** functions.


In addition, the **cmath** module provides the complex implementation of the **isclose()** method available for floats.

```python
import cmath

a = 1 + 5j
print(cmath.sqrt(a))
```

The standard **math** module functions will not work with complex numbers:

```python
import math
print(math.sqrt(a))
```

#### Polar / Rectangular Conversions


The **cmath.phase()** function can be used to return the phase (or argument) of  any complex number.


The standard **abs()** function supports complex numbers and will return the magnitude (euclidean norm) of the complex number.

```python
a = 1 + 1j
```

```python
r = abs(a)
phi = cmath.phase(a)
print('{0} = ({1},{2})'.format(a, r, phi))
```

Complex numbers in polar coordinates can be converted to rectangular coordinates using the **math.rect()** function:

```python
r = math.sqrt(2)
phi = cmath.pi/4
print(cmath.rect(r, phi))
```

#### Euler's Identity and the **isclose()** function


e<sup>i &pi;</sup> + 1 = 0

```python
RHS = cmath.exp(cmath.pi * 1j) + 1
print(RHS)
```

Which, because of limited precision is not quite zero.


However, the result is very close to zero.

We can use the **isclose()** method of the **cmath** module, which behaves similarly to the **math.isclose()** method. Since we are testing for closeness of two numbers close to zero, we need to make sure an absolute tolerance is also specified:

```python
cmath.isclose(RHS, 0, abs_tol=0.00001)
```

If we had not specified an absolute tolerance:

```python
cmath.isclose(RHS, 0)
```
