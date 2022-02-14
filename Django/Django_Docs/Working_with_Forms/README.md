# Working with forms

<hr>

**About this document**

This document provides an introduction to the basics of web forms and how they are handled in Django. For a more detailed look at specific areas of the forms API, see [The Forms API](https://docs.djangoproject.com/en/4.0/ref/forms/api/), [Form fields](https://docs.djangoproject.com/en/4.0/ref/forms/fields/), and [Form and field validation](https://docs.djangoproject.com/en/4.0/ref/forms/validation/).

<hr>

Unless you're planning to build websites and applications that do nothing but publish content, and don't accept input from your visitors, you're going to need to understand and use forms.

Django provides a range of tools and libraries to help you build forms to accept input from site visitors, and then process and respond to the input.

## HTML forms

In HTML, a form is a collection of elements inside `<form>...</form>` that allow a visitor to do things like enter text, select options, manipulate objects or controls, and so on, and then sned that information back to the server.

Some of these form interface elements -- text input or checkboxes -- are built into HTML itself. Others are much more complex; an interface that pops up a date picker or allows you to move a slider or manipulate controls will typically use JavaScript and CSS as well as HTML form `<input>` elements to achieve these effects.

As well as its `<input>` elements, a form must specify two things:

* *where*: the URL to which the data corresponding to the user's input should be returned.
* *how*: the HTTP method the data should be returned by.

As an example, the login form for the Django admin contains several `<input>` elements: one of `type="text"` for the username, one of `type="password"` for the password, and one of `type="submit"` for the "Log in" button. It also contains some hidden text fields that the user doesn't see, which Django uses to determine what to do next.

It also tells the browser that the form data should be sent to the URL specified in the `<form>`'s `action` attribute -- `/admin/` -- and that it should be sent using the HTTP mechanism specified by the `method` attribute -- `post`.

When the `<input type="submit" value="Log in">` element is triggered, the data is returned to `/admin/`.

### `GET` and `POST`

`GET` and `POST` are the only HTTP methods to use when dealing with forms.

Django's login form is returned using the `POST` method, in which the browser bundles up the form data, encodes it for transmission, sends it to the server, and then receives back its response.

`GET`, by contrast, bundles the submitted data into a string, and uses this to compose a URL. The URL contains the address where the data must be sent, as well as the data keys and values. You can see this in action if you do a search in the Django documentation, which will produce a URL of the form `https://docs.djangoproject.com/search/?q=forms&release=1`.

`GET` and `POST` are typically used for different purposes.

Any request that could be used to change the state of the system -- for example, a request that makes changes in the database -- should use `POST`. `GET` should be used only for requests that do not affect the state of the system.

`GET` would also be unsuitable for a password form, because the password would appear in the URL, and thus, also in browser history and server logs, all in plain text. Neither would it be suitable for large quantities of data, or for binary data, such as an image. A web application that uses `GET` requests for admin forms is a security risk: it can be easy for an attacker to mimic a form's request to gain access to sensitive parts of the system. `POST`, coupled with other protections like Django's [CSRF protection](https://docs.djangoproject.com/en/4.0/ref/csrf/) offers more control over access.

On the other hand, `GET` is suitable for things like a web search form, because the URLs that represent a `GET` request can easily be bookmarked, shared, or resubmitted.

## Django's role in forms

Handling forms is a complex business. Consider Django's admin, where numerous items of data of several different types may need to be prepared for display in a form, rendered as HTML, edited using a convenient interface, returned to the server, validated and cleaned up, and then saved or passed on for further processing.

Django's form functionality can simplify and automate vast portions of this work, and can also do it more securely than most programmers would be able to do in code they wrote themselves.

Django handles three distinct parts of the work involved in forms:

* preparing and restructuring data to make it ready for rendering.
* creating HTML forms for the data.
* receiving and processing submitted forms and data from the client.

It is *possible* to write code that does all of this manually, but Django can take care of it all for you.

## Forms in Django

We've described HTML forms briefly, but an HTML `<form>` is just one part of the machinery required.

In the context of a web application, "form" might refer to that HTML `<form>`, or to the Django [`Form`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form) that produces it, or to the structured data returned when it is submitted, or to the end-to-end working collection of these parts.

### The Django `Form` class

At the heart of this system of components is Django's [`Form`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form) class. In much the same way that a Django model describes the logical structure of an object, its behavior, and the way its parts are represented to us, a `Form` class describes a form and determines how it works and appears.

In a similar way that a model class's fields map to database fields, a form class's fields map to HTML form `<input>` elements. (A [`ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#django.forms.ModelForm) maps a model class's fields to HTML form `<input>` elements via a `Form`; this is what the Django admin is based upon.)

A form's fields are themselves classes; they manage form data and perform validation when a form is submitted. A [`DateField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.DateField) and a [`FileField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FileField) handle very different kinds of data and have to do different things with it.

A form field is represented to a user in the browser as an HTML "widget" -- a piece of user interface machinery. Each field type has an appropriate default [Widget class](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/), but these can be overridden as required.

### Instantiating, processing, and rendering forms

When rendering an object in Django, we generally:

1. Get hold of it in the view (fetch it from the database, for example).
2. Pass it to the template context.
3. Expand it to HTML markup using tempalte variables.

Rendering a form in a template involves nearly the same work as rendering any other kind of object, but there are some key differences.

In the case of a model instance that contained no data, it would rarely if ever be useful to do anything with it in a template. On the other hand, it makes perfect sense to render an unpopulated form -- that's what we do when we want the user to populate it.

So when we handle a model instance in a view, we typically retrieve it from the database. When we're dealing with a form, we typically instantiate it in the view.

When we instantiate a form, we can opt to leave it empty or prepopulate it, for example, with:

* data from a saved model instance (as in the case of admin form for editing).
* data that we have collated from other sources.
* data received from a previous HTML form submission.

The last of these cases is the most interesting, because it's what makes it possible for users not just to read a website, but to send information back to it, too.

## Building a form

### The work that needs to be done

Suppose you want to create a simple form on your website, in order to obtain the user's name. You'd need something like this in your template:
```
<form action="/your-name/" method="post">
    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
    <input type="submit" value="OK">
</form>
```
This tells the browser to return the form data to the URL `/your-name/`, using the `POST` method. It will display a text field, labeled "Your name:", and a button marked "OK". If the template context contains a `current_name` variable, that will be used to pre-fill the `your_name` field.

You'll need a view that renders the tempalte containing the HTML form, and that can supply the `current_name` field as appropriate.

When the form is submitted, the `POST` request which is sent to the server will contain the form data.

Now you'll also need a view corresponding to that `/your-name/` URL which will find the appropriate key/value pairs in the request, and then process them.

This is a very simple form. In practice, a form might contain dozens or hundreds of fields, many of which might need to be prepopulated, and we might expect the user to work through the edit-submit cycle several times before concluding the operation.

We might require some validation to occur in the browser, even before the form is submitted; we might want to use much more complex fields, that allow the user to do things like pick dates from a calendar and so on.

At this point, it's much easier to get Django to do most of this work for us.

### Building a form in Django

#### The `Form` class

We already know what we want our HTML form to look like. Our starting point for it in Django is this:

`forms.py`
```
from django import forms

class NameForm(forms.Form):
    your_name = forms.CharField(label='Your name', max_length=100)
```
This defines a [`Form`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form) class with a single field (`your_name`). We've applied a human-friendly label to the field, which will appear in the `<label>` when it's rendered (although in this case, the [`label`])(https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.Field.label) we specified is actually the same one that would be generated automatically if we had omitted it).

The field's maximum allowable length is defined by [`max_length`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField.max_length). This does two things. It puts a `maxlength="100"` on the HTML `<input>` (so the browser should prevent the user from entering more than that number of characters in the first place). It also means that when Django receives the form back from the browser, it will validate the length of the data.

A `Form` instance has an [`is_valid()`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form.is_valid) method, which runs validation routines for all its fields. When this method is called, if all fields contain valid data, it will:

* return `True`.
* place the form's data in its [`cleaned_data`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form.cleaned_data) attribute.

The whole form, when rendered for the first time, will look like:
```
<label for="your_name">Your name: </label>
<input id="your_name" type="text" name="your_name" maxlength="100" required>
```
Note that it **does not** include the `<form>` tags, or a submit button. We'll have to provide those ourselves in the template.

#### The view

Form data sent back to a Django website is processed by a view, generally the same view which published the form. This allows us to reuse some of the same logic.

To handle the form, we need to instantiate it in the view for the URL where we want to be published:

`views.py`
```
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import NameForm

def get_name(request):
    # if this is a POST request, we need to process the form data
    if request.method == 'POST':
        # create a form instance and populate it with data from the request:
        form = NameForm(request.POST)
        # check whether it's valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required
            # ...
            # redirect to a new URL:
            return HttpResponseRedirect('/thanks/')

    # if a GET (or any other method) we'll create a blank form
    else:
        form = NameForm()

    return render(request, 'name.html', {'form': form})
```
If we arrive at this view with a `GET` request, it will create an empty form instance and place it in the template context to be rendered. This is what we can expect to happen the first time we visit the URL.

If the form is submitted using a `POST` request, the view will once again create a form instance and populate it with data from the request: `form = NameForm(request.POST)`. This is called "binding data to the form" (it is now a *bound* form).

We call the form's `is_valid()` method; if it's not `True`, we go back to the template with the form. This time the form is no longer empty (*unbound*) so the HTML form will be populated with the data previously submitted, where it can be edited and corrected as required.

If `is_valid()` is `True`, we'll now be able to find all the validated form data in its `cleaned_data` attribute. We can use this data to update the database or do other processing before sending an HTTP redirect to the browser telling it where to go next.

#### The template

We don't need to do much in our `name.html` template:
```
<form action="/your-name/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```
All the form's fields and their attributes will be unpacked into HTML markup from that `{{ form }}` by Django's template language.

<hr>

**Forms and Cross Site Request Forgery protection**

Django ships with an easy-to-use [protection against Corss Site Request Forgeries](https://docs.djangoproject.com/en/4.0/ref/csrf/). When submitting a form via `POST` with CSRF protection enabled, you must use the [`csrf_token`](https://docs.djangoproject.com/en/4.0/ref/templates/builtins/#std:templatetag-csrf_token) template tag as in the preceding example. However, since CSRF protection is not directly tied to forms in templates, this tag is omitted from the following examples in this document.

<hr>

**HTML5 input types and browser validation**

If your form includes a [`URLField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.URLField), an [`EmailField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.EmailField) or any integer field type, Django will use the `url`, `email` and `number` HTML5 input types. By default, browsers may apply their own validation on these fields, which may be stricter than Django's validation. if you would like to disable this behavior, set the `novalidate` attribute on the `form` tag, or specify a different widget on the field, like [`TextInput`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.TextInput).

<hr>

We now have a working web form, described by a Django [`Form`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form), processed by a view, and rendered as an HTML `<form>`.

That's all you need to get started, but the forms framework puts a lot more at your fingertips. Once you understand the basics of the process described above, you should be prepared to understand other features of the forms system and ready to learn a bit more about the underlying machinery.

## More about Django `Form` classes

All form classes are created as subclasses of either [`django.forms.Form`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form) or [`django.forms.ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#django.forms.ModelForm). You can think of `ModelForm` as a subclass of `Form`. `Form` and `ModelForm` actually inherit common functionality from a (private) `BaseForm` class, but this implementation detail is rarely important.

<hr>

**Models and Forms**

In fact, if your form is going to be used to directly add or edit a Django model, a [`ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/) can save you a great deal of time, effort, and code, because it will build a form, along with the appropriate fields and their attributes, from a `Model` class.

<hr>

### Bound and unbound form instances

The distinction between [Bound and unbound forms](https://docs.djangoproject.com/en/4.0/ref/forms/api/#ref-forms-api-bound-unbound) is important:

* An unbound form has no data associated with it. When rendered to the user, it will be empty or will contain default values.
* A bound form has submitted data, and hence can be used to tell if that data is valid. If an invalid bound form is rendered, it can include inline error messages telling the user what data to correct.

The form's [`is_bound`](https://docs.djangoproject.com/en/4.0/ref/forms/api/#django.forms.Form.is_bound) attribute will tell you whether a form has data bound to it or not.

### More on fields

Consider a more useful form than our minimal example above, which we could use to implement "contact me" functionality on a personal website:

`forms.py`
```
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```
Our eariler form used a single field, `your_name`, a [`CharField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField). In this case, our form has four fields: `subject`, `message`, `sender`, and `cc_myself`. `CharField`, [`EmailField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.EmailField), and [`BooleanField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.BooleanField) are just three of the available field types; a full list can be found in [Form fields](https://docs.djangoproject.com/en/4.0/ref/forms/fields/).

#### Widgets

Each form field has a corresponding [Widget class](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/), which in turn corresponds to an HTML form widget such as `<input type="text">`.

In most cases, the field will have a sensible default widget. For example, by default, a [`CharField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.CharField) will have a [`TextInput`](https://docs.djangoproject.com/en/4.0/ref/forms/widgets/#django.forms.TextInput) widget, that produces an `<input type="text">` in the HTML. If you needed `<textarea>` instead, you'd specify the appropriate widget when defining your form field, as we have done for the `message` field.

#### Field data

Whatever the data submitted with a form, once it has been successfully validated by calling `is_valid()` (and `is_valid()` has returned `True`), the validated form data will be in the `form.cleaned_data` dictionary. This data  will nave been nicely converted into Python types for you.

<hr>

**Note**: You can still access the unvalidated data directly from `request.POST` at this point, but the validated data is better.

<hr>

In the contact form example above, `cc_myself` will be a Boolean value. Likewise, fields such as [`IntegerField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.IntegerField) and [`FloatField`](https://docs.djangoproject.com/en/4.0/ref/forms/fields/#django.forms.FloatField) convert values to a Python `int` and `float` respectively.

Here's how the form data could be processed in the view that handles this form:

`views.py`
```
from django.core.mail import send_mail

if form.is_valid():
    subject = form.cleaned_data['subject']
    message = form.cleaned_data['message']
    sender = form.cleaned_data['sender']
    cc_myself = form.cleaned_data['cc_myself']

    recipients = ['info@example.com']
    if cc_myself:
        recipients.append(sender)

    send_mail(subject, message, sender, recipients)
    return HttpResponseRedirect('/thanks/')
```

<hr>

**Tip**

For more on sending email from Django, see [Sending email](https://docs.djangoproject.com/en/4.0/topics/email/).

<hr>

Some field types need some extra handling. For example, files that are uploaded using a form need to be handled differently (they can be retrieved from `request.FILES`, rather than `request.POST`). For details of how to handle file uploads with your form, see [Binding uploaded files to a form](https://docs.djangoproject.com/en/4.0/ref/forms/api/#binding-uploaded-files).

## Working with form templates

All you need to do to get your form into a template is to place the form instance into the template context. So if your form is called `form` in the context, `{{ form }}` will render its `<label>` and `<input>` elements appropriately.

### Form rendering options

<hr>

**Additional form template furniture**

Don't forget that a form's output does *not* include the surrounding `<form>` tags, or the form's `submit` control. You will have to provide these yourself.

<hr>

There are other output options, though, for the `<label>`/`<input>` pairs:

* `{{ form.as_table }}` will render them as table cells wrapped in `<tr>` tags.
* `{{ form.as_p }}` will render then wrapped in `<p>` tags.
* `{{ form.as_ul }}` will render them wrapped in `<li>` tags.

Note that you'll have to provide the surrounding `<table>` or `<ul>` elements yourself.

Here's the output of `{{ form.as_p }}` for our `ContactForm` instance:
```
<p><label for="id_subject">Subject:</label>
    <input id="id_subject" type="text" name="subject" maxlength="100 required>
</p>
<p><label for="id_message">Message:</label>
    <textarea name="message" id="id_message" required></textarea>
</p>
<p><label for="id_sender">Sender:</label>
    <input type="email" name="sender" id="id_sender" required>
</p>
<p><label for="id_cc_myself">Cc myself:</label>
    <input type="checkbox" name="cc_myself" id="id_cc_myself">
</p>
```
Note that each form has an ID attribute set to `id_<field-name>`, which is referenced by the accompanying label tag. This is important in ensuring that forms are accessible to assistive technology such as screen reader software. You can also [customize the way in which labels and ids are generated](https://docs.djangoproject.com/en/4.0/ref/forms/api/#ref-forms-api-configuring-label).

See [Outputting forms as HTML](https://docs.djangoproject.com/en/4.0/ref/forms/api/#ref-forms-api-outputting-html) for more on this.

### Rendering fields manually

