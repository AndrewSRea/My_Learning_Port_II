# Writing your first Django app - Part 7

This tutorial begins where [Tutorial 6](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_6#writing-your-first-django-app---part-6) left off. We're continuing the Web-poll application and will focus on customizing Django's automatically-generated admin site that we first explored in [Tutorial 2](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_App_Part_2#writing-your-first-django-app---part-2).

<hr>

**Where to get help**: If you're having trouble going through this tutorial, please head over to the [Getting Help](https://docs.djangoproject.com/en/3.1/faq/help/) section of the FAQ.

<hr>

## Customize the admin form

By registering the `Question` model with `admin.site.register(Question)`, Django was able to construct a default form representation. Often, you'll want to customize how the admin form looks and works. You'll do this by telling Django the options you want when you register the object.

Let's see how this works by reordering the fields on the edit form. Replace the `admin.site.register(Question)` line with:

`polls/admin.py`

```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

admin.site.register(Question, QuestionAdmin)
```

You'll follow this pattern -- create a model admin class, then pass it as the second argument to `admin.site.register()` -- any time you need to change the admin options for a model.

This particular change above makes the "Publication date" come before the "Question" field:

![Image of the "Change question" page on the admin site](https://docs.djangoproject.com/en/3.1/_images/admin07.png)

This isn't impressive with only two fields, but for admin forms with dozens of fields, choosing an intuitive order is an important usability detail.

And speaking of forms with dozens of fields, you might want to split the form up into fieldsets:

`polls/admin.py`

```
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question, QuestionAdmin)
```

The first element of each tuple in [`fieldsets`](https://docs.djangoproject.com/en/3.1/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets) is the title of the fieldset. Here's what our form looks like now:

![Image of the updated "Change question" page on the admin site](https://docs.djangoproject.com/en/3.1/_images/admin08t.png)

## Adding related objects

OK, we have our Question admin page, but a `Question` has multiple `Choices`, and the admin page doesn't display choices. 

Yet.

There are two ways to solve this problem. The first is to register `Choice` with the admin just as we did with `Question`:

`polls/admin.py`

```
from django.contrib import admin

from .models import Choice, Question
# ...
admin.site.register(Choice)
```

Now "Choices" is an available option in the Django admin. The "Add choice" form looks like this:

![Image of the "Choices" page on the admin site](https://docs.djangoproject.com/en/3.1/_images/admin09.png)

In that form, the "Question" field is a select box containing every question in the database. Django knows that a [`ForeignKey`](https://docs.djangoproject.com/en/3.1/ref/models/fields/#django.db.models.ForeignKey) should be represented in the admin as a `<select>` box. In our case, only one question exists at this point.

Also, note the "Add Another" link next to "Question". Every object with a `ForeignKey` relationship to another gets this for free. When you click "Add Another", you'll get a popup window with the "Add question" form. If you add a question in that window and click "Save", Django will save the question to the database and dynamically add it as the selected choice on the "Add choice" form you're looking at.

