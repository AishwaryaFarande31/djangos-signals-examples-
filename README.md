# djangos-signals-examples-

# Django Signals Examples

This README provides explanations and code examples demonstrating various behaviors of Django signals, specifically addressing:

1. Whether Django signals are executed synchronously or asynchronously by default.
2. Whether Django signals run in the same thread as the caller.
3. Whether Django signals run within the same database transaction as the caller.

## Overview

### What are Django Signals?
Django signals allow decoupled applications to get notified when certain actions occur in a Django app. They enable functions (signal handlers) to be triggered by specific events, like saving a model instance.

### Prerequisites
To follow along with these examples, you’ll need:
- Python with Django installed (`pip install django`)
- Basic knowledge of Django models, signals, and transactions.

## Examples and Explanations

### Example 1: Synchronous or Asynchronous Execution

#### Explanation
Django signals are executed **synchronously** by default. This means that when a signal is triggered, the corresponding signal handler runs immediately, and the main function will wait until the signal handler has completed.

In this example, we introduce a 2-second delay in the signal handler using `time.sleep()`. When a user is created in the main function, the total time taken for the operation includes this delay, confirming that the signal handler runs synchronously.

#### Code Snippet

```python
import time
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

# Signal handler with delay
@receiver(post_save, sender=User)
def user_saved_handler(sender, instance, **kwargs):
    time.sleep(2)  # Simulate a delay in the signal handler
    print("Signal handler executed.")

def main():
    print("Main function started.")
    user = User.objects.create(username="test_user")
    print("User created, main function ended.")

start_time = time.time()
main()
end_time = time.time()

print("Total time:", end_time - start_time)
```

#### Summary
The total runtime will reflect the delay introduced by `user_saved_handler`, demonstrating that the signal handler runs immediately and is not asynchronous by default.

---

### Example 2: Threading Behavior

#### Explanation
Django signals run in the **same thread** as the caller function by default. This means that when a signal is triggered, it executes within the same thread that called it, rather than starting a new thread.

In this example, both the main function and the signal handler print the thread name. Observing the same thread name for both confirms that the signal handler runs in the same thread as the main function, demonstrating thread consistency.

#### Code Snippet

```python
import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def user_saved_handler(sender, instance, **kwargs):
    print(f"Signal handler running in thread: {threading.current_thread().name}")

def main():
    print(f"Main function running in thread: {threading.current_thread().name}")
    user = User.objects.create(username="test_user")

main()
```

#### Summary
The output will show the same thread name for both the main function and the signal handler, confirming that the signal handler runs in the same thread as the caller.

---

### Example 3: Transaction Behavior

#### Explanation
Django signals operate within the **same database transaction** as the main function by default. If an error occurs in the signal handler, it will cause the entire transaction to roll back, undoing any database changes made by the main function.

In this example, a signal handler creates a profile for a user and then raises an intentional error (e.g., `ValueError`). This will trigger a rollback of the transaction initiated in the main function, and the user creation will also be undone, demonstrating that both the main function and signal handler share the same transaction.

#### Code Snippet

```python
from django.db import transaction, models
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)

@receiver(post_save, sender=User)
def create_profile(sender, instance, **kwargs):
    Profile.objects.create(user=instance)
    raise ValueError("Intentional error to test transaction rollback")

def main():
    try:
        with transaction.atomic():
            user = User.objects.create(username="test_user")
    except Exception as e:
        print(f"Transaction rolled back due to error: {e}")

main()
```

#### Summary
The `ValueError` in `create_profile` will cause the transaction to roll back, so the user creation will also be undone. This demonstrates that both the main function and signal handler are part of the same transaction.

---

## Conclusion

These examples illustrate that Django signals are executed synchronously, run within the same thread as the caller, and share the same transaction context by default. This behavior should be considered when using signals for critical tasks in Django applications.
```


