# Templates

Being a web framework, Django needs a convenient way to generate HTML dynamically. The most common approach relies on templates. A template contains the static parts of the desired HTML output as well as some special syntax describing how dynamic content will be inserted. For a hands-on example of creating HTML pages with templates, see [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Getting_Started/Tutorial_3#writing-your-first-django-app---part-3).

A Django project can be configured with one or several template engines (or even zero if you don't use templates). Django ships built-in backends for its own template system, creatively called the Django template language (DTL), and for the popular alternative [Jinja2](https://jinja.palletsprojects.com/en/3.0.x/). Backends for other template languages may be available from third parties. You can also write your own custom backend; see [Custom template backend](https://docs.djangoproject.com/en/4.0/howto/custom-template-backend/).

Django defines a standard API for loading and rendering templates regardless of the backend. Loading consists of finding the template for a given identifier and preprocessing it, usually compiling it to an in-memory representation. Rendering means interpolating the template with context data and returning the resulting string.

The [Django template language](https://docs.djangoproject.com/en/4.0/ref/templates/language/) is Django's own template system. Until Django 1.8, it was the only built-in option available. It's a good template library even though it's fairly opinionated and sports a few idiosyncracies. If you don't have a pressing reason to choose another backend, you should use the DTL, especially if you're writing a pluggable application and you intend to distribute templates. Django's contrib apps that include templates, like [`django.contrib.admin`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/), use the DTL.

For historical reasons, both the generic support for template engines and the implementation of the Django template language live in the `django.template` namespace.

<hr>

:warning: **Warning**: The template system isn't safe against untrusted template authors. For example, a site shouldn't allow its users to provide their own templates, since template authors can do things like perform XSS attacks and access properties of template variables that may contain sensitive information.

<hr>