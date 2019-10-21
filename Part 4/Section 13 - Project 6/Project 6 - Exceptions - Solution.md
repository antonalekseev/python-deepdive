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

### Project 6 - Solution


```
1. Supplier exceptions
    a. Not manufactured anymore
    b. Production delayed
    c. Shipping delayed
    
2. Checkout exceptions
    a. Inventory type exceptions
        - out of stock
    b. Pricing exceptions
        - invalid coupon code
        - cannot stack coupons
```

```python
from datetime import datetime 

class WidgetException(Exception):
    message = 'Generic Widget exception.'
    
    def __init__(self, *args, customer_message=None):
        super().__init__(args)
        if args:
            self.message = args[0]
        self.customer_message = customer_message if customer_message is not None else self.message
        
    def log_exception(self):
        exception = {
            "type": type(self).__name__,
            "message": self.message,
            "args": self.args[1:]
        }
        print(f'EXCEPTION: {datetime.utcnow().isoformat()}: {exception}')
```

```python
ex1 = WidgetException('some custom message', 10, 100)
ex2 = WidgetException(customer_message='A custom user message.')
```

```python
ex1.log_exception()
```

```python
ex2.log_exception()
```

Now we can create our hierarchy, and override the appropriate values for `message` to make it more specific:

```python
class SupplierException(WidgetException):
    message = 'Supplier exception.'

class NotManufacturedException(SupplierException):
    message = 'Widget is no longer manufactured by supplier.'
    
class ProductionDelayedException(SupplierException):
    message = 'Widget production has been delayed by supplier.'
    
class ShippingDelayedException(SupplierException):
    message = 'Widget shipping has been delayed by supplier.'
    
class CheckoutException(WidgetException):
    message = 'Checkout exception.'
    
class InventoryException(CheckoutException):
    message = 'Checkout inventory exception.'
    
class OutOfStockException(InventoryException):
    message = 'Inventory out of stock'
    
class PricingException(CheckoutException):
    message = 'Checkout pricing exception.'
    
class InvalidCouponCodeException(PricingException):
    message = 'Invalid checkout coupon code.'
    
class CannotStackCouponException(PricingException):
    message = 'Cannot stack checkout coupon codes.'
```

And now we can use any of these exceptions in our code, and use the defined "logger" we implemented:

```python
try:
    raise CannotStackCouponException()
except WidgetException as ex:
    ex.log_exception()
    raise
```

Next let's add the http status codes we want to assign to each exception type.

```python
from http import HTTPStatus
```

```python
class WidgetException(Exception):
    message = 'Generic Widget exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
    def __init__(self, *args, customer_message=None):
        super().__init__(*args)
        if args:
            self.message = args[0]
        self.customer_message = customer_message if customer_message is not None else self.message
        
    def log_exception(self):
        exception = {
            "type": type(self).__name__,
            "message": self.message,
            "args": self.args[1:]
        }
        print(f'EXCEPTION: {datetime.utcnow().isoformat()}: {exception}')
```

Before we redefine our child classes, let's also implement the `to_json` function that we can use to send back to our users:

```python
import json
```

```python
class WidgetException(Exception):
    message = 'Generic Widget exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
    def __init__(self, *args, customer_message=None):
        super().__init__(*args)
        if args:
            self.message = args[0]
        self.customer_message = customer_message if customer_message is not None else self.message
        
    def log_exception(self):
        exception = {
            "type": type(self).__name__,
            "message": self.message,
            "args": self.args[1:]
        }
        print(f'EXCEPTION: {datetime.utcnow().isoformat()}: {exception}')
        
    def to_json(self):
        response = {
            'code': self.http_status.value,
            'message': '{}: {}'.format(self.http_status.phrase, self.customer_message),
            'category': type(self).__name__,
            'time_utc': datetime.utcnow().isoformat()            
        }
        return json.dumps(response)
```

```python
e = WidgetException('same custom message for log and user')
```

```python
e.log_exception()
```

```python
json.loads(e.to_json())
```

```python
e = WidgetException('custom internal message', customer_message='custom user message')
```

```python
e.log_exception()
```

```python
e.to_json()
```

Now for the bonus exercise - I asked you to try and log the stack trace as well.


To do that we could cannot simply use the `str` or `repr` of the  `__traceback__` property of the exception:

```python
try:
    raise WidgetException('custom error message')
except WidgetException as ex:
    print(repr(ex.__traceback__))
```

Instead we can use the `traceback` module:

```python
import traceback
```

```python
try:
    raise ValueError
except ValueError:
    try:
        raise WidgetException('custom error message')
    except WidgetException as ex:
        print(list(traceback.TracebackException.from_exception(ex).format()))
```

So we can use that to implement logging the traceback. What would be nice too would be to expose the formatted traceback in our exception class while we're at it.

Since tracebacks can be huge, we're not going to materialize the traceback generator in that property (we'll still have to when we log the exception):

```python
class WidgetException(Exception):
    message = 'Generic Widget exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
    def __init__(self, *args, customer_message=None):
        super().__init__(*args)
        if args:
            self.message = args[0]
        self.customer_message = customer_message if customer_message is not None else self.message
        
    @property
    def traceback(self):
        return traceback.TracebackException.from_exception(self).format()
    
    def log_exception(self):
        exception = {
            "type": type(self).__name__,
            "message": self.message,
            "args": self.args[1:],
            "traceback": list(self.traceback)
        }
        print(f'EXCEPTION: {datetime.utcnow().isoformat()}: {exception}')
        
    def to_json(self):
        response = {
            'code': self.http_status.value,
            'message': '{}: {}'.format(self.http_status.phrase, self.customer_message),
            'category': type(self).__name__,
            'time_utc': datetime.utcnow().isoformat()            
        }
        return json.dumps(response)
```

```python
try:
    raise WidgetException('custom internal message', customer_message='custom user message')
except WidgetException as ex:
    ex.log_exception()
    print('------------')
    print(ex.to_json())
```

What's nice now, is that we could just print the traceback wihout logging the exception:

```python
try:
    a = 1 / 0
except ZeroDivisionError:
    try:
        raise WidgetException()
    except WidgetException as ex:
        print(''.join(ex.traceback))
```

Now we can define our exception sub types, including the http status for each:

```python
class SupplierException(WidgetException):
    message = 'Supplier exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR

class NotManufacturedException(SupplierException):
    message = 'Widget is no longer manufactured by supplier.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class ProductionDelayedException(SupplierException):
    message = 'Widget production has been delayed by supplier.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class ShippingDelayedException(SupplierException):
    message = 'Widget shipping has been delayed by supplier.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class CheckoutException(WidgetException):
    message = 'Checkout exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class InventoryException(CheckoutException):
    message = 'Checkout inventory exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class OutOfStockException(InventoryException):
    message = 'Inventory out of stock'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class PricingException(CheckoutException):
    message = 'Checkout pricing exception.'
    http_status = HTTPStatus.INTERNAL_SERVER_ERROR
    
class InvalidCouponCodeException(PricingException):
    message = 'Invalid checkout coupon code.'
    http_status = HTTPStatus.BAD_REQUEST
    
class CannotStackCouponException(PricingException):
    message = 'Cannot stack checkout coupon codes.'
    http_status = HTTPStatus.BAD_REQUEST
```

```python
e = InvalidCouponCodeException('User tried to use an old coupon', customer_message='Sorry. This coupon has expired.')
```

```python
e.log_exception()
```

```python
e.to_json()
```

As you can see our traceback was empty above (the exception is present, but there is no call stack) - because we did not actually raise the exception!

```python
try:
    raise ValueError
except ValueError:
    try:
        raise InvalidCouponCodeException(
            'User tried to use an old coupon', customer_message='Sorry. This coupon has expired.'
        )
    except InvalidCouponCodeException as ex:
        ex.log_exception()
        print('------------')
        print(ex.to_json())
        print('------------')
        print(''.join(ex.traceback))
```
