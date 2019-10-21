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

### Properties


To be clear, here we are examining **instance** properties. That is, we define the property in the class we are defining, but the property itself is going to be **instance** specific, i.e. different instances will support different values for the property. Just like instance attributes. The main difference is that we will use accessor method to get, set (and optionally) delete the associated instance value.


As I mentioned in the lecture, because properties use the same dotted notation (and the same `getattr`, `setattr` and `delattr` functions), we do not need to **start** with properties. Often a bare attribute works just fine, and if, later, we decide we need to manage getting/setting/deleting the attribute value, we can switch over to properties without breaking our class interface. This is unlike languages like Java - and hence why in those languages it is recommended to **always** use getter and setter functions. *Not so* in Python!


A **property** in Python is essentially a class instance - we'll come back to what that class looks like when we study descriptors. For now, we are going to use the `property` function in Python which is a convenience callable essentially.


Let's start with a simple example and a bare attribute:

```python
class Person:
    def __init__(self, name):
        self.name = name
```

So this class has a single instance **attribute**, `name`.

```python
p = Person('Alex')
```

And we can access and modify that attribute using either dotted notation or the `getattr` and `setattr` methods:

```python
p.name
```

```python
getattr(p, 'name')
```

p.name = 'John'

```python
p.name
```

```python
setattr(p, 'name', 'Eric')
```

```python
p.name
```

Now suppose we wan't to disallow setting an empty string or `None` for the name. Also, we'll require the name to be a string.

To do that we are going to create an instance method that will handle the logic and setting of the value. We also create an instance method to retrieve the attribute value.

We'll use `_name` as the instance variable to store the name.

```python
class Person:
    def __init__(self, name):
        self.set_name(name)
        
    def get_name(self):
        return self._name
    
    def set_name(self, value):
        if isinstance(value, str) and len(value.strip()) > 0:
            # this is valid
            self._name = value.strip()
        else:
            raise ValueError('name must be a non-empty string')
```

```python
p = Person('Alex')
```

```python
try:
    p.set_name(100)
except ValueError as ex:
    print(ex)
```

```python
p.set_name('Eric')
```

```python
p.get_name()
```

Of course, our users can still manipulate the atribute directly if they want by using the "private" attribute `_name`. You can't stop anyone from doing this in Python - they should know better than to do that, but we're all good programmers, and know what and what not to do!


The way we set up our initializer, the validation will work too:

```python
try:
    p = Person('')
except ValueError as ex:
    print(ex)
```

So this works, but it's a bit of pain to use the method names. So let's turn this into a property instead. We start with the class we just had and tweak it a bit:

```python
class Person:
    def __init__(self, name):
        self.name = name  # note how we are actually setting the value for name using the property!
        
    def get_name(self):
        return self._name
    
    def set_name(self, value):
        if isinstance(value, str) and len(value.strip()) > 0:
            # this is valid
            self._name = value.strip()
        else:
            raise ValueError('name must be a non-empty string')
            
    name = property(fget=get_name, fset=set_name)
```

```python
p = Person('Alex')
```

```python
p.name
```

```python
p.name = 'Eric'
```

```python
try:
    p.name = None
except ValueError as ex:
    print(ex)
```

So now we have the benefit of using accessor methods, without having to call the methods explicitly.

In fact, even `getattr` and `setattr` will work the same way:

```python
setattr(p, 'name', 'John')  # or p.name = 'John'
```

```python
getattr(p, 'name')  # or simply p.name
```

Now let's examine the instance dictionary:

```python
p.__dict__
```

You'll see we can find the underlying "private" attribute we are using to store the name. But the property itself (`name`) is not in the dictionary.

The property was defined in the class:

```python
Person.__dict__
```

And you can see it's type is `property`.

So when we type `p.name` or `p.name = value`, Python recognizes that `'name` is a `property` and therefore uses the accessor methods. (How it does we'll see later when we study descriptors).

What's interesting is that even if we muck around with the instance dictionary, Python does not get confused - (and as usual in Python, just because you **can** do something does not mean you **should**!)

```python
p = Person('Alex')
```

```python
p.name
```

```python
p.__dict__
```

```python
p.__dict__['name'] = 'John'
```

```python
p.__dict__
```

As you can see, we now have `name` in our instance dictionary.

Let's retrieve the `name` via dotted notation:

```python
p.name
```

That's obviously still using the getter method.

And setting the name:

```python
p.name = 'Raymond'
```

```python
p.__dict__
```

As you can see, it used the setter method.

And the same thing happens with `setattr` and `getattr`:

```python
getattr(p, 'name')
```

```python
setattr(p, 'name', 'Python')
```

```python
p.__dict__
```

As you can see the `setattr` method used the property setter.


For completeness, let's see how the deleter method works:

```python
class Person:
    def __init__(self, name):
        self._name = name
        
    def get_name(self):
        print('getting name')
        return self._name
    
    def set_name(self, value):
        print('setting name')
        self._name = value
        
    def del_name(self):
        print('deleting name')
        del self._name  # or whatever "cleanup" we want to do
        
    name = property(get_name, set_name, del_name)
```

```python
p = Person('Alex')
```

```python
p.__dict__
```

```python
p.name
```

```python
p.name = 'Eric'
```

```python
p.__dict__
```

```python
del p.name
```

```python
p.__dict__
```

Now, the property still exists (since it is defined in the class) - all we did was remove the underlying value for the property (the `_name` instance attribute):

```python
try:
    p.name
except AttributeError as ex:
    print(ex)
```

As you can see the issue is that the getter function is trying to find `_name` in the attribute, which no longer exists. So the getter and setter still exist (i.e. the property still exists), so we can still assign to it:

```python
p.name = 'Alex'
```

```python
p.name
```

The last param in `property` is just a docstring. So we could do this:

```python
class Person:
    """This is a Person object"""
    def __init__(self, name):
        self._name = name
        
    def get_name(self):
        return self._name
    
    def set_name(self, value):
        self._name = value
        
    name = property(get_name, set_name, doc="The person's name.")
```

```python
p = Person('Alex')
```

```python
help(Person.name)
```

```python
help(Person)
```
