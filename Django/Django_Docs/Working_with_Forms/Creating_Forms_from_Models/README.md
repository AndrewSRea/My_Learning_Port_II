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
