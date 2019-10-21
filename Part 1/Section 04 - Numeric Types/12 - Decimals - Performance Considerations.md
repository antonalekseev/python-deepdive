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

### Decimals: Performance Considerations


#### Memory Footprint


Decimals take up a lot more memory than floats.

```python
import sys
from decimal import Decimal
```

```python
a = 3.1415
b = Decimal('3.1415')
```

```python
sys.getsizeof(a)
```

24 bytes are used to store the float 3.1415

```python
sys.getsizeof(b)
```

104 bytes are used to store the Decimal 3.1415


#### Computational Performance


Decimal arithmetic is also much slower than float arithmetic (on a CPU, and even more so if using a GPU)


We can do some rough timings to illustrate this.


First we look at the performance difference creating floats vs decimals:

```python
import time
from decimal import Decimal

def run_float(n=1):
    for i in range(n):
        a = 3.1415
        
def run_decimal(n=1):
    for i in range(n):
        a = Decimal('3.1415')

```

Timing float and Decimal operations:

```python
n = 10000000
```

```python
start = time.perf_counter()
run_float(n)
end = time.perf_counter()
print('float: ', end-start)

start = time.perf_counter()
run_decimal(n)
end = time.perf_counter()
print('decimal: ', end-start)
```

We make a slight variant here to see how addition compares between the two types:

```python
def run_float(n=1):
    a = 3.1415
    for i in range(n):
        a + a
        
def run_decimal(n=1):
    a = Decimal('3.1415')
    for i in range(n):
        a + a
        
start = time.perf_counter()
run_float(n)
end = time.perf_counter()
print('float: ', end-start)

start = time.perf_counter()
run_decimal(n)
end = time.perf_counter()
print('decimal: ', end-start)
```

How about square roots:

(We drop the n count a bit)

```python
n = 5000000

import math

def run_float(n=1):
    a = 3.1415
    for i in range(n):
        math.sqrt(a)
        
def run_decimal(n=1):
    a = Decimal('3.1415')
    for i in range(n):
        a.sqrt()
        
start = time.perf_counter()
run_float(n)
end = time.perf_counter()
print('float: ', end-start)

start = time.perf_counter()
run_decimal(n)
end = time.perf_counter()
print('decimal: ', end-start)
```
