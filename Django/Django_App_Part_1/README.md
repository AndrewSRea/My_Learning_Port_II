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

<blockquote>

**Where to get help:** If you're having trouble going through this tutorial, please head over to the [Getting Help](https://docs.djangoproject.com/en/4.0/faq/help/) section of the FAQ.

</blockquote>

## Creating a project

If this is your first time using Django, you'll have to take care of some initial setup. Namely, you'll need to auto-generate some code that establishes a Django [project](https://docs.djangoproject.com/en/4.0/glossary/#term-project) -- a collection of settings for an instance of Django, including database configuration, Django-specific options and application-specific settings.

From the command line, **`cd`** into a directory where you'd like to store your code, then run the following command:
```
$ django-admin startproject mysite
```
This will create a **mysite** directory in your current directory. If it didn't work, see [Problems running django-admin](https://docs.djangoproject.com/en/4.0/faq/troubleshooting/#troubleshooting-django-admin).

<blockquote>

**Note**: These command line instructions are based upon using a macOS/Linux bash terminal. If you are using Windows, I would suggest following [Django's tutorial documentation](https://docs.djangoproject.com/en/4.0/intro/tutorial01/) directly, and clicking the Windows icons above each command prompt to switch to Command Prompt commands.

</blockquote>