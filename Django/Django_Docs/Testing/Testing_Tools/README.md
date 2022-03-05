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

### Making requests

Use the `django.test.Client` class to make requests.

##### `class Client(enforce_csrf_checks=False, json_encoder=DjangoJSONEncoder, **defaults)`

It requires no arguments at time of construction. However, you can use keyword arguments to specify some default headers. For example, this will send a `User-Agent` HTTP header in each request:
```
>>> c = Client(HTTP_USER_AGENT='Mozilla/5.0')
```
The values from the `extra` keyword arguments pased to [`get()`](), [`post()`](), etc., have precedence over the defaults passsed to the class constructor. <!-- below -->

The `enforce_csrf_checks` argument can be used to test CSRF protection (see above).

The `json_encoder` argument allows setting a custom JSON encoder for the JSON serialization that's described in `post()`.

The `raise_request_exception` argument allows controlling whether or not exceptions raised during the request should also be raised in the test. Defaults to `True`.

Once you have a `Client` instance, you can call any of the following methods:

* ##### `get(path, data=None, follow=False, secure=False, **extra)`

Makes a GET request on the provided `path` and returns a `Response` object, which is documented below.

The key-value pairs in the `data` dictionary are used to create a GET data payload. For example:
```
>>> c = Client()
>>> c.get('/customers/details/', {'name': 'fred', 'age': 7})
```
...will result in the evaluation of a GET request equivalent to:
```
/customers/details/?name=fred&age=7
```
The `extra` keyword arguments parameter can be used to specify headers to be sent in the request. For example:
```
>>> c = Client()
>>> c.get('/customers/details/', {'name': 'fred', 'age': 7},
...       HTTP_ACCEPT='application/json')
```
...will send the HTTP header `HTTP_ACCEPT` to the details view, which is a good way to test code paths that use the [`django.http.HttpRequest.accepts()`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.accepts) method.

<hr>

**CGI specification**

The headers sent via `**extra` should follow [CGI](https://www.w3.org/CGI/) specification. For example, emulating a different "Host" header as sent in the HTTP request from the browser to the server should be passed as `HTTP_HOST`.

<hr>

If you already have the GET arguments in URL-encoded form, you can use that encoding instead of using the data argument. For example, the previous GET request could also be posed as:
```
>>> c = Client()
>>> c.get('/customers/details/?name=fred&age=7')
```
If you provide a URL with both an encoded GET data and a data argument, the data argument will take precedence.

If you set `follow` to `True`, the client will follow any redirects and a `redirect_chain` attribute will be set in the response object containing tuples of the intermediate URLs and status codes.

If you had a URL `/redirect_me/` that redirects to `/next/`, that redirected to `/final/`, this is what you'd see:
```
>>> response = c.get('/redirect_me/', follow=True)
>>> response.redirect_chain
[('http://testserver/next/', 302), ('http://testserver/final/', 302)]
```
If you set `secure` to `True`, the client will emulate an HTTPS request.

* ##### `post(path, data=None, content_type=MULTIPART_CONTENT, follow=False, secure=False, **extra)`

Makes a POST request on the provided `path` and returns a `Response` object, which is documented below.

The key-value pairs in the `data` dictionary are used to submit POST data. For example:
```
>>> c = Client()
>>> c.post('/login/', {'name': 'fred', 'password': 'secret'})
```
...will result in the evaluation of a POST request to this URL:
```
/login/
```
...with this POST data:
```
name=fred&password=secret
```
If you provide `content_type` as *application/json*, the `data` is serialized using [`json.dumps()`](https://docs.python.org/3/library/json.html#json.dumps) if it's a dict, list, or tuple. Serialization is performed with [`DjangoJSONEncoder`](https://docs.djangoproject.com/en/4.0/topics/serialization/#django.core.serializers.json.DjangoJSONEncoder) by default, and can be overridden by providing a `json_encoder` argument to [`Client`](). <!-- above --> This serialization also happens for [`put()`](), [`patch()`](), and [`delete()`]() requests. <!-- below -->

If you provide any other `content_type` (e.g. *text/html* for an XML payload), the contents of `data` are sent as-is in the POST request, using `content_type` in the HTTP `Content-Type` header.

If you don't provide a value for `content_type`, the values in `data` will be transmitted with a content type of *multipart/form-data*. In this case, the key-value pairs in `data` will be encoded as a multipart message and used to create the POST data payload.

To submit multiple values for a given key -- for example, to specify the selections for a `<select multiple>` -- provide the values as a list or tuple for the required key. For example, this value of `data` would submit three selected values for the field named `choices`:
```
{'choices': ('a', 'b', 'd')}
```
Submitting files is a special case. To POST a file, you need only provide the file name as a key, and a file handle to the file you wish to upload as a value. For example:
```
>>> c = Client()
>>> with open('wishlist.doc', 'rb') as fp:
...     c.post('/customers/wishes/', {'name': 'fred', 'attachment': fp})
```
(The name `attachment` here is not relevant; use whatever name your file-processing code expects.)

You may also provide any file-like object (e.g. [`StringIO`](https://docs.python.org/3/library/io.html#io.StringIO) or [`BytesIO`](https://docs.python.org/3/library/io.html#io.BytesIO)) as a file handle. If you're uploading to an [`ImageField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ImageField), the object needs a `name` attribute that passes the [`validate_image_file_extension`](https://docs.djangoproject.com/en/4.0/ref/validators/#django.core.validators.validate_image_file_extension) validator. For example:
```
>>> from io import BytesIO
>>> img = BytesIO(b'mybinarydata')
>>> img.name = 'myimage.jpg'
```
Note that if you wish to use the same file handle for multiple `post()` calls, then you will need to manually reset the file pointer between posts. The easiest way to do this is to manually close the file after it has been provided to `post()`, as demonstrated above.

You should also ensure that the file is opened in a way that allows the data to be read. If your file contains binary data such as an image, this means you will need to open the file in `rb` (read binary) mode.

The `extra` argument acts the same as for [`client.get()`](). <!-- "get()" above -->

If the URL you request with a POST contains encoded parameters, these parameters will be made available in the `request.GET` data. For example, if you were to make the request:
```
>>> c.post('/login/?visitor=true', {'name': 'fred', 'password': 'secret'})
```
...the view handling this request could interrogate `request.POST` to retrieve the username and password, and could interrogate `request.GET` to determine if the user was a visitor.

If you set `follow` to `True`, the client will follow any redirects and a `redirect_chain` attribute will be set in the response object containing tuples of the intermediate URLs and status codes.

If you set `secure` to `True`, the client will emulate an HTTPS request.