# Django shortcut functions

The package `django.shortcuts` collects helper functions and classes that "span" multiple levels of MVC. In other words, these functions/classes introduce controlled coupling for convenience's sake.

## `render()`

#### `render(request, template_name, context=None, content_type=None, status=None, using=None)`

Combines a given template with a given context dictionary and returns an [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse) object with that rendered text.

Django does not provide a shortcut function which returns a [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) because the constructor of `TemplateResponse` offers the same level of convenience as `render()`.

### Required arguments

#### `request`

The request object used to generate this response.

#### `template_name`

The full name of a template to use or sequence of template names. If a sequence is given, the first template that exists will be used. See the [template loading documentation](https://docs.djangoproject.com/en/4.0/topics/templates/#template-loading) for more information on how templates are found. <!-- possible future folder? -->

### Optional arguments

#### `context`

A dictionary of values to add to the template context. By default, this is an empty dictionary. If a value in the dictionary is callable, the view will call it just before rendering the template.

#### `content_type`

The MIME type to use for the resulting document. Defaults to `'text/html'`.

#### `status`

The status code for the response. Defaults to `200`.

#### `using`

The [`NAME`]() of a template engine to use for loading the template.

### Example

The following example renders the template `myapp/index.html` with the MIME type *application/xhtml+xml*:
```
from django.shortcuts import render

def my_view(request):
    # View code here...
    return render(request, 'myapp/index.html', {
        'foo': 'bar',
    }, content_type='application/xhtml+xml')
```
This example is equivalent to:
```
from django.http import HttpResponse
from django.template import loader

def my_view(request):
    # View code here...
    t = loader.get_template('myapp/index.html')
    c = {'foo': 'bar'}
    return HttpResponse(t.render(c, request), content_type='application/xhtml+xml')
```

## `redirect()`

#### `redirect(to, *args, permanent=False, **kwargs)`

Returns an [`HttpResponseRedirect`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponseRedirect) to the appropriate URL for the arguments passed.

The arguments could be:

* A model: the model's [`get_absolute_url()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.get_absolute_url) function will be called.
* A view name, possible with arguments: [`reverse()`](https://docs.djangoproject.com/en/4.0/ref/urlresolvers/#django.urls.reverse) will be used to reverse-resolve the name.
* An absolute or relative URL, which will be used as-is for the redirect location.

By default, issues a temporary redirect; pass `permanent=True` to issue a permanent redirect.

### Examples

You can use the `redirect()` function in a number of ways.

1. By passing some object; that object's `get_absolute_url()` method will be called to figure out the redirect URL:

    ```
    from django.shortcuts import redirect

    def my_view(request):
        ...
        obj = MyModel.objects.get(...)
        return redirect(obj)
    ```

2. By passing the name of a view and optionally some positional or keyword arguments; the URL will be reverse resolved using the `reverse()` method:

    ```
    def my_view(request):
        ...
        return redirect('some-view-name', foo='bar')
    ```

3. By passing a hardcoded URL to redirect to:

    ```
    def my_view(request):
        ...
        return redirect('/some/url/')
    ```
    This also works with full URLs:
    ```
    def my_view(request):
        ...
        return redirect('https://example.com/')
    ```

By default, [`redirect()`]() <!-- above --> returns a temporary redirect. All of the above forms accept a `permanent` argument; if set to `True`, a permanent redirect will be returned:
```
def my_view(request):
    ...
    obj = MyModel.objects.get(...)
    return redirect(obj, permanent=True)
```

## `get_object_or_404()`

#### `get_object_or_404(klass, *args, **kwargs)`

Calls [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) on a given model manager, but it raises [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) instead of the model's [`DoesNotExist`](https://docs.djangoproject.com/en/4.0/ref/models/class/#django.db.models.Model.DoesNotExist) exception.

### Required arguments

#### `klass`

A [`Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model) class, a [`Manager`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Managers#managers), or a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) instance from which to get the object.

#### `**kwargs`

Lookup parameters, which should be in the format accepted by `get()` and `filter()`.

### Example

The following example gets the object with the primary key of 1 from `MyModel`:
```
from django.shortcuts import get_object_or_404

def my_view(request):
    obj = get_object_or_404(MyModel, pk=1)
```
This example is equivalent to:
```
from django.http import Http404

def my_view(request):
    try:
        obj = MyModel,objects.get(pk=1)
    except MyModel.DoesNotExist:
        raise Http404("No MyModel matches the given query.")
```
The most common use case is to pass a [`Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model), as shown above. However, you can also pass a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) instance:
```
queryset = Book.objects.filter(title__startswith='M')
get_object_or_404(queryset, pk=1)
```
The above example is a bit contrived since it's equivalent to doing:
```
get_object_or_404(Book, title__startswith='M', pk=1)
```
...but it can be useful if you are passed the `queryset` variable from somewhere else.

Finally, you can also use a [`Manager`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Managers#managers). This is useful, for example, if you have a [custom manager](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Managers#custom-managers):
```
get_object_or_404(Book.dahl_objects, title='Matilda')
```
You can also use [related managers](https://docs.djangoproject.com/en/4.0/ref/models/relations/#related-objects-reference):
```
author = Author.objects.get(name='Roald Dahl')
get_object_or_404(author.book_set, title='Matilda')
```
Note: As with `get()`, a [`MultipleObjectsReturned`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.MultipleObjectsReturned) exception will be raised if more than one object is found.

## `get_list_or_404()`

#### `get_list_or_404(klass, *args, **kwargs)`

Returns the result of [`filter()`]() on a given model manager cast to a list, raising [`Http404`]() if the resulting list is empty.

### Required arguments

#### `klass`

A [`Model`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model) class, a [`Manager`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Managers#managers), or a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet) instance from which to get the list.

#### `**kwargs`

Lookup parameters, which should be in the format accepted by `get()` and `filter()`.

### Example

The following example gets all published objects from `MyModel`:
```
from django.shortcuts import get_list_or_404

def my_view(request):
    my_objects = get_list_or_404(MyModel, published=True)
```
This example is equivalent to:
```
from django.http import Http404

def my_view(request):
    my_objects = list(MyModel.objects.filter(published=True))
    if not my_objects:
        raise Http404("No MyModel matches the given query.")
```

<hr>

:exclamation: **Attention**: The next page in Django's **Handling_HTTP_Requests** module is [Generic views](https://docs.djangoproject.com/en/4.0/topics/http/generic-views/), which just has a link to see the [Built-in class-based views API](https://docs.djangoproject.com/en/4.0/ref/class-based-views/) module. The **Built-in class-based views API** module is quite lengthy so you should just follow the link provided here as the module is too lengthy to add to this **My_Learning_Port_II** repository.

<hr>

[[Previous page]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/File_Uploads#file-uploads) - [[Top]](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Shortcut_Functions#django-shortcut-functions) - [[Next page: Middleware]]()