# Testing tools

Django provides a small set of tools that come in handy when writing tests.

## The test client

The test client is a Python class that acts as a dummy web browser, allowing you to test your views and interact with your Django-powered application programmatically.

Some of the things you can do with the test client are:

* Simluate GET and POST requests on a URL and observe the response -- everything from low-level HTTP (result headers and status codes) to page content.
* See the chain of redirects (if any) and check the URL and status code at each step.
* Test that a given request is rendered by a given Django template, with a template context that contains certain values.

Note that the test client is not intended to be a replacement for [Selenium](https://www.selenium.dev/) or other "in-browser" frameworks. Django's test client has a different focus. In short:

* Use Django's test client to establish that the correct template is being rendered and that the template is passed the correct context data.
* Use in-browser frameworks like [Selenium](https://www.selenium.dev/) to test *rendered* HTML and the *behavior* of web pages, namely JavaScript functionality. Django also provides special support for those frameworks; see the section on [`LiveServerTestCase`]() for more details. <!-- below -->

A comprehensive test suite should use a combination of both test types.

### Overview and a quick example

To use the test client, instantiate `django.test.Client` and retrieve web pages:
```
>>> from django.test import Client
>>> c = Client()
>>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
>>> response.status_code
200
>>> response = c.get('/customer/details/')
>>> response.content
b'<!DOCTYPE html...'
```
As this example suggests, you can instantiate `Client` from within a session of the Python interactive interpreter.

Note a few important things about how the test client works:

* The test client does *not* require the web server to be running. IN fact, it will run just fine with no web server running at all! That's because it avoids the overhead of HTTP and deals directly with the Django framework. This helps make the unit tests run quickly.

* When retrieving pages, remember to specify the *path* of the URL, not the whole domain. For example, this is correct:

    ```
    >>> c.get('/login/')
    ```

    This is incorrect:

    ```
    >>> c.get('https://www.example.com/login/')
    ```

    The test client is not capable of retrieving web pages that are not powered by your Django project. If you need to retrieve other web pages, use a Python standard library module such as [`urllib`](https://docs.python.org/3/library/urllib.html#module-urllib).

* To resolve URLs, the test client uses whatever URLconf is pointed-to by your [`ROOT_URLCONF`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ROOT_URLCONF) setting.

* Although the above example would work in the Python interactive interpreter, some of the test client's functionality, notably the template-related functionality, is only available *while tests are running*.

    The reason for this is that Django's test runner performs a bit of black magic in order to determine which template was loaded by a given view. This black magic (essentially a patching of Django's template system in memory) only happens during test running.

* By default, the test client will disable any CSRF checks performed by your site.

    If, for some reason, you *want* the test client to perform CSRF checks, you can create an instance of the test client that enforces CSRF checks. To do this, pass in the `enforce_csrf_checks` argument when you construct your client:

    ```
    >>> from django.test import Client
    >>> csrf_client = Client(enforce_csrf_checks=True)
    ```