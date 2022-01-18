# Quick install guide

Before you can use Django, you'll need to get it installed. We have a [complete installation guide]() <!-- link to internal folder? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/install/) --> that covers all the possibilities; this guide will guide you to a minimal installation that'll work while you walk through the introduction.

## Install Python

Being a Python web framework, Django requires Python. See [What Python version can I use with Django?](https://docs.djangoproject.com/en/4.0/faq/install/#faq-python-version-support) for details. Python includes a lightweight database called [SQLite](https://www.sqlite.org/index.html) so you won't need to set up a database just yet.

Get the latest version of Python at [https://www.python.org/downloads/](https://www.python.org/downloads/) or with your operating system's package manager.

You can verify that Python is installed by typing `python` from your shell; you should see something like:
```
Python 3.x.y
[GCC 4.x] on Linux
Type "help", "copyright", "credits" or "license" for more information.
```

## Set up a database

This step is only necessary if you'd like to work with a "large" database engine like PostgreSQL, MariaDB, MySQL, or Oracle. To install such a database, consult the [database installation information](). <!-- link to internal folder? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/install/#database-installation) -->

## Install Django

You've got three options to install Django:

* [Install an official release](). This is the best approach for most users. <!-- link to internal folder? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/install/#installing-official-release) -->
* Install a version of Django [provided by your operating system distribution](). <!-- link to internal folder? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/install/#installing-distribution-package) -->
* [Install the latest development version](). This option is for enthusiasts who want the latest-and-greatest features and aren't afraid of running brand new code. You might encounter new bugs in the development version, but reporting them helps the development of Django. Also, releases of third-party packages are less likely to be compatible with the development version than with the latest stable release. <!-- link to internal folder? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/install/#installing-development-version) -->

<hr>

:attention: **Always refer to the documentation that corresponds to the version of Django you're using!**

If you do either of the first two steps, keep an eye out for parts of the documentation marked **new in development version**. That phrase flags features that are only available in development versions of Django, and they likely won't work with an official release.

<hr>

## Verifying

To verify that Django can be seen by Python, type `python` from your shell. Then at the Python prompt, try to import Django:
```
>>> import Django
>> print(django.get_version())
4.0
```
You may have another version of Django installed.

## That's it!

That's it -- you can now [move onto the tutorial]().

<hr>

[[Previous page]]() - [[Top]]() - [[Next page]]()