# Writing your first Django app - Part 1

Let's learn by example.

Throughout this tutorial, we'll walk you through the creation of a basic poll application.

It'll consist of two parts:

* A public site that lets people view polls and vote in them.
* An admin site that lets you add, change, and delete polls.

We'll assume you have [Django installed](https://docs.djangoproject.com/en/4.0/intro/install/) already. You can tell Django is installed and which version by running the following command in a shell prompt (indicated by the `$` prefix):
```
$ python -m django --version
```
If Django is installed, you should see the version of your installation. If it isn't, you'll get an error telling "No module named django".

This tutorial is written for Django 4.0, which supports Python 3.8 and later (at the time of this writing). If the Django version doesn't match, you can refer to the [Django's own tutorial](https://docs.djangoproject.com/en/4.0/intro/tutorial01/) for your version of Django by using the version switcher at the bottom right corner of Django's tutorial page (the little box marked "Documentation version: 4.0"), or update Django to the newest version. If you're using an older version of Python, check [What Python version can I use with Django?](https://docs.djangoproject.com/en/4.0/faq/install/#faq-python-version-support) to find a compatible version of Django.

See [How to install Django](https://docs.djangoproject.com/en/4.0/topics/install/) for advice on how to remove older versions of Django and install a newer one.

<hr>

**Where to get help:** If you're having trouble going through this tutorial, please head over to the [Getting Help](https://docs.djangoproject.com/en/4.0/faq/help/) section of the FAQ.

<hr>

## Creating a project

If this is your first time using Django, you'll have to take care of some initial setup. Namely, you'll need to auto-generate some code that establishes a Django [project](https://docs.djangoproject.com/en/4.0/glossary/#term-project) -- a collection of settings for an instance of Django, including database configuration, Django-specific options and application-specific settings.

From the command line, `cd` into a directory where you'd like to store your code, then run the following command (make sure you're in your [virtual environment in your terminal first](https://github.com/AndrewSRea/My_Learning_Port/tree/main/JavaScript/Server-Side_Website_Programming/Django_Web_Framework/Django_Development_Environment#setting-up-a-django-development-environment)):
```
$ django-admin startproject mysite
```
This will create a `mysite` directory in your current directory. If it didn't work, see [Problems running django-admin](https://docs.djangoproject.com/en/4.0/faq/troubleshooting/#troubleshooting-django-admin).

<hr>

**Note**: These command line instructions are based upon using a macOS/Linux bash terminal. If you are using Windows, I would suggest following [Django's tutorial documentation](https://docs.djangoproject.com/en/4.0/intro/tutorial01/) directly, and clicking the Windows icons above each command prompt to switch to Command Prompt commands.

<hr>

**Note**: You'll need to avoid naming projects after built-in Python or Django components. In particular, this means you should avoid using names like `django` (which will conflict with Django itself) or `test` (which conflicts with a built-in Python package).

<hr>

**Where should this code live?**

If your background is in plain old PHP (with no use of modern frameworks), you're probably used to putting code under the web server's document root (in a place such as `/var/www`). With Django, you don't do that. It's not a good idea to put any of this Python code within your web server's document root, because it risks the possibility that people may be able to view your code over the web. That's not goof for security.

Put your code in some directory **outside** of the document root, such as `/home/mycode`.

<hr>

Let's look at what [`startproject`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startproject) created:
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```
These files are:

* The outer `mysite/` root directory is a container for your project. Its name doesn't matter to Django; you can rename it to anything you like.
* `manage.py`: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about `manage.py` in [`django-admin` and `manage.py`](https://docs.djangoproject.com/en/4.0/ref/django-admin/).
* The inner `mysite/` directory is the actual Python package for your project. Its name is the Python package name you'll need to use to import anything inside it (e.g. `mysite.urls`).
* `mysite/__init__.py`: An empty file that tells Python that this directory should be considered a Python package. If you're a Python beginner, read [more about packages](https://docs.python.org/3/tutorial/modules.html#tut-packages) in the official Python docs.
* `mysite/settings.py`: Settings/configuration for this Django project. [Django settings](https://docs.djangoproject.com/en/4.0/topics/settings/) will tell you all about how settings work.
* `mysite/urls.py`: The URL declarations for this Django project; a "table of contents" of your Django-powered site. You can read more about URLs in [URL dispatcher](https://docs.djangoproject.com/en/4.0/topics/http/urls/).
* `mysite/asgi.py`: An entry-point for ASGI-compatible web servers to serve your project. See [How to deploy with ASGI](https://docs.djangoproject.com/en/4.0/howto/deployment/asgi/) for more details.
* `mysite/wsgi.py`: An entry-point for WSGI-compatible web servers to serve your project. See [How to deploy with WSGI](https://docs.djangoproject.com/en/4.0/howto/deployment/wsgi/) for more details.

## The development server

Let's verify your Django project works. Change into the outer `mysite` directory, if you haven't already, and run the following commands:
```
python manage.py runserver
```
You'll see the following output on the command line:
```
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

December 28, 2021 - 15:50:53
Django version 4.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

<hr>

**Note**: Ignore the warning about unapplied database migrations for now; we'll deal with the database shortly.

<hr>

You've started the Django development server, a lightweight web server written purely in Python. We've included this with Django so you can develop things rapidly, without having to deal with configuring a production server -- such as Apache -- until you're ready for production.

Now's a good time to note: **don't** use this server in anything resembling a production environment. It's intended only for use while developing. (We're in the business of making web frameworks, not web servers.)

Now that the server's running, visit [http://127.0.0.1:8000/](http://127.0.0.1:8000/) with your web browser. You'll see a "Congratulations!" page, with a rocket taking off. It worked!

<hr>

**Changing the port**

By default, the [`runserver`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver) command starts the development server on the internal IP at port 8000.

If you want to change the server's port, pass it as a command-line argument. For instance, this command starts the server on port 8080:
```
$ python manage.py runserver 8080
```
If you want to change the server's IP, pass it along with the port. For example, to listen on all available public IPs (which is useful if you are running Vagrant or want to show off your work on other computers on the network), use:
```
$ python manage.py runserver 0:8000
```
`0` is a shortcut for `0.0.0.0`. Full docs for the development server can be found in the [`runserver`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)` reference.

<hr>

**Automatic reloading of [`runserver`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)**

The development server automatically reloads Python code for each request as needed. You don't need to restart the server for code changes to take effect. However, some actions like adding files don't trigger a restart, so you'll have to restart the server in these cases.

<hr>

## Creating the Polls app

Now that your environment -- a "project" -- is set up, you're set to start doing work.

Each application you write in Django consists of a Python package that follows a certain convention. Django comes with a utility that automatically generates the basic directory structure of an app, so you can focus on writing code rather than creating directories.

<hr>

**Projects vs. apps**

What's the difference between a project and an app? An app is a web application that does something -- e.g., a blog system, a database of public records, or a small poll app. A project is a collection of configuration and apps for a particular website. A project can contain multiple apps. An app can be in multiple projects.

<hr>

Your apps can live anywhere on your [Python path](https://docs.python.org/3/tutorial/modules.html#tut-searchpath). In this tutorial, we'll create our poll app in the same directory as your `manage.py` file so that it can be imported as its own top-level module, rather than a submodule of of `mysite`.

To create your app, make sure you're in the same directory as `manage.py` and type this command:
```
$ python manage.py startapp polls
```
That'll create a directory `polls`, which is laid out like this:
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
This directory structure will house the poll application.

## Write your first view

Let's write the first view. Open the file **polls/views.py** and put the following Python code in it:

`polls/views.py`

```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

This is the simplest view possible in Django. To call the view, we need to map it to a URL -- and for this, we need a URLconf.

To create a URLconf in the polls directory, create a file called `urls.py`. You app directory should now look like this:
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```
In the `polls/urls.py` file, include the following code:

`polls/urls.py`

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

The next step is to point the root URLconf at the `polls.urls` module. In `mysite/urls.py`, add an import for `django.urls.include` and insert an [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) in the `urlpatterns` list, so you have:

`mysite/urls.py`

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

The [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) function allows referencing other URLconfs. Whenever Django encounters [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

The idea behind [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) is to make it easy to plug-and-play URLs. Since polls are in their own URLconf (`polls/urls.py`), they can be placed under "/polls/", or under "/fun_polls/", or under "/content/polls/", or any other path root, and the app will still work.

<hr>

**When to use [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include)**

You should always use `include()` when you include other URL patterns. `admin.site.urls` is the only exception to this.

<hr>

You have now wired an `index` view into the URLconf. Verify it's working with the following command:
```
$ python manage.py runserver
```
Go to [http://localhost:8000/polls/](http://localhost:8000/polls/) in your browser, and you should see the text *"Hello, world. You're at the polls index."*, which you defined in the `index` view.

<hr>

**Page not found?**

If you get an error page here, check that you're going to [http://localhost:8000/polls/](http://localhost:8000/polls/) and not [http://localhost:8000/](http://localhost:8000/).

<hr>

The [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) function is passed four arguments, two required: `route` and `view`, and two optional: `kwargs` and `name`. At this point, it's worth reviewing what these arguments are for:

### [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: route

`route` is a string that contains a URL pattern. When processing a request, Django starts at the first pattern in `urlpatterns` and makes its way down the list, comparing the requested URL against each pattern until it finds one that matches.

Patterns don't search `GET` and `POST` parameters, or the domain name. For example, in a request to `https://www.example.com/myapp/`, the URLconf will look for `myapp/`. In a request to `https://www.example.com/myapp/?page=3`, the URLconf will also look for `myapp/`.

### [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: view

When Django finds a matching pattern, it calls the specified view function with an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) object as the first argument and any "captured" values from the route as keyword arguments. We'll give an example of this in a bit.

### [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: kwargs

Arbitrary keyword arguments can be passed in a dictionary to the target view. We aren't going to use this feature of Django in the tutorial.

### [`path()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.path) argument: name

Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.

When you're comfortable with the basic request and response flow, read [Part 2 of this tutorial](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2) to start working with the database.

<hr>

[[Back to the Django opening page]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django#django) - [[Top]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_1#writing-your-first-django-app---part-1) - [[Next page: Part 2]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2)