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

### Data Attributes


Let's focus on data attributes first (non-callables).

As we saw before we can have class attributes - they live in the class dictionary:

```python
class BankAccount:
    apr = 1.2
```

```python
BankAccount.__dict__
```

```python
BankAccount.apr
```

Now when we create instances of that class:

```python
acc_1 = BankAccount()
acc_2 = BankAccount()
```

The instance dictionaries are currently empty:

```python
acc_1.__dict__, acc_2.__dict__
```

Yet, these instances do have an `apr` attribute:

```python
acc_1.apr, acc_2.apr
```

Where is that value coming from? The class the objects were created from!


In fact, if we modify the class attribute:

```python
BankAccount.apr = 2.5
```

We'll see this reflected in the instances as well:

```python
acc_1.apr, acc_2.apr
```

And if we a a class attribute to `BankAccount`:

```python
BankAccount.account_type = 'Savings'
```

```python
acc_1.account_type, acc_2.account_type
```

As you can see modifying attributes in the **class** are reflected in the instances too - that's because Python does not find an `apr` attribute in the instance dic tionary, so next it looks in the class that was used to create the instance.

Which raises the question, what happens if we add `apr` to the **instance** dictionary?

```python
acc_1.apr = 0
```

Well that did not raise an exception - so what's happening now:

```python
acc_1.__dict__, acc_2.__dict__
```

As you can see, we actually create an entry for `apr` in the state dictionary of `acc_1`.

Now that we have it there, it we try to get the attribute value `apr` for `acc_1`, Python will find it in the instance dictionary, so it will use that instead!

```python
acc_1.apr, acc_2.apr
```

In effect, the instance attribute `apr` is **hiding** the class attribute.

You'll notice also that `acc_2` was **not** affected - this is because we did not modify `acc_2`'s dictionary, just the dictionary for `acc_1`.

And the `getattr` and `setattr` functions work the same way as dotted notation:

```python
acc_1 = BankAccount()
print(acc_1.__dict__)
print(acc_1.apr)
print(getattr(acc_1, 'apr'))
```

```python
setattr(acc_1, 'apr', 0)
print(acc_1.__dict__)
print(acc_1.apr)
print(getattr(acc_1, 'apr'))
```

We can even add instance attributes directly to an instance:

```python
acc_1.bank = 'Acme Savings & Loans'
```

```python
acc_1.__dict__
```

But this is specific to the instance, and only that specific instance:

```python
acc_2 = BankAccount()
```

```python
acc_2.__dict__
```

As you can see `acc_2` has an empty instance dictionary.


So it is really important to distingush between **class attributes** and **instance attributes**.

**Class attributes** are like attributes that are "common" to all instances - because the attribute does not live in the instance, but in the class itself.

On the other hand, **instance attributes** are specific to each instance, and values for the same attribute can be different across multiple instances, as we just saw with `acc_1.apr` and `acc_2.apr`.


So, in summary, classes and instances each have their own state - usually maintained in a dictionary, available through `__dict__`. Irrespective of where the state is stored, when we look up an attribute on an instance, Python will first look for the attribute in the instance's local state. If it does not find it there, it will next look for it in the class of the instance.


One other thing to note is the difference in type between class and instance `__dict__`.

Classes as we saw, return a `mapping proxy` object:

```python
BankAccount.__dict__
```

But instances, return a real dictionary:

```python
acc_1.__dict__
```

So with instances, unlike with classes, we can manipulate that dictionary directly:

```python
class Program:
    language = 'Python'
```

```python
p = Program()
```

```python
p.__dict__
```

```python
p.__dict__['version'] = '3.7'
```

```python
p.__dict__
```

```python
p.version, getattr(p, 'version')
```

But once again, this only affects that specific **instance**.

```python

```
