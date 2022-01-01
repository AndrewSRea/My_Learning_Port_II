# Writing your first Django app - Part 4

This tutorial begins where [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_3#writing-your-first-django-app---part-3) left off. We're continuing the Web-poll application and will focus on form processing and cutting down our code.

<hr>

**Where to get help**: If you're having trouble going through this tutorial, please head over to the [Getting Help](https://docs.djangoproject.com/en/3.1/faq/help/) section of the FAQ.

<hr>

## Write a minimal form

Let's update our poll detail template ("polls/detail.html") from the last tutorial, so that the template contains an HTML `<form>` element:

`polls/templates/polls/detail.html`

```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice"{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

A quick rundown:

* The above template displays a radio button for each question choice. The `value` of each radio button is the associated question choice's ID. The `name` of each radio button is "`choice`". That means when somebody selects one of the radio buttons and submits the form, it'll send the `POST` data `choice=#` where # is the ID of the selected choice. This is the basic concept of HTML forms.
* We set the form's `action` to `{% url 'polls:vote' question.id %}`, and we set `method="post"`. Using `method="post"` (as opposed to `method="get"`) is very important, because the act of submitting this form will alter data server-side. Whenever you create a form that alters data server-side, use `method="post"`. This tip isn't specific to Django; it's good Web development practice in general.
* `forloop.counter` indicates how many times the [`for`](https://docs.djangoproject.com/en/3.1/ref/templates/builtins/#std:templatetag-for) tag has gone through its loop.
* Since we're creating a `POST` form (which can have the effect of modifying data), we need to worry about Cross Site Request Forgeries. Thankfully, you don't have to worry too hard, because Django comes with a helpful system for protecting against it. In short, all `POST` forms that are targeted at internal URLs should use the [`{% csrf_token %}`](https://docs.djangoproject.com/en/3.1/ref/templates/builtins/#std:templatetag-csrf_token) template tag.

Now let's create a Django view that handles the submitted data and does something with it. Remember, in [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_3#writing-your-first-django-app---part-3), we created a URLconf for the polls application that includes this line:

`polls/urls.py`

```
path('<int:question_id>/vote/', views.vote, name='vote'),
```

We also created a dummy implementation of the `vote()` function. Let's create a real version. Add the following to `polls/views.py`:

`polls/views.py`

```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form
        return render(request, 'polls/detail.html', {
            'question': question,
            'error message': "You didn't select a choice.",
        })
    else: 
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

This code includes a few things we haven't covered yet in this tutorial:

* [`request.POST`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest.POST) is a dictionary-like object that lets you access submitted data by key name. In this case, `request.POST['choice']` returns the ID of the selected choice, as a string. [`request.POST`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest.POST) values are always strings.
    - Note that Django also provides [`request.GET`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest.GET) for accessing `GET` data in the same way -- but we're explicitly using [`request.POST`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest.POST) in our code, to ensure that data is only altered via a `POST` call.
* `request.POST['choice']` will raise [`KeyError`](https://docs.python.org/3/library/exceptions.html#KeyError) if `choice` wasn't provided in `POST` data. The above code checks for [`KeyError`](https://docs.python.org/3/library/exceptions.html#KeyError) and redisplays the question form with an error message if `choice` isn't given.
* After incrementing the choice count, the code returns an [`HttpResponseRedirect`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponseRedirect) rather than a normal [`HttpResponse`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponse). [`HttpResponseRedirect`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponseRedirect) takes a single argument: the URL to which the user will be redirected (see the following point for how we construct the URL in this case). As the Python comment above points out, you should always return an [`HttpResponseRedirect`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponseRedirect) after successfully dealing with `POST` data. This tip isn't specific to Django; it's good Web development practice in general.
* We are using the [`reverse()`](https://docs.djangoproject.com/en/3.1/ref/urlresolvers/#django.urls.reverse) function in the [`HttpResponseRedirect`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpResponseRedirect) constructor in this example. This function helps avoid having to hardcode a URL in the view function. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view. In this case, using the URLconf we set up in [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_3#writing-your-first-django-app---part-3), this [`reverse()`](https://docs.djangoproject.com/en/3.1/ref/urlresolvers/#django.urls.reverse) call will return a string like `'/polls/3/results/'`, where the `3` is the value of `question.id`. This redirected URL will then call the `'results'` view to display the final page.

As mentioned in [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_3#writing-your-first-django-app---part-3), `request` is an [`HttpRequest`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest) object. For more on [HttpRequest`](https://docs.djangoproject.com/en/3.1/ref/request-response/#django.http.HttpRequest) objects, see the [request and response documentation](https://docs.djangoproject.com/en/3.1/ref/request-response/).

After somebody votes in a question, the `vote()` view redirects to the results page for the question. Let's write that view:

`polls/views.py`

```
from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or _404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

This is almost exactly the same as the `detail()` view from [Tutorial 3](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_3#writing-your-first-django-app---part-3). The only difference is the template name. We'll fix this redundancy later.

Now, create a `polls/results.html` template:

`polls/templates/polls/results.html`

```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %} 
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %} 
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

Now, go to `/polls/1/` in your browser and vote in the question. You should see a results page that gets updated each time you vote. If you submit the form with having chosen a choice, you should see the error message.

<hr>

**Note**: The code for our `vote()` view does have a small problem. It first gets the `selected_choice` object from the database, then computes the new value of `votes`, and then saves it back to the database. If two users of your website try to vote at *exactly the same time, this might go wrong: The same value, let's say 42, will be retrieved for `votes`. Then for both users, the new value of 43 is computed and saved, but 44 would be the expected value.

This is called a *race condition*. If you are interested, you can read [Avoiding race conditions using `F()`](https://docs.djangoproject.com/en/3.1/ref/models/expressions/#avoiding-race-conditions-using-f) to learn how you can solve this issue.

<hr>