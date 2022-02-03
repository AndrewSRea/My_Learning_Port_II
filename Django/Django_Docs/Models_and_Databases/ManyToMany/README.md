# Examples of model relationship API usage

## Many-to-many relationships

To define a many-to-many relationship, use [`ManyToManyField`](https://docs.djangoproject.com/en/4.0/ref/models/fields/#django.db.models.ManyToManyField).

In this example, an `Article` can be published in multiple `Publication` objects, and a `Publication` has multiple `Article` objects:
```
from django.db import models

class Publication(models.Model):
    title = models.CharField(max_length=30)

    class Meta:
        ordering = ['title']

    def __str__(self):
        return self.title

class Article(models.Model):
    headline = models.CharField(max_length=100)
    publications = models.ManyToManyField(Publication)

    class Meta:
        ordering = ['headline']

    def __str__(self):
        return self.headline
```
What follows are examples of operations that can be performed using the Python API facilities.

Create a few `Publications`.
```
>>> p1 = Publication(title='The Python Journal')
>>> p1.save()
>>> p2 = Publication(title='Science News')
>>> p2.save()
>>> p3 = Publication(title='Science Weekly')
>>> p3.save()
```
Create an `Article`.
```
>>> a1 = Article(headline='Django lets you build web apps easily')
```
You can't associate it with a `Publication` until it's been saved:
```
>>> a1.publications.add(p1)
Traceback (most recent call last):
...
ValueError: "<Article: Django lets you build web apps easily>" needs to have a value for field "id" before this many-to-many relationship can be used.
```
Save it!
```
>>> a1.save()
```
Associate the `Article` with a `Publication`:
```
>>> a1.publications.add(p1)
```
Create another `Article`, and set it to appear in the `Publications`:
```
>>> a2 = Article(headline='NASA uses Python')
>>> a2.save()
>>> a2.publications.add(p1, p2)
>>> a2.publications.add(p3)
```
Adding a second time is OK, it will not duplicate the relation:
```
>>> a2.publications.add(p3)
```
Adding an object of the wrong type raises `TypeError`:
```
>>> a2.publications.add(a1)
Traceback (most recent call last):
...
TypeError: 'Publication' instance expected
```
Create and add a `Publication` to an `Article` in one step using `create()`:
```
>>> new_publication = a2.publications.create(title='Highlights for Children')
```
`Article` objects have access to their related `Publication` objects:
```
>>> a1.publications.all()
<QuerySet [<Publication: The Python Journal>]>
>>> a2.publications.all()
<QuerySet [<Publication: Highlights for Children>, <Publication: Science News>, <Publication: Science Weekly>, <Publication: The Python Journal>]>
```
`Publication` objects have access to their related `Article` objects:
```
>>> p2.article_set.all()
<QuerySet [<Article: NASA uses Python>]>
>>> p1.article_set.all()
<QuerySet [<Article: Django lets you build web apps easily>, <Article: NASA uses Python>]>
>>> Publication.objects.get(id=4).article_set.all()
<QuerySet [<Article: NASA uses Python>]>
```
Many-to-many relationships can be queried using [lookups across relationships](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#lookups-that-span-relationships):