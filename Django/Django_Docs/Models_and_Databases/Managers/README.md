# Managers

#### *`class`* `Manager`

A `Manager` is the interface through which database query operations are provided to Django models. At least one `Manager` exists for every model in a Django application.

The way `Manager` classes work is documented in [Making queries](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#making-queries); this document specifically touches on model options that customize `Manager` behavior.

## Manager names

By default, Django adds a `Manager` with the name `objects` to every Django model class. However, if you want to use `objects` as a field name, or if you want to use a name other than `objects` for the `Manager`, you can rename it on a per-model basis. To rename the `Manager` for a given class, define a class attribute of type `models.Manager()` on that model. For example:
```
from django.db import models

class Person(models.Model):
    # ...
    people = models.Manager()
```
Using this example model, `Person.objects` will generate an `AttributeError` exception, but `Person.people.all()` will provide a list of all `Person` objects.

## Custom managers

You can use a custom `Manager` in a particular model by extending the base `Manager` class and instantiating your custom `Manager` in your model.

There are two reasons you might want to customize a `Manager`: to add extra `Manager` methods, and/or to modify the initial `QuerySet` the `Manager` returns.

### Adding extra manager methods

Adding extra `Manager` methods is the preferred way to add "table-level" functionality to your models. (For "row-level" functionality -- i.e., functions that act on a single instance of a model object -- use [Model methods](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Models#model-methods), not custom `Manager` methods.)

For example, this custom `Manager` adds a method `with_counts()`:
```
from django.db import models
from django.db.models.functions import Coalesce 

class PollManager(models.Manager):
    def with_counts(self):
        return self.annotate(
            num_responses=Coalesce(models.Count("response"), 0)
        )

class OpinionPoll(models.Model):
    question = models.CharField(max_length=200)
    objects = PollManager()

class Response(models.Model):
    poll = models.ForeignKey(OpinionPoll, on_delete=models.CASCADE)
    # ...
```
With this example, you'd use `OpinionPoll.objects.with_counts()` to get a `QuerySet` of `OpinionPoll` objects with the extra `num_responses` attribute attached.

A custom `Manager` method can return anything you want. It doesn't have to return a `QuerySet`.

Another thing to note is that `Manager` methods can access `self.model` to get the model class to which they're attached.

### Modifying a manager's initial `QuerySet`

A `Manager`'s base `QuerySet` returns all objects in the system. For example, using this model:
```
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)
```
...the statement `Book.objects.all()` will return all books in the database.

You can override a `Manager`'s base `QuerySet` by overriding the `Manager.get_queryset()` method. `get_queryset()` should return a `QuerySet` with the properties you require.

For example, the following model has *two* `Manager`s -- one that returns all objects, and one that returns only the books by Roald Dahl:
```
# First, define the Manager subclass.
class DahlBookManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(author='Roald Dahl')

# Then hook it into the Book model explicitly.
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)

    objects = models.Manager()         # The default manager.
    dahl_objects = DahlBookManager()   # The Dahl-specific manager.
```
With this sample model, `Book.objects.all()` will return all books in the database, but `Book.dahl_objects.all()` will only return the ones written by Roald Dahl.

Because `get_queryset()` returns a `QuerySet` object, you can use `filter()`, `exclude()`, and all the other `QuerySet` methods on it. So these statements are all legal:
```
Book.dahl_objects.all()
Book.dahl_objects.filter(title='Matilda')
Book.dahl_objects.count()
```
This example also pointed out another interesting technique: using multiple managers on the same model. You can attach as many `Manager()` instances to a model as you'd like. This is a non-repetitive way to define common "filters" for your models.

For example:
```
class AuthorManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(role='A')

class EditorManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(role='E')

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    role = models.CharField(max_length=1, choices=[('A', _('Author')), ('E', _('Editor'))])
    people = models.Manager()
    authors = AuthorManager()
    editors = EditorManager()
```
This example allows you to request `Person.authors.all()`, `Person.editors.all()`, and `Person.people.all()`, yielding predictable results.

### Default managers

#### `Model._default_manager`

If you use custom `Manager` objects, take note that the first `Manager` Django encounters (in the order in which they're defined in the model) has a special status. Django interprets the first `Manager` defined in a class as the "default" `Manager`, and several parts of Django (including [`dumpdata`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-dumpdata)) will use that `Manager` exclusively for that model. As a result, it's a good idea to be careful in your choice of default manager in order to avoid a situation where overriding `get_queryset()` results in an inability to retrieve objects you'd like to work with.

You can specify a custom default manager using [`Meta.default_manager_name`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.default_manager_name).

If you're writing some code that must handle an unknown model, for example, in a third-party app that implements a generic view, use this manager (or [`_base_manager`]()) rather than assuming the model has an `objects` manager. <!-- below -->