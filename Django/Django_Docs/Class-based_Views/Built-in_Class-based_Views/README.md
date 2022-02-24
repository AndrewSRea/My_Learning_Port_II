# Built-in class-based generic views

Writing web applications can be monotonous, because we repeat certain patterns again and again. Django tries to take away some of that monotony at the model and template layers, but web developers also experience this boredom at the view level.

Django's *generic views* were developed to ease that pain. They take certain common idioms and patterns found in view development and abstract them so that you can quickly write common views of data without having to write too much code.

We can recognize certain common tasks, like displaying a list of objects, and write code that displays a list of *any* object. Then the model in question can be passed as an extra argument to the URLconf.

Django ships with generic views to do the following:

* Display list and detail pages for a single object. If we were creating an application to manage conferences, then a `TalkListView` and a `RegisteredUserListView` would be examples of list views. A single talk page is an example of what we call a "detail" view.
* Present date-based objects in year/month/day archive pages, associated detail, and "latest" pages.
* Allow users to create, update, and delete objects -- with or without authorization.

Taken together, these views provide interfaces to perform the most common tasks developers encounter.

## Extending generic views

There's no question that using generic views can speed up development substantially. In most projects, however, there comes a moment when the generic views no longer suffice. Indeed, the most common question asked by new Django developers is how to make generic views handle a wider array of situations.

This is one of the reasons generic views were redesigned for the 1.3 release -- previously, they were view functions with a bewildering array of options; now, rather than passing in a large amount of configuration in the URLconf, the recommended way to extend generic views is to subclass them, and override their attributes or methods.

That said, generic views will have a limit. If you find you're struggling to implement your view as a subclass of a generic view, then you may find it more effective to write just the code you need, using your own class-based or functional views.

More examples of generic views are available in some third party applications, or you could write your own as needed.

## Generic views of objects

[`TemplateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.TemplateView) certainly is useful, but Django's generic views really shine when it comes to presenting views of your database content. Because it's such a common task, Django comes with a handful of built-in generic views to help generate list and detail views of objects.

Let's start by looking at some examples of showing a list of objects or an individual object.

We'll be using these models:
```
# models.py
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    class Meta:
        ordering = ["-name"]

    def __str__(self):
        return self.name

class Author(models.Model):
    salutation = models.CharField(max_length=10)
    name = models.CharField(max_length=200)
    email = models.EmailField()
    headshot = models.ImageField(upload_to='author_headshots')

    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField('Author')
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    publication_date = models.DateField()
```
Now we need to define a view:
```
# views.py
from django.views.generic import ListView
from books.models import Publisher

class PublisherListView(ListView):
    model = Publisher
```
Finally, hook that view into your urls:
```
# urls.py
from django.urls import path
from books.view import PublisherListView

urlpatterns = [
    path('publishers/', PublisherListView.as_view()),
]
```
That's all the Python code we need to write. We still need to write a template, however. We could explicitly tell the view which template to use by adding a `template_name` attribute to the view, but in the absence of an explicit template Django will infer one from the object's name. In this case, the inferred template will be `"books/publisher_list.html"` -- the "books" part comes from the name of the app that defines the model, while the "publisher" bit is the lowercased version of the model's name.

<hr>

**Note**: Thus, when (for example) the `APP_DIRS` option of a `DjangoTemplates` backend is set to `True` in [`TEMPLATES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TEMPLATES), a template location could be: `/path/to/project/books/templates/books/publisher_list.html`.

<hr>

This template will be rendered against a context containing a variable called `object_list` that contains all the publisher objects. A template might look like this:
```
{% extends "base.html" %}

{% block content %}
    <h2>Publishers</h2>
    <ul>
        {% for publisher in object_list %}
            <li>{{ publisher.name }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```
That's really all there is to it. All the cool features of generic views come from changing the attributes set on the generic view. The [generic views reference](https://docs.djangoproject.com/en/4.0/ref/class-based-views/) documents all the generic views and their options in detail; the rest of this document will consider some of the common ways you might customize and extend generic views.