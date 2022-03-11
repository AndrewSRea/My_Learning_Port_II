# Advanced testing topics

## The request factory

##### `class RequestFactory`

The `RequestFactory` shares the same API as the test client. However, instead of behaving like a browser, the `RequestFactory` provides a way to generate a request instance that can be used as the first argument to any view. This means you can test a view function the same way as you would test any other function -- as a black box, with exactly known inputs, testing for specific outputs.

The API for the `RequestFactory` is a slightly restricted subset of the test client API:

* It only has access to the HTTP methods [`get()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#getpath-datanone-followfalse-securefalse-extra), [`post()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#postpath-datanone-content_typemultipart_content-followfalse-securefalse-extra), [`put()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#putpath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra), [`delete()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#deletepath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra), [`head()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#headpath-datanone-followfalse-securefalse-extra), [`options()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#optionspath-data-content_typeapplicationoctet-stream-followfalse-securefalse-extra), and [`trace()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#tracepath-followfalse-securefalse-extra).
* These methods accept all the same arguments *except* for `follow`. Since this is just a factory for producing requests, it's up to you to handle the response.
* It does not support middleware. Session and authentication attributes must be supplied by the test itself if required for the view to function properly.

### Example

The following is a unit test using the request factory:
```
from django.contrib.auth.models import AnonymousUser, User
from django.test import RequestFactory, TestCase

from .views import MyView, my_view

class SimpleTest(TestCase):
    def setUp(self):
        # Every test needs access to the request factory.
        self.factory = RequestFactory()
        self.user = User.objects.create_user(
            username='jacob', email='jacob@...', password='top_secret')

    def test_details(self):
        # Create an instance of a GET request.
        request = self.factory.get('/customer/details')

        # Recall that middleware are not supported. You can simulate a 
        # logged-in user by setting request.user manually.
        request.user = self.user

        # Or you can simulate an anonymous user by setting request.user to
        # an AnonymousUser instance.
        request.user = AnonymousUser()

        # Test my_view() as if it were deployed at /customer/details
        response = my_view(request)
        # Use this syntax for class-based views.
        response = MyView.as_view()(request)
        self.assertEqual(response.status_code, 200)
```

### `AsyncRequestFactory`

`RequestFactory` creates WSGI-like requests. If you want to create ASGI-like requests, including having a correct ASGI `scope`, you can instead use `django.test.AsyncRequestFactory`.

This class is directly API-compatible with `RequestFactory`, with the only difference being that it returns `ASGIRequest` instances rather than `WSGIRequest` instances. All of its methods are still synchronous callables.

## Testing class-based views

In order to test class-based views outside of the request/response cycle, you must ensure that they are configured correctly, by calling [`setup()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View.setup) after instantiation.

For example, assuming the following class-based view:
```
# views.py

from django.views.generic import TemplateView

class HomeView(TemplateView):
    template_name = 'my/home.html'

    def get_context_data(self, **kwargs):
        kwargs['environment'] = 'Production'
        return super().get_context_data(**kwargs)
```
You may directly test the `get_context_data()` method by first instantiating the view, then passing a `request` to `setup()`, before proceeding with your test's code:
```
# tests.py

from django.test import RequestFactory, TestCase
from .views import HomeView

class HomePageTest(TestCase):
    def test_environment_set_in_context(self):
        request = RequestFactory().get('/')
        view = HomeView()
        view.setup(request)

        context = view.get_context_data()
        self.assertIn('environment', context)
```

## Tests and multiple host names

The [`ALLOWED_HOSTS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ALLOWED_HOSTS) setting is validated when running tests. This allows the test client to differentiate between internal and external URLs.

Projects that support multitenancy or otherwise alter business logic based on the request's host and use custom host names in tests must include those hosts in `ALLOWED_HOSTS`.

The first option to do so is to add the hosts to your settings file. For example, the test suite for docs.djangoproject.com includes the following:
```
from django.test import TestCase

class SearchFormTestCase(TestCase):
    def test_empty_get(self):
        response = self.client.get('/en/dev/search/', HTTP_HOST='docs.djangoproject.dev:8000')
        self.assertEqual(response.status_code, 200)
```
...and the settings file includes a list of the domains supported by the project:
```
ALLOWED_HOSTS = [
    'www.djangoproject.dev',
    'docs.djangoproject.dev',
    ...
]
```
Another option is to add the required hosts to [`ALLOWED_HOSTS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ALLOWED_HOSTS) using [`override_settings()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#override_settingskwargs) or [`modify_settings()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Testing/Testing_Tools#modify_settingsargs-kwargs). This option may be preferable in standalone apps that can't package their own settings file or for projects where the list of domains is not static (e.g., subdomains for multitenancy). For example, you could write a test for the domain `http://otherserver/` as follows:
```
from django.test import TestCase, override_settings

class MultiDomainTestCase(TestCase):
    @override_settings(ALLOWED_HOSTS=['otherserver'])
    def test_other_domain(self):
        response = self.client.get('http://otherserver/foo/bar/')
```
Disabling `ALLOWED_HOSTS` checking (`ALLOWED_HOSTS = ['*']`) when running tests prevents the test client from raising a helpful error message if you follow a redirect to an external URL.