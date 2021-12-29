# Writing your first Django app - Part 2

This tutorial begins where [Tutorial 1](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_1#writing-your-first-django-app---part-1) left off. We'll set up the database, create your first model, and get a quick introduction to Django's automatically-generated admin site.

<hr>

**Where to get help**: If you're having trouble going through this tutorial, please head over to the [Getting help](https://docs.djangoproject.com/en/4.0/faq/help/) section of the FAQ.

<hr>

## Database setup

Now, open up **mysite/settings.py**. It's a normal Python module with module-level variables representing Django settings.

By default, the configuration uses SQLite. If you're new to databases, or you're just interested in trying Django, this is the easiest choice. SQLite is included in Python, so you won't need to install anything else to support your database. When starting your first project, however, you may want to use a more scalable database like PostgreSQL, to avoid database-switching headaches down the road.

If you wish to use another database, install the appropriate [database bindings](https://docs.djangoproject.com/en/4.0/topics/install/#database-installation) and change the following keys in the **[DATABASES](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES) 'default'** item to match your database connection settings:

* **[ENGINE](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASE-ENGINE)** - Either **'django.db.backends.sqlite3'**, **'django.db.backends.postgresql'**, **'django.db.backends.mysql'**, or **'django.db.backends.oracle'**. Other backends are [also available](https://docs.djangoproject.com/en/4.0/ref/databases/#third-party-notes).
* **[NAME](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-NAME)** - The name of your database. If you're using SQLite, the database will be a file on your computer; in that case, **[NAME](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-NAME)** should be the full absolute path, including filename, of that file. The default value, **BASE_DIR / 'db.sqlite3'**, will store the file in your project directory.

If you are not using SQLite as your database, additional settings such as **[USER](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USER)**, **[PASSWORD](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD)**, and **[HOST](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-HOST)** must be added. For more details, see the reference documentation for **[DATABASES](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DATABASES)**.

<hr>

**For database other than SQLite**

If you're using a database besides SQLite, make sure you've created a database by this point. Do that with **"CREATE DATABASE database_name;"** within your database's interactive prompt.

Also, make sure that the database user provided in **mysite/settings.py** has "create database" privileges. This allows automatic creation of a [test database](https://docs.djangoproject.com/en/4.0/topics/testing/overview/#the-test-database) which will be needed in a later tutorial.

If you're using SQLite, you don't need to create anything beforehand -- the database file will be created automatically when it is needed.

<hr>

While you're editing **mysite/settings.py**, set **[TIME_ZONE](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TIME_ZONE)** to your time zone.

Also, note the **[INSTALLED_APPS](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS)** setting at the top of the file. That holds the names of all Django applications that are activated in this Django instance. Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.