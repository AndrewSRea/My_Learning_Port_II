# Models

A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you're storing. Generally, each model maps to a single database table.

The basics:

* Each model is a Python class that subclasses [`django.db.models.Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model).
* Each attribute of the model represents a database field.
* With all of this, Django gives you an automatically-generated database-access API; see [Making queries](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#making-queries).

## Quick example

This example model defiones a `Person`, which has a `first_name` and `last_name`:
```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```
`first_name` and `last_name` are [fields]() <!-- below --> of the model. Each field is specified as a class attribute, and each attribute maps to a database column.

The above `Person` model would create a database table like this:
```
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```
Some technical notes:

* The name of the table, `myapp_person`, is automatically derived from some model metadata but can be overridden. See [Table names]() for more details. <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/ref/models/options/#table-names) -->
* An `id` field is added automatically, but this behavior can be overridden. See [Automatic primary key fields](). <!-- below -->
* The `CREATE TABLE` SQL in this example is formatted using PostgreSQL syntax, but it's worth noting Django uses SQL tailored to the database backend specified in your [settings file](). <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/topics/settings/) -->

## Using models

Once you have defined your models, you need to tell Django you're going to *use* those models. Do this by editing your settings file and changing the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) setting to add the name of the module that contains your `models.py`.

For example, if the models for your application live in the module `myapp.models` (the package structure that is created for an application by the [`manage.py startapp`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startapp) script), [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) should read, in part:
<!-- not sure if `manage.py startapp` will be a future folder or not -->
```
INSTALLED_APPS = [
    # ...
    'myapp',
    # ...
]
```
When you add new apps to [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS), be sure to run [`manage.py migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate), optionally making migrations for them first with [`manage.py makemigrations`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemigrations). <!-- again, not sure if `manage.py migrate / makemigrations` will be a future folder -->

## Fields

The most important part of a model -- and the only required part of a model -- is the list of database fields it defines. Fields are specified by class attributes. Be careful not to choose fields names that conflict with the [models API](https://docs.djangoproject.com/en/4.0/ref/models/instances/) like `clean`, `save`, or `delete`. <!-- "models API" a future folder? -->

Example:
```
from django.db import models

class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

class Album(models.Model):
    artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```

### Field types 

Each field in your model should be an instance of the appropriate [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) class. Django uses the field class types to determine a few things:

* The column type, which tells the database what kind of data to store (e.g. `INTEGER`, `VARCHAR`, `TEXT`).
* The default HTML [widget]() to use when rendering a form field (e.g. `<input type="text">`, `<select>`). <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/ref/forms/widgets/) -->
* The minimal validation requirements, used in Django's admin and in automatically-generated forms.

Django ships with dozens of built-in field types; you can find the complete list in the [model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#model-field-types). You can easily write your own fields if Django's built-in ones don't do the trick; see [How to create custom model fields](). <!-- link to internal file? Or to DjangoProject? (https://docs.djangoproject.com/en/4.0/howto/custom-model-fields/) -->

### Field options

Each field takes a certain set of field-specific arguments (documented in the [model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#model-field-types)). For example, [`CharField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField) (and its subclasses) require a [`max_length`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField.max_length) argument which specifies the size of the `VARCHAR` database field used to store the data.

There's also a set of common arguments available to all field types: All are optional. They're fully explained in the [reference](), but here's a quick summary of the most often-used ones:

**[`null`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.null)**

If `True`, Django will store empty values as `NULL` in the database. Default is `False`.

**[`blank`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.blank)**

If `True`, the field is allowed to be blank. Default is `False`.

Note that this is different than `null`. `null` is purely database-related, whereas `blank` is validation-related. If a field has `blank=True`, form validation will allow entry of an empty value. If a field has `blank=False`, the field will be required.

**[`choices`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.choices)**

A [sequence](https://docs.python.org/3/glossary.html#term-sequence) of 2-tuples to use as choices for this field. If this is given, the default form widget will be a select box instead of the standard text field and will limit choices to the choices given.

A `choices` list looks like this:
```
YEAR_IN_SCHOOL_CHOICES = [
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
]
```

<hr>

**Note**: A new migration is created each time the order of `choices` changes.

<hr>

The first elements in each tuple is the value that will be stored in the database. The second element is displayed by the field's form widget.

Given a model instance, the display value for a field with `choices` can be accessed using the [`get_FOO_display()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.get_FOO_display) method. for example:
```
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```
```
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```
You can also use enumeration classes to define `choices` in a concise way:
```
from django.db import models

class Runner(models.Model):
    