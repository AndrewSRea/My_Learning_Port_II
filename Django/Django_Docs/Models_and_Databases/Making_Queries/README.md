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

### Lookups that span relationships

Django offers a powerful and intuitive way to "follow" relationships in lookups, taking care of the SQL `JOIN`s for you automatically, behind the scenes. To span a relationship, use the field name of related fields across models, separated by double underscores, until you get to the field you want.

This example retrieves all `Entry` objects with a `Blog` whose `name` is `'Beatles Blog'`:
```
>>> Entry.objects.filter(blog__name='Beatles Blog')
```
This spanning can be as deep as you'd like.

It works backwards, too. While it [can be customized](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey.related_query_name), by default you refer to a "reverse" relationship in a lookup using the lowercase name of the model.

This example retrieves all `Blog` objects which have at least one `Entry` whose `headline` contains `'Lennon'`:
```
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```
If you are filtering across multiple relationships and one of the intermediate models doesn't have a value that meets the filter condition, Django will treat it as if there is an empty (all values are `NULL`), but valid, object there. All this means is that no error will be raised. For example, in this filter:
```
Blog.objects.filter(entry__authors__name='Lennon')
```
...(if there was a related `Author` model), if there was no `author` associated with an entry, it would be treated as if there was also no `name` attached, rather than raising an error because of the missing `author`. Usually this is exactly what you want to have happen. The only case where it might be confusing is if you were using [`isnull`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-isnull). Thus:
```
Blog.objects.filter(entry__authors__name__isnull=True)
```
...will return `Blog` objects that have an empty `name` on the `author` and also those which have an empty `author` on the `entry`. If you don't want those latter objects, you could write:
```
Blog.objects.filter(entry__authors__isnull=False,
entry__authors__name__isnull=True)
```

#### Spanning multi-valued relationships

When spanning to a [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField) or a reverse [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey) (such as from `Blog` to `Entry`), filtering on multiple attributes raises the question of whether to require each attribute to coincide in the same related object. We might seek blogs that have an entry from 2008 with *"Lennon"* in its headline, or we might seek blogs that merely have any entry from 2008 as well as some newer or older entry with *"Lennon"* in its headline.

To select all blogs containing at least one entry from 2008 having *"Lennon"* in its headline (the same entry satisfying both conditions), we would write:
```
Blog.objects.filter(entry__headline__contains='Lennon',
entry__pub_date__year=2008)
```
Otherwise, to perform a more permissive query selecting any blogs with merely *some* entry with *"Lennon"* in its headline and *some* entry from 2008, we would write:
```
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```
Suppose there is only one blog that has both entries containing *"Lennon"* and entries from 2008, but that none of the entries from 2008 contained *"Lennon"*. The first query would not return any blogs, but the second query would return that one blog. (This is because the entries selected by the second filter may or may not be the same as the entries in the first filter. We are filtering the `Blog` items with each filter statement, not the `Entry` items.) In short, if each condition needs to match the same related object, then each should be contained in a single [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) call.

<hr>

**Note**: As the second (more permissive) query chains multiple filters, it performs multiple joins to the primary model, potentially yielding duplicates.
```
>>> from datetime import date
>>> beatles = Blog.objects.create(name='Beatles Blog')
>>> pop = Blog.objects.create(name='Pop Music Blog')
>>> Entry.objects.create(
...     blog=beatles,
...     headline='New Lennon Biography',
...     pub_date=date(2008, 6, 1),
... )
<Entry: New Lennon Biography>
>>> Entry.objects.create(
...     blog=beatles,
...     headline='New Lennon Biography in Paperback',
...     pub_date=date(2009, 6, 1),
... )
<Entry: New Lennon Biography in Paperback>
>>> Entry.objects.create(
...     blog=pop,
...     headline='Best Albums of 2008',
...     pub_date=date(2008, 12, 15),
... )
<Entry: Best Albums of 2008>
>>> Entry.objects.create(
...     blog=pop,
...     headline='Lennon Would Have Loved Hip Hop',
...     pub_date=date(2020, 4, 1),
... )
<Entry: Lennon Would Have Loved Hip Hop>
>>> Blog.objects.filter(
...     entry__headline__contains='Lennon',
...     entry__pub_date__year=2008,
... )
<QuerySet [<Blog: Beatles Blog>]>
>>> Blog.objects.filter(
...     entry__headline__contains='Lennon',
... ).filter(
...     entry__pub_date__year=2008,
... )
<QuerySet [<Blog: Beatles Blog>, <Blog: Beatles Blog>, <Blog: Pop Music Blog>]>
```

<hr>

**Note**: The behavior of [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) for queries that span multi-value relationships, as described above, is not implemented equivalently for [`exclude()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.exclude). Instead, the conditions in a single `exclude()` call will not necessarily refer to the same item.

For example, the following query qould exclude blogs that contain *both* entries with *"Lennon"* in the headline *and* entries published in 2008:
```
Blog.objects.exclude(
    entry__headline__contains='Lennon',
    entry__pub_date__year=2008,
)
```
However, unlike the behavior when using `filter()`, this will not limit blogs based on entries that satisfy both conditions. In order to do that, i.e. to select all blogs that do not contain entries published with *"Lennon"* that were published in 2008, you need to make two queries:
```
blog.objects.exclude(
    entry__in=Entry.objects.filter(
        headline__contains='Lennon',
        pub_date__year=2008,
    ),
)
```

### Filters can reference fields on the model

In the examples given so far, we have constructed filters that compare the value of a model field with a constant. But what if you want to compare the value of a model field with another field on the same model?

Django provides [`F` expressions](https://docs.djangoproject.com/en/4.0/ref/models/expressions/#django.db.models.F) to allow such comparisons. Instances of `F()` act as a reference to a model field within a query. These references can then be used in query filters to compare the values of two different fields on the same model instance.

For example, to find a list of all blog entries that have had more comments than pingbacks, we construct an `F()` object to reference the pingback count, and use that `F()` object in the query:
```
>>> from django.db.models import F
>>> Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks'))
```
Django supports the use of addition, subtraction, multiplication, division, modulo, and power arithmetic with `F()` objects, both with constants and with other `F()` objects. To find all the blog entries with more than *twice* as many comments as pingbacks, we modify the query:
```
>>> Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks') * 2)
```
To find all the entries where the rating of the entry is less than the sum of the pingback count and comment count, we would issue the query:
```
>>> Entry.objects.filter(rating__lt=F('number_of_comments') + F('number_of_pingbacks'))
```
You can also use the double underscore notation to span relationships in an `F()` object. An `F()` object with a double underscore will introduce any joins needed to access the related object. For example, to retrieve all the entries where the author's name is the same as the blog name, we could issue the query:
```
>>> Entry.objects.filter(authors__name=F('blog__name'))
```
For date and date/time fields, you can add or subtract a [`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta) object. The following would return all entries that were modified more than 3 days after they were published:
```
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```
The `F()` objects support bitwise operations by `.bitand()`, `.bitor()`, `.bitxor()`, `.bitrightshift()`, and `.bitleftshift()`. For example:
```
>>> F('somefield').bitand(16)
```

<hr>

**Oracle**

Oracle doesn't support bitwise XOR operation.

<hr>

### Expressions can reference transforms

Django supports using transforms in expressions.

For example, to find all `Entry` objects published in the same year as they were last modified:
```
>>> Entry.objects.filter(pub_date__year=F('mod_date__year'))
```
To find the earliest year an entry was published, we can issue the query:
```
>>> Entry.objects.aggregate(first_published_yearMin('pub_date__year'))
```
This example finds the value of the highest rated entry and the total number of comments on all entries for each year:
```
>>> Entry.objects.values('pub_date__year').annotate(
...     top_rating=Subquery(
...         Entry.objects.filter(
...             pub_date__year=OuterRef('pub_date__year'),
...         ).order_by('-rating').values('rating')[:1]
...     ),
...     total_comments=Sum('number_of_comments'),
... )
```

### The `pk` lookup shortcut

For convenience, Django provides a `pk` lookup shortcut, which stands for "primary key".

In the example `Blog` model, the primary key is the `id` field, so these three statements are equivalent:
```
>>> Blog.objects.get(id__exact=14)   # Explicit form
>>> Blog.objects.get(id=14)          # __exact is implied
>>> Blog.objects.get(pk=14)          # pk implies id__exact
```
The use of `pk` isn't limited to `__exact` queries -- any query term can be combined with `pk` to perform a query on the primary key of a model:
```
# Get blogs entries with id 1, 4, and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```
`pk` lookups also work across joins. For example, these three statements are equivalent:
```
>>> Entry.objects.filter(blog__id__exact=3)   # Explicit form
>>> Entry.objects.filter(blog__id=3)          # __exact is implied
>>> Entry.objects.filter(blog__pk=3)          # __pk implies __id__exact
```

### Escaping percent signs and underscores in `LIKE` statements

The field lookups that equate to `LIKE` SQL statements (`iexact`, `contains`, `icontains`, `startswith`, `istartswith`, `endswith`, and `iendswith`) will automatically escape the two special characters used in `LIKE` statements -- the percent sign and the underscore. (In a `LIKE` statement, the percent sign signifies a multiple-character wildcard and the underscore signifies a single-character wildcard.)

This means things should work intuitively, so the abstraction doesn't leak. For example, to retrieve all the entries that contain a percent sign, use the percent sign as any other character:
```
>>> Entry.objects.filter(headline__contains='%')
```
Django takes care of the quoting for you; the resulting SQL will look something like this:
```
SELECT ... WHERE headline LIKE '%\%%';
```
Same goes for underscores. Both percentage signs and underscores are handled for you transparently.

### Caching and `QuerySet`s

Each [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) contains a cache to minimize database access. Understanding how it works will allow you to write the most efficient code.

In a newly created `QuerySet`, the cache is empty. The first time a `QuerySet` is evaluated -- and, hence, a database query happens -- Django saves the query results in the `QuerySet`'s cache and returns the results that have been explicitly requested (e.g. the next element, if the `QuerySet` is being iterated over). Subsequent evaluations of the `QuerySet` reuse the cached results.

Keep this caching behavior in mind, because it may bite you if you don't use your `QuerySet`s correctly. For example, the following will create two `QuerySet`s, evaluate them, and throw them away:
```
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```
That means the same database query will be executed twice, effectively doubling your database load. Also, there's a possibility the two lists may not include the same database records, because an `Entry` may have been added or deleted in the split second between the two requests.

To avoid this problem, save the `QuerySet` and reuse it:
```
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset])   # Evaluate the query set.
>>> print([p.pub_date for p in queryset])   # Reuse the cache from the evaluation.
```

### When `QuerySet`s are not cached

Querysets do not always cache their results. When evaluating only *part* of the queryset, the cache is checked, but if it is not populated, then the items returned by the subsequent query are not cached. Specifically, this means that [limiting the queryset](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#limiting-querysets) using an array slice or an index will not populate the cache.

For example, repeatedly getting a certain index in a queryset object will query the database each time:
```
>>> queryset = Entry.objects.all()
>>> print(queryset[5])   # Queries the database
>>> print(queryset[5])   # Queries the database again
```
However, if the entire queryset has already been evaluated, the cache will be checked instead:
```
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset]   # Queries the database
>>> print(queryset[5])   # Uses cache
>>> print(queryset[5])   # Uses cache
```
Here are some examples of other actions that will result in the entire queryset being evaluated and therefore populate the cache:
```
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

<hr>

**Note**: Simply printing the queryset will not populate the cache. This is because the call to `__repr__()` only returns a slice of the entire queryset.

<hr>

## Querying `JSONField`

Lookups implementation is different in [`JSONField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.JSONField), mainly due to the existence of key transformations. To demonstrate, we will use the following example model:
```
from django.db import models

class Dog(models.Model):
    name = models = models.CharField(max_length=200)
    date = models.JSONField(null=True)

    def __str__(self):
    return self.name
```

### Storing and querying for `None`

As with other fields, storing `None` as the field's value will store it as SQL `NULL`. While not recommended, it is possible to store JSON scalar `null` instead of SQL `NULL` by using [`Value('null')`](https://docs.djangoproject.com/en/4.0/ref/models/expressions/#django.db.models.Value).

Whichever of the values is stored, when retrieved from the database, the Python representation of the JSON scalar `null` is the same as SQL `NULL`, i.e. `None`. Therefore, it can be hard to distinguish between them. 

This only applies to `None` as the top-level value of the field. If `None` is inside a [`list`](https://docs.python.org/3/library/stdtypes.html#list) or [`dict`](https://docs.python.org/3/library/stdtypes.html#dict), it will always be interpreted as JSON `null`.

When querying, `None` value will always be interpreted as JSON `null`. To query for SQL `NULL`, use [`isnull`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-isnull):
```
>>> Dog.objects.create(name='Max', data=None)   # SQL NULL
<Dog: Max>
>>> Dog.objects.create(name='Archie', data=Value('null'))   # JSON null
<Dog: Archie>
>>> Dog.objects.filter(data=None)
<QuerySet [<Dog: Archie>]>
>>> Dog.objects.filter(data=Value('null'))
<QuerySet [<Dog: Archie>]>
>>> Dog.objects.filter(date__isnull=True)
<QuerySet [<Dog: Max>]>
>>> Dog.objects.filter(data__isnull=False)
<QuerySet [<Dog: Archie>]>
```
Unless you are sure you wish to work with SQL `NULL` values, consider setting `null=False` and providing a suitable default for empty values, such as `default=dict`.

<hr>

**Note**: Storing JSON scalar `null` does not violate [`null=False`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.null).

<hr>

### Key, index, and path transforms

To query based on a given dictionary key, use that key as the lookup name:
```
>>> Dog.objects.create(name='Rufus', data={
...     'breed': 'labrador',
...     'owner': {
...         'name': 'Bob',
...         'other_pets': [{
...             'name': 'Fishy',
...          }],
...     },
... })
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': None})
<Dog: Meg>
>>> Dog.objects.filter(data__breed='collie')
<QuerySet [<Dog: Meg>]>
```
Multiple keys can be chained together to form a path lookup:
```
>>> Dog.objects.filter(data__owner__name='Bob')
<QuerySet [<Dog: Rufus>]>
```
If the key is an integer, it will be interpreted as an index transform in an array:
```
>>> Dog.objects.filter(data__owner__other_pets__0__name='Fishy')
<QuerySet [<Dog: Rufus>]>
```
If the key you wish to query by clashes with the name of another lookup, use the [`contains`](https://docs.djangoproject.com/en/4.0/topics/db/queries/#std:fieldlookup-jsonfield.contains) lookup instead.

To query for missing keys, use the `isnull` lookup:
```
>>> Dog.objects.create(name='Shep', data={'breed': 'collie'})
<Dog: Shep>
>>> Dog.objects.filter(data__owner__isnull=True)
<QuerySet [<Dog: Shep>]>
```

<hr>

**Note**: The lookup examples given above implicitly use the [`exact`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-exact) lookup. Key, index, and path transforms can also be chained with: [`icontains`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-icontains), [`endswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-endswith), [`iendswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-iendswith), [`iexact`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-iexact), [`regex`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-regex), [`iregex`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-iregex), [`startswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-startswith), [`istartswith`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-istartswith), [`lt`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-lt), [`lte`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-lte), [`gt`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-gt), and [`gte`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-gte), as well as with [Containment and key lookups](). <!-- below -->

<hr>

**Note**: Due to the way in which key-path queries work, [`exclude()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.exclude) and [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) are not guaranteed to produce exhaustive sets. If you want to include objects that do not have the path, add the `isnull` lookup.

<hr>

:warning: **Warning**: Since any string could be a key in a JSON object, any lookup other than those listed below will be interpreted as a key lookup. No errors are raised. Be extra careful for typing mistakes, and always check your queries work as you intend.

<hr>

**MariaDB and Oracle users**

Using [`order_by()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.order_by) on key, index, or path transforms will sort the objects using the string representation of the values. This is because MariaDB and Oracle Database do not provide a function that converts JSON values into their equivalent SQL values.

<hr>

**Oracle users**

On Oracle Database, using `None` as the lookup value in an `exclude()` query will return objects that do not have `null` as the value at the given path, including objects that do not have the path. On other database backends, the query will return objects that have the path and the value is not `null`.

<hr>

**PostgreSQL users**

On PostgreSQL, if only one key or index is used, the SQL operator `->` is used. If multiple operators are used, then the `#>` operator is used.

<hr>

### Containment and key lookups













## Deleting objects

The delete method...



### Many-to-many relationships

Both ends...