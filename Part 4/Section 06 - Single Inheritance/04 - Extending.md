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

### Extending


So far we have seen inheriting and overriding methods from a parent class.

We can also provide additional functionality in child classes. This is very straightforward, we simply define methods (or attributes) in the child class.

In fact we have already done this multiple times - whenever we create a class and define attributes and methods, we are essentially extending the functionality of the `object` class!

```python
class Person:
    pass
```

```python
class Student(Person):
    def study(self):
        return 'study... study... study...'
```

```python
p = Person()
```

```python
try:
    p.study()
except AttributeError as ex:
    print(ex)
```

```python
s = Student()
```

```python
isinstance(s, Person)
```

```python
s.study()
```

Now, think back to what happened when we provided an override in a child class and called the method from inside a method in the parent class.

Since the method being called was bound to an instance of the child class we ended up calling the override method in the child class.

The same thing happens here:

```python
class Person:
    def routine(self):
        return self.eat() + self.study() + self.sleep()
        
    def eat(self):
        return 'Person eats...'
    
    def sleep(self):
        return 'Person sleeps...'
        
```

Now we have a problem here! We call `self.study()` in the `routine` method of `Person`, but of course that method does not exist.


We get this exception if we try to call `routine`:

```python
p = Person()

try:
    p.routine()
except AttributeError as ex:
    print(ex)
```

But watch what happens if we create a `Student` class that inherits from `Person` and extends that class by implementing a `study` method:

```python
class Student(Person):
    def study(self):
        return 'Student studies...'
```

```python
s = Student()
```

```python
s.routine()
```

So, `Person` does not implement `sleep`, but `Student` does. In this case, since we are directly calling `sleep` from the `Person` class we really want that method to exist. Or we could check if the instance has that method before we call it.

Let's do the latter first:

```python
class Person:
    def routine(self):
        result = self.eat()
        if hasattr(self, 'study'):
            result += self.study()
        result += self.sleep()
        return result
    
    def eat(self):
        return 'Person eats...'
    
    def sleep(self):
        return 'Person sleeps...'
```

```python
p = Person()
```

```python
p.routine()
```

So that works, and if our child class implements the `study` method:

```python
class Student(Person):
    def study(self):
        return 'Student studies...'
```

```python
s = Student()
```

```python
s.routine()
```

There are times when we want our base class to be used as a base class only, and not really directly. This starts getting into abstract classes, so I won't cover it now beyond a few basics.


Suppose we want our "base" class to be something that is used via inheritance, and not really directly. If you've studied Java OOP, you probably are aware of this coconcept alreads: **abstract** classes.

Abstract classes are basically classes that are not meant to be instantiated directly, but instead used in some inheritance chain.

For now, we can achieve this quite simply in Python by actually implementing the method in the "base" class, but returning a `NotImplemented` value, letting the users of our class know that they need to implement the functionality by overriding the method.


We could do it this way:

```python
class Person:
    def __init__(self, name):
        self.name = name
        
    def routine(self):
        return NotImplemented
```

```python
p = Person('Alex')
```

```python
p.routine()
```

And now we can extend this class, providing an override for that method:

```python
class Student(Person):
    def routine(self):
        return 'Eat...Study...Sleep'
```

```python
class Teacher(Person):
    def routine(self):
        return 'Eat...Teach...Sleep'
```

```python
s = Student('Alex')
```

```python
t = Teacher('Fred')
```

```python
s.routine()
```

```python
t.routine()
```

The drawback of our current approach is that we can still create instances of the `Person` class - but doing so does not make much sense since we really need the `routine` method to be defined.

To address this properly we will need to look at the framework Python provides for abstract base classes (*ABC*), but is beyond our current scope.


Everything I have explained concerning the method being always bound to the instance applies equally well to any instance or class attribute.

Let's look at an example of this:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {self.apr}'
        
```

```python
a = Account(123, 100)
```

```python
a.apr, a.account_type, a.calc_interest()
```

```python
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account'
```

```python
s = Savings(234, 200)
```

```python
s.apr, s.account_type, s.calc_interest()
```

Notice how the `calc_interest` method defined in the `Account` class used the correct instance value for `account_type` as well as the class level variable `apr`.


Now let's look at the class variable a bit closer.

You'll notice that I referenced it by using `self.apr`.


Now as we know, we can also access class attributes directly from the class, not just from the instance:

```python
Account.apr, Savings.apr
```

But we have to be careful here when we use it in the `calc_interest` method:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {Account.apr}'
        
        
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account' 
```

```python
s = Savings(123, 100)
s.calc_interest()
```

Notice how even though this was a `Savings` account, we still used the `apr` defined in the `Account` class. That's because we explicitly used `Account.apr`.

This is why I chose to use `self.apr` in the first example. We can also use the `__class__` method to recover the actual class of the specific instance:

```python
a = Account(123, 100)
s = Savings(234, 200)
```

```python
a.__class__
```

```python
s.__class__
```

Fairly often we need to get a handle on the class of the instance, but we cannot assume it is necessarily the class our code is *defined* in, as was the case in his example. Even though `calc_interest` is defined in the `Account` class, it is actually bound to an instance of the `Savings` class when we call `s.calc_interest()`.

So we can also do it this way:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {self.__class__.apr}'
        
        
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account' 
```

```python
a = Account(123, 100)
s = Savings(234, 200)
```

```python
a.calc_interest(), s.calc_interest()
```

So why use this `self.__class__.apr` technique instead of using `self.apr`? Basically if we want to protect from someone shadowing the `apr` class attribute with an instance attribute:


Remember that instances can define instance attributes that can shadow class attributes:

```python
s1 = Savings(123, 100)
```

```python
s1.__dict__
```

```python
s1.apr
```

```python
s2 = Savings(234, 200)
s2.apr = 10
```

```python
s2.__dict__
```

```python
s2.apr
```

So now watch what happens when we use the `self.apr`:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {self.apr}'
        
        
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account' 
```

```python
s1 = Savings(123, 100)
s2 = Savings(234, 200)
s1.apr = 10
```

```python
s1.calc_interest(), s2.calc_interest()
```

As you can see `self.apr` used the "overriding" instance attribute for the class attribute `apr`.


If instead we use `self.__class__.apr`:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {self.__class__.apr}'
        
        
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account' 
```

```python
s1 = Savings(123, 100)
s2 = Savings(234, 200)
s1.apr = 10
```

```python
s1.calc_interest(), s2.calc_interest()
```

As you can see we forced our code to use the **class** attribute. Depending on what you are designing, you may want to choose one or the other.


More often, we use `type(a)` instead of `a.__class__`, like so:

```python
class Account:
    apr = 3.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number
        self.balance = balance
        self.account_type = 'Generic Account'
        
    def calc_interest(self):
        return f'Calc interest on {self.account_type} with APR = {type(self).apr}'
        
        
class Savings(Account):
    apr = 5.0
    
    def __init__(self, account_number, balance):
        self.account_number = account_number  # We'll revisit this later - this is clumsy
        self.balance = balance
        self.account_type = 'Savings Account' 
```

And it works exactly the same way:

```python
a = Account(100, 100)
s1 = Savings(101, 100)
s2 = Savings(102, 100)
```

```python
s2.apr = 10
```

```python
a.calc_interest()
```

```python
s1.calc_interest()
```

```python
s2.calc_interest()
```

```python

```
