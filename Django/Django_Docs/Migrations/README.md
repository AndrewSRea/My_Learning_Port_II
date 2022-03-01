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

This process generally works well, but it can be slow and occassionally buggy. It is not recommended that you run and migrate SQLite in a production environment unless you are very aware of the risks and its limitations; the support Django ships with is designed to allow developers to use SQLite on their local machines to develop less complex Django projects without the need for a full database.

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