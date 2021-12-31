# Writing your first Django app - Part 3

This tutorial begins where [Tutorial 2](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2) left off. We're continuing the Web-poll application and will focus on creating the public interface -- "views".

<hr>

**Where to get help**: If you're having trouble going through this tutorial, please head over to the [Getting Help](https://docs.djangoproject.com/en/3.1/faq/help/) section of the FAQ.

## Overview

A view is a "type" of Web page in your Django application that generally serves a specific function and has a specific template. For example, in a blog application, you might have the following views:

* Blog homepage - displays the latest few entries.
* Entry "detail" page - permalink page for a single entry.
* Year-based archive page - displays all months with entries in the given year.
* Month-based archive page - displays all days with entries in the given month.
* Day-based archive page - displays all entries in the given day.
* Comment action - handles posting comments to a given entry.

In our poll application, we'll have the following four views:

* Question "index" page - displays the latest few questions.
* Question "detail" page - displays a question text, with no results but with a form to vote.
* Question "results" page - displays results for a particular question.
* Vote action - handles voting for a particular choice in a particular question.

In Django, web pages and other content are delivered by views. Each view is represented by a Python function (or method, in the case of class-based views). Django will choose a view by examining the URL that's requested (to be precise, the part of the URL after the domain name).

Now in your time on the web, you may have come across such beauties as `ME2/Sites/dirmod.htm?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B`. You will be pleased to know that Django allows us much more elegant *URL patterns* than that.

A URL pattern is the general form of a URL -- for example: `/newsarchive/<year>/<month>/`.

To get from a URL to a view, Django uses what are known as "URLconfs". A URLconf maps URL patterns to views.

This tutorial provides basic information in the use of URLconfs, and you can refer to [URL dispatcher](https://docs.djangoproject.com/en/3.1/topics/http/urls/) for more information.

## Writing more views

Now let's add a few more views to `polls/views.py`. These views are slightly different because they take an argument:

`polls/views.py`

```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

Wire these new views into the `polls.urls` module by adding the following [`path()`](https://docs.djangoproject.com/en/3.1/ref/urls/#django.urls.path) calls:

`polls/urls.py`

```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Take a look in your browser, at "/polls/34/". It'll run the `detail()` method and display whatever ID you provide in the URL. Try "/polls/34/results/" and "/polls/34/vote/", too -- these will display the placeholder results and voting pages.

When somebody requests a page from your website -- say, "/polls/34/", Django will load the `mysite.urls` Python module because it's pointed to by the [`ROOT_URLCONF`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-ROOT_URLCONF) setting. It finds the variable named `urlpatterns` and traverses the patterns in order. After finding the match at `'polls/'`, it strips off the matching text (`"polls/"`) and sends the remaining text -- `"34/"` -- to the `polls.urls` URLconf for further processing. There it matches `'<int:question_id>/'`, resulting in a call to the `detail()` view like so:
```
detail(request=<HttpRequest object>, question_id=34)
```
The `question_id=34` part comes from `<int:question_id>`. Using angle brackets "captures" part of the URL and sends it as a keyword argument to the view function. The `question_id` part of the string defines the name that will be used to identify the matched pattern, and the `int` part is a converter that determines what patterns should match this part of the URL path. The colon (`:`) separates the converter and pattern name.

## Write views that actually do something

Each view is responsible for doing one of two things: returning an [`HttpResponse`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponse) object containing the content for the requested page, or raising an exception such as [`Http404`](https://docs.djangoproject.com/en/3.1/topics/http/views/#django.http.Http404). The rest is up to you.

Your view can read records from a database, or not. It can use a template system such as Django's -- or a third-party Python template system -- or not. It can generate a PDF file, output XML, create a ZIP file on the fly, anything you want, using whatever Python libraries you want.

All Django wants is that [`HttpResponse`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponse). Or an exception.

Because it's convenient, let's use Django's own database API, which we covered in [Tutorial 2](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2). Here's one stab at a new `index()` view, which displays the latest 5 poll questions in the system, separated by commas, according to publication date:

`polls/views.py`

```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged.
```

There's a problem here, though: the page's design is hard-coded in the view. If you want to change the way the page looks, you'll have to edit this Python code. So let's use Django's template system to separate the design from Python by creating a template that the view can use.

First, create a directory called `templates` in your `polls` directory. Django will look for templates in there.

Your project's [`TEMPLATES`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-TEMPLATES) setting describes how Django will load and render templates. The default settings file configures a `DjangoTemplates` backend whose [`APP_DIRS`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-TEMPLATES-APP_DIRS) option is set to `True`. By convention, `DjangoTemplates` looks for a "templates" subdirectory in each of the [`INSTALLED_APPS`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-INSTALLED_APPS).

Within the `templates` directory you have just created, create another directory called `polls`, and within that create a file called `index.html`. In other words, your template should be at `polls/templates/polls/index.html`. Because of how the `app_directories` template loader works as described above, you can refer to this template within Django as `polls/index.html`.

<hr>

**Template namespacing**

Now we *might* be able to get away with putting our templates directly in `polls/templates` (rather than creating another `polls` subdirectory), but it would actually be a bad idea. Django will choose the first template it finds whose name matches, and if you had a template with the same name in a *different* application, Django would be unable to distinguish between them. We need to be able to point Django at the right one, and the best way to ensure this is by *namespacing* them. That is, by putting those templates inside *another* directory named for the application itself.

<hr>

Put the following code in that template:

`polls/templates/polls/index.html`

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

<hr>

**Note**: To make this tutorial shorter, all template examples use incomplete HTML. In your own projects, you should use [complete HTML documents](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Getting_started#anatomy_of_an_html_document).

<hr>

:attention: **Attention**: It's strange that the Django documentation would refer to using "normal" HTML documentation above, when Django has its own [template language](https://docs.djangoproject.com/en/4.0/ref/templates/language/) for creating HTML templates.

<hr>

Now, let's update our `index` view in `polls/views.py` to use the template:

`polls/views.py`

```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

That code loads the template called `polls/index.html` and passes it a context. The context is a dictionary mapping template variable names to Python objects.

Load the page by pointing your browser at "/polls/", and you should see a bulleted-list containing the "What's up" question from [Tutorial 2](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2). The link points to the question's detail page.

### A shortcut: [`render()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.render)

It's a very common idiom to load a template, fill a context and return an [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse) object with the result of the rendered template. Django provides a shortcut. Here's the full `index()` view, rewritten:

`polls/views.py`

```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

Note that once we've done this in all these views, we no longer need to import [`loader`](https://docs.djangoproject.com/en/4.0/topics/templates/#module-django.template.loader) and [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse) (you'll want to keep `HttpResponse` if you still have the stub methods for `detail`, `results`, and `vote`).

The [`render()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.render) function takes the request object as its first argument, a template name as its second argument, and a dictionary as its optional third argument. It returns an [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse) object of the given template rendered with the given context.

## Raising a 404 error

Now, let's tackle the question detail view -- the page that displays the question text for a given poll. Here's the view:

`polls/views.py`

```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

The new concept here: The view raises the [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) exception if a question with the requested ID doesn't exist.

We'll discuss what you could put in that `polls/detail.html` template a bit later, but if you'd like to quickly get the above example working, a file containing just:

`polls/templates/polls/detail.html`

```
{{ question }}
```

...will get you started for now.

### A shortcut: [`get_object_or_404()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404)

It's a very common idiom to use [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) and raise [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) if the object doesn't exist. Django provides a shortcut. Here's the `detail()` view, rewritten:

`polls/views.py`

```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

The [`get_object_or_404()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404) function takes a Django model as its first argument and an arbitrary number of keyword arguments, which it passes to the [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get) function of the model's manager. It raises [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) if the object doesn't exist.

<hr>

**Philosophy**

Why do we use a helper function [`get_object_or_404()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404) instead of automatically catching the [`ObjectDoesNotExist`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.ObjectDoesNotExist) exceptions at a higher level, or having the model API raise [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) instead of [`ObjectDoesNotExist`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.ObjectDoesNotExist)?

Because that would couple the model layer to the view layer. One of the foremost design goals of Django is to maintain loose coupling. Some controlled coupling is introduced in the [`django.shortcuts`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#module-django.shortcuts) module.

<hr>

There's also a [`get_list_or_404()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.get_list_or_404) function, which works just as [`get_object_or_404()`](https://docs.djangoproject.com/en/4.0/topics/http/shortcuts/#django.shortcuts.get_object_or_404) -- except using [`filter()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.filter) instead of [`get()`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet.get). It raises [`Http404`](https://docs.djangoproject.com/en/4.0/topics/http/views/#django.http.Http404) if the list is empty.