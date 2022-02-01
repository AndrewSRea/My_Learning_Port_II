# Multiple databases

This topic guide describes Django's support for interacting with multiple databases. Most of the rest of Django's documentation assumes you are interacting with a single database. If you want to interact with multiple databases, you'll need to take some additional steps.

<hr>

**See also**

See [Multi-database support]() <!-- possible future folder? (https://docs.djangoproject.com/en/4.0/topics/testing/tools/#testing-multi-db) --> for information about testing with multiple databases.

<hr>

## Defining your databases

The first step to using more than one database with Django is to tell Django about the database servers you'll be using. This is done using the [`DATABASES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) setting. This setting maps database aliases, which are a way to refer to a specific database throughout Django, to a dictionary of settings for that specific connection. The settings in the inner dictionaries are described fully in the `DATABASES` documentation.

Databases can have any alias you choose. However, the alias `default` has special significance. Django uses the database with the alias of `default` when no other database has been selected.

The following is an example `settings.py` snippet defining two databases -- a default PostgreSQL database and a MySQL database called `users`:
```
DATABASES = {
    'default': {
        'NAME': 'app_data',
        'ENGINE': 'django.db.backends.postgresql',
        'USER': 'postgres_user',
        'PASSWORD': 's3krit'
    },
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'priv4te'
    }
}
```
If the concept of a `default` database doesn't make sense in the context of your project, you need to be careful to always specify the database that you want to use. Django requires that a `default` database entry be defined, but the parameters dictionary can be left blank if it will not be used. To do this, you must set up [`DATABASE_ROUTERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASE_ROUTERS) for all of your apps' models, including those in any contrib and third-party apps you're using, so that no queries are routed to the default database. The following is an example `settings.py` snippet defining two non-default databases, with the `default` entry intentionally left empty:
```
DATABASES = {
    'default': {},
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'superS3cret'
    },
    'customers': {
        'NAME': 'customer_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_cust',
        'PASSWORD': 'veryPriv@te'
    }
}
```
If you attempt to access a database that you haven't defined in your [`DATABASES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) setting, Django will raise a `django.utils.connection.ConnectionDoesNotExist` exception.

## Synchronizing your databases

The [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) management command operates on one database at a time. By default, it operates on the `default` database, but by providing the [`--database`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-migrate-database) option, you can tell it to synchronize a different database. So, to synchronize all models onto all databases in the first example above, you would need to call:
```
$ ./manage.py migrate
$ ./manage.py migrate --database=users
```
If you don't want every application to be synchronized onto a particular database, you can deinfe a [database router]() <!-- below --> that implements a policy constraining the availability of particular models.

If, as in the second example above, you've left the `default` database empty, you must provide a database name each time you run `migrate`. Omitting the database name would raise an error. For the second example:
```
$ ./manage.py migrate --database=users
$ ./manage.py migrate --database=customers
```

### Using other management commands

Most other `django-admin` commands that interact with the database operate in the same way as [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) -- they only ever operate on one database at a time, using [`--database`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-migrate-database) to control the database used.

An exception to this rule is the [`makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations) command. It validates the migration history in the databases to catch problems with the existing migration files (which could be caused by editing them) before creating new migrations. By default, it checks only the `default` database, but it consults the [`allow_migrate()`]() <!-- below --> method of [routers]() if any are installed.

## Automatic database routing

The easiest way to use multiple databases is to set up a database routing scheme. The default routing scheme ensures that objects remain "sticky" to their original database (i.e., an object retrieved from the `foo` database will be saved on the same database). The default routing scheme ensures that if a database isn't specified, all queries fall back to the `default` database.

You don't have to do anything to activate the default routing scheme -- it is provided "out of the box" on every Django project. However, if you want to implement more interesting database allocation behaviors, you can define and install your own database routers.

### Database routers

A database Router is a class that provides up to four methods:

##### `db_for_read(model, **hints)`

Suggest the database that should be used for read operations for objects of type `Model`.

If a database operation is able to provide any additional information that might assist in selecting a database, it will be provided in the `hints` dictionary. Details on valid hints are provided [below](). <!-- below "Hints" -->

Returns `None` if there is no suggestion.

##### `db_for_write(model, **hints)`

Suggest the database that should be used for writes of objects of type `Model`.

If a database operation is able to provide any additional information that might assist in selecting a database, it will be provided in the `hints` dictionary. Details on valid hints are provided [below](). <!-- "Hints" -->

Returns `None` if there is no suggestion.

##### `allow_relation(obj1, obj2, **hints)`

Returns `True` if a relation between `obj1` and `obj2` should be allowed, `False` if the relation should be prevented, or `None` if the router has no opinion. This is purely a validation operation, used by foreign key and many to many operations to determine if a relation should be allowed between two objects.

If no router has an opinion (i.e. all routers return `None`), only relations within the same database are allowed.

##### `allow_migrate(db, app_label, model_name=None, **hints)`

Determine if the migration operation is allowed to run on the database with alias `db`. Return `True` if the operation should run, `False` if it shouldn't run, or `None` if the router has no opinion.

The `app_label` positional argument is the label of the application being migrated.

`model_name` is set by most migration operations to the value of `model._meta.model_name` (the lowercased version of the model `__name__`) of the model being migrated. Its value is `None` for the [`RunPython`](https://docs.djangoproject.com/en/4.0/ref/migration-operations/#django.db.migrations.operations.RunPython) and [`RunSQL`](https://docs.djangoproject.com/en/4.0/ref/migration-operations/#django.db.migrations.operations.RunSQL) operations unless they provide it using hints.

`hints` are used by certain operations to communicate additional information to the router.