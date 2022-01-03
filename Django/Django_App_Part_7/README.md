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

But, really, this is an inefficient way of adding `Choice` objects to the system. It'd be better if you could add a bunch of Choices directly when you create the `Question` object. Let's make that happen.

Remove the `register()` call for the `Choice` model. Then, edit the `Question` registration code to read:

`polls/admin.py`

```
from django.contrib import admin

from .models import Choice, Question


class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse]}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

This tells Django: "`Choice` objects are edited on the `Question` admin page. By default, provide enough fields for 3 choices."

Load the "Add question" page to see how that looks:

![Image of the updated "Add question" page on the admin site](https://docs.djangoproject.com/en/3.1/_images/admin10t.png)

It works like this: There are three slots for related Choices -- as specified by `extra` -- and each time you come back to the "Change" page for an already-created object, you get another three extra slots.

At the end of the three current slots, you will find an "Add another Choice" link. If you click on it, a new slot will be added. If you want to remove the added slot, you can click on the "X" to the top right of the added slot. This image shows an added slot:

![Image of an added slot on the "Add question" page of the admin site](https://docs.djangoproject.com/en/3.1/_images/admin14t.png)

One small problem, though. It takes a lot of screen space to display all the fields for entering related `Choice` objects. For that reason, Django offers a tabular way of displaying inline related objects. To use it, change the `ChoiceInline` declaration to read:

`polls/admin.py`

```
class ChoiceInline(admin.TabularInline):
    # ...
```

With that `TabularInline` (instead of `StackedInline`), the related objects are displayed in a more compact, table-based format:

![Image of the updated "Choices" page with "TabularInline" objects on the page](https://docs.djangoproject.com/en/3.1/_images/admin11t.png)

Note that there is an extra "Delete?" column that allows removing rows added using the "Add Another Choice" button and rows that have already been saved.

## Customize the admin change list