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