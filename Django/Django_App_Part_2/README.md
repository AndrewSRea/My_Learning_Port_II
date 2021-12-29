# Writing your first Django app - Part 2

This tutorial begins where [Tutorial 1](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_1#writing-your-first-django-app---part-1) left off. We'll set up the database, create your first model, and get a quick introduction to Django's automatically-generated admin site.

<hr>

**Where to get help**: If you're having trouble going through this tutorial, please head over to the [Getting help](https://docs.djangoproject.com/en/4.0/faq/help/) section of the FAQ.

<hr>

## Database setup

Now, open up `mysite/settings.py`. It's a normal Python module with module-level variables representing Django settings.

By default, the configuration uses SQLite. If you're new to databases, or you're just interested in trying Django, this is the easiest choice. SQLite is included in Python, so you won't need to install anything else to support your database. When starting your first project, however, you may want to use a more scalable database like PostgreSQL, to avoid database-switching headaches down the road.

If you wish to use another database, install the appropriate [database bindings](https://docs.djangoproject.com/en/4.0/topics/install/#database-installation) and change the following keys in the [`DATABASES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) `'default'` item to match your database connection settings:

* [`ENGINE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASE-ENGINE) - Either `'django.db.backends.sqlite3`, `'django.db.backends.postgresql'`, `'django.db.backends.mysql'`, or `'django.db.backends.oracle'`. Other backends are [also available](https://docs.djangoproject.com/en/4.0/ref/databases/#third-party-notes).
* [`NAME`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-NAME) - The name of your database. If you're using SQLite, the database will be a file on your computer; in that case, [`NAME`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-NAME) should be the full absolute path, including filename, of that file. The default value, `BASE_DIR / 'db.sqlite3'`, will store the file in your project directory.

If you are not using SQLite as your database, additional settings such as [`USER`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USER), [`PASSWORD`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD), and [`HOST`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-HOST) must be added. For more details, see the reference documentation for [`DATABASES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES).

<hr>

**For database other than SQLite**

If you're using a database besides SQLite, make sure you've created a database by this point. Do that with `"CREATE DATABASE database_name;"` within your database's interactive prompt.

Also, make sure that the database user provided in `mysite/settings.py` has "create database" privileges. This allows automatic creation of a [test database](https://docs.djangoproject.com/en/4.0/topics/testing/overview/#the-test-database) which will be needed in a later tutorial.

If you're using SQLite, you don't need to create anything beforehand -- the database file will be created automatically when it is needed.

<hr>

While you're editing `mysite/settings.py`, set [`TIME_ZONE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TIME_ZONE) to your time zone.

Also, note the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) setting at the top of the file. That holds the names of all Django applications that are activated in this Django instance. Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.

By default, [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) contains the following apps, all of which come with Django:

* [`django.contrib.admin`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#module-django.contrib.admin) - The admin site. You'll use it shortly.
* [`django.contrib.auth`](https://docs.djangoproject.com/en/4.0/topics/auth/#module-django.contrib.auth) - An authentication system.
* [`django.contrib.contenttypes`](https://docs.djangoproject.com/en/4.0/ref/contrib/contenttypes/#module-django.contrib.contenttypes) - A framework for content types.
* [`django.contrib.sessions`](https://docs.djangoproject.com/en/4.0/topics/http/sessions/#module-django.contrib.sessions) - A session framework.
* [`django.contrib.messages`](https://docs.djangoproject.com/en/4.0/ref/contrib/messages/#module-django.contrib.messages) - A messaging framework.
* [`django.contrib.staticfiles`](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles) - A framework for managing static files.

These applications are included by default as a convenience for the common case.

Some of these applications make use of, at least, one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:
```
$ python manage.py migrate
```
The [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) command looks at the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) setting and creates any necessary database tables according to the database settings in your `mysite/settings.py` file and the database migrations shipped with the app (we'll cover those later). You'll see a message for each migration it applies. If you're interested, run the command-line client for your database and type: `\dt` (PostgreSQL), `SHOW TABLES;` (MariaDB, MySQL), `.tables` (SQLite), or `SELECT TABLE_NAME FROM USER_TABLES;` (Oracle) to display the tables Django created.

<hr>

**For the minimalists**

Like we said above, the default applications are included for the common case, but not everybody needs them. If you don't need any or all of them, feel free to comment-out or delete the appropriate line(s) from [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) before running [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate). The [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate) command will only run migrations for apps in [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS).

<hr>

## Creating models

Now we'll define your models -- essentially, your database layout, with additional metadata.

<hr>

**Philosophy**

A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you're storing. Django follows the [DRY Principle](https://docs.djangoproject.com/en/4.0/misc/design-philosophies/#dry). The goal is to define your data model in one place and automatically derive things from it.

This includes the migrations -- unlike in Ruby On Rails, for example, migrations are entirely derived from your models file, and are essentially a history that Django can roll through to update your database schema to match your current models.

<hr>

In our poll app, we'll create two models: `Question` and `Choice`. A `Question` has a question and a publication date. A `Choice` has two fields: the text of the choice and a vote tally. Each `Choice` is associateds with a `Question`.

These concepts are represented by Python classes. Edit the `polls/models.py` file so it looks like this:

`polls/models.py`

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Here, each model is represented by a class that subclasses [`django.db.models.Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model). Each model has a number of class variables, each of which represents a database field in the model.

Each field is represented by an instance of a [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) class -- e.g., [`CharField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField) for character fields, and [`DateTimeField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DateTimeField) for datetimes. This tells Django what type of data each field holds.

The name of each [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) instance (e.g. `question_text` or `pub_date`) is the field's name, in machine-friendly format. You'll use this value in your Python code, and your database will use it as the column name.

You can use an optional first positional argument to a [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) to designate a human-readable name. That's used in a couple of introspective parts of Django, and it doubles as documentation. If this field isn't provided, Django will use...