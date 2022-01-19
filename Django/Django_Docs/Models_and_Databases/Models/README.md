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
`first_name` and `last_name` are [fields](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Models#fields) of the model. Each field is specified as a class attribute, and each attribute maps to a database column.

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
    MedalType = models.TextChoices('MedalType', 'GOLD SILVER BRONZE')
    name = models.CharField(max_length=60)
    medal = models.CharField(blank=True, choices=MedalType.choices, max_length=10)
```
Further examples are available in the [model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#field-choices).

**[`default`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.default)**

The default value for the field. This can be a value or a callable object. If callable, it will be called every time a new object is created.

**[`help_text`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.help_text)**

Extra "help" text to be displayed with the form widget. It's useful for documentation even if your field isn't used on a form.

**[`primary_key`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.primary_key)**

If `True`, this field is the primary key for the model.

If you don't specify `primary_key=True` for any fields in your model, Django will automatically add an [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.IntegerField) to hold the primary key, so you don't need to set `primary_key=True` on any of your fields unless you want to override the default primary-key behavior. For more, see [Automatic primary key fields](). <!-- below -->

The primary key field is read-only. If you change the value of the primary key on an existing object and then save it, a new object will be created alongside the old one. For example:
```
from django.db import models

class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)
```
```
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
<QuerySet ['Apple', 'Pear']>
```

**[`unique`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.unique)**

If `True`, this field must be unique throughout the table.

Again, these are just short descriptions of the most common field options. Full details can be found in the [common model field option reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#common-model-field-options).

### Automatic primary key fields

By default, Django gives each model an auto-incrementing primary key with the type specified per app in [`AppConfig.default_auto_field](https://docs.djangoproject.com/en/4.0/ref/applications/#django.apps.AppConfig.default_auto_field) or globally in the [`DEFAULT_AUTO_FIELD`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DEFAULT_AUTO_FIELD) setting. For example:
```
id = models.BigAutoField(primary_key=True)
```
If you'd like to specify a custom primary key, specify `primary_key=True` on one of your fields. If Django sees you've explicitly set [`Field.primary_key`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.primary_key), it won't add the automatic `id` column.

Each model requires exactly one field to have `primary_key=True` (either explicitly declared or automatically added).

### Verbose field names

Each field type, except for [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey), [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField), and [`OneToOneField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.OneToOneField), takes an optional first positional argument -- a verbose name. If the verbose name isn't given, Django will automatically create it using the field's attribute name, converting underscores to spaces.

In this example, the verbose name is `"person's first name"`:
```
first_name = models.CharField("person's first name", max_length=30)
```
In this example, the verbose name is `"first name"`:
```
first_name = models.CharField(max_length=30)
```
`ForeignKey`, `ManyToManyField`, and `OneToOneField` require the first argument to be a model class, so use the [`verbose_name`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.verbose_name) keyword argument:
```
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
)
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
)
```
The convention is not to capitalize the first letter of the `verbose_name`. Django will automatically capitalize the first letter where it needs to.

### Relationships

Clearly, the power of relational databases lies in relating tables to each other. Django offers ways to define the three most common types of database relationships: many-to-one, many-to-many, and one-to-one.

#### Many-to-one relationships

To define a many-to-one relationship, use [`django.db.models.ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey). You use it just like any other [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) type: by including it as a class attribute of your model.

`ForeignKey` requires a positional argument: the class to which the model is related.

For example, if a `Car` model has a `Manufacturer` -- that is, a `Manufacturer` makes multiple cars but each `Car` only has one `Manufacturer` -- use the following definitions:
```
from django.db import models

class Manufacturer(models.Model):
    # ...
    pass

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
    # ...
```
You can also create [recursive relationships](https://docs.djangoproject.com/en/4.0/ref/models/fields/#recursive-relationships) (an object with a many-to-one relationship to itself) and [relationships to models not yet defined](https://docs.djangoproject.com/en/4.0/ref/models/fields/#lazy-relationships); see [the model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#ref-foreignkey) for details.

It's suggested, but not required, that the anme of a `ForeignKey` field (`manufacturer` in the example above) be the name of the model, lowercase. You can call the field whatever you want. For example:
```
class Car(models.Model):
    company_that_makes_it = models.ForeignKey(
        Manufacturer,
        on_delete=models.CASCADE,
    )
    # ...
```

<hr>

**See also**: `ForeignKey` fields accept a number of extra arguments which are explained in [the model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#foreign-key-arguments). These options help define how the relationship should work; all are optional.

For details on accessing backwards-related objects, see the [Following relationships backward example](). <!-- in the "Making queries" module -->

For sample code, see the [Many-to-one relationship model example](https://docs.djangoproject.com/en/4.0/topics/db/examples/many_to_one/).

<hr>

#### Many-to-many relationships

To define a many-to-many relationship, use [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField). You use it just like any other [`Field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field) type: by including it as a class attribute of your model.

`ManyToManyField` requires a positional argument: the class to which the model is related.

For example, if a `Pizza` has multiple `Topping` objects -- that is, a `Topping` can be on multiple pizzas and each `Pizza` has multiple toppings -- here's how you'd represent that:
```
from django.db import models

class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```
As with [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey), you can also create [recursive relationships](https://docs.djangoproject.com/en/4.0/ref/models/fields/#recursive-relationships) (an object with a many-to-many relationship to itself) and [relationships to models not yet defined](https://docs.djangoproject.com/en/4.0/ref/models/fields/#lazy-relationships).

It's suggested, but not required, that the name of a `ManyToManyField` (`toppings` in the example above) be a plural describing the set of related model objects.

It doesn't matter which model has the `ManyToManyField`, but you should only put it in one of the models -- not both.

Generally, `ManyToManyField` instances should go in the object that's going to be edited on a form. In the above example, `toppings` is in `Pizza` (rather than `Topping` having a `pizzas` `ManyToManyField`) because it's more natural to think about a pizza having toppings than a topping being on multiple pizzas. The way it's set up above, the `Pizza` form would let users select the toppings.

<hr>

**See also**: See the [Many-to-many relationship model example](https://docs.djangoproject.com/en/4.0/topics/db/examples/many_to_many/) for a full example.

<hr>

`ManyToManyField` fields also accept a number of extra arguments which are explained in [the model field reference](https://docs.djangoproject.com/en/4.0/ref/models/fields/#manytomany-arguments). These options help define how the relationship should work; all are optional.

### Extra fields on many-to-many relationships

When you're only dealing with many-to-many relationships such as mixing and matching pizzas and toppings, a standard [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField) is all you need. However, sometimes you may need to associate data with the relationship between two models.

For example, consider the case of an application tracking the musical groups which musicians belong to. There is a many-to-many relationship between a person and the groups of which they are a member, so you could use a `ManyToManyField` to represent this relationship. However, there is a lot of detail about the membership that you might want to collect, such as the date at which the person joined the group.

For these situations, Django allows you to specify the model that will be used to govern the many-to-many relationship. You can then put extra fields on the intermediate model. The intermediate model is associated with the `ManyToManyField` using the [`through`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField.through) argument to point to the model that will act as an intermediary. For our musician example, the code would look something like this:
```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __str__(self):
        return self.name
    
class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```
When you set up the intermediary model, you explicitly specify foreign keys to the models that are involved in the many-to-many relationship. This explicit declaration defines how the two models are related.

There are a few restrictions on the intermediate model:

* Your intermediate model must contain one -- and *only* one -- foreign key to the source model (this would be `Group` in our example), or you must explicitly specify the foreign keys Django should use for the relationship using [`ManyToManyField.through_fields`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField.through_fields). If you have more than one foreign key and `through_fields` is not specified, a validation error will be raised. A similar restriction applies to the foreign key to the target model (this would be `Person` in our example).
* For a model which has a many-to-many relationship to itself through an intermediary model, two foreign keys to the same model are permitted, but they will be treated as the two (different) sides of the many-to-many relationship. If there are *more* than two foreign keys though, you must also specify `through_fields` as above, or a validation error will be raised.