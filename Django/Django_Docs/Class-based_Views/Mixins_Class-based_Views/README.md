# Using mixins with class-based views

<hr>

:exclamation: **Caution**: This is an advanced topic. A working knowledge of [Django's class-based views](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Class-based_Views#class-based-views) is advised before exploring these techniques.

<hr>

Django's built-in class-based views provide a lot of functionality, but some of it you may want to use separately. For instance, you may want to write a view that renders a template to make the HTTP response, but you can't use [`TemplateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.TemplateView); perhaps you need to render a template only on `POST`, with `GET` doing something else entirely. While you could use [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) directly, this will likely result in duplicate code.

For this reason, Django also provides a number of mixins that provide more discrete functionality. Template rednering, for instance, is encapsulated in the [`TemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin). The Django reference documentation contains [full documentation of all the mixins](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins/).

## Context and template responses

Two central mixins are provided that help in providing a consistent interface to working with templates in class-based views.

##### [`TemplateResponseMixin`]((https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin)

Every built-in view which returns a [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) will call the [`render_to_response()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.render_to_response) method that `TemplateResponseMixin` provides. Most of the time, this will be called for you (for instance, it is called by the `get()` method implemented by both [`TemplateView'](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.TemplateView) and [`DetailView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.detail.DetailView)); similarly, it's unlikely that you'll need to override it, although if you want your response to return something not rendered via a Django template, then you'll want to do it. For an example of this, see the [`JSONResponseMixin` example](). <!-- below -->

`render_to_response()` itself calls [`get_template_names()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.get_template_names), which, by default, will look up [`template_name`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.template_name) on the class-based view; two other mixins ([`SingleObjectTemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectTemplateResponseMixin) and [`MultipleObjectTemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectTemplateResponseMixin)) override this to provide more flexible defaults when dealing with actual objects.

##### [`ContextMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.ContextMixin)

Every built-in view which needs context data, such as for rendering a template (including `TemplateResponseMixin` above), should call [`get_context_data()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.ContextMixin.get_context_data) passing any data they want to ensure is in there as keyword arguments. `get_context_data()` returns a dictionary; in `ContextMixin`, it returns its keyword arguments, but it is common to override this to add more members to the dictionary. You can also use the [`extra_context`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.ContextMixin.extra_context) attribute.

## Building up Django's generic class-based views

Let's look at how two of Django's generic class-based views are built out of mixins providing discrete functionality. We'll consider [`DetailView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.detail.DetailView), which renders a "detail" view of an object, and [`ListView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.list.ListView), which will render a list of objects, typically from a queryset, and optionally paginate them. This will introduce us to four mixins which between them provide useful functionality when working with either a single Django object, or multiple objects.

There are also mixins involved in the generic edit views ([`FormView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.FormView), and the model-specific views [`CreateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.CreateView), [`UpdateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.UpdateView) and [`DeleteView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-editing/#django.views.generic.edit.DeleteView)), and in the date-based generic views. These are covered in the [mixin reference documentation](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins/).

### `DetailView`: working with a single Django object

To show the detail of an object, we basically need to do two things: we need to look up the object and then we need to make a [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) with a suitable template, and that object as context.

To get the object, [`DetailView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.detail.DetailView) relies on [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin), which provides a [`get_object()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_object) method that figures out the object based on the URL of the request (it looks for `pk` and `slug` keyword arguments as declared in the URLconf, and looks the object up either from the [`model`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.model) attribute on the view, or the [`queryset`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.queryset) attribute if that's provided). `SingleObjectMixin` also overrides [`get_context_data()`](), which is used across all Django's built-in class-based views to supply context data for template renders.

To then make a `TemplateResponse`, `DetailView` uses [`SingleObjectTemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectTemplateResponseMixin), which extends [`TemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin), overriding [`get_template_names()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.get_template_names) as discussed above. It actually provides a fairly sophisticated set of options, but the main one that most people are going to use is `<app_label>/<model_name>_detail.html`. The `_detail` part can be changed by setting [`template_name_suffix`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectTemplateResponseMixin.template_name_suffix) on a subclass to something else. (For instance, the [generic edit views](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Class-based_Views/Form_Handling_Class-based_Views#form-handling-with-class-based-views) use `_form` for create and update views, and `_confirm_delete` for delete views.)

### `ListView`: working with many Django objects

Lists of objects follow roughly the same pattern: we need a (possibly paginated) list of objects, typically a [`QuerySet`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#django.db.models.query.QuerySet), and then we need to make a [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) with a suitable template using that list of objects. 

To get the objects, [`ListView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.list.ListView) uses [`MultipleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin), which provides both [`get_queryset()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.get_queryset) and [`paginate_queryset()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.paginate_queryset). Unlike with [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin), there's no need to key off parts of the URL to figure out the queryset to work with, so the default uses the [`queryset`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.queryset) or [`model`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.model) attribute on the view class. A common reason to override `get_queryset()` here would be to dynamically vary the objects, such as depending on the current user or to exclude posts in the future for a blog.

`MultipleObjectMixin` also overrides [`get_context_data()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.ContextMixin.get_context_data) to include appropriate context variables for pagination (providing dummies if pagination is disable). It relies on `object_list` being passed in as a keyword argument, which `ListView` arranges for it.

To make a `TemplateResponse`, `ListView` then uses [`MultipleObjectTemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectTemplateResponseMixin); as with [`SingleObjectTemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectTemplateResponseMixin) above, this overrides `get_template_names()` to provide [a range of options](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectTemplateResponseMixin), with the most commonly-used being `<app_label>/<model_name>_list.html`, with the `_list` part again being taken from the [`template_name_suffix`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectTemplateResponseMixin.template_name_suffix) attribute. (The date-based generic views use suffixes such as `_archive`, `_archive_year`, and so on to use different templates for the various specialized date-based list views.)

## Using Django's class-based view mixins

Now we've seen how Django's generic class-based views use the provided mixins, let's look at other ways we can combine them. We're still going to be combining them with either built-in class-based views, or other generic class-based views, but there are a range of rarer problems you can solve than are provided for by Django out of the box.

<hr>

:warning: **Warning**: Not all mixins can be used together, and not all generic class-based views can be used with all other mixins. Here we present a few examples that do work; if you want to bring together other functionality, then you'll have to consider interactions between attributes and methods that overlap between the different classes you're using, and how [method resolution order](https://www.python.org/download/releases/2.3/mro/) will affect which versions of the methods will be called in what order.

The reference documentation for Django's [class-based views](https://docs.djangoproject.com/en/4.0/ref/class-based-views/) and [class-based view mixins](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins/) will help you in understanding which attributes and methods are likely to cause conflict between different classes and mixins.

If in doubt, it's often better to back off and base your work on [`View`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/flattened-index/#View) or [`TemplateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/flattened-index/#TemplateView), perhaps with[`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin) and [`MultipleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin). Although you will probably end up writing more code, it is more likely to be clearly understandable to someone else coming to it later, and with fewer interactions to worry about, you will save yourself some thinking. (Of course, you can always dip into Django's implementation of the generic class-based views for inspiration on how to tackle problems.)

<hr>

### Using `SingleObjectMixin` with `View`

If we want to write a class-based view that responds only to `POST`, we'll subclass [`View`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View) and write a `post()` method in the subclass. However, if we want our processing to work on a particular object, identified from the URL, we'll want the functionality provided by [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin).

We'll demonstrate this with the `Author` model we used in the [generic class-based views introduction](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Class-based_Views/Built-in_Class-based_Views#built-in-class-based-generic-views):
```
# views.py
from django.http import HttpResponseForbidden, HttpResponseRedirect
from django.urls import reverse
from django.views import View
from django.views.generic.detail import SingleObjectMixin
from books.models import Author

class RecordInterestView(SingleObjectMixin, View):
    """Records the current user's interest in an author."""
    model = Author

    def post(self, request, *args, **kwargs):
        if not request.user.is_authenticated:
            return HttpResponseForbidden()

        # Look up the author we're interested in.
        self.object = self.get_object()
        
        # Actually record interest somehow here!

        return HttpResponseRedirect(reverse('author-detail', kwargs={'pk': self.object.pk}))
```
In practice, you'd probably want to record the interest in a key-value store rather than in a relational database, so we've left that bit out. The only bit of the view that needs to worry about using [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin) is where we want to look up the author we're interested in, which it does with a call to `self.get_object()`. Everything else is taken care of for us by the mixin.

We can hook this into our URLs easily enough:
```
# urls.py
from django.urls import path
from books.views import RecordInterestView

urlpatterns = [
    # ...
    path('author/<int:pk>/interest/', RecordInterestView.as_view(), name='author-interest'),
]
```
Note the `pk` named group, which [`get_object()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_object) uses to look up the `Author` instance. You could also use a slug, or any of the other features of `SingleObjectMixin`.

### Using `SingleObjectMixin` with `ListView` 

[`ListView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.list.ListView) provides built-in pagination, but you might want to paginate a list of objects that are all linked (by a foreign key) to another object. In our publishing example, you might want to paginate through all the books by a particular publisher.

One way to do this is to combine `ListView` with [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin), so that the queryset for the paginated list of books can hang off the publisher found as the single object. In order to do this, we need to have two different querysets:

##### `Book` queryset for use by `ListView`

Since we have access to the `Publisher` whose books we want to list, we override `get_queryset()` and use the `Publisher`'s [reverse foreign key manager](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#following-relationships-backward).

##### `Publisher` queryset for use in [`get_object()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_object)

We'll rely on the default implementation of `get_object()` to fetch the correct `Publisher` object. However, we need to explicitly pass a `queryset` argument because otherwise the default implementation of `get_object()` would call `get_queryset()` which we have overridden to return `Book` objects instead of `Publisher` ones.

<hr>

**Note**: We have to think carefully about `get_context_data()`. Since both [`SingleObjectMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin) and [`ListView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/flattened-index/#ListView) will put things in the context data under the value of `context_object_name` if it's set, we'll instead explicitly ensure the `Publisher` is in the context data. `ListView` will add in the suitable `page_obj` and `paginator` for us providing we remember to call `super()`.

<hr>

Now we can write a new `PublisherDetailView`:
```
from django.views.generic import ListView
from django.views.generic.detail import SingleObjectMixin
from books.models import Publisher

class PublisherDetailView(SingleObjectMixin, ListView):
    paginate_by = 2
    template_name = "books/publisher_detail.html"

    def get(self, request, *args, **kwargs):
        self.object = self.get_object(queryset=Publisher.objects.all())
        return super().get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['publisher'] = self.object
        return context

    def get_queryset(self):
        return self.object.book_set.all()
```
Notice how we set `self.object` within `get()` so we can use it again later in `get_context_data()` and `get_queryset()`. If you don't set `template_name`, the template will default to the normal [`ListView`]() choice, which in this case would be `"books/book_list.html"` because it's a list of books; `ListView` knows nothing about [`SingleObjectMixin`](), so it doesn't have any clue this view is anything to do with a `Publisher`.

The `paginate_by` is deliberately small in the example so you don't have to create lots of books to see the pagination working! Here's the tempalte you'd want to use:
```
{% extends "base.html" %}

{% block content %}
    <h2>Publisher {{ publisher.name }}</h2>

    <ol>
    {% for book in page_obj %}
        <li>{{ book.title }}</li>
    {% endfor %}
    </ol>

    <div class="pagination">
        <span class="step-links">
            {% if page_obj.has_previous %}
                <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
            {% endif %}

            <span class="current">
                Page {{ page_obj.number }} of {{ paginator.num_pages }}.
            </span>

            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">Next</a>
            {% endif %}
        </span>
    </div>
{% endblock %}
```
