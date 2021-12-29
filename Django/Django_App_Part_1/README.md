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

From the command line, **`cd`** into a directory where you'd like to store your code, then run the following command (make sure you're in your [virtual environment in your terminal first](https://github.com/AndrewSRea/My_Learning_Port/tree/main/JavaScript/Server-Side_Website_Programming/Django_Web_Framework/Django_Development_Environment#setting-up-a-django-development-environment)):
```
$ django-admin startproject mysite
```
This will create a **mysite** directory in your current directory. If it didn't work, see [Problems running django-admin](https://docs.djangoproject.com/en/4.0/faq/troubleshooting/#troubleshooting-django-admin).

<hr>

**Note**: These command line instructions are based upon using a macOS/Linux bash terminal. If you are using Windows, I would suggest following [Django's tutorial documentation](https://docs.djangoproject.com/en/4.0/intro/tutorial01/) directly, and clicking the Windows icons above each command prompt to switch to Command Prompt commands.

<hr>

**Note**: You'll need to avoid naming projects after built-in Python or Django components. In particular, this means you should avoid using names like **django** (which will conflict with Django itself) or **test** (which conflicts with a built-in Python package).

<hr>

**Where should this code live?**

If your background is in plain old PHP (with no use of modern frameworks), you're probably used to putting code under the web server's document root (in a place such as **/var/www**). With Django, you don't do that. It's not a good idea to put any of this Python code within your web server's document root, because it risks the possibility that people may be able to view your code over the web. That's not goof for security.

Put your code in some directory **outside** of the document root, such as **/home/mycode**.

<hr>

Let's look at what **[startproject](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startproject)** created:
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

* The outer **mysite/** root directory is a container for your project. Its name doesn't matter to Django; you can rename it to anything you like.
* **manage.py**: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about **manage.py** in [django-admin and manage.py](https://docs.djangoproject.com/en/4.0/ref/django-admin/).
* The inner **mysite/** directory is the actual Python package for your project. Its name is the Python package name you'll need to use to import anything inside it (e.g. **mysite.urls**).
* **mysite/__init__.py**: An empty file that tells Python that this directory should be considered a Python package. If you're a Python beginner, read [more about packages](https://docs.python.org/3/tutorial/modules.html#tut-packages) in the official Python docs.
* **mysite/settings.py**: Settings/configuration for this Django project. [Django settings](https://docs.djangoproject.com/en/4.0/topics/settings/) will tell you all about how settings work.
* **mysite/urls.py**: The URL declarations for this Django project; a "table of contents" of your Django-powered site. You can read more about URLs in [URL dispatcher](https://docs.djangoproject.com/en/4.0/topics/http/urls/).
* **mysite/asgi.py**: An entry-point for ASGI-compatible web servers to serve your project. See [How to deploy with ASGI](https://docs.djangoproject.com/en/4.0/howto/deployment/asgi/) for more details.
* **mysite/wsgi.py**: An entry-point for WSGI-compatible web servers to serve your project. See [How to deploy with WSGI](https://docs.djangoproject.com/en/4.0/howto/deployment/wsgi/) for more details.

## The development server

Let's verify your Django project works. Change into the outer **mysite** directory, if you haven't already, and run the following commands:
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

By default, the **[runserver](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)** command starts the development server on the internal IP at port 8000.

If you want to change the server's port, pass it as a command-line argument. For instance, this command starts the server on port 8080:
```
$ python manage.py runserver 8080
```
If you want to change the server's IP, pass it along with the port. For example, to listen on all available public IPs (which is useful if you are running Vagrant or want to show off your work on other computers on the network), use:
```
$ python manage.py runserver 0:8000
```
**0** is a shortcut for **0.0.0.0**. Full docs for the development server can be found in the **[runserver](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)** reference.

<hr>

**Automatic reloading of [runserver](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-runserver)**

The development server automatically reloads Python code for each request as needed. You don't need to restart the server for code changes to take effect. However, some actions like adding files don't trigger a restart, so you'll have to restart the server in these cases.

<hr>