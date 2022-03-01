# Migrations

Migrations are Django's way of propagating changes you make to your models (adding a field, deleting a model, etc.) into your database schema. They're designed to be mostly automatic, but you'll need to know when to make migrations, when to run them, and the common problems you might run into.

## The commands

There are several commands which you will use to interact with migrations and Django's handling of database schema:

* [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate), which is responsible for applying and unapplying migrations.
* [`makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations), which is responsible for creating new migrations based on the changes you have made to your models.
* [`sqlmigrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-sqlmigrate), which displays the SQL statements for a migration.
* [`showmigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-showmigrations), which lists a project's migrations and their status.

You should think of migrations as a version control system for your database schema. `makemigrations` is responsible for packaging up your model changes into individual migration files -- analogous to commits -- and `migrate` is responsible for applying those to your database.

The migration files for each app live in a "migrations" directory inside of that app, and are designed to be committed to, and distributed as part of, its codebase. You should be making them once on your development machine and then running the same migrations on your colleagues' machines, your staging machines, and eventually your production machines.

<hr>

**Note**: It is possible to override the name of the package which contains the migrations on a per-app basis by modifying the [`MIGRATION_MODULES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIGRATION_MODULES) setting.

<hr>

Migrations will run the same way on the same dataset and produce consistent results, meaning that what you see in development and staging is, under the same circumstances, exactly what will happen in production.

Django will make migrations for any change to your models or fields -- even options that don't affect the database -- as the only way it can reconstruct a field correctly is to have all the changes in the history, and you might need those options in some data migrations later on (for example, if you've set custom validators).

## Backend support

Migrations are supported on all backends that Django ships with, as well as any third-party backends if they have programmed in support for schema alteration (done via the [`SchemaEditor`](https://docs.djangoproject.com/en/4.0/ref/schema-editor/) class).

However, some databases are more capable than others when it comes to schema migrations; some of the caveats are covered below.

### PostgreSQL

PostgreSQL is the most capable of all the databases here in terms of schema support.

The only caveat is that prior to PostgreSQL 11, adding columns with default values causes a full rewrite of the table, for a time proportional to its size. For this reason, it's recommended you always create new columns with `null-True`, as this way they will be added immediately.

### MySQL

MySQL lacks support for transactions around schema alteration operations, meaning that if a migration fails to apply, you will have to manually unpick the changes in order to try again (it's impossible to roll back to an earlier point).

In addition, MySQL will fully rewrite tables for almost every schema operation and generally takes a time proportional to the number of rows in the table to add or remove columns. On slower hardware, this can be worse than a minute per million rows -- adding a few columns to a table with just a few million rows could lock your site up for over ten minutes.

Finally, MySQL has relatively small limits on name lengths for columns, tables, and indexes, as well as a limit on the combined size of all columns an index covers. This means that indexes that are possible on other backends will fail to be created under MySQL.

### SQLite

SQLite has very little built-in schema alteration support, and so Django attempts to emulate it by:

* Creating a new table with the new schema.
* Copying the data across.
* Dropping the old table.
* Renaming the new table to match the original name.

This process generally works well, but it can be slow and occasionally buggy. It is not recommended that you run and migrate SQLite in a production environment unless you are very aware of the risks and its limitations; the support Django ships with is designed to allow developers to use SQLite on their local machines to develop less complex Django projects without the need for a full database.

## Workflow

Django can create migrations for you. Make changes to your models -- say, add a field and remove a model -- and then run [`makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations):
```
$ python manage.py makemigrations
Migrations for 'books':
    books/migrations/0003_auto.py:
        - Alter field author on book
```
Your models will be scanned and compared to the versions currently contained in your migration files, and then a new set of migrations will be written out. Make sure to read the output to see what `makemigrations` thinks you have changed -- it's not perfect, and for complex changes it might not be detecting what you expect.

Once you have your new migration files, you should apply them to your database to make sure they work as expected:
```
$ python manage.py migrate
Operations to perform:
    Apply all migrations: books
Running migrations:
    Rendering model states... DONE
    Applying books.0003_auto... OK
```
Once the migration is applied, commit the migration and the models change to your version control system as a single commit -- that way, when other developers (or your production servers) check out the code, they'll get both the changes to your models and the accompanying migration at the same time.

If you want to give the migration(s) a meaningful name instead of a generated one, you can use the [`makemigrations --name`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-makemigrations-name) option:
```
$ python manage.py makemigrations --name changed_my_model your_app_label
```

### Version control

Because migrations are stored in version control, you'll occasionally come across situations where you and another developer have both committed a migration to the same app at the same time, resulting in two migrations with the same number.

Don't worry -- the numbers are just there for developers' reference, Django just cares that each migration has a different name. Migrations specify which other migrations they depend on -- including earlier migrations in the same app -- in the file, so it's possible to detect when there's two new migrations for the same app that aren't ordered.

When this happens, Django will prompt you and give you some options. If it thinks it's safe enough, it will offer to automatically linearize the two migrations for you. If not, you'll have to go in and modify the migrations yourself -- don't worry, this isn't difficult, and is explained more in [Migration files]() below.

## Transactions

On databases that support DDL transactions (SQLite and PostgreSQL), all migration operations will run inside a single transaction by default. In contrast, if a database doesn't support DDL transactions (e.g. MySQL, Oracle), then all operations will run without a transaction.

You can prevent a migration from running in a transaction by setting the `atomic` attribute to `False`. For example:
```
from django.db import migrations

class Migration(migrations.Migration):
    atomic = False
```
It's also possible to execute parts of the migration inside a transaction using [`atomic()`](https://docs.djangoproject.com/en/4.0/topics/db/transactions/#django.db.transaction.atomic) or by passing `atomic=True` to [`RunPython`](https://docs.djangoproject.com/en/4.0/ref/migration-operations/#django.db.migrations.operations.RunPython). See [Non-atomic migrations](https://docs.djangoproject.com/en/4.0/howto/writing-migrations/#non-atomic-migrations) for more details.

## Dependencies

While migrations are per-app, the tables and relationships implied by your models are too complex to be created for one app at a time. When you make a migration that requires something else to run -- for example, you add a `ForeignKey` in your `books` app to your `authors` app -- the resulting migration will contain a dependency on a migration in `authors`.

This means that when you run the migrations, the `authors` migration runs first and creates the table the `ForeignKey` references, and then the migration that makes the `ForeignKey` column runs afterward and creates the constraint. If this didn't happen, the migration would try to create the `ForeignKey` column without the table it's referencing existing and your database would throw an error.

This dependency behavior afects most migration operations where you restrict to a single app. Restricting to a single app (either in `makemigrations` or `migrate`) is a best-efforts promise, and not a guarantee; any other apps that need to be used to get dependencies correct will be.

Apps without migrations must not have relations (`ForeignKey`, `ManyToManyField`, etc.) to apps with migrations. Sometimes it may work, but it's not supported.

## Migration files

Migrations are stored as an on-disk format, referred to here as "migration files". These files are actually normal Python files with an agreed-upon object layout, written in a declarative style.

A basic migration file looks like this:
```
from django.db import migrations, models

class Migration(migrations.Migration):

    dependencies = [('migrations', '0001_initial')]

    operations = [
        migrations.DeleteModel('Tribble'),
        migrations.AddField('Author', 'rating', models.IntegerField(default=0)),
    ]
```
What Django looks for when it loads a migration file (as a Python module) is a subclass of `django.db.migrations.Migration` called `Migration`. It then inspects this object for four attributes, only two of which are used most of the time:

* `dependencies`, a list of migrations this one depends on.
* `operations`, a list of `Operation` classes that define what this migration does.

The operations are the key; they are a set of declarative instructions which tell Django what schema changes need to be made. Django scans them and builds an in-memory representation of all of the schema changes to all apps, and uses this to generate the SQL which makes the schema changes.

That in-memory structure is also used to work out what the differences are between your models and the current state of your migrations; Django runs through all the changes, in order, on an in-memory set of models to come up with the state of your models last time you ran `makemigrations`. It then uses these models to compare against the ones in your `models.py` files to work out what you have changed.

You should rarely, if ever, need to edit migration files by hand, but it's entirely possible to write them manually if you need to. Some of the more complex operations are not autodetectable and are only available via a hand-written migration, so don't be scared about editing them if you have to.

### Custom fields

You can't modify the number of positional arguments in an already migrated custom field without raising a `TypeError`. The old migration will call the modified `__init__` method with the old signature. So if you need a new argument, please create a keyword argument and add something like `assert 'argument_name' in kwargs` in the constructor.

### Model managers

You can optionally serialize managers into migrations and have them available in [`RunPython`](https://docs.djangoproject.com/en/4.0/ref/migration-operations/#django.db.migrations.operations.RunPython) operations. This is done by defining a `use_in_migrations` attribute on the manager class:
```
class MyManager(models.Manager):
    use_in_migrations = True

class MyModel(models.Model):
    objects = MyManager()
```
If you are using the [`from_queryset()`](https://docs.djangoproject.com/en/4.0/topics/db/managers/#django.db.models.from_queryset) function to dynamically generate a manager class, you need to inherit from the generated class to make it importable:
```
class MyManager(MyBaseManager.from_queryset(CustomQuerySet)):
    use_in_migrations = True

class MyModel(models.Model):
    objects = MyManager()
```
Please refer to the notes about [Historical models]() <!-- below --> in migrations to see the implications that come along.

### Initial migrations

##### `Migration.initial`

The "initial migrations" for an app are the migrations that create the first version of that app's tables. Usually an app will have one initial migration, but in some cases of complex model interdependencies it may have two or more.

Initial migrations are marked with an `initial = True` class attribute on the migration class. If an `initial` class attribute isn't found, a migration will be considered "initial" if it is the first migration in the app (i.e. if it has no dependencies on any other migration in the same app).

When the [`migrate --fake-initial`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-migrate-fake-initial) option is used, these initial migrations are treated specially. For an initial migration that creates one or more tables (`CreateModel` operation), Django checks that all of those tables already exist in the database and fake-applies the migration if so. Similarly, for an initial migration that adds one or more fields (`AddField` operation), Django checks that all of the respective columns already exist in the database and fake-applies the migration if so. Without `--fake-initial`, initial migrations are treated no differently from any other migration.

### History consistency

As previously discussed, you may need to linearize migrations manually when two development branches are joined. While editing migration dependencies, you can inadvertently create an inconsistent history state where a migration has been applied but some of its dependencies haven't. This is a strong indication that the dependencies are incorrect, so Django will refuse to run migrations or make new migrations until it's fixed. When using multiple databases, you can use the [`allow_migrate()`](https://docs.djangoproject.com/en/4.0/topics/db/multi-db/#allow_migrate) method of [database routers](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Multiple_Databases#automatic-database-routing) to control which databases [`makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations) checks for consistent history.

## Adding migrations to apps

New apps come preconfigured to accept migrations, and so you can add migrations by running `makemigrations` once you've made some changes.

If your app already has models and database tables, and doesn't have migrations yet (for example, you created it against a previous Django version), you'll need to convert it to use migrations by running:
```
$ python manage.py makemigrations your_app_label
```
This will make a new initial migration for your app. Now, run `python manage.py migrate --fake-initial`, and Django will detect that you have an initial migration *and* that the tables it wants to create already exist, and will mark the migration as already applied. (Without the [`migrate -- fake-initial`)(https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-migrate-fake-initial) flag, the command would error out because the tables it wants to create already exist.)

Note that this only works given two things:

* You have not changed your models since you made their tables. For migrations to work, you must make the initial migration *first* and then make changes, as Django compares changes against migration files, not the database.
* You have not manually edited your database -- Django won't be able to detect that your database doesn't match your models, you'll just get errors when migrations try to modify those tables.

## Reversing migrations

Migrations can be reversed with [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) by passing the number of the previous migration. For example, to reverse migration `books.0003`:
```
$ python manage.py migrate books 0002
Operations to perform:
    Target specific migration: 0002_auto, from books
Running migrations:
    Rendering model states... DONE
    Unapplying books.0003_auto... OK
```
If you want to reverse all migrations applied for an app, use the name `zero`:
```
$ python manage.py migrate books zero
Operations to perform:
    Unapply all migrations: books
Running migrations:
    Rendering model states... DONE
    Unapplying books.0002_auto... OK
    Unapplying books.0001_initial... OK
```
A migration is irreversible if it contains any irreversible operations. Attempting to reverse such migrations will raise `IrreversibleError`:
```
$ python manage.py migrate books 0002
Operations to perform:
    Target specific migration: 0002_auto, from books
Running migrations:
    Rendering model states... DONE
    Unapplying books.0003_auto...Traceback (most recent call last):
django.db.migrations.exception.IrreversibleError: Operation <RunSQL  sql='DROP
TABLE demo_books'> in books.0003_auto is not reversible
```

## Historical models

When you run migrations, Django is working from historical versions of your models stored in the migration files. If you write Python code using the [`RunPython`](https://docs.djangoproject.com/en/4.0/ref/migration-operations/#django.db.migrations.operations.RunPython) operation, or if you have `allow_migrate` methods on your database routers, you **need to use** these historical model versions rather than importing them directly.

<hr>

:warning: **Warning**: If you import models directly rather than using the historical models, your migrations *may work initially* but will fail in the future when you try to rerun old migrations (commonly, when you set up a new installation and run through all the migrations to set up the database).

This means that historical model problems may not be immediately obvious. If you run into this kind of failure, it's OK to edit the migration to use the historical models rather than direct imports and commit those changes.

<hr>

Because it's impossible to serialize arbitrary Python code, these historic models will not have any custom methods that you have defined. They will, however, have the same fields, relationships, managers (limited to those with `use_in_migrations = True`) and `Meta` options (also versioned, so they may be different from your current ones).

<hr>

:warning: **Warning**: This means that you will NOT have custom `save()` methods called on objects when you access them in migrations, and you will NOT have any custom constructors or instance methods. Plan appropriately!

<hr>

