# Database access optimization

Django's database layer provides various ways to help developers get the most out of their databases. This document gathers together links to the relevant documentation, and adds various tips, organized under a number of headings that outline the steps to take when attempting to optimize your database usage.

## Profile first

As general programming practice, this goes without saying. Find out [what queries you are doing and what they are costing you](https://docs.djangoproject.com/en/4.0/faq/models/#faq-see-raw-sql-queries). Use [`QuerySet.explain()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.explain) to understand how specific `QuerySet`s are executed by your database. You may also want to use an external project like [django-debug-toolbar](https://github.com/jazzband/django-debug-toolbar/), or a tool that monitors your database directly.

Remember that you may be optimizing for speed or memory or both, depending on your requirements. Sometimes optimizing for one will be detrimental to the other, but sometimes they will help each other. Also, work that is done by the database process might not have the same cost (to you) as the same amount of work done in your Python process. It is up to you to decide what your priorities are, where the balance must lie, and profile all of these as required since this will depend on your application and server.

With everything that follows, remember to profile after every change to ensure that the change is a benefit, and a big enough benefit given the decrease in readability of your code. **All** of the suggestions below come with the caveat that in your circumstances the general principle might not apply, or might even be reversed.

## Use standard DB optimization techniques

...including:

* [Indexes](https://en.wikipedia.org/wiki/Database_index). This is a number one priority, *after* you have determined from profiling what indexes should be added. Use [`Meta.indexes`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.indexes) or [`Field.db_index`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.db_index) to add these from Django. Consider adding indexes to fields that you frequently query using [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter), [`exclude()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.exclude), [`order_by()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.order_by), etc., as indexes may help to speed up lookups. Note that determining the best indexes is a complex database-dependent topic that will depend on your particular application. The overhead of maintaining an index may outweigh any gains in query speed.

* Appropriate use of field types.

We will assume you have done the things listed above. The rest of this document focuses on how to use Django in such a way that you are not doing unnecessary work. This document also does not address other optimization techniques that apply to all expensive operations, such as [general purpose caching](https://docs.djangoproject.com/en/4.0/topics/cache/).

## Understand `QuerySet`s

Understanding [QuerySets](https://docs.djangoproject.com/en/4.0/ref/models/querysets/) is vital to getting good performance with simple code. In particular:

### Understand `QuerySet` evaluation

To avoid performance problems, it is important to understand:

* ...that [QuerySets are lazy](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#querysets-are-lazy).
* ...when [they are evaluated](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#when-querysets-are-evaluated).
* ...how [the data is held in memory](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#caching-and-querysets).

### Understand cached attributes

As well as caching of the whole `QuerySet`, there is caching of the result of attributes on ORM objects. In general, attributes that are not callable will be cached. For example, assuming the [example blog models](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#making-queries):
```
>>> entry = Entry.objects.get(id=1)
>>> entry.blog   # Blog object is retrieved at this point
>>> entry.blog   # cached version, no DB access
```
But in general, callable attributes cause DB lookups every tijme:
```
>>> entry = Entry.objects.get(id=1)
>>> entry.authors.all()   # query performed
>>> entry.authors.all()   # query performed again
```
Be careful when reading template code -- the template system does not allow use of parentheses, but will call callables automatically, hiding the above distinction.

Be careful with your own custom properties -- it is up to you to implement caching when required. For example, using the [`cached_property`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.functional.cached_property) decorator.

### Use the `with` template tag

To make use of the caching behavior of `QuerySet`, you may need to use the [`with`](https://docs.djangoproject.com/en/4.0/ref/templates/builtins/#std:templatetag-with) template tag.

### Use `iterator()`

When you have a lot of objects, the caching behavior of the `QuerySet` can cause a large amount of memory to be used. In this case, [`iterator()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.iterator) may help.

### Use `explain()`

[`QuerySet.explain()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.explain) gives you detailed information about how the database executes a query, including indexes and joins that are used. These details may help you find queries that could be rewritten more efficiently, or identify indexes that could be added to improve performance.

## Do database work in the database rather than in Python

For instance:

* At the most basic level, use [filter and exclude](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#methods-that-return-new-querysets) to do filtering in the database.
* Use [`F` expressions](https://docs.djangoproject.com/en/4.0/ref/models/expressions/#f-expressions) to filter based on other fields within the same model.
* Use [annotate to do aggregation in the database](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Aggregation#aggregation).

If these aren't enough to generate the SQL you need:

### Use `RawSQL`

A less portable but more powerful method is the [`RawSQL`](https://docs.djangoproject.com/en/4.0/ref/models/expressions/#django.db.models.expressions.RawSQL) expression, which allows some SQL to be explicitly added to the query. If that still isn't powerful enough:

### Use raw SQL

Write your own [custom SQL to retrieve data or populate models](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Raw_SQL_Queries#performing-raw-sql-queries). Use `django.db.connection.queries` to find out what Django is writing for you and start from there.

## Retrieve individual objects using a unique, indexed column

There are two reasons to use a column with [`unique`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.unique) or [`db_index`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.db_index) when using [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) to retrieve individual objects. First, the query will be quicker because of the underlying database index. Also, the query could run much slower if multiple objects match the lookup; having a unique constraint on the column guarantees this will never happen.

So using the [example blog models](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#making-queries):
```
>>> entry = Entry.objects.get(id=10)
```
...will be quicker than:
```
>>> entry = Entry.objects.get(headline="News Item Title")
```
...because `id` is indexed by the database and is guaranteed to be unique.

Doing the following is potentially quite slow:
```
>>> entry = Entry.objects.get(headline__startswith="News")
```
First of all, `headline` is not indexed, which will make the underlying database fetch slower.

Second, the lookup doesn't guarantee that only one object will be returned. If the query matches more than one object, it will retrieve and transfer all of them from the database. This penalty could be substantial if hundreds or thousands of records are returned. The penalty will be compounded if the database lives on a separate server, where network overhead and latency also play a factor.

## Retrieve everything at once if you know you will need it

Hitting the database multiple times for different parts of a single "set" of data that you will need all parts of is, in general, less efficient than retrieving it all in one query. This is particularly important if you have a query that is executed in a loop, and could therefore end up doing many database queries,l when only one was needed. So:

### Use `QuerySet.select_related()` and `prefetch_related()`

Understand [`select_related()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.select_related) and [`prefetch_related()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.prefetch_related) thoroughly, and use them:

* ...in [managers and default managers](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Managers#managers) where appropriate. Be aware when your manager is and is not used; sometimes this is tricky so don't make assumptions.
* ...in view code or other layers, possibly making use of [`prefetch_related_objects()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.prefetch_related_objects) where needed.

## Don't retrieve things you don't need 

### Use `QuerySet.values()` and `values_list()`

When you only want a `dict` or `list` of values, and don't need ORM model objects, make appropriate usage of [`values()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.values). These can be useful for replacing model objects in template code -- as long as the dicts you supply have the same attributes as those used in the template, you are fine.

### Use `QuerySet.defer()` and `only()`

Use [`defer()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.defer) and [`only()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.only) if there are database columns you know that you won't need (or won't need in most cases) to avoid loading them. Note that if you *do* use them, the ORM will have to go and get them in a separate query, making this a pessimization if you use it inappropriately.

Don't be too aggressive in deferring fields without profiling as the database has to read most of the non-text, non-VARCHAR data from the disk for a single row in the results, even if it ends up only using a few columns. The `defer()` and `only()` methods are most useful when you can avoid loading a lot of text data or for fields that might take a lot of processing to convert back to Python. As always, profile first, then optimize.

### Use `QuerySet.contains(obj)`

...if you only want to find out if `obj` is in the queryset, rather than `if obj in queryset`.

### Use `QuerySet.count()`

...if you only wnt the count, rather than doing `len(queryset)`.

### Use `QuerySet.exists()`

...if you only want to find out if, at least, one result exists, rather than `if queryset`.

But:

### Don't overuse `count()` and `exists()`

If you are going to need other data from the `QuerySet`, evaluate it immediately.

For example, assuming an Email model that has a `subject` attribute and a many-to-many relation to User, the following code is optimal:
```
if display_emails:
    emails = user.emails.all()
    if emails:
        print('You have', len(emails), 'emails:')
        for email in emails:
            print(email.subject)
    else:
        print('You do not have any emails.')
```
It is optimal because:

1. Since QuerySets are lazy, this does no database queries if `display_emails` is `False`.
2. Storing `user.emails.all()` in the `emails` variable allows its result cache to be reused.
3. The line `if emails` causes `QuerySet.__bool__()` to be called, which causes the `user.emails.all()` query to be run on the database. If there aren't any results, it will return `False`, otherwise `True`.
4. The use of `len(emails)` calls `QuerySet.__len__()`, reusing the result cache.
5. The `for` loop iterates over the already filled cache.

In total, this code does either one or zero database queries. The only deliberate optimization performed is using the `emails` variable. Using `QuerySet.exists()` for the `if` or `QuerySet.count()` for the count would each cause additional queries.