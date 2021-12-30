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