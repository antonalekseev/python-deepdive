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

### Decimals

```python
import decimal
```

```python
from decimal import Decimal
```

Decimals have context, that can be used to specify rounding and precision (amongst other things)


Contexts can be local (temporary contexts) or global (default)


#### Global Context

```python
g_ctx  = decimal.getcontext()
```

```python
g_ctx.prec
```

```python
g_ctx.rounding
```

We can change settings in the global context:

```python
g_ctx.prec = 6
```

```python
g_ctx.rounding = decimal.ROUND_HALF_UP
```

And if we read this back directly from the global context:

```python
decimal.getcontext().prec
```

```python
decimal.getcontext().rounding
```

we see that the global context was indeed changed.


#### Local Context


The ``localcontext()`` function will return a context manager that we can use with a ``with`` statement:

```python
with decimal.localcontext() as ctx:
    print(ctx.prec)
    print(ctx.rounding)
```

Since no argument was specified in the ``localcontext()`` call, it provides us a context manager that uses a copy of the global context.


Modifying the local context has no effect on the global context

```python
with decimal.localcontext() as ctx:
    ctx.prec = 10
    print('local prec = {0}, global prec = {1}'.format(ctx.prec, g_ctx.prec))
```

#### Rounding

```python
decimal.getcontext().rounding
```

The rounding mechanism is ROUND_HALF_UP because we set the global context to that earlier in this notebook. Note that normally the default is ROUND_HALF_EVEN.

So we first reset our global context rounding to that:

```python
decimal.getcontext().rounding = decimal.ROUND_HALF_EVEN
```

```python
x = Decimal('1.25')
y = Decimal('1.35')
print(round(x, 1))
print(round(y, 1))
```

Let's change the rounding mechanism in the global context to ROUND_HALF_UP:

```python
decimal.getcontext().rounding = decimal.ROUND_HALF_UP
```

```python
x = Decimal('1.25')
y = Decimal('1.35')
print(round(x, 1))
print(round(y, 1))
```

As you may have realized, changing the global context is a pain if you need to constantly switch between different precisions and rounding algorithms. Also, it could introduce bugs if you forget that you changed the global context somewhere further up in your module.

For this reason, it is usually better to use a local context manager instead:


First we reset our global context rounding to the default:

```python
decimal.getcontext().rounding = decimal.ROUND_HALF_EVEN
```

```python
x = Decimal('1.25')
y = Decimal('1.35')
print(round(x, 1), round(y, 1))
with decimal.localcontext() as ctx:
    ctx.rounding = decimal.ROUND_HALF_UP
    print(round(x, 1), round(y, 1))
print(round(x, 1), round(y, 1))
```
