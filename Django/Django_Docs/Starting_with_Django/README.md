# Django at a glance

Because Django was developed in a fast-paced newsroom environment, it was designed to make common web development tasks fast and easy. Here's an informal overview of how to write a database-driven web app with Django.

The goal of this document is to give you enough technical specifics to understand how Django works, but this isn't intended to be a tutorial or reference -- but we've got both! When you're ready to start a project, you can [start with the tutorial](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Tutorial/Django_App_Part_1#writing-your-first-django-app---part-1) or [dive right into more detailed documentation]().

## Design your model

Although you can use Django without a database, it comes with an [object-relational mapper](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) in which you describe your database layout in Python code.

The [data-model syntax]() <!-- link to internal folder? Or to DjangoProjects? (https://docs.djangoproject.com/en/4.0/topics/db/models/) --> offers many rich ways of representing your models -- so far, it's been solving many years' worth of database-schema problems. Here's a quick example (within a `models.py` folder):
```
from django.db import models

class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

    def __str__(self):
        return self.headline
```

## Install it

Next, run the Django command-line utilities to create the database tables automatically:
```
$ python manage.py makemigrations
$ python manage.py migrate
```
The [`makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations) command looks at all your available models and creates migrations for whichever tables don't already exist. [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) runs the migrations and creates tables in your database, as well as optionally providing [much richer schema control](). <!-- link to internal folder? Or to DjangoProjects? (https://docs.djangoproject.com/en/4.0/topics/migrations/) -->

## Enjoy the free API

With that, you've got a free, and rich, [Python API]() to access your data. The API is created on the fly, no code generation necessary.

(The code displayed below is being input into a Terminal using a [Python shell](https://docs.djangoproject.com/en/4.0/ref/django-admin/#shell). Terminal commands in a Python shell are preceeded by `>>>`, and the output from each command is listed right below each command.)
```
# Import the models we created from our "news" app
>>> from news.models import Article, Reporter

# No reporters are in the system yet.
>>> Reporter.objects.all()
<QuerySet []>

# Create a new Reporter.
>>> r = Reporter(full_name='John Smith')

# Save the object into the database. You have to call save() explicitly.
>>> r.save()

# Now it has an ID.
>>> r.id
1

# Now the new reporter is in the database.
>>> Reporter.objects.all()
<QuerySet [<Reporter: John Smith>]>

# Fields are represented as attributes on the Python object.
>>> r.full_name
'John Smith'

# Django provides a rich database lookup API.
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith='John')
<Reporter: John Smith>
>>> Reporter.objects.get(full_name___contains='mith')
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.

# Create an article.
>>> from datetime import date
>>> a = Article(pub_date=date.today(), headline='Django is cool'
...     content='Yeah.', reporter=r)
>>> a.save()

# Now the article is in the database.
>>> Article.objects.all()
<QuerySet [<Article: Django is cool>]>

# Article objects get API access to related Reporter objects.
>>> r = a.reporter
>>> r.full_name
'John Smith'

# And vice versa: Reporter objects get API access to Article objects.
>>> r.article_set.all()
<QuerySet [<Article: Django is cool>]>

# The API follows relationships as far as you need, performing efficient
# JOINs for you behind the scenes.
# This finds all articles by a reporter whose name starts with "John".
>>> Article.objects.filter(reporter__full_name__startswith='John')
<QuerySet [<Article: Django is cool>]>

# Change an object by altering its attributes and calling save().
>>> r.full_name = 'Billy Goat'
>>> r.save()

# Delete an object with delete().
>>> r.delete()
```
