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

### Partial Functions

```python
from functools import partial
```

```python
def my_func(a, b, c):
    print(a, b, c)
```

```python
f = partial(my_func, 10)
```

```python
f(20, 30)
```

We could have done this using another function (or a lambda) as well:

```python
def partial_func(b, c):
    return my_func(10, b, c)
```

```python
partial_func(20, 30)
```

or, using a lambda:

```python
fn = lambda b, c: my_func(10, b, c)
```

```python
fn(20, 30)
```

Any of these ways is fine, but sometimes partial is just a cleaner more consise way to do it.

Also, it is quite flexible with parameters:

```python
def my_func(a, b, *args, k1, k2, **kwargs):
    print(a, b, args, k1, k2, kwargs)
```

```python
f = partial(my_func, 10, k1='a')
```

```python
f(20, 30, 40, k2='b', k3='c')
```

We can of course do the same thing using a regular function too:

```python
def f(b, *args, k2, **kwargs):
    return my_func(10, b, *args, k1='a', k2=k2, **kwargs)
```

```python
f(20, 30, 40, k2='b', k3='c')
```

As you can see in this case, using **partial** seems a lot simpler.


Also, you are not stuck having to specify the first argument in your partial:

```python
def power(base, exponent):
    return base ** exponent
```

```python
power(2, 3)
```

```python
square = partial(power, exponent=2)
```

```python
square(4)
```

```python
cube = partial(power, exponent=3)
```

```python
cube(2)
```

You can even call it this way:

```python
cube(base=3)
```

#### Caveat


We can certainly use variables instead of literals when creating partials, but we have to be careful.

```python
def my_func(a, b, c):
    print(a, b, c)
```

```python
a = 10
f = partial(my_func, a)
```

```python
f(20, 30)
```

Now let's change the value of the variable **a** and see what happens:

```python
a = 100
```

```python
f(20, 30)
```

As you can see, the value for **a** is fixed once the partial has been created.

In fact, the memory address of **a** is baked in to the partial, and **a** is immutable.


If we use a mutable object, things are different:

```python
a = [10, 20]
f = partial(my_func, a)
```

```python
f(100, 200)
```

```python
a.append(30)
```

```python
f(100, 200)
```

#### Use Cases


We tend to use partials in situation where we need to call a function that actually requires more parameters than we can supply.

Often this is because we are working with exiting libraries or code, and we have a special case.


For example, suppose we have points (represented as tuples), and we want to sort them based on the distance of the point from some other fixed point:

```python
origin = (0, 0)
```

```python
l = [(1,1), (0, 2), (-3, 2), (0,0), (10, 10)]
```

```python
dist2 = lambda x, y: (x[0]-y[0])**2 + (x[1]-y[1])**2
```

```python
dist2((0,0), (1,1))
```

```python
sorted(l, key = lambda x: dist2((0,0), x))
```

```python
sorted(l, key=partial(dist2, (0,0)))
```

Another use case is when using **callback** functions. Usually these are used when running asynchronous operations, and you provide a callable to another callable which will be called when the first callable completes its execution.

Very often, the asynchronous callable will specify the number of variables that the callback function must have - this may not be what we want, maybe we want to add some additional info.


We'll look at asynchronous processing later in this course.


Often we can also use partial functions to make our life a bit easier.

Consider a situation where we have some generic `email()` function that can be used to notify someone when various things happen in our application. But depending on what is happening we may want to notify different people. Let's see how we may do this:

```python
def sendmail(to, subject, body):
    # code to send email
    print('To:{0}, Subject:{1}, Body:{2}'.format(to, subject, body))
```

Now, we may haver different email adresses we want to send notifications to, maybe defined in a config file in our app. Here, I'll just use hardcoded variables:

```python
email_admin = 'palin@python.edu'
email_devteam = 'idle@python.edu;cleese@python.edu'
```

Now when we want to send emails we would have to write things like:

```python
sendmail(email_admin, 'My App Notification', 'the parrot is dead.')
sendmail(';'.join((email_admin, email_devteam)), 'My App Notification', 'the ministry is closed until further notice.')
```

We could simply our life a little using partials this way:

```python
send_admin = partial(sendmail, email_admin, 'For you eyes only')
send_dev = partial(sendmail, email_devteam, 'Dear IT:')
send_all = partial(sendmail, ';'.join((email_admin, email_devteam)), 'Loyal Subjects')
```

```python
send_admin('the parrot is dead.')
send_all('the ministry is closed until further notice.')
```

Finally, let's make this a little more complex, with a mixture of positional and keyword-only arguments:

```python
def sendmail(to, subject, body, *, cc=None, bcc=email_devteam):
    # code to send email
    print('To:{0}, Subject:{1}, Body:{2}, CC:{3}, BCC:{4}'.format(to, 
                                                                  subject, 
                                                                  body, 
                                                                  cc, 
                                                                  bcc))
```

```python
send_admin = partial(sendmail, email_admin, 'General Admin')
send_admin_secret = partial(sendmail, email_admin, 'For your eyes only', cc=None, bcc=None)
```

```python
send_admin('and now for something completely different')
```

```python
send_admin_secret('the parrot is dead!')
```

```python
send_admin_secret('the parrot is no more!', bcc=email_devteam)
```

```python
def pow(base, exponent):
    return base ** exponent
```

```python
cube = partial(pow, exponent=3)
```

```python
cube(2)
```

```python
cube(2, 4)
```

```python
cube(2, exponent=4)
```

```python

```
