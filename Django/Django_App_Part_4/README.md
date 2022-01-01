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

Now let's create a Django view that handles the submitted data and does something with it. Remember, in [Tutorial 3](), we created a URLconf for the polls application that includes this line:

`polls/urls.py`

```
path