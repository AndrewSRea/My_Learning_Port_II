# Advanced tutorial: How to write reusable apps

This advanced tutorial begins where [Tutorial 7](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_7#writing-your-first-django-app---part-7) left off. We'll be turning our Web-poll into a standalone Python package you can reuse in new projects and share with other people.

If you haven't recently completed Tutorials 1-7, we encourage you to review these so that your example project matches the one described below.

## Reusability matters

It's a lot of work to design, build, test, and maintain a web application. Many Python and Django projects share common problems. Wouldn't it be great if we could save some of this repeated work?

Reusability is the way of life in Python. [The Python Package Index (PyPI)](https://pypi.org/) has a vast range of packages you can use in your own Python programs. Check out [Django Packages](https://djangopackages.org/) for existing reusable apps you could incorporate in your project. Django itself is also a normal Python package. This means that you can take existing Python packages or Django apps and compose them into your own web project. You only need to write the parts that make your project unique.

Let's say you were starting a new project that needed a polls app like the one we've been working on. How do you make this app reusable? Luckily, you're well on the way already. In [Tutorial 1](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_1#write-your-first-view), we saw how we could decouple polls from the project-level URLconf using an `include`. In this tutorial, we'll take further steps to make the app easy to use in new projects and ready to publish for others to install and use.

<hr>

**Package? App?**

A Python [package](https://docs.python.org/3/glossary.html#term-package) provides a way of grouping related Python code for easy reuse. A package contains one or more files of Python code (also known as "modules").

A package can be imported with `import foo.bar` or `from foo import bar`. For a directory (like `polls`) to form a package, it must contain a special file `__init__.py`, even if this file is empty.

A Django *application* is a Python package that is specifically intended for use in a Django project. An application may use common Django conventions, such as having `models`, `test`, `urls`, and `views` submodules.

Later on we use the term *packaging* to describe the process of making a Python package easy for others to install. It can be a little confusing, we know.

<hr>

## Your project and your reusable app

After the previous tutorials, our project should look like this:
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html
```
