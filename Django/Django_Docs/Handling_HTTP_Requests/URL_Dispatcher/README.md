# URL dispatcher

A clean, elegant URL scheme is an important detail in a high-quality web application. Django lets you design URLs however you want, with no framework limitations.

See [Cool URIs don't change](https://www.w3.org/Provider/Style/URI), by World Wide Web creator Tim Berners-Lee, for excellent arguments on why URLs should be clean and usable.

## Overview

To design URLs for an app, you create a Python module informally called a **URLconf** (URL configuration). This module is pure Python code and is a mapping between URL path expressions to Python functions (your views).

This mapping can be as short or as long as needed. It can reference other mappings. And, because it's pure Python code, it can be constructed dynamically.

Django also provides a way to translate URLs according to the active language. See the [internationalization documentation](https://docs.djangoproject.com/en/4.0/topics/i18n/translation/#url-internationalization) for more information.

## How Django processes a request

When a user requests a page from your Django-powered site, this is the algorithm the system follows to determine which Python code to execute:

1. Django determines the root URLconf module to use. Ordinarily, this is the value of the [`ROOT_URLCONF`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ROOT_URLCONF) setting, but if the incoming `HttpRequest` object has a [`urlconf`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.urlconf) attribute (set by middleware), its value will be used in place of the `ROOT_URLCONF` setting.
2. Django loads that Python module and looks for the variable `urlpatterns`. This should be a [sequence](https://docs.python.org/3/glossary.html#term-sequence) of [`django.urls.path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) and/or [`django.urls.re_path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.re_path) instances.
3. Django runs through each URL pattern, in order, and stops at the first one that matches the requested URL, matching against [`path_info`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.path_info).
4. Once one of the URL patterns matches, Django imports and calls the given view, which is a Python function (or a [class-based view]()). <!-- a future module --> The view gets passed the following arguments:
    - An instance of [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest).
    - If the matched URL pattern contained no named groups, then the matches from the regular expression are provided as positional arguments.
    - The keyword arguments are made up of any named parts matched by the path expression that are provided, overridden by any arguments specified in the optional `kwargs` argument to `django.urls.path()` or `django.urls.re_path()`.
5. If no URL pattern matches, or if an exception is raised during any point in this process, Django invokes an appropriate error-handling view. See [Error handling]() below. <!-- below -->

## Example

Here's a sample URLconf:
```
from django.urls import path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
]
```

Notes: 

* To capture a value from the URL, use angle brackets.
* Captured values can optionally include a converter type. For example, use `<int:name>` to capture an integer parameter. If a converter isn't included, any string, excluding a `/` character, is matched.
* There's no need to add a leading slash, because every URL has that. For example, it's `articles`, not `/articles`.

Example requests:

* A request to `/articles/2005/03/` would match the third entry in the list. Django would call the function `views.month_archive(request, year=2005, month=3)`.
* `/articles/2003/` would match the first pattern in the list, not the second one, because the patterns are tested in order, and the first one is the first test to pass. Feel free to exploit the ordering to insert special cases like this. Here, Django would call the function `views.special_case_2003(request)`.
* `/articles/2003` would not match any of these patterns, because each pattern requires that the URL end with a slash.
* `/articles/2003/03/building-a-django-site/` would match the final pattern. Django would call the function `views.article_detail(request, year=2003, month=3, slug="building-a-django-site")`.

## Path converters

The following path converters are available by default:

* `str` - Matches any non-empty string, excluding the path separator, `'/'`. This is the default if a converter isn't included in the expression.
* `int` - Matches zero or any positive integer. Returns an `int`.
* `slug` - Macthes any slug string consisting of ASCII letters or numbers, plus the hyphen and underscore characters. For example, `building-your-1st-django-site`.
* `uuid` - Matches a formatted UUID. To prevent multiple URLs from mapping to the same page, dashes must be included and letters must be lowercase. For example, `075194d3-6885-417e-a8a8-6c931e272f00`. Returns a [UUID](https://docs.python.org/3/library/uuid.html#uuid.UUID) instance.
* `path` - Matches any non-empty string, including the path separator, `'/'`. This allows you to match against a complete URL path rather than a segment of a URL path as with `str`.

## Registering custom path converters

For more complex matching requirements, you can define your own path converters.

A converter is a class that includes the following:

* A `regex` class attribute, as a string.
* A `to_python(self, value)` method, which handles converting the matched string into the type that should be passed to the view function. It should raise `ValueError` if it can't convert the given value. A `ValueError` is interpreted as no match and as a consequence, a 404 response is sent to the user unless another URL pattern matches.
* A `to_url(self, value)` method, which handles converting the Python type into a string to be used in the URL. It should raise `ValueError` if it can't convert the given value. A `ValueError` is interpreted as no match and as a consequence, [`reverse()`](https://docs.djangoproject.com/en/4.0/ref/urlresolvers/#django.urls.reverse) will raise [`NoReverseMatch`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.urls.NoReverseMatch) unless another URL pattern matches.

For example:
```
class FourDigitYearConverter:
    regex = '[0-9]{4}'

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return '%04d' % value
```
Register custom converter classes in your URLconf using [`register_converter()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.register_converter):
```
from django.urls import path, register_converter

from . import converters, views

register_converter(converters.FourDigitYearConverter, 'yyyy')

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    path('articles/<yyyy:year>/', views.year_archive),
    ...
]
```

## Using regular expressions

If the paths and converters syntax isn't sufficient for defining your URL patterns, you can also use regular expressions. To do so, use [`re_path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.re_path) instead of [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path).

In Python regular expressions, the syntax for named regular expressions is `(?P<name>pattern)`, where `name` is the name of the group and `patterns` is some pattern to match.

Here's the example URLconf from earlier, rewritten using regular expressions:
```
from django.urls import path, re_path

from . import views

urlpatterns = [
    path('articles/2003/', views.special_case_2003),
    re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
    re_path(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<slug>[\w-]+)/$', views.article_detail),
]
```
This accomplishes roughly the same thing as the previous example, except:

* The exact URLs that will match are slightly more constrained. For example, the year 10000 will no longer match since the year integers are constrained to be exactly four digits long.
* Each captured argument is sent to the view as a string, regardless of what sort of match the regular expression makes.

When switching from using `path()` to `re_path()` or vice versa, it's particularly important to be aware that the type of the view arguments may change, and so you may need to adapt your views.

### Using unnamed regular expression groups

As well as the named group syntax, e.g. `(?P<year>[0-9]{4})`, you can also use the shorter unnamed group, e.g. `([0-9]{4})`.

This usage isn't particularly recommended as it makes it easier to accidentally introduce errors between the intended meaning of a match and the arguments of the view.

In either case, using only one style within a given regex is recommended. When both styles are mixed, any unnamed groups are ignored and only named groups are passed to the view function.

### Nested arguments

Regular expressions allow nested arguments, and Django will resolve them and pass them to the view. When reversing, Django will try to fill in all outer captured arguments, ignoring any nested captured arguments. Consider the following URL patterns which optionally take a page argument:
```
from django.urls import re_path

urlpatterns = [
    re_path(r'^blog/(page-(\d+)/)?$', blog_articles),                   # bad
    re_path(r'^comments/(?:page-(?P<page_number>\d+)/)?$', comments),   # good
]
```
Both patterns use nested arguments and will resolve: for example, `blog/page-2/` will result in a match to `blog_articles` with two positional arguments: `page-2/` and `2`. The second pattern for `comments` will match `comments/page-2/` with keyword argument `page_number` set to 2. The outer argument in this case is a non-capturing argument `(?:...)`.

The `blog_articles` view needs the outermost captured argument to be reversed, `page-2/` or no arguments in this case, while `comments` can be reversed with either no arguments or a value for `page_number`.

Nested captured arguments create a strong coupling between the view arguments and the URL as illustrated by `blog_articles`: the view receives part of the URL (`page-2/`) instead of only the value the view is interested in. This coupling is even more pronounced when reversing, since to reverse the view we need to pass the piece of URL instead of the page number.

As a rule of thumb, only capture the values the view needs to work with and use non-capturing arguments when the regular expression needs an argument but the view ignores it.

## What the URLconf searches against

THe URLconf searches against the requested URL, as a normal Python string. This does not include `GET` or `POST` parameters, or the domain name.

For example, in a request to `https://www.example.com/myapp/`, the URLconf will look for `myapp/`.

In a request to `https://www.example.com/myapp/?page=3`, the URLconf will look for `myapp/`.

The URLconf doesn't look at the request method. In other words, all request methods -- `POST`, `GET`, `HEAD`, etc. -- will be routed to the same function for the same URL.

## Specifying defaults for view arguments

A convenient trick is to specify default parameters for your views' arguments. Here's an example URLconf and view:
```
# URLconf
from django.urls import path

from . import views

urlpatterns = [
    path('blog/, views.page),
    path('blog/pages<int:num>/', views.page),
]

# View (in blog/views.py)
def page(request, num=1):
    # Output the appropriate page of blog entries, according to num.
    ...
```
In the above example, both URL patterns point to the same view -- `views.page` -- but the first pattern doesn't capture anything from the URL. If the first pattern matches, the `page()` function will use its default argument for `num`, `1`. If the second pattern matches, `page()` will use whatever `num` value was captured.

## Performance

Each regular expression in a `urlpatterns` is compiled the first time it's accessed. This makes the system blazingly fast.

## Syntax of the `urlpatterns` variable

`urlpatterns` should be a [sequence](https://docs.python.org/3/glossary.html#term-sequence) of [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) and/or [`re_path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.re_path) instances.

## Error handling

When Django can't find a match for the requested URL, or when an exception is raised, Django invokes an error-handling view.

The views to use for these cases are specified by four variables. Their default values should suffice for most projects, but further customization is possible by overriding their default values.

See the documentation on [customizing error views]() for the full details. <!-- next page -->

Such values can be set in your root URLconf. Setting these variables in any other URLconf will have no effect.

Values must be callables, or strings representing the full Python import path to the view that should be called to handle the error condition at hand.

The variables are:

* `handler400` - See [`django.conf.urls.handler400](https://docs.djangoproject.com/en/4.0/ref/urls/#handler400).
* `handler403` - See [`django.conf.urls.handler403](https://docs.djangoproject.com/en/4.0/ref/urls/#handler403).
* `handler404` - See [`django.conf.urls.handler404](https://docs.djangoproject.com/en/4.0/ref/urls/#handler404).
* `handler500` - See [`django.conf.urls.handler500](https://docs.djangoproject.com/en/4.0/ref/urls/#handler500).

## Including other URLconfs 

At any point, your `urlpatterns` can "include" other URLconf modules. This essentially "roots" a set of URLs below other ones. 

For example, here's an excerpt of the URLconf for the [Django website](https://www.djangoproject.com/) itself. It includes a number of other URLconfs:
```
from django.urls import include, path

urlpatterns = [
    # ... snip ...
    path('community/', include('aggregator.urls')),
    path('contact/', include('contact.urls')),
    # ... snip ...
]
```
Whenever Django encounters [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

Another possibility is to include additional URL patterns by using a list of [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) instances. For example, consider this URLconf:
```
from django.urls import include, path

from apps.main import views as main_views
from credit import views as credit_views

extra_patterns = [
    path('reports/', credit_views.report),
    path('reports/<int:id>/', credit_views.report),
    path('charge/', credit_views.charge),
]

urlpatterns = [
    path('', main_views.homepage),
    path('help/', include('apps.help.urls')),
    path('credit/', include(extra_patterns)),
]
```
In this example, the `/credit/reports/` URL will be handled by the `credit_views.report()` Django view.

This can be used to remove redundancy from URLconfs where a single pattern prefix is used repeatedly. For example, consider this URLconf:
```
from django.urls import path
from . import views

urlpatterns = [
    path('<page_slug>-<page_id>/history/', views.history),
    path('<page_slug>-<page_id>/edit/', views.edit),
    path('<page_slug>-<page_id>/discuss/', views.discuss),
    path('<page_slug>-<page_id>/permissions/', views.permissions),
]
```
We can improve this by stating the common path prefix only once and grouping the suffixes that differ:
```
from django.urls import include, path
from . import views

urlpatterns = [
    path('<page_slug>-<page_id>/', include([
        path('history/', views.history),
        path('edit/', views.edit),
        path('discuss/', views.discuss),
        path('permissions/', views.permissions),
    ])),
]
```

### Captured permissions

An included URLconf receives any captured parameters from parent URLconfs, so the following example is valid:
```
# In settings/urls/main.py
from django.urls import include, path

urlpatterns = [
    path('<username>/blog/', include('foo.urls.blog')),
]

# In foo/urls/blog.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.blog.index),
    path('archive/', views.blog.archive),
]
```
In the above example, the captured `"username"` variable is passed to the included URLconf, as expected.