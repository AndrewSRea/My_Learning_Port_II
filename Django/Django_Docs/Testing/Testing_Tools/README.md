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
* Use in-browser frameworks like [Selenium](https://www.selenium.dev/) to test *rendered* HTML and the *behavior* of web pages, namely JavaScript functionality. Django also provides special support for those frameworks; see the section on [`LiveServerTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-liveservertestcase) for more details.

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
The values from the `extra` keyword arguments pased to [`get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra), [`post()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#postpath-datanone-content_typemultipart_content-followfalse-securefalse-extra), etc., have precedence over the defaults passsed to the class constructor.

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
    If you provide `content_type` as *application/json*, the `data` is serialized using [`json.dumps()`](https://docs.python.org/3/library/json.html#json.dumps) if it's a dict, list, or tuple. Serialization is performed with [`DjangoJSONEncoder`](https://docs.djangoproject.com/en/4.0/topics/serialization/#django.core.serializers.json.DjangoJSONEncoder) by default, and can be overridden by providing a `json_encoder` argument to [`Client`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-clientenforce_csrf_checksfalse-json_encoderdjangojsonencoder-defaults). This serialization also happens for [`put()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#putpath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra), [`patch()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#patchpath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra), and [`delete()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#deletepath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra) requests.

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

    The `extra` argument acts the same as for [`client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

    If the URL you request with a POST contains encoded parameters, these parameters will be made available in the `request.GET` data. For example, if you were to make the request:
    ```
    >>> c.post('/login/?visitor=true', {'name': 'fred', 'password': 'secret'})
    ```
    ...the view handling this request could interrogate `request.POST` to retrieve the username and password, and could interrogate `request.GET` to determine if the user was a visitor.

    If you set `follow` to `True`, the client will follow any redirects and a `redirect_chain` attribute will be set in the response object containing tuples of the intermediate URLs and status codes.

    If you set `secure` to `True`, the client will emulate an HTTPS request.

* ##### `head(path, data=None, follow=False, secure=False, **extra)`

    Makes a HEAD request on the provided `path` and returns a `Response` object. This method works just like [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra), including the `follow`, `secure`, and `extra` arguments, except it does not return a message body.

* ##### `options(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes an OPTIONS request on the provided `path` and returns a `Response` object. Useful for testing RESTful interfaces.

    When `data` is provided, it is used as the request body, and a `Content-Type` header is set to `content_type`.

    The `follow`, `secure`, and `extra` arguments act the same as for [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

* ##### `put(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a PUT request on the provided `path` and returns a `Response` object. Useful for testing RESTful interfaces.

    When `data` is provided, it is used as the request body, and a `Content-Type` header is set to `content_type`.

    The `follow`, `secure`, and `extra` arguments act the same as for [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

* ##### `patch(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a PATCH request on the provided `path` and returns a `Response` object. Useful for testing RESTful interfaces.

    The `follow`, `secure`, and `extra` arguments act the same as for [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

* ##### `delete(path, data='', content_type='application/octet-stream', follow=False, secure=False, **extra)`

    Makes a DELETE request on the provided `path` and returns a `Response` object. Useful for testing RESTful interfaces.

    When `data` is provided, it is used as the request body, and a `Content-Type` header is set to `content_type`.

    The `follow`, `secure`, and `extra` arguments act the same as for [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

* ##### `trace(path, follow=False, secure=False, **extra)`

    Makes a TRACE request on the provided `path` and returns a `Response` object. Useful for simulating diagnostic probes.

    Unlike the other request methods, `data` is not provided as a keyword parameter in order to comply with [RFC 7231#section-4.3.8](https://datatracker.ietf.org/doc/html/rfc7231.html#section-4.3.8), which mandates that TRACE requests must not have a body.

    The `follow`, `secure`, and `extra` arguments act the same as for [`Client.get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra).

* ##### `login(**credentials)`

    If your site uses Django's [authentication system]() <!-- next module --> and you deal with logging in users, you can use the test client's `login()` method to simulate the effect of a user logging into the site.

    After you call this method, the test client will have all the cookies and session data required to pass any login-based tests that may form part of a view.

    The format of the `credentials` argument depends on which [authentication backend](https://docs.djangoproject.com/en/4.0/topics/auth/customizing/#authentication-backends) you're using (which is configured by your [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS) setting). If you're using the standard authentication backend provided by Django (`ModelBackend`), `credentials` should be the user's username and password, provided as keyword arguments:
    ```
    >>> c = Client()
    >>> c.login(username='fred', password='secret')

    # Now you can access a view that's only available to logged-in users.
    ```
    If you're using a different authentication backend, this method may require different credentials. It requires whichever credentials are required by your backend's `authenticate()` method.

    `login()` returns `True` if the credentials were accepted and login was successful.

    Finally, you'll need to remember to create user accounts before you can use this method. As we explained above, the test runner is executed using a test database, which contains no users by default. As a result, user accounts that are valid on your production site will not work under test conditions. You'll need to create users as part of the test suite -- either manually (using the Django model API) or with a test fixture. Remember that if you want your test user to have a password, you can't set the user's password by setting the password attribute directly -- you must use the [`set_password()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.set_password) function to store a correctly hashed password. Alternatively, you can use the [`create_user()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.UserManager.create_user) helper method to create a new user with a correctly hashed password.

* ##### `force_login(user, backend=None)`

    If your site uses Django's [authentication system](), <!-- next module --> you can use the `force_login()` method to simulate the effect of a user logging into the site. Use this method instead of [`login()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#logincredentials) when a test requires a user be logged in and the details of how a user logged in aren't important.

    Unlike `login()`, this method skips the authentication and verification steps: inactive users ([`is_active=False`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.is_active)) are permitted to login and the user's credentials don't need to be provided.

    The user will have its `backend` attribute set to the value of the `backend` argument (which should be a dotted Python path string), or to `settings.AUTHENTICATION_BACKENDS[0]` if a value isn't provided. The [`authenticate()`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.authenticate) function called by [`login()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#logincredentials) normally annotates the user like this.

    This method is faster than `login()` since the expensive password hashing algorithms are bypassed. Also, you can speed up `login()` by [using a weaker hasher while testing](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Writing_Running_Tests#speeding-up-the-tests).

* ##### `logout()`

    If your site uses Django's [authentication system](), <!-- next module --> the `logout()` method can be used to simulate the effect of a user logging out of your site.

    After you call this method, the test client will have all the cookies and session data cleared to defaults. Subsequent requests will appear to come from an [`AnonymousUser`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.AnonymousUser).

### Testing responses

The `get()` and `post()` methods both return a `Response` object. This `Response` object is *not* the same as the `HttpResponse` object returned by Django views; the test response object has some additional data useful for test code to verify.

Specifically, a `Response` object has the following attributes:

##### `class Response`

* ##### `client`

    The test client that was used to make the request that resulted in the response.

* ##### `content`

    The body of the response, as a bytestring. This is the final page content as rendered by the view, or any error message.

* ##### `context`

    The template `Context` instance that was used to render the template that produced the response content.

    If the rendered page used multiple templates, then `context` will be a list of `Context` objects, in the order in which they were rendered.

    Regardless of the number of templates used during rendering, you can retrieve context values using the `[]` operator. For example, the context variable `name` could be retrieved using:
    ```
    >>> response = client.get('/foo/')
    >>> response.context['name']
    'Arthur'
    ```

    <hr>

    **Not using Django templates?**

    This attribute is only populated when using the [DjangoTemplates](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Templates#class-djangotemplates) backend. If you're using another template engine, [`context_data`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.SimpleTemplateResponse.context_data) may be a suitable alternative on responses with that attribute.

    <hr>

* ##### `exc_info`

    A tuple of three values that provides information about the unhandled exception, if any, that occurred during the view.

    The values are (type, value, traceback), the same as returned by Python's [`sys.exc_info()`](https://docs.python.org/3/library/sys.html#sys.exc_info). Their meanings are:

    * *type*: The type of the exception.
    * *value*: The exception instance.
    * *traceback*: A traceback object which encapsulates the call stack at the point where the exception originally occurred.

    If no exception occurred, the `exc_info` will be `None`.

* ##### `json(**kwargs)`

    The body of the response, parsed as JSON. Extra keyword arguments are passed to [`json.loads()`](https://docs.python.org/3/library/json.html#json.loads). For example:
    ```
    >>> response = client.get('/foo/')
    >>> response.json()['name']
    'Arthur'
    ```
    If the `Content-Type` header is not `"application/json"`, then a [`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError) will be raised when trying to parse the response.

* ##### `request`

    The request data that stimulated the response.

* ##### `wsgi_request`

    The `WSGIRequest` instance generated by the test handler that generated the response.

* ##### `status_code`

    The HTTP status of the response, as an integer. For a full list of defined codes, see the [IANA status code registry](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml).

* ##### `templates`

    A list of `Template` instances used to render the final content, in the order they were rendered. For each template in the list, use `template.name` to get the template's file name, if the template was loaded from a file. (The name is a string such as `'admin/index.html'`.)

    <hr>

    **Not using Django templates?**

    This attribute is only populated when using the [DjangoTemplates](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Templates#class-djangotemplates) backend. If you're using another template engine, [`template_name`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.SimpleTemplateResponse.template_name) may be a suitable alternative if you only need the name of the template used for rendering.

    <hr>

* `resolver_match`

    An instance of [`ResolverMatch`](https://docs.djangoproject.com/en/4.0/ref/urlresolvers/#django.urls.ResolverMatch) for the reponse. You can use the [`func`](https://docs.djangoproject.com/en/4.0/ref/urlresolvers/#django.urls.ResolverMatch.func) attribute, for example, to verify the view that server the response:
    ```
    # my_view here is a function-based view
    self.assertEqual(response.resolver_match.func, my_view)

    # class-based views need to be compared by name, as the functions
    # generated by as_view() won't be equal
    self.assertEqual(response.resolve_match.func.__name__, MyView.as_view().__name__)
    ```
    If the given URL is not found, accessing this attribute will raise a [`Resolver404`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.urls.Resolver404) exception.

As with a normal response, you can also access the headers through [`HttpResponse.headers`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse.headers). For example, you could determine the content type of a response using `response.headers['Content-Type']`.

### Exceptions

If you point the test client at a view that raises an exception and `Client.raise_request_exception` is `True`, that exception will be visible in the test case. You can then use a standard `try ... except` block or [`assertRaises()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaises) to test for exceptions.

The only exceptions that are not visible to the test client are [`Http404`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Writing_Views#class-djangohttphttp404), [`PermissionDenied`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.PermissionDenied), [`SystemExit`](https://docs.python.org/3/library/exceptions.html#SystemExit), and [`SuspiciousOperation`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.SuspiciousOperation). Django catches these exceptions internally and converts them into the appropriate HTTP response codes. In these cases, you can check `reponse.status_code` in your test.

If `Client.raise_request_exception` is `False`, the test client will return a 500 response as would be returned to a browser. The response has the attribute [`exc_info`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#exc_info) to provide information about the unhandled exception.

### Persistent state

The test client is stateful. If a response returns a cookie, then that cookie will be stored in the test client and sent with all subsequent `get()` and `post()` requests.

Expiration policies for these cookies are not followed. If you want a cookie to expire, either delete it manually or create a new `Client` instance (which will effectively delete all cookies).

A test client has two attributes that store persistent state information. You can access these properties as part of a test condition.

* ##### `Client.cookies`

    A Python [`SimpleCookie`](https://docs.python.org/3/library/http.cookies.html#http.cookies.SimpleCookie) object, containing session information. See the documentation of the [`http.cookies`](https://docs.python.org/3/library/http.cookies.html#module-http.cookies) module for more.
    
* ##### `Client.session`

    A dictionary-like object containing session information. See the [session documentation](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Sessions#how-to-use-sessions) for full details.

    To modify the session and then save it, it must be stored in a variable first (because a new `SessionStore` is created every time this property is accessed):
    ```
    def test_something(self):
        session = self.client.session
        session['somekey'] = 'test'
        session.save()
    ```

### Setting the language

When testing applications that support internationalization and localization, you might want to set the language for a test client request. The method for doing so depends on whether or not the [`LocaleMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.middleware.locale.LocaleMiddleware) is enabled.

If the middleware is enabled, the language can be set by creating a cookie with a name of [`LANGUAGE_COOKIE_NAME`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-LANGUAGE_COOKIE_NAME) and a value of the language code:
```
from django.conf import settings

def test_language_using_cookie(self):
    self.client.cookies.load({settings.LANGUAGE_COOKIE_NAME: 'fr'})
    response = self.client.get('/')
    self.assertEqual(response.content, b"Bienvenue sur mon site.")
```
...or by including the `Accept-Language` HTTP header in the request:
```
def test_language_using_header(self):
    response = self.client.get('/', HTTP_ACCEPT_LANGUAGE='fr')
    self.assertEqual(response.content, b"Bienvenue sur mon site.")
```
More details are in [How Django discovers language preference](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#how-django-discovers-language-preference).

If the middleware isn't enabled, the active langauge may be set using [`translation.override()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.override):
```
from django.utils import translation

def test_language_using_override(self):
    with translation.override('fr'):
        response = self.client.get('/')
    self.assertEqual(reponse.content, b"Bienvenue sur mon site.")
```
More details are in [Explicitly setting the active language](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#explicitly-setting-the-active-language).

### Example

The following is a unit test using the test client:
```
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def setUp(self):
        # Every test needs a client.
        self.client = Client()

    def test_details(self):
        # Issue a GET request.
        response = self.client.get('/customer/details/')

        # Check that the response is 200 OK.
        self.assertEqual(response.status_code, 200)

        # Check that the rendered context contains 5 customers.
        self.assertEqual(len(response.context['customers']), 5)
```

<hr>

**See also**

[`django.test.RequestFactory`]() <!-- next module: "class RequestFactory" -->

<hr>

## Provided test case classes

Normal Python unit test classes extend a base class of [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase). Django provides a few extensions of this base class:

![Image showing the hierarchy of Django unit testing classes](https://docs.djangoproject.com/en/4.0/_images/django_unittest_classes_hierarchy.svg)

Hierarchy of Django unit testing classes

You can convert a normal `unittest.TestCase` to any of the subclasses: change the base class of your test from `unittest.TestCase` to the subclass. All of the standard Python unit test functionality will be available, and it will be augmented with some useful additions as described in each section below.

### `SimpleTestCase`

##### `class SimpleTestCase`

A subclass of [`unittest.TestCase`](https://docs.python.org/3/library/unittest.html#unittest.TestCase) that adds this functionality:

* Some useful assertions like:
    - Checking that a callable [raises a certain exception](). <!-- all below (?) -->
    - Checking that a callable [triggers a certain warning]().
    - Testing form field [rendering and error treatment]().
    - Testing [HTML responses for the presence/lack of a given fragment]().
    - Verifying that a template [has/hasn't been used to generate a given response content]().
    - Verifying that two [URLs]() are equal.
    - Verifying an HTTP [`redirect`]() is performed by the app.
    - Robustly testing two [HTML fragments]() for equality/inequality or [containment]().
    - Robustly testing two [XML fragments]() for equality/inequality.
    - Robustly testing two [JSON fragments]() for equality.
* The ability to run tests with [modified settings]().
* Using the [`client`]() [`Client`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-clientenforce_csrf_checksfalse-json_encoderdjangojsonencoder-defaults).

If your tests make any database queries, use subclasses [`TransactionTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-transactiontestcase) or [`TestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-testcase).

##### `SimpleTestCase.databases`

[`SimpleTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-simpletestcase) disallows database queries by default. This helps to avoid executing write queries which will affect other tests since each `SimpleTestCase` test isn't run in a transaction. If you aren't concerned about this problem, you can disable this behavior by setting the `database` class attribute to `'__all__'` on your test class.

<hr>

:warning: **Warning**: `SimpleTestCase` and its subclasses (e.g. `TestCase`, ...) rely on `setUpClass()` and `tearDownClass()` to perform some class-wide initialization (e.g., overriding settings). If you need to override those methods, don't forget to call the `super` implementation:
```
class MyTestCase(TestCase):

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        ...

    @classmethod
    def tearDownClass(cls):
        ...
        super().tearDownClass()
```
Be sure to account for Python's behavior if an exception is raised during `setUpClass()`. If that happens, neither the tests in the class nor `tearDownClass()` are run. In the case of [`django.test.TestCase`](), <!-- below --> this will leak the transaction created in `super()` which results in various symptoms including a segmentation fault on some platforms (reported on macOS). If you want to intentionally raise an exception such as [`unittest.SkipTest`](https://docs.python.org/3/library/unittest.html#unittest.SkipTest) in `setUpClass()`, be sure to do it before calling `super()` to avoid this.

<hr>

### `TransactionTestCase`

##### `class TransactionTestCase`

`TransactionTestCase` inherits from [`SimpleTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-simpletestcase) to add some database-specific features:

* Resetting the database to a known state at the beginning of each test to ease testing and using the ORM.
* Database [`fixtures`]().
* Test [skipping based on database backend features]().
* The remaining specialized [`assert*`]() methods. <!-- all below -->

Django's [`TestCase`]() <!-- below --> class is a more commonly used subclass of `TransactionTestCase` that makes use of database transaction facilities to speed up the process of resetting the database behaviors cannot be tested within a Django `TestCase` class. For instance, you cannot test that a block of code is executing within a transaction, as is required when using [`select_for_update()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.select_for_update). In those cases, you should use `TransactionTestCase`.

`TransactionTestCase` and `TestCase` are identical except for the manner in which the database is reset to a known state and the ability for test code to test the effects of commit and rollback:

* A `TransactionTestCase` resets the database after the test runs by truncating all tables. A `TransactionTestCase` may call commit and rollback and observe the effects of these calls on the database.
* A `TestCase`, on the other hand, does not truncate tables after a test. Instead, it encloses the test code in a database transaction that is rolled back at the end of the test. This guarantees that the rollback at the end of the test restores the database to its initial state.

<hr>

:warning: **Warning**: `TestCase` running on a database that does not support rollback (e.g., MySQL with the MyISAM storage engine), and all instances of `TransactionTestCase`, will roll back at the end of the test by deleting all data from the test database.

Apps [will not see their data reloaded](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Writing_Running_Tests#rollback-emulation); if you need this functionality (for example, third-party apps should enable this), you can set `serialized_rollback = True` inside the `TestCase` body.

<hr>

### `TestCase`

##### `class TestCase`

This is the most common class to use for writing tests in Django. It inherits from [`TransactionTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-transactiontestcase) (and by extention, [`SimpleTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-simpletestcase)). If your Django application doesn't use a database, use `SimpleTestCase`.

The class:

* Wraps the tests within two nested [`atomic()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Database_Transactions#atomicusingnone-savepointtrue-durablefalse) blocks: one for the whole class and one for each test. Therefore, if you want to test some specific database transaction behavior, use [`TransactionTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-transactiontestcase).
* Checks deferrable database constraints at the end of each test.

It also provides an additional method:

##### `classmethod TestCase.setUpTestData()`

The class-level `atomic` block described above allows the creation of initial data at the class level, once for the whole `TestCase`. This technique allows for faster tests as compared to using `setUp()`.

For example:
```
from django.test import TestCase

class MyTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up data for the whole TestCase
        cls.foo = Foo.objects.create(bar="Test")
        ...

    def test1(self):
        # Some test using self.foo
        ...

    def test2(self):
        # Some other test using self.foo
        ...

```
Note that is the tests are run on a database with no transaction support (for instance, MySQL with the MyISAM engine), `setUpTestData()` will be called before each test, negating the speed benefits.

##### `classmethod TestCase.captureOnCommitCallbacks(using=DEFAULT_DB_ALIAS, execute=False)`

Returns a context manager that captures [`transaction.on_commit()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Database_Transactions#on_commitfunc-usingnone) callbacks for the given database connection. It returns a list that contains, on exit of the context, the captured callback functions. From this list, you can make assertions on the callbacks or call them to invoke their side effects, emulating a commit.

`using` is the alias of the database connection to capture callbacks for.

If `execute` is `True`, all the callbacks will be called as the context manager exits, if no exception occurred. This emulates a commit after the wrapped block of code.

For example:
```
from django.core import mail
from django.test import TestCase


class ContactTests(TestCase):
    def test_post(self):
        with self.captureOnCommitCallbacks(execute=True) as callbacks:
            response = self.client.post(
                '/contact/',
                {'message': 'I like your site'},
            )

        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(callbacks), 1)
        self.assertEqual(len(mail.outbox), 1)
        self.assertEqual(mail.outbox[0].subject, 'Contact Form')
        self.assertEqual(mail.outbox[0].body, 'I like your site')
```

### `LiveServerTestCase`

##### `class LiveServerTestCase`

`LiveServerTestCase` does basically the same as [`TransactionTestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-transactiontestcase) with one extra feature: it launches a live Django server in the background on setup, and shuts it down on teardown. This allows the use of automated test clients other than the [Django dummy client](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#the-test-client) such as, for example, the [Selenium](https://www.selenium.dev/) client, to execute a series of functional tests inside a browser and simulate a real user's actions.

The live server listens on `localhost` and binds to port 0, which uses a free port assigned by the operating system. The server's URL can be accessed with `self.live_server_url` during the tests.

To demonstrate how to use `LiveServerTestCase`, let's write a Selenium test. First of all, you need to install the [selenium package](https://pypi.org/project/selenium/) into your Python path:
```
$ python -m pip install selenium
```
Then, add a `LiveServerTestCase`-based test to your app's tests module (for example: `myapp/tests.py`). For this example, we'll assume you're using the [`staticfiles`](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles) app and want to have static files served during the execution of your tests similar to what we get at development time with `DEBUG=True`, i.e. without having to collect them using [`collectstatic`](). We'll use the [`StaticLiveServerTestCase`](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#django.contrib.staticfiles.testing.StaticLiveServerTestCase) subclass which provides that functionality. Replace it with `django.test.LiveServerTestCase` if you don't need that.

The code for this test may look as follows:
```
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium.webdriver.firefox.webdriver import WebDriver

class MySeleniumTests(StaticLiveServerTestCase):
    fixtures = ['user-data.json']

    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.selenium = WebDriver()
        cls.selenium.implicitly_wait(10)

    @classmethod
    def tearDownClass(cls):
        cls.selenium.quit()
        super().tearDownClass()

    def test_login(self):
        self.selenium.get('%s%s' % (self.live_server_url, '/login/'))
        username_input = self.selenium.find_element_by_name("username")
        username_input.send_keys('myuser')
        password_input = self.selenium.find_element_by_name("password")
        self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
```
Finally, you may run the test as follows:
```
$ ./manage.py test myapp.tests.MySeleniumTests.test_login
```
This example will automatically open Firefox, then go to the login page, enter the credentials and press the "Log in" button. Selenium offers other drivers in case you do not have Firefox installed or wish to use another browser. The example above is just a tiny fraction of what the Selenium client can do; check out the [full reference](https://selenium-python.readthedocs.io/api.html) for more details.

<hr>

**Note**: When using an in-memory SQLite database to run the tests, the same database connection will be shared by two threads in parallel: the thread in which the live server is run and the thread in which the test case is run. It's important to prevent simultaneous database queries via this shared connection by the two threads, as that may sometimes randomly cause the tests to fail. So you need to ensure that the two threads don't access the database at the same time. In particular, this means that in some cases (for example, just after clicking a link or submitting a form), you might need to check that a response is received by Selenium and that the next page is loaded before proceeding with further test execution. Do this, for example, by making Selenium wait until the `<body>` HTML tag is found in the response (requires Selenium > 2.13):
```
def test_login(self):
    from selenium.webdriver.support.wait import WebDriverWait
    timeout = 2
    ...
    self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
    # Wait until the response is received
    WebDriverWait(self.selenium, timeout).until(
        lambda driver: driver.find_element_by_tag_name('body'))
```
The tricky thing here is that there's really no such thing as a "page load", especially in modern web apps that generate HTML dynamically after the server generates the initial document. So, checking for the presence of `<body>` in the response might  not necessarily be appropriate for all use cases. Please refer to the [Selenium FAQ](https://web.archive.org/web/20160129132110/http://code.google.com/p/selenium/wiki/FrequentlyAskedQuestions#Q:_WebDriver_fails_to_find_elements_/_Does_not_block_on_page_loa) and [Selenium documentation](https://www.selenium.dev/documentation/webdriver/waits/#explicit-wait) for more information.

<hr>

## Test case features

### Default test client

##### `SimpleTestCase.client`

Every test case in a `django.test.*TestCase` instance has access to an instance has access to an instance of a Django test client. This client can be accessed as `self.client`. This client is recreated for each test, so you don't have to worry about state (such as cookies) carrying over from one test to another.

This means, instead of instantiating a `Client` in each test:
```
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def test_details(self):
        client = Client()
        response = client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)

    def test_index(self):
        client = Client()
        response = client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)
```
...you can refer to `self.client`, like so:
```
from django.test import TestCase

class SimpleTest(TestCase):
    def test_details(self):
        response = self.client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)

    def test_index(self):
        response = self.client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)
```

### Customizing the test client

##### `SimpleTestCase.client_class`

If you want to use a different `Client` class (for example, a subclass with customized behavior), use the `client_class` class attribute:
```
from django.test import Client, TestCase

class MyTestClient(Client):
    # Specialized methods for your environment
    ...

class MyTest(TestCase):
    client_class = MyTestClient

    def test_my_stuff(self):
        # Here self.client is an instance of MyTestClient...
        call_some_test_code()
```

### Fixture loading

##### `TransactionTestCase.fixtures`

A test case for a database-backed website isn't much use if there isn't any data in the database. Tests are more readable and it's more maintainable to create objects using the ORM, for example, in [`TestCase.setUpTestData()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#classmethod-testcasesetuptestdata). However, you can also use fixtures.

A fixture is a collection of data that Django knows how to import into a database. For example, if your site has user accounts, you might set up a fixture of fake user accounts in order to populate your database during tests.

The most straightforward way of creating a fixture is to use the [`manage.py dumpdata`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-dumpdata) command. This assumes you already have some data in your database. See the [`dumpdata` documentation](https://docs.djangoproject.com/en/4.0/ref/django-admin/#dumpdata) for more details.

Once you've created a fixture and placed it in a `fixtures` directory in one of your [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS), you can use it in your unit tests by specifying a `fixtures` class attribute on your [`django.test.TestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-testcase) subclass:
```
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    fixtures = ['mammals.json', 'birds']

    def setUp(self):
        # Test definitions as before.
        call_setup_methods()

    def test_fluffy_animals(self):
        # A test that uses the fixtures.
        call_some_test_code()
```
Here's specifically what will happen:

* At the start of each test, before `setUp()` is run, Django will flush the database, returning the database to the state it was in directly after [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) was called.
* Then, all the named fixtures are installed. In this example, Django will install any JSON fixture named `mammals`, followed by any fixture named `birds`. See the [`loaddata`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-loaddata) documentation for more details on defining and installing fixtures.

For performance reasons, [`TestCase`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#class-testcase) loads fixtures once for the entire test class, before [`setUpTestData()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#classmethod-testcasesetuptestdata), instead of before each test, and it uses transactions to clean the database before each test. In any case, you can be certain that the outcome of a test will not be affected by another test or by the order of test execution.

By default, fixtures are only loaded into the `default` database. If you are using multiple databases and set [`TransactionTestCase.databases`](), <!-- below --> fixtures will be loaded into all specified databases.

### URLconf configuration

If your application provides views, you may want to include tests that use the test client to exercise those views. However, an end user is free to deploy the views in your application at any URL of their choosing. This means that your tests can't rely upon the fact that your views will be available at a particular URL. Decorate your test class or test method with `@override_settings(ROOT_URLCONF=...)` for URLconf configuration.

### Multi-database support

##### `TransactionTestCase.database`

Django sets up a test database corresponding to every database that is defined in the [`DATABASES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) definition in your settings and referred to by at least one test through `databases`.

However, a big part of the time taken to run a Django `TestCase` is consumed by the call to `flush` that ensures that you have a clean database at the start of each test run. If you have multiple databases, multiple flushes are required (one for each database), which can be a time consuming activity -- especially if your tests don't need to test multi-database activity.

As an optimization, Django only flushes the `default` database at the start of each test run. If your setup contains multiple databases, and you have a test that requires every database to be clean, you can use the `databases` attribute on the test suite to request extra databases to be flushed.

For example:
```
class TestMyViews(TransactionTestCase):
    databases = {'default', 'other'}

    def test_index_page_view(self):
        call_some_test_code()
```
This test case will flush the `default` and `other` test databases before running `test_index_page_view`. You can also use `'__all__'` to specify that all of the test databases must be flushed.

The `databases` flag also controls which databases the [`TransactionTestCase.fixtures`]() are loaded into. By default, fixtures are only loaded into the `default` database.

Queries against databases not in `databases` will give assertion errors to prevent state leaking between tests.

##### `TestCase.databases`

By default, only the `default` database will be wrapped in a transaction during a `TestCase`'s execution and attempts to query other databases will result in assertion errors to prevent state leaking between tests.

Use the `databases` class attribute on the test class to request transaction wrapping against non-`default` databases.

For example:
```
class OtherDBTests(TestCase):
    databases = {'other'}

    def test_other_db_query(self):
        ...
```
This test will only allow queries against the `other` database. Just like for [`SimpleTestCase.databases`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#simpletestcasedatabases) and [`TransactionTestCase.databases`](), the `'__all__'` constant can be used to specify that the test should allow queries to all databases.