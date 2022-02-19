# Creating forms from models

## `ModelForm`

##### `class ModelForm`

If you're building a database-driven app, chances are you'll have forms that map closely to Django models. For instance, you might have a `BlogComment` model, and you want to create a form that lets people submit comments. In this case, it would be redundant to define the field types in your form, because you've already defined the fields in your model.

For this reason, Django provides a helper class that lets you create a `Form` class from a Django model.

For example:
```
>>> from django.forms import ModelForm
>>> from myapp.models import Article

# Create the form class.
>>> class ArticleForm(ModelForm):
...     class Meta:
...         model = Article
...         fields = ['pub_date', 'headline', 'content', 'reporter']

# Creating a form to add an article.
>>> form = ArticleForm()

# Creating a form to change an existing article.
>>> article = Article.objects.get(pk=1)
>>> form = ArticleForm(instance=article)
```

### Field types

The generated `Form` class will have a form field for every model field specified, in the order specified in the `fields` attribute.

Each model field has a corresponding default form field. For example, a `CharField` on a model is represented as a `CharField` on a form. A model `ManyToManyField` is represented as a `MultipleChoiceField`. Here is the full list of conversions:

| Model field | Form field |
| --- | --- |
| [`AutoField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.AutoField) | Not represented in the form |
| [`BigAutoField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.BigAutoField) | Not represented in the form |
| [`BigIntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.BigIntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) with `min_value` set to -9223372036854775808 and `max_value` set to 9223372036854775807. |
| [`BinaryField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.BinaryField) | [`CharField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField), if [`editable`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.editable) is set to `True` on the model field, otherwise not represented in the form. |
| [`BooleanField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.BooleanField) | [`BooleanField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.BooleanField), or [`NullBooleanField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.NullBooleanField) if `null=True`. |
| [`CharField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.CharField) | [`CharField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField) with `max_length` set to the model field's `max_length` and [`empty_value`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField.empty_value) set to `None` if `null=True`. |
| [`DateField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DateField) | [`DateField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.DateField) |
| [`DateTimeField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DateTimeField) | [`DateTimeField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.DateTimeField) |
| [`DecimalField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DecimalField)| [`DecimalField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.DecimalField) |
| [`DurationField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.DurationField) | [`DurationField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.DurationField) |
| [`EmailField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.EmailField) | [`EmailField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.EmailField) |
| [`FileField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FileField) | [`FileField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FileField) |
| [`FilePathField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FilePathField) | [`FilePathField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FilePathField) |
| [`FloatField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.FloatField) | [`FloatField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FloatField) |
| [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey) | [`ModelChoiceField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.ModelChoiceField) (see below) |
| [`ImageField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ImageField) | [`ImageField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.ImageField) |
| [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.IntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) |
| `IPAddressField` | `IPAddressField` |
| [`GenericIPAddressField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.GenericIPAddressField) | [`GenericIPAddressField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.GenericIPAddressField) |
| [`JSONField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.JSONField) | [`JSONField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.JSONField) |
| [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField) | [`ModelMultipleChoiceField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.ModelMultipleChoiceField) (see below) |
| [`PositiveBigIntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.PositiveBigIntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) |
| [`PositiveIntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.PositiveIntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) |
| [`PositiveSmallIntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.PositiveSmallIntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) |
| [`SlugField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.SlugField) | [`SlugField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.SlugField) |
| [`SmallAutoField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.SmallAutoField) | Not represented in the form |
| [`SmallIntegerField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.SmallIntegerField) | [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) |
| [`TextField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.TextField) | [`CharField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField) with `widget=forms.Textarea` |
| [`TimeField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.TimeField) | [`TimeField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.TimeField) |
| [`URLField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.URLField) | [`URLField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.URLField) |
| [`UUIDField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.UUIDField) | [`UUIDField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.UUIDField) |

As you might expect, the `ForeignKey` and `ManyToManyField` model field types are special cases:

* `ForeignKey` is represented by `django.forms.ModelChoiceField`, which is a `ChoiceField` whose choices are a model `QuerySet`.
* `ManyToManyField` is represented by `django.forms.ModelMultipleChoiceField`, which is a `MultipleChoiceField` whose choices are a model `QuerySet`.

In addition, each generated form field has attributes set as follows:

* If the model field has `blank=True`, then `required` is set to `False` on the form field. Otherwise, `required=True`.
* The form field's `label` is set to the `verbose_name` of the model field, with the first character capitalized.
* The form field's `help_text` is set to the `help_text` of the model field.
* If the model field has `choices` set, then the form field's `widget` will be set to `Select`, with choices coming from the model field's `choices`. The choices will normally include the blank choice which is selected by default. If the field is required, this forces the user to make a selection. The blank choice will not be included if the model field has `blank=False` and an explicit `default` value (the `default` value will be initially selected instead).

Finally, note that you can override the form field used for a given model field. See [Overriding the default fields]() below.

### A full example

Consider this set of models:
```
from django.db import models
from django.forms import ModelForm

TITLE_CHOICES = [
    ('MR', 'Mr.'),
    ('MRS', 'Mrs.'),
    ('MS', 'Ms.'),
]

class Author(models.Model):
    name = models.CharField(max_length=100)
    title = models.CharField(max_length=3, choices=TITLE_CHOICES)
    birth_date = models.DateField(blank=True, null=True)

    def __str__(self):
        return self.name

class Book(models.Model):
    name = model.CharField(max_length=100)
    authors = models.ManyToManyField(Author)

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ['name', 'title', 'birth_date']

class BookForm(ModelForm):
    class Meta:
        model = Book
        fields = ['name', 'authors']
```
With these models, the `ModelForm` subclasses above would be roughly equivalent to this (the only difference being the `save()` method, which we'll discuss in a moment):
```
from django import forms

class AuthorForm(forms.Form):
    name = forms.CharField(max_length=100)
    title = forms.Charfield(
        max_length = 3,
        widget=forms.Select(choices=TITLE_CHOICES),
    )
    birth_date = forms.DateField(required=False)

class BookForm(forms.Form):
    name = forms.CharField(max_length=100)
    authors = forms.ModelMultipleChoiceField(queryset=Author.objects.all())
```

## Validation on a `ModelForm`

There are two main steps involved in validating a `ModelForm`:

1. [Validating the form]() <!-- possible future folder? (https://docs.djangoproject.com/en/4.0/ref/forms/validation/) -->
2. [Validating the model instance](https://docs.djangoproject.com/en/4.0/ref/models/instances/#validating-objects)

Just like normal form validation, model form validation is triggered implicitly when calling [`is_valid()`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form.is_valid) or accessing the [`errors`]() attribute and explicitly when calling `full_clean()`, although you will typically not use the latter method in practice.

`Model` validation ([`Model.full_clean()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.full_clean)) is triggered from within the form validation step, right after the form's `clean()` method is called.

<hr>

:warning: **Warning**: The cleaning process modifies the model instance passed to the `ModelForm` constructor in various ways. For instance, any date fields on the model are converted into actual date objects. Failed validation may leave the underlying model instance in an inconsistent state and therefore it's not recommended to reuse it.

<hr>

#### Overriding the `clean()` method

You can override the `clean()` method on a model form to provide additional validation in the same way you can on a normal form.

A model form instance attached to a model object will contain an `instance` attribute that gives its methods access to that specific model instance.

<hr>

:warning: **Warning**: The `ModelForm.clean()` method sets a flag that makes the [model validation](https://docs.djangoproject.com/en/4.0/ref/models/instances/#validating-objects) step validate the uniqueness of model fields that are marked as `unique`, `unique_together` or `unique_for_date|month|year`.

If you would like to override the `clean()` method and maintain this validation, you must call the parent class's `clean()` method.

<hr>

#### Interaction with model validation

As part of the validation process, `ModelForm` will call the `clean()` method of each field on your model that has a corresponding field on your form. If you have excluded any model fields, validation will not be run on those fields. See the [form validation]() <!-- possible future folder? (https://docs.djangoproject.com/en/4.0/ref/forms/validation/) --> documentation for more on how field cleaning and validation work.

The model's `clean()` method will be called before any uniqueness checks are made. See [Validation objects](https://docs.djangoproject.com/en/4.0/ref/models/instances/#validating-objects) for more information on the model's `clean()` hook.

#### Considerations regarding model's `error_messages`

Error messages defined at the [`form field`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.Field.error_messages) level or at the [form `Meta`]() <!-- below --> level always take precedence over the error messages defined at the [`model field`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.error_messages) level.

Error messages defined on `model fields` are only used when the `ValidationError` is raised during the [model validation](https://docs.djangoproject.com/en/4.0/ref/models/instances/#validating-objects) step and no corresponding error messages are defined at the form level.

You can override the error messages from `NON_FIELD_ERRORS` raised by model validation by adding the [`NON_FIELD_ERRORS`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.NON_FIELD_ERRORS) key to the `error_messages` dictionary of the `ModelForm`'s inner `Meta` class:
```
from django.core.exceptions import NON_FIELD_ERRORS
from django.forms import ModelForm

class ArticleForm(ModelForm):
    class Meta:
        error_messages = {
            NON_FIELD_ERRORS: {
                'unique_together': "%(model_name)s's %(field_labels)s are not unique.",
            }
        }
```

### The `save()` method

Every `ModelForm` also has a `save()` method. This method creates and saves a database object from the data bound to the form. A subclass of `ModelForm` can accept an existing model instance as the keyword argument `instance`; if this is supplied, `save()` will update that instance. If it's not supplied, `save()` will create a new instance of the specified model:
```
>>> from myapp.models import Article
>>> from myapp.forms import ArticleForm

# Create a form instance from POST data.
>>> f = ArticleForm(request.POST)

# Save a new Article object from the form's data.
>>> new_article = f.save()

# Create a form to edit an existing Article, but use POST data to populate the form.
>>> a = Article.objects.get(pk=1)
>>> f = ArticleForm(request.POST, instance=a)
>>> f.save()
```
Note that if the form [hasn't been validated](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#validation-on-modelform), calling `save()` will do so by checking `form.errors`. A `ValueError` will be raised if the data in the form doesn't validate -- i.e., if `form.errors` evaluates to `True`.

If an optional field doesn't appear in the form's data, the resulting model instance uses the model field [`default`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.Field.default), if there is one, for that field. This behavior doesn't apply to fields that use [`CheckboxInput`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.CheckboxInput), [`CheckboxSelectMultiple`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.CheckboxSelectMultiple), or [`SelectMultiple`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.SelectMultiple) (or any custom widget whose [`value_omitted_from_data()`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.Widget.value_omitted_from_data) method always returns `False`) since an unchecked checkbox and unselected `<select multiple>` don't appear in the data of an HTML form submission. Use a custom form field or widget if you're designing an API and want the default fallback behavior for a field that uses one of these widgets.

This `save()` method accepts an optional `commit` keyword argument, which accepts either `True` or `False`. If you call `save()` with `commit=False`, then it will return an object that hasn't yet been saved to the database. In this case, it's up to you to call `save()` on the resulting model instance. This is useful if you want to do custom processing on the object before saving it, or if you want to use one of the specialized [model saving options](https://docs.djangoproject.com/en/4.0/ref/models/instances/#ref-models-force-insert). `commit` is `True` by default.