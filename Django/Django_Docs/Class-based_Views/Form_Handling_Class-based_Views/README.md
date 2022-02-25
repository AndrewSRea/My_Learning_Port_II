# Form handling with class-based views

Form processing generally has 3 paths:

* Initial `GET` (blank or prepopulated form)
* `POST` with invalid data (typically redisplay form with errors)
* `POST` with valid data (process the data and typically redirect)

Implementing this yourself often results in a lot of repeated boilerplate code (see [Using a form in a view](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Working_with_Forms#the-view)). To help avoid this, Django provides a collection of generic class-based views for form processing.

## Basic forms

Given a contact form:
```
# forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField()
    message = forms.CharField(widget=forms.Textarea)

    def send_email(self):
        # send email using the self.cleaned_data dictionary
        pass
```
The view can be constructed using a `FormView`:
```
# views.py
from myapp.forms import ContactForm
from django.views.generic.edit import FormView

class ContactFormView(FormView):
    template_name = 'contact.html'
    form_class = ContactForm
    success_url = '/thanks/'

    def form_valid(self, form):
        # This method is called when valid form data has been POSTed.
        # It should return an HttpResponse.
        form.send_email()
        return super().form_valid(form)
```
Notes:

* `FormView` inherits [`TemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin) so [`template_name`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.template_name) can be used here.
* The default implementation for [`form_valid()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.FormMixin.form_valid) simply redirects to the [`success_url`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.FormMixin.success_url).

## Model forms

Generic views really shine when working with models. These generic views will automatically create a [`ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#django.forms.ModelForm), so long as they can work out which model class to use:

* If the [`model`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.ModelFormMixin.model) attribute is given, that model class will be used.
* If [`get_object()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_object) returns an object, the class of that object will be used.
* If a [`queryset`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.queryset) is given, the model for that queryset will be used.

Model form views provide a [`form_valid()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.ModelFormMixin.form_valid) implementation that saves the model automatically. You can override this if you have any special requirements; see below for examples.

You don't even need to provide a `success_url` for [`CreateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.CreateView) or [`UpdateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.UpdateView) -- they will use [`get_absolute_url()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.get_absolute_url) on the model object if available.

If you want to use a custom [`ModelForm`](https://docs.djangoproject.com/en/4.0/topics/forms/modelforms/#django.forms.ModelForm) (for instance, to add extra validation), set [`form_class`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.FormMixin.form_class) on your view.

<hr>

**Note**: When specifying a custom form class, you must still specify the model, even though the `form_class` may be a `ModelForm`.

<hr>

First, we need to add [`get_absolute_url()`](https://docs.djangoproject.com/en/4.0/ref/models/instances/#django.db.models.Model.get_absolute_url) to our `Author` class:
```
# models.py
from django.db import models
from django.urls import reverse

class Author(models.Model):
    name = models.CharField(max_length=200)

    def get_absolute_url(self):
        return reverse('author-detail', kwargs={'pk': self.pk})
```
Then we can use [`CreateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/flattened-index/#CreateView) and friends to do the actual work. Notice how we're just configuring the generic class-based views here; we don't have to write any logic ourselves:
```
# views.py
from django.urls import reverse_lazy
from django.views.generic.edit import CreateView, DeleteView, UpdateView
from myapp.models import Author

class AuthorCreateView(CreateView):
    model = Author
    fields = ['name']

class AuthorUpdateView(UpdateView):
    model = Author
    fields = ['name']

class AuthorDeleteView(DeleteView):
    model = Author
    success_url = reverse_lazy('author-list')
```