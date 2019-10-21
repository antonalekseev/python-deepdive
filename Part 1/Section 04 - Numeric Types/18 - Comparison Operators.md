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

### Comparison Operators


#### Identity and Membership Operators


The **is** and **is not** operators will work with any data type since they are comparing the memory addresses of the objects (which are integers)

```python
0.1 is (3+4j)
```

```python
'a' is [1, 2, 3]
```

The **in** and **not in** operators are used with iterables and test membership:

```python
1 in [1, 2, 3]
```

```python
[1, 2] in [1, 2, 3]
```

```python
[1, 2] in [[1,2], [2,3], 'abc']
```

```python
'key1' in {'key1': 1, 'key2': 2}
```

```python
1 in {'key1': 1, 'key2': 2}
```

We'll come back to these operators in later sections on iterables and mappings.


#### Equality Operators


The **==** and **!=** operators are value comparison operators. 

They will work with mixed types that are comparable in some sense.

For example, you can compare Fraction and Decimal objects, but it would not make sense to compare string and integer objects.

```python
1 == '1'
```

```python
from decimal import Decimal
from fractions import Fraction
```

```python
Decimal('0.1') == Fraction(1, 10)
```

```python
1 == 1 + 0j
```

```python
True == Fraction(2, 2)
```

```python
False == 0j
```

#### Ordering Comparisons


Many, but not all data types have an ordering defined.

For example, complex numbers do not.

```python
1 + 1j < 2 + 2j
```

Mixed type ordering comparisons is supported, but again, it needs to make sense:

```python
1 < 'a'
```

```python
Decimal('0.1') < Fraction(1, 2)
```

#### Chained Comparisons


It is possible to chain comparisons.

For example, in **a < b < c**, Python simply **ands** the pairwise comparisons: **a < b and b < c**

```python
1 < 2 < 3
```

```python
1 < 2 > -5 < 50 > 4
```

```python
1 < 2 == Decimal('2.0')
```

```python
import string
'A' < 'a' < 'z' > 'Z' in string.ascii_letters 
```
