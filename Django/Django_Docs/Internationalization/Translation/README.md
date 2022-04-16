# Translation

## Overview

In order to make a Django project translatable, you have to add a minimal number of hooks to your Python code and templates. These hooks are called [translation strings](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Internationalization#translation-string). They tell Django: "This text should be translated into the end user's language, if a translation for this text is available in that language." It's your responsibility to mark translatable strings; the system can only translate strings it knows about.

Django then provides utilities to extract the translation strings into a [message file](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Internationalization#message-file). This file is a convenient way for translators to provide the equivalent of the translation strings in the target language. Once the translators have filled in the message file, it must be compiled. This process relies on the GNU `gettext` toolset.

Once this is done, Django takes care of translating web apps on the fly in each available language, according to users' language preferences.

Django's internationalization hooks are on by default, and that means there's a bit of i18n-related overhead in certain places of the framework. If you don't use internationalization, you should take the two seconds to set [`USE_I18N = False`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_I18N) in your settings file. Then Django will make some optimizations so as not to load the internationalization machinery.

<hr>

**Note**: Make sure you've activated translation for your project (the fastest way is to check if [`MIDDLEWARE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIDDLEWARE) includes [`django.middleware.locale.LocaleMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.middleware.locale.LocaleMiddleware)). If you haven't yet, see [How Django discovers language preference](). <!-- "How Django discovers language preference" below -->

<hr>

## Internationalization: in Python code

### Standard translation

Specify a translation string by using the function [`gettext()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.gettext). It's convention to import this as a shorter alias, `_`, to save typing.

<hr>

**Note**: Python's standard library `gettext` module installs `_()` into the global namespace, as an alias for `gettext()`. In Django, we have chosen not to follow this practice, for a couple of reasons:

1. Sometimes, you should use [`gettext_lazy()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.gettext_lazy) as the default translation method for a particular file. Without `_()` in the global namespace, the developer has to think about which is the most appropriate translation function.
2. The underscore character (`_`) is used to represent "the previous result" in Python's interactive shell and doctest tests. Installing a global `_()` function causes interference. Explicitly importing `gettext()` as `_()` avoids this problem.

<hr>

**What functions may be aliased as `_`?**

Because of how `xgettext` (used by [`makemessages`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemessages)) works, only functions that take a single string argument can be imported as `_`:

* [`gettext()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.gettext)
* [`gettext_lazy()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.gettext_lazy)

<hr>

In this example, the text `"Welcome to my site."` is marked as a translation string:
```
from django.http import HttpResponse
from django.utils.translation import gettext as _

def my_view(request):
    output = _("Welcome to my site.")
    return HttpResponse(output)
```
You could code this without using the alias. This example is identical to the previous one:
```
from django.http import HttpResponse
from django.utils.translation import gettext

def my_view(request):
    output = gettext("Welcome to my site.")
    return HttpResponse(output)
```
Translation works on computed values. This example is identical to the previous two:
```
def my_view(request):
    words = ['Welcome', 'to', 'my', 'site.']
    output = _(' '.join(words))
    return HttpResponse(output)
```
Translation works on variables. Again, here's an identical example:
```
def my_view(request):
    sentence = 'Welcome to my site.'
    output = _(sentence)
    return HttpResponse(output)
```
(The caveat with using variables or computed values, as in the previous two examples, is that Django's translation-string-detecting utility, [`django-admin makemessages`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-makemessages), won't be able to find these strings. More on `makemessages` later.)

The strings you pass to `_()` or `gettext()` can take placeholders, specified with Python's standard named-string interpolation syntax. Example:
```
def my_view(request, m, d):
    output = ('Today is %(month)s %(day)s.') % {'month': m, 'day': d}
    return HttpResponse(output)
```
This technique lets language-specific translation reorder the placeholder text. For example, an English translation may be `"Today is November 26."`, while a Spanish translation may be `"Hoy es 26 de noviembre."` -- with the month and the day placeholders swapped.

For this reason, you should use named-string interpolation (e.g., `%(day)s`) instead of positional interpolation (e.g., `%s` or `%d`) whenever you have more than a single parameter. If you used positional interpolation, translations wouldn't be able to reorder placeholder text.

Since string extraction is done by the `xgettext` command, only syntaxes supported by `gettext` are supported by Django. In particular, Python [f-strings](https://docs.python.org/3/reference/lexical_analysis.html#f-strings) are not yet supported by `xgettext`, and JavaScript template strings need `gettext` 0.21+.

### Comments for translators

If you would like to give translators hints about a translatable string, you can add a comment prefixed with the `Translators` keyword on the line preceding the string, e.g.:
```
def my_view(request):
    # Translators: This message appears on the home page only
    output = gettext("Welcome to my site.")
```
The comment will then appear in the resulting `.po` file associated with the translatable construct located below it and should also be displayed by most translation tools.

<hr>

**Note**: Just for completeness, this is the corresponding fragment of the resulting `.po` file:

```
#. Translators: This message appears on the home page only
# path/to/python/file.py:123
msgid "Welcome to my site."
msgstr ""
```

<hr>

This also works in templates. See [Comments for translators in templates]() for more details. <!-- same title below -->

### Marking strings as no-op

Use the function [`django.utils.translation.gettext_noop()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.gettext_noop) to mark a string as a translation string without translating it. The string is later translated from a variable.

Use this if you have constant strings that should be stored in the source language because they are exchanged over systems or users -- such as strings in a database -- but should be translated at the last possible point in time, such as when the string is presented to the user.

### Pluralization

Use the function [`django.utils.translation.ngettext()`]() to specify pluralized messages.

`ngettext()` takes three arguments: the singular translation string, the plural translation string, and the number of objects.

This function is useful when you need your Django application to be localizable to languages where the number and complexity of [plural forms]() is greater than the two forms used in English ('object' for the singular and 'objects' for all the cases where `count` is different from one, irrespective of its value).

For example:
```
from django.http import HttpResponse
from django.utils.translation import ngettext

def hello_world(request, count):
    page = ngettext(
        'there is %(count)d object',
        'there are %(count)d objects',
        count,
    ) % {
        'count': count,
    }
    return HttpResponse(page)
```
In this example, the number of object is passed to the translation languages as the `count` variable.

Note that pluralization is complicated and works differently in each language. Comparing `count` to 1 isn't always the correct rule. This code looks sophisticated, but will produce incorrect results for some languages:
```
from django.utils.translation import ngettext 
from my app.models import Report

count = Report.objects.count()
if count == 1:
    name = Report._meta.verbose_name
else:
    name = Report._meta.verbose_name_plural

text = ngettext(
    'There is %(count)d %(name)s available.',
    'There are %(count)d %(name)s available.',
    count,
) % {
    'count': count,
    'name': name
}
```
Don't try to implement your own singular-or-plural logic; it won't be correct. In a case like this, consider something like the following:
```
text = ngettext(
    'There is %(count)d %(name)s object available.',
    'There are %(count)d %(name)s objects available.',
    count,
) % {
    'count': count,
    'name': Report._meta.verbose_name,
}
```

<hr>

**Note**: When using `ngettext()`, make sure you use a single name for every extrapolated variable included in the literal. In the examples above, note how we used the `name` Python variable in both translation strings. This example, besides being incorrect in some languages as noted above, would fail:
```
text = ngettext(
    'There is %(count)d %(name)s available.',
    'There are %(count)d %(plural_name)s available.',
    count,
) % {
    'count': Report.objects.count(),
    'name': Report._meta.verbose_name,
    'plural_name': Report._meta.verbose_name_plural,
}
```
You would get an error when running [`django-admin compilemessages`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-compilemessages):
```
a format specification for argument 'name', as in 'msgstr[0]`, doesn't exist in 'msgid'
```

<hr>

### Contextual markers

Sometimes words have several meanings, such as `"May"` in English, which refers to a month name and to a verb. To enable translators to translate these words correctly in different contexts, you can use the [`django.utils.translation.pgettext()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.pgettext) function, or the [`django.utils.translation.npgettext()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.translation.npgettext) function if the string needs pluralization. Both take a context string as the first variable.

In the resulting `.po` file, the string will then appear as often as there are different contextual markers for the same string (the context will appear on the `msgctxt` line), allowing the translator to give a different translation for each of them.

For example:
```
from django.utils.translation import pgettext

month = pgettext("month name", "May")
```
or:
```
from django.db import models
from django.utils.translation import pgettext_lazy

class MyThing(models.Model):
    name = models.CharField(help_text=pgettext_lazy(
        'help text for MyThing model', 'This is the help text'))
```
...will appear in the `.po` file as:
```
msgctxt "month name"
msgid "May"
msgstr ""
```
Contextual markers are also supported by the [`translate`]() <!-- "`translate` template tag" below --> and [`blocktranslate`]() <!-- "`blocktranslate` template tag" below --> template tags.

### Lazy translation

Use the lazy versions of translation functions in [`django.utils.translation`](https://docs.djangoproject.com/en/4.0/ref/utils/#module-django.utils.translation) (easily recognizable by the `lazy` suffix in their names) to translate string lazily -- when the value is accessed rather than when they're called.

These functions store a lazy reference to the string -- not the actual translation. The translation itself will be done when the string is used in a string context, such as in template rendering.

This is essential when calls to these functions are located in code paths that are executed at module load time.

This is something that can easily happen when defining models, forms, and model forms, because Django implements these such that their fields are actually class-level attributes. For that reason, make sure to use lazy translations in the following cases:

#### Model fields and relationships `verbose_name` and `help_text` option values

For example, to translate the help text of the *name* field in the following model, do the following:
```
from django.db import models
from django.utils.translation import gettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(help_text=_('This is the help text'))
```
You can mark names of [`ForeignKey`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ForeignKey), [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField) or [`OneToOneField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.OneToOneField) relationship as translatable by using their [`verbose_name`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.verbose_name) options:
```
class MyThing(models.Model):
    kind = models.ForeignKey(
        ThingKind,
        on_delete=models.CASCADE,
        related_name='kinds',
        verbose_name=_('kind'),
    )
```
Just like you would do in `verbose_name`, you should provide a lowercase verbose name text for the relation as Django will automatically titlecase it when required.

#### Model verbose names values

It is recommended to always provide explicit [`verbose_name`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.verbose_name) and [`verbose_name_plural`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.verbose_name_plural) options rather than relying on the fallback English-centric and somewhat naive determination of verbose names Django performs by looking at the model's class name:
```
from django.db import models
from django.utils.translation import gettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(_('name'), help_text=('This is the help text'))

    class Meta:
        verbose_name = _('my thing')
        verbose_name_plural = _('my things')
```

#### Model methods `description` argument to the `@display` decorator

For model methods, you can provide translations to Django and the admin site with the `description` argument to the [`display()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.display) decorator:
```
from django.contrib import admin
from django.db import models
from django.utils.translation import gettext_lazy as _

class MyThing(models.Model):
    kind = models.ForeignKey(
        ThingKind,
        on_delete=models.CASCADE,
        related_name='kinds',
        verbose_name=_('kind'),
    )

    @admin.display(description=_('Is it a mouse?'))
    def is_mouse(self):
        return self.kind.type == MOUSE_TYPE
```

### Working with lazy translation objects

The result of a `gettext_lazy()` call can be used wherever you would use a string (a [`str`](https://docs.python.org/3/library/stdtypes.html#str) object) in other Django code, but it may not work with arbitrary Python code. For example, the following won't work because the [requests](https://pypi.org/project/requests/) library doesn't handle `gettext_lazy` objects:
```
body = gettext_lazy("I \u2764 Django")   # (Unicode :heart:)
requests.post('https://example.com/send', data={'body': body})
```
You can avoid such problems by casting `gettext_lazy()` objects to text strings before passing them to non-Django code:
```
requests.post('https://example.com/send', data={'body': str(body)})
```
If you don't like the long `gettext_lazy` name, you can alias it as `_` (underscore), like so:
```
from django.db import models
from django.utils.translation import gettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(help_text=_('This is the help text'))
```
Using `gettext_lazy()` and `ngettext_lazy()` to mark strings in models and utility functions is a common operation. When you're working with these objects elsewhere in your code, you should ensure that you don't accidentally convert them to strings, because they should be converted as late as possible (so that the correct locale is in effect). This necessitates the use of the helper function described next.

#### Lazy translations and plural

When using lazy translation for a plural string (`n[p]gettext_lazy`), you generally don't know the `number` argument at the time of the string definition. Therefore, you are authorized to pass a key name instead of an integer as the `number` argument. Then `number` will be looked up in the dictionary under that key during string interpolation. Here's an example:
```
from django import forms
from django.core.exceptions import ValidationError
from django.utils.translation import ngettext_lazy

class MyForm(forms.Form):
    error_message = ngettext_lazy("You only provided %(num)d argument", "You only provided %(num)d arguments", 'num')

    def clean(self):
        # ...
        if error:
            raise ValidationError(self.error_message % {'num': number})
```
If the string contains exactly one unnamed placeholder, you can interpolate directly with the `number` argument:
```
class MyForm(forms.Form):
    error_message = ngettext_lazy(
        "You provided %d argument",
        "You provided %d arguments",
    )

    def clean(self):
        # ...
        if error:
            raise ValidationError(self.error_message % number)
```

#### Formatting strings: `format_lazy()`

Python's [`str.format()`](https://docs.python.org/3/library/stdtypes.html#str.format) method will not work when wither the `format_string` or any of the arguments to `str.format()` contains lazy translation objects. Instead, you can use [`django.utils.text.format_lazy()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.text.format_lazy), which creates a lazy object that runs the `str.format()` method only when the result is included in a string. For example:
```
from django.utils.text import format_lazy
from django.utils.translation import gettext_lazy
...
name = gettext_lazy('John Lennon')
instrument = gettext_lazy('guitar')
result = format_lazy('{name}: {instrument}', name=name, instrument=instrument)
```
In this case, the lazy translations in `result` will only be converted to strings when `result` itself is used in a string (usually at template rendering time).

#### Other uses of lazy in delayed translations

For any other case where you would like to delay the translation, but have to pass the translatable string as an argument to another function, you can wrap this function inside a lazy call yourself. For example:
```
from django.utils.functional import lazy
from django.utils.safestring import mark_safe
from django.utils.translation import gettext_lazy as _

mark_safe_lazy = lazy(mark_safe, str)
```
And the later:
```
lazy_string = mark_safe_lazy(_("<p>My <strong>string!</strong></p>"))
```

### Localized names of languages

##### [`get_language_info(lang_code)`](https://docs.djangoproject.com/en/4.0/_modules/django/utils/translation/#get_language_info)

The `get_language_info()` function provides detailed information about languages:
```
>>> from django.utils.translation import activate, get_language_info
>>> activate('fr')
>>> li = get_language_info('de')
>>> print(li['name'], li['name_local'], li['name_translated'], li['bidi'])
German Deutsch Allemande False
```
The `name`, `name_local`, and `name_translated` attributes of the dictionary contain the name of the language in English, in the language itself, and in your current active language respectively. The `bidi` attribute is `True` only for bi-directional languages.

The source of the language information is the `django.conf.locale` module. Similar access to this information is available for template code. See below.