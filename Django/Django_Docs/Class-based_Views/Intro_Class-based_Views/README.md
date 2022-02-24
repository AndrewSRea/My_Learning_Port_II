# Introduction to class-based views

Class-based views provide an alternative way to implement views as Python objects instead of functions. They do not replace function-based views, but have certain differences and advantages when compared to function-based views:

* Organization of code related to specific HTTP methods (`GET`, `POST`, etc.) can be addressed by separate methods instead of conditional branching.
* Object oriented techniques such as mixins (multiple inheritance) can be used to factor code into reusable components.

## The relationship and history of generic views, class-based views, and class-based generic views

In the beginning, there was only the view function contract, Django passed your function an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) and expected back an [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse). This was the extent of what Django provided.

Early on, it was recognized that there were common idioms and patterns found in view development. Function-based generic views were introduced to abstract these patterns and ease view development for the common cases.

The problem with function-based generic views is that while they covered the simple cases well, there was no way to extend or customize them beyond some configuration options, limiting their usefulness in many real-world applications.

Class-based generic views were created with the same objective as function-based generic views, to make view development easier. However, the way the solution is implemented, through the use of mixins, provides a toolkit that results in class-based generic views being more extensible and flexible than their function-based counterparts.

If you have tried function-based generic views in the past and found them lacking, you should not think of class-based generic view as a class-based equivalent, but rather as a fresh approach to solving the original problems that generic views were meant to solve.

The toolkit of base classes and mixins that Django uses to build class-based generic views are built for maximum flexibility, and as such have many hooks in the form of default method implementations and attributes that you are unlikely to be concerned with in the simplest use cases. For example, instead of limiting you to a class-based attribute for `form_class`, the implementation uses a `get_form` method, which calls a `get_form_class` method, which in its default implementation returns the `form_class` attribute of the class. This gives you several options for specifying what form to use, from an attribute, to a fully dynamic, callable hook. These options seem to add hollow complexity for siomple situations, but without them, more advanced designs would be limited.

## Using class-based views

At its core, a class-based view allows you to respond to different HTTP request methods with different class instance methods, instead of with conditionally branching code inside a single view function.

So where the code to handle HTTP `GET` in a view function would look something like:
```
from django.http import HttpResponse

def my_view(request):
    if request.method == 'GET':
        # <view logic>
        return HttpResponse('result')
```
In a class-based view, this would become:
```
from django.http import HttpResponse
from django.view import View

class MyView(View):
    def get(self, request):
        # <view logic>
        return HttpResponse('result')
```
Because Django's URL resolver expects to send the request and associated arguments to a callable function, not a class, class-based views have an [`as_view()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View.as_view) class method which returns a function that can be called when a request arrives for a URL matching the associated pattern. The function creates an instance of the class, calls [`setup()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View.setup) to initialize its attributes, and then calls its [`dispatch()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View.dispatch) method. `dispatch` looks at the request to determine whether it is a `GET`, `POST`, etc., and relays the request to a matching method if one is defined, or raises [`HttpResponseNotAllowed`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponseNotAllowed) if not:
```
# urls.py
from django.urls import path
from myapp.views import MyView

urlpatterns = [
    path('about/', MyView.as_view()),
]
```
It is worth noting that what your method returns is identical to what you return from a function-based view, namely some form of [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse). This means that [http shortcuts](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Shortcut_Functions#django-shortcut-functions) or [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) objects are valid to use inside a class-based view.

While a minimal class-based view does not require any class attributes to perform its job, class attributes are useful in many class-based designs, and there are two ways to configure or set class attributes.

The first is the standard Python way of subclassing and overriding attributes and methods in the subclass. So that if your parent class has an attribute `greeting` like this:
```
from django.http import HttpResponse
from django.views import View

class GreetingView(View):
    greeting = "Good Day"

    def get(self, request):
        return HttpResponse(self.greeting)
```
You can override that in a subclass:
```
class MorningGreetingView(GreetingView):
    greeting = "Morning to ya"
```
Another option is to configure class attributes as keyword arguments to the [`as_view()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View.as_view) call in the URLconf:
```
urlpatterns = [
    path('about/', GreetingView.as_view(greeting="G'day")),
]
```

<hr>

**Note**: While your class is instantiated for each request dispatched to it, class attributes set through the `as_view()` entry point are configured only once at the time your URLs are imported.

<hr>

## Using mixins

Mixins are a form of multiple inheritance where behaviors and attributes of multiple parent classes can be combined.

For example, in the generic class-based views there is a mixin called [`TemplateResponseMixin`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin) whose primary purpose is to define the method [`render_to_response()`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.render_to_response). When combined with the behavior of the [`View`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View) base class, the result is a [`TemplateView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.TemplateView) class that will dispatch requests to the appropriate matching methods (a behavior defined in the `View` base class), and that has a `render_to_response()` method that uses a [`template_name`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.template_name) attribute to return a [`TemplateResponse`](https://docs.djangoproject.com/en/4.0/ref/template-response/#django.template.response.TemplateResponse) object (a behavior defined in the `TemplateResponseMixin`).

Mixins are an excellent way of reusing code across multiple classes, but they come with some cost. The more your code is scattered among mixins, the harder it will be to read a child class and know what exactly it is doing, and the harder it will be to know which methods from which mixins to override if you are subclassing something that has a deep inheritance tree.

Note also that you can only inherit from one generic view -- that is, only one parent class may inherit from [`View`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/base/#django.views.generic.base.View) and the rest (if any) should be mixins. Trying to inherit from more than one class that inherits from `View` -- for example, trying to use a form at the top of a list and combining [`ProcessFormView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/mixins-editing/#django.views.generic.edit.ProcessFormView) and [`ListView`](https://docs.djangoproject.com/en/4.0/ref/class-based-views/generic-display/#django.views.generic.list.ListView) -- won't work as expected.