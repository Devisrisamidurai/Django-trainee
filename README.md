# Django Signals and Custom Classes in Python

## Django Signals Questions

### Question 1: Are Django signals executed synchronously or asynchronously?

By default, Django signals are executed **synchronously**. This means that when a signal is triggered, it runs immediately within the same execution flow.

#### Code to Prove:

```python
import time
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def my_signal_handler(sender, instance, **kwargs):
    print("Signal handler started...")
    time.sleep(3)  # Simulating a delay
    print("Signal handler finished!")

# Trigger the signal
user = User.objects.create(username="test_user")

print("User creation completed!")
```

#### Expected Output:
```
Signal handler started...
(Script pauses for 3 seconds)
Signal handler finished!
User creation completed!
```
Since the script waits for the signal to complete, this proves that Django signals are **synchronous by default**.

---

### Question 2: Do Django signals run in the same thread as the caller?

Yes, Django signals run in the same thread as the caller unless explicitly configured to run in a different thread.

#### Code to Prove:

```python
import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def my_signal_handler(sender, instance, **kwargs):
    print(f"Signal running in thread: {threading.current_thread().name}")

# Print the main thread name
print(f"Main thread: {threading.current_thread().name}")

# Trigger the signal
user = User.objects.create(username="test_user")
```

#### Expected Output:
```
Main thread: MainThread
Signal running in thread: MainThread
```
Since both the main execution and the signal handler run in `MainThread`, this confirms that Django signals run in the **same thread as the caller**.

---

### Question 3: Do Django signals run in the same database transaction as the caller?

By default, Django signals **run in the same database transaction** as the caller unless explicitly modified using `transaction.on_commit`.

#### Code to Prove:

```python
from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

class TestModel(models.Model):
    name = models.CharField(max_length=100)

@receiver(post_save, sender=TestModel)
def my_signal_handler(sender, instance, **kwargs):
    print("Signal received. Creating another object...")
    TestModel.objects.create(name="Created inside signal")  # This should be rolled back

try:
    with transaction.atomic():
        print("Creating first object...")
        obj = TestModel.objects.create(name="Initial Object")
        raise ValueError("Simulating an error after signal execution")
except ValueError as e:
    print("Transaction rolled back!")

# Check if the second object (inside the signal) still exists
print("Objects in DB:", TestModel.objects.all().values_list("name", flat=True))
```

#### Expected Output:
```
Creating first object...
Signal received. Creating another object...
Transaction rolled back!
Objects in DB: []  # Everything is rolled back
```
Since the object created inside the signal is **rolled back when the error occurs**, it confirms that signals **run in the same database transaction by default**.

---

## Custom Classes in Python (Rectangle Class)

### Task:
Create a `Rectangle` class with the following requirements:
- Requires `length: int` and `width: int` to be initialized.
- Allows iteration over an instance of the `Rectangle` class.
- When iterated, first returns `{'length': <VALUE>}` followed by `{'width': <VALUE>}`.

#### Implementation:

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {"length": self.length}
        yield {"width": self.width}

# Example usage:
rect = Rectangle(10, 5)

for item in rect:
    print(item)
```

#### Expected Output:
```
{'length': 10}
{'width': 5}
```

This class:
1. **Initializes with `length` and `width` as integers.**
2. **Allows iteration** using the `__iter__` method.
3. **Yields** the length first (`{"length": <value>}`), followed by the width (`{"width": <value>}`).

---

This concludes the explanations and implementations for Django Signals and Custom Classes in Python. 



