# Performing raw SQL queries

Django gives you two ways of performing raw SQL queries: you can use [`Manager.raw()`]() <!-- below --> to [perform raw queries and return model instances](), <!-- below --> or you can avoid the model layer entirely and [execute custom SQL directly](). <!-- below -->

<hr>

**Explore the ORM before using raw SQL!**

The Django ORM provides many tools to express queries without writing raw SQL. For example:

* The [QuerySet API](https://docs.djangoproject.com/en/4.0/ref/models/querysets/) is extensive.
* You can [`annotate`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.annotate) and [aggregate](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Aggregation#aggregation) using many built-in [database functions](https://docs.djangoproject.com/en/4.0/ref/models/database-functions/). Beyond those, you can create [custom query expressions](https://docs.djangoproject.com/en/4.0/ref/models/expressions/).

Before using raw SQL, explore [the ORM](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases#models-and-databases). Ask on one of [the support channels](https://docs.djangoproject.com/en/4.0/faq/help/) to see if the ORM supports your use case.

<hr>

:warning: **Warning**

You should be very careful whenever you write raw SQL. Every time you use it, you should properly escape any parameters that the user can control by using `params` in order to protect against SQL injection attacks. Please read more about [SQL injection protection](https://docs.djangoproject.com/en/4.0/topics/security/#sql-injection-protection).

<hr>

## Performing raw queries

The `raw()` manager method can be used to perform raw SQL queries that return model instances.

##### `Manager.raw(raw_query, params=(), translations=None)`

This method takes a raw SQL query, executes it, and returns a `django.db.models.query.RawQuerySet` instance. This `RawQuerySet` instance can be iterated over like a normal [`QuerySet`]() to provide object instances.

This is best illustrated with an example. Suppose you have the following model:
```
class Person(models.Model):
    first_name = models.CharField(...)
    last_name = models.CharField(...)
    birth_date = models.DateField(...
```
You could then execute custom SQL like so:
```
>>> for p in Person.objects.raw('SELECT * FROM myapp_person'):
...     print(p)
John Smith
Jane Jones
```
This example isn't very exciting -- it's exactly the same as running `Person.objects.all()`. However, `raw()` has a bunch of other options that make it very powerful.

<hr>

**Model table names**

Where did the name of the `Person` table come from in that example?

By default, Django figures out a database table name by joining the model's "app label" -- the name you used in `manage.py startapp` -- to the model's class name, with an underscore between them. In the example, we've assumed that the `Person` model lives in an app named `myapp`, so its table would be `myapp_person`.

For more details, check out the documentation for the [`db_table`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.db_table) option, which also lets you manually set the database table name.

<hr>

:warning: **Warning**

No checking is done on the SQL statement that is passed in to `.raw()`. Django expects that the statement will return a set of rows from the database, but does nothing to enforce that. If the query does not return rows, a (possibly cryptic) error will result.

<hr>

:warning: **Warning**

If you are performing queries on MySQL, note that MySQL's silent type coercion may cause unexpected results when mixing types. If you query on a string type column, but with an integer value, MySQL will coerce the types of all values in the table to an integer before performing the comparison. For example, if your table contains the value `'abc'`, `'def'`, and you query for `WHERE mycolumn=0`, both rows will match. To prevent this, perform the correct typecasting before using the value in a query.

<hr>