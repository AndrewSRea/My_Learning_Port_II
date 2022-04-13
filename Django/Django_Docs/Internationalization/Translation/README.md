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