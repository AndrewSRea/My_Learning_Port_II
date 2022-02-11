# Middleware

Middleware is a framework of hooks into Django's request/response processing. It's a light, low-level "plugin" system for globally altering Django's input or output.

Each middleware component is responsible for doing some specific function. For example, Django includes a middleware component, [`AuthenticationMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.contrib.auth.middleware.AuthenticationMiddleware), that associates users with requests using sessions.

This document explains how middleware works, how you activate middleware, and how to write your own middleware. Django ships with some built-in middleware you can use right out of the box. They're documented in the [built-in middleware reference](https://docs.djangoproject.com/en/4.0/ref/middleware/).

## Writing your own middleware

A middleware factory is a callable that takes a `get_response` callable and returns a middleware. A middleware is a callable that takes a request and returns a response, just like a view.

A middleware can be written as a function that looks like this:
```
def simple_middleware(get_response):
    # One-time configuration and initialization.

    def middleware(request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response

    return middleware
```
Or it can be written as a class whose instances are callable, like this:
```
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.

    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = self.get_response(request)

        # Code to be executed for each request/response after
        # the view is called.

        return response
```
The `get_reponse` callable provided by Django might be the actual view (if this is the last listed middleware) or it might be the next middleware in the chain. The current middleware doesn't need to know or care what exactly it is, just that it represents whatever comes next.

The above is a slight simplification -- the `get_response` callable for the last middleware in the chain won't be the actual view but rather a wrapper method from the handler which takes care of applying [view middleware](), <!-- below --> calling the view with appropriate URL arguments, and applying [template-response]() <!-- below --> and [exception]() <!-- below --> middleware.

Middleware can either support only synchronous Python (the default), only asynchronous Python, or both. See [Asynchronous support]() for details of how to advertise what you support, and know what kind of request you are getting. <!-- below -->

Middleware can live anywhere on your Python path.

### `__init__(get_response)`

Middleware factories must accept a `get_response` argument. You can also initialize some global state for the middleware. Keep in mind a couple of caveats:

* Django initializes your middleware with only the `get_response` argument, so you can't define `__init__()` as requiring any other arguments.
* Unlike the `__call__()` method which is called once per request,`__init__()` is called only *once*, when the web server starts.

### Marking middleware as unused

It's sometimes useful to determine at startup time whether a piece of middleware should be used. In these cases, your middleware's `__init__()` method may raise [`MiddlewareNotUsed`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.MiddlewareNotUsed). Django will then remove that middleware from the middleware process and log a debug message to the [`django.request`](https://docs.djangoproject.com/en/4.0/ref/logging/#django-request-logger) logger when [`DEBUG`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DEBUG) is `True`.

## Activating middleware

To activate a middleware component, add it to the [`MIDDLEWARE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIDDLEWARE) list in your Django settings.

In `MIDDLEWARE`, each middleware component is represented by a string: the full Python path to the middleware factory's class or function name. For example, here's the default value created by [`django-admin startproject`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startproject):
```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```
A Django installation doesn't require any middleware -- `MIDDLEWARE` can be empty, if you'd like -- but it's strongly suggested that you, at least, use [`CommonMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.middleware.common.CommonMiddleware).

The order in `MIDDLEWARE` matters because a middleware can depend on other middleware. For instance, [`AuthenticationMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.contrib.auth.middleware.AuthenticationMiddleware) stores the authenticated user in the session; therefore, it must run after [`SessionMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.contrib.sessions.middleware.SessionMiddleware). See [Middleware ordering](https://docs.djangoproject.com/en/4.0/ref/middleware/#middleware-ordering) for some common hints about ordering of Django middleware classes.

## Middleware order and layering

During the request phase, before calling the view, Django applies middleware in the order it's defined in [`MIDDLEWARE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIDDLEWARE), top-down.

You can think of it like an onion: each middleware class is a "layer" that wraps the view, which is in the core of the onion. If the request passes through all the layers of the onion (each one calls `get_response` to pass the request in to the next layer), all the way to the view at the core, the response will then pass through every layer (in reverse order) on the way back out.

If one of the layers decides to short-circuit and return a response without ever calling its `get_response`, none of the layers of the onion inside that layer (including the view) will see the request or the response. The response will only return through the same layers that the request passed in through.