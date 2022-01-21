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

**Note**: `Manager`s are accessible only via model classes, rather than from model instances, to enforce a separation between "table-level" operations and "record-level" operations.

<hr>

The `Manager` is the main source of `QuerySet`s for a model. For example, `Blog.objects.all()` returns a `QuerySet` that contains all `Blog` objects in the database.

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

The lookup parameters (`**kwargs` in the above function definitions) should be in the format described in [Field lookups](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#field-lookups) below.

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

#### Filtered `QuerySet`s are unique

Each time you refine a `QuerySet`, you get a brand-new `QuerySet` that is in no way bound to the previous `QuerySet`. Each refinement creats a separate and distinct `QuerySet` that can be stored, used, and reused.

Example:
```
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```
These three `Queryset`s are separate. The first is a base `QuerySet` containing all entries that contain a headline starting with "What". The second is a subset of the first, with an additional criteria that excludes records whose `pub_date` is today or in the future. The third is a subset of the first, with an additional criteria that selects only the records whose `pub_date` is today or in the future. The initial `QuerySet` (`q1`) is unaffected by the refinement process.

#### `QuerySet`s are lazy

`QuerySet`s are lazy -- the act of creating a `QuerySet` doesn't involve any database activity. You can stack filters together all day long, and Django won't actually run the query until `QuerySet` is *evaluated*. Take a look at this example:
```
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```
Though this looks like three database hits, in fact it hits the database only once, at the last line (`print(q)`). In general, the results of a `QuerySet` aren't fetched from the database until you "ask" for them. When you do, the `QuerySet` is *evaluated* by accessing the database. For more details on exactly when evaluation takes place, see [When QuerySets are evaluated](). <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/ref/models/querysets/#when-querysets-are-evaluated) -->

### Retrieving a single object with `get()`

[`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) will always give you a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet), even if only a single object matches the query -- in this case, it will be a `QuerySet` containing a single elements.

If you know there is only one object that matches your query, you can use the [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) method on a [`Manager`](https://docs.djangoproject.com/en/4.0/topics/db/managers/#django.db.models.Manager) which returns the object directly:
```
>>> one_entry = Entry.objects.get(pk=1)
```
You can use any query expression with `get()`, just like with `filter()` -- again, see [Field lookups](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#field-lookups) below.

Note that there is a difference between using `get()`, and using `filter()` with a slice of `[0]`. If there are no results that match the query, `get()` will raise a `DoesNotExist` exception. This exception is an attribute of the model class that the query is being performed on -- so in the code above, if there is no `Entry` object with a primary key of 1, Django will raise `Entry.DoesNotExist`.

Similarly, Django will complain if more than one item matches the `get()` query. In this case, it will raise [`MultipleObjectsReturned`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.MultipleObjectsReturned), which again is an attribute of the model class itself.

### Other `QuerySet` methods

Most of the time, you'll use [`all()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.all), [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get), [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter), and [`exclude()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.exclude) when you need to look up objects from the database. However, that's far from all there is; see the [`QuerySet` API Reference](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#queryset-api) for a complete list of all the various `QuerySet` methods.

### Limiting `QuerySet`s

Use a subset of Python's array-slicing syntax to limit your `QuerySet` to a certain number of results. This is the equivalent of SQL's `LIMIT` and `OFFSET` clauses.

For example, this returns the first 5 objects (`LIMIT 5`):
```
>>> Entry.objects.all()[:5]
```
This returns the sixth through tenth objects (`OFFSET 5 LIMIT 5`):
```
>>> Entry.objects.all()[5:10]
```
Negative indexing (i.e. `Entry.objects.all()[-1]`) is not supported.

Generally, slicing a `QuerySet` returns a new `QuerySet` -- it doesn't evaluate the query. An exception is if you use the "step" parameter of Python slice syntax. For example, this would actually execute the query in order to return a list of every *second* object of the first 10:
```
>>> Entry.objects.all()[:10:2]
```
Further filtering or ordering of a sliced queryset is prohibited due to the ambiguous nature of how that might work.

To retrieve a *single* object rather than a list (e.g. `SELECT foo FROM bar LIMIT 1`), use an index instead of a slice. For example, this returns the first `Entry` in the database, after ordering entries alphabetically by headline:
```
>>> Entry.objects.order_by('headline')[0]
```
This is roughly equivalent to:
```
>>> Entry.objects.order_by('headline')[0:1].get()
```
Note, however, that the first of these will raise `IndexError` while the second will raise `DoesNotExist` if no objects match the given criteria. See [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) for more details.

### Field lookups

Field lookups are how you specify the meat of an SQL `WHERE` clause. They're specified as keyword arguments to the [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) methods [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter), [`exclude()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.exclude), and [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get).

Basic lookups' keyword arguments take the form `field__lookuptype=value`. (That's a double-underscore.) For example:
```
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```
...translates (roughly) into the following SQL:
```
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

<hr>

**How is this possible?**

Python has the ability to define functions that accept arbitrary name-value arguments whose names and values are evaluated at runtime. For more information, see [Keyword Arguments](https://docs.python.org/3/tutorial/controlflow.html#tut-keywordargs) in the official Python tutorial.

<hr>

The field specified in a lookup has to be the name of a model field. There's one exception, though: in case of a [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey), you can specify the field name suffixed with `_id`. In this case, the value parameter is expected to contain the raw value of the foreign model's primary key. For example:
```
>>> Entry.objects.filter(blog_id=4)
```
If you pass an invalid keyword argument, a lookup function will raise `TypeError`.

The database API supports about two dozen lookup types; a complete reference can be found in the [field lookup reference](). To give you a taste of what's available, here's some of the more common lookups you'll probably use:

**[`exact`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-exact)**

An "exact" match. For example:
```
>>> Entry.objects.get(headline__exact="Cat bites dog")
```
...would generate SQL along these lines:
```
SELECT ... WHERE headline = 'Cat bites dog';
```
If you don't provide a lookup type -- that is, if your keyword argument doesn't contain a double underscore -- the lookup type is assumed to be `exact`.

For example, the following two statements are equivalent:
```
>>> Blog.objects.get(id__exact=14)   # Explicit form
>>> Blog.objects.get(id=14)          # __exact is implied
```

This is for convenience, because `exact` lookups are the common case.

**[`iexact`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-iexact)**

A case-insensitive match. So, the query:
```
>>> Blog.objects.get(name__iexact="beatles blog")
```
...would match a `Blog` title `"Beatles Blog"`, `"beatles blog"`, or even `"BeAtlES blOG"`.

**[`contains`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-contains)**

Case-sensitive containment test. For example:
```
Entry.objects.get(headline__contains='Lennon')
```
...roughly translates to this SQL:
```
SELECT ... WHERE headline LIKE '%Lennon%';
```
Note this will match the headline `'Today Lennon honored'` but not `'today lennon honored'`.

There's also a case-insensitive version, **[`icontains`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-icontains)**.

**[`startswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-startswith)**, **[`endswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-endswith)**

Starts-with and ends-with search, respectively. There are also case-insensitive versions called **[`istartswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-istartswith)** and **[`iendswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-iendswith)**.

Again, this only scratches the surface. A complete reference can be found in the [field lookup reference](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#field-lookups).













## Deleting objects

The delete method...



### Many-to-many relationships

Both ends...