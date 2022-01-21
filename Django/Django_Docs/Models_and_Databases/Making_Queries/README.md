# Making queries

Once you've created your [data models](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Models#models), Django automatically gives you a database-abstraction API that lets you create, retrieve, update, and delete objects. This document explains how to use this API. Refer to the [data model reference](https://docs.djangoproject.com/en/4.0/ref/models/) for full details of all the various model lookup options.

Throughout this guide (and in the reference), we'll refer to the following models, which comprise a blog application:
```
from datetime import date 

from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField(default=date.today)
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField(default=0)
    number_of_pingbacks = models.IntegerField(default=0)
    rating = models.IntegerField(default=5)

    def __str__(self):
        return self.headline
```

## Creating objects

To represent database-table data in Python objects, Django uses an intuitive system: A model class represents a data table, and an instance of that class represents a particular record in the database table. 

To create an object, instantiate it using keyword arguments to the model class, then call [`save()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.save) to save it to the database.

Assuming models live in a file `mysite/blog/models.py`, here's an example:
```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b = save()
```
This performs an `INSERT` SQL statement behind the scenes. Django doesn't hit the database until you explicitly call `save()`.

The `save()` method has no return value.

<hr>

**See also**

`save()` takes a number of advanced options not described here. See the documentation for [`save()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.save) for complete details.

To create and save an object in a single step, use the [`create()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.create) method.

<hr>

## Saving changes to objects

To save changes to an object that's already in the database, use `save()`.

Given a `Blog` instance `b5` that has already been saved to the database, this example changes its name and updates its record in the database.
```
>>> b5.name = 'New name'
>>> b5.save()
```
This performs an `UPDATE` SQL statement behind the scenes. Django doesn't hit the database until you explicitly call `save()`.

### Saving `ForeignKey` and `ManyToManyField` fields

Updating a [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey) field works works exactly the same way as saving a normal field -- assign an object of the right type to the field in question. This example updates the `Blog` attribute of an `Entry` instance `entry`, assuming appropriate instances of `Entry` and `Blog` are already saved to the database (so we can retrieve them below).
```
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```
Updating a [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField) works a little differently -- use the [`add()`](https://docs.djangoproject.com/en/4.0/ref/models/relations/#django.db.models.fields.related.RelatedManager.add) method on the field to add a record to the relation. This example adds the `Author` instance `joe` to the `entry` object:
```
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```
To add multiple records to a `ManyToManyField` in one go, include multiple arguments in the call to `add()`, like this:
```
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```
Django will complain if you try to assign or add an object of the wrong type.

## Retrieving objects

To retrieve objects from your database, construct a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) via a [`Manager`]() on your model class. <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/db/managers/#django.db.models.Manager) -->

A `QuerySet` represents a collection of objects from your database. It can have zero, one, or many *filters*. Filters narrow down the query results based on the given parameters. In SQL terms, a `QuerySet` equates a `SELECT` statement, and a filter is a limiting clause such as `WHERE` or `LIMIT`.

You get a `QuerySet` by using your model's `Manager`. Each model has at least one `Manager`, and it's called [`objects`]() by default. Access it directly via the model class, like so: <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/ref/models/class/#django.db.models.Model.objects) -->
```
>>> Blog.objects
<django.db.models.manager.Manager object at... >
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

<hr>

**Note**: `Managers` are accessible only via model classes, rather than from model instances, to enforce a separation between "table-level" operations and "record-level" operations.

<hr>

The `Manager` is the main source of `QuerySets` for a model. For example, `Blog.objects.all()` returns a `QuerySet` that contains all `Blog` objects in the database.

### Retrieving all objects

The simplest way to retrieve objects from a table is to get all of them. To do this, use the [`all()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.all) method on a `Manager`:
```
>>> all_entries = Entry.objects.all()
```
The `all()` method returns a `QuerySet` of all the objects in the database.

### Retrieving specific objects with filters

The `QuerySet` returned by `all()` describes all objects in the database table. Usually, though, you'll need to select only a subset of the complete set of objects. 

To create such a subset, you refine the initial `QuerySet`, adding filter conditions. The two most common ways to refine a `QuerySet` are:

**`filter(\*\*kwargs)`**

Returns a new `QuerySet` containing objects that match the given lookup parameters.

**`exclude(\*\*kwargs)`**

Returns a new `QuerySet` containing objects that do *not* match the given lookup parameters.

The lookup parameters (`**kwargs` in the above function definitions) should be in the format described in [Field lookups]() below.

For example, to get a `QuerySet` of blog entries from the year 2006, use [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) like so:
```
Entry.objects.filter(pub_date__year=2006)
```
With the default manager class, it is the same as:
```
Entry.objects.all().filter(pub_date__year=2006)
```

#### Chaining filters

The result of refining a `QuerySet` is itself a `QuerySet`, so it's possible to chanin refinements together. For example:
```
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```
This takes the initial `QuerySet` of all entries in the database, adds a filter, then an exclusion, then another filter. The final result is a `QuerySet` containing all entries with a headline that starts with "What", that were published between January 20, 2005, and the current day.
















## Deleting objects

The delete method...



### Many-to-many relationships

Both ends...