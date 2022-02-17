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
| [`AutoField`]() | Not represented in the form |
| [`BigAutoField`]() | Not represented in the form |
| [`BigIntegerField`]() | [`IntegerField`]() with `min_value` set to -9223372036854775808 and `max_value` set to 9223372036854775807. |
| [`BinaryField`]() | [`CharField`](), if [`editable`]() is set to `True` on the model field, otherwise not represented in the form. |
| [`BooleanField`]() | [`BooleanField`](), or [`NullBooleanField`]() if `null=True`. |
| [`CharField`]() | [`CharField`]() with `max_length` set to the model field's `max_length` and [`empty_value`]() set to `None` if `null=True`. |
| [`DateField`]() | [`DateField`]() |
| [`DateTimeField`]() | [`DateTimeField`]() |
| [`DecimalField`]() [`DecimalField`]() |
| [`DurationField`]() | [`DurationField`]() |
| [`EamilField`]() | [`EmailField`]() |
| [`FileField`]() | [`FileField`]() |
| [`FilePathField`]() | [`FilePathField`]() |
| [`FloatField`]() | [`FloatField`]() |
| [`ForeignKey`]() | [`ModelChoiceField`]() (see below) |
| [`ImageField`]() | [`ImageField`]() |
| [`IntegerField`]() | [`IntegerField`]() |
| `IPAddressField` | `IPAddressField` |
| [`GenericIPAddressField`]() | [`GenericIPAddressField`]() |
| [`JSONField`]() | [`JSONField`]() |
| [`ManyToManyField`]() | [`ModelMultipleChoiceField`]() (see below) |
| [`PositiveBigIntegerField`]() | [`IntegerField`]() |
| [`PositiveIntegerField`]() | [`IntegerField`]() |
| [`PositiveSmallIntegerField`]() | [`IntegerField`]() |
| [`SlugField`]() | [`SlugField`]() |
| [`SmallAutoField`]() | Not represented in the form |
| [`SmallIntegerField`]() | [`IntegerField`]() |
| [`TextField`]() | [`CharField`]() with `widget=forms.Textarea` |
| [`TimeField`]() | [`TimeField`]() |
| [`URLField`]() | [`URLField`]() |
| [`UUIDField`]() | [`UUIDField`]() |

As you might expect, the `ForeignKey` and `ManyToManyField` model field types are special cases:

* `ForeignKey` is represented by `django.forms.ModelChoiceField`, which is a `ChoiceField` whose choices are a model `QuerySet`.
* `ManyToManyField` is represented by `django.forms.ModelMultipleChoiceField`, which is a `MultipleChoiceField` whose choices are a model `QuerySet`.

In addition, each generated form field has attributes set as follows:

* If the model field has `blank=True`, then `required` is set to `False` on the form field. Otherwise, `required=True`.
* The form field's `label` is set to the `verbose_name` of the model field, with the first character capitalized.
* The form field's `help_text` is set to the `help_text` of the model field.
* If the model field has `choices` set, then the form field's `widget` will be set to `Select`, with choices coming from the model field's `choices`. The choices will normally include the blank choice which is selected by default. If the field is required, this forces the user to make a selection. The blank choice will not be included if the model field has `blank=False` and an explicit `default` value (the `default` value will be initially selected instead).

Finally, note that you can override the form field used for a given model field. See [Overriding the default fields]() below.
