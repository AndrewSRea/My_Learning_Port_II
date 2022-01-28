# Database transactions

Django give you a few ways to control how database transactions are managed.

## Managing database transactions

### Django's default transaction behavior

Django's default behavior is to run in autocommit mode. Each query is immediately committed to the database, unless a transaction is active. [See below for details](). <!-- "Why Django uses autocommit" -->

Django uses transactions or savepoints automatically to guarantee the integrity of ORM operations that require multiple queries, especially [`delete()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#deleting-objects) and [`update()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#updating-multiple-objects-at-once) queries.

Django's [`TestCase`](https://docs.djangoproject.com/en/4.0/topics/testing/tools/#django.test.TestCase) class also wraps each test in a transaction for performance reasons.

### Tying transactions to HTTP requests

A common way to handle transactions on the web is to wrap each request in a transaction. Set [`ATOMIC_REQUESTS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASE-ATOMIC_REQUESTS) to `True` in the configuration of each database for which you want to enable this behavior.

It works like this. Before calling a view function, Django starts a transaction. If the response is produced without problems, Django commits the transaction. If the view produces an exception, Django rolls back the transaction.

You may perform subtransactions using savepoints in your view code, typically with the [`atomic()`]() <!-- below --> context manager. However, at the end of the view, either all or none of the changes will be committed.

<hr>

:warning: **Warning**

While the simplicity of this transaction model is appealing, it also makes it inefficient when traffic increases. Opening a transaction for every view has some overhead. The impact on performance depends on the query pattern of your application and on how well your database handles locking.

<hr>

**Per-request transactions and streaming responses**

When a view returns a [`StreamingHttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.StreamingHttpResponse), reading the contents of the response will often execute code to generate the content. Since the view has already returned, such code runs outside of the transaction.

Generally speaking, it isn't advisable to write to the database while generating a streaming response, since there's no sensible way to handle errors after starting to send the response.

<hr>

In practice, this feature wraps every view function in the [`atomic()`]() <!-- below --> decorator described below.

Note that only the execution of your view is enclosed in the transactions. Middleware runs outside of the transaction, and so does the rendering of template responses.

When [`ATOMIC_REQUESTS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASE-ATOMIC_REQUESTS) is enabled, it's still possible to prevent views from running in a transaction.

##### `non_atomic_requests(using=None)`

This decorator will negate the effect of `ATOMIC_REQUESTS` for a given view:
```
from django.db import transaction

@transaction.non_atomic_requests
def my_view(request):
    do_stuff()

@transaction.non_atomic_requests(using='other')
def my_other_view(request):
    do_stuff_on_the_other_database()
```
It only works if it's applied to the view itself.

### Controlling transactions explicitly

Django provides a single API to control database transactions.

##### `atomic(using=None, savepoint=True, durable=False)`

Atomicity is the defining property of database transactions. `atomic` allows us to create a block of code within which the atomicity on the database is guaranteed. If the block of code is successfully completed, the changes are committed to the database. If there is an exception, the changes are rolled back.

`atomic` blocks can be nested. In this case, when an inner block completes successfully, its effects can still be rolled back if an exception is raised in the outer block at a later point.

It is sometimes useful to ensure an `atomic` block is always the outermost `atomic` block, ensuring that any database changes are committed when the block is exited without errors. This is known as durability and can be achieved by setting `durable=True`. If the `atomic` block is nested within another, it raises a `RuntimeError`.

`atomic` is usable both as a [decorator](https://docs.python.org/3/glossary.html#term-decorator):
```
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()
```
...and as a [context manager](https://docs.python.org/3/glossary.html#term-context-manager):
```
from django.db import transaction

def viewfunc(request):
    # This code executes in autocommit mode (Django's default).
    do_stuff()

    with transaction.atomic()
        # This code executes inside a transaction.
        do_more_stuff()
```
