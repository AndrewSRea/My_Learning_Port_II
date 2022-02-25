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