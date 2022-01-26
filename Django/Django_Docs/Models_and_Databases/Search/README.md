# Search

A common task for web applications is to search some data in the database with user input. In a simple case, this could be filtering a list of objects by a category. A more complex use case might require searching with weighting, categorization, highlighting, multiple languages, and so on. This document explains some of the possible use cases and the tools you can use.

We'll refer to the same models using in [Making queries](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Making_Queries#making-queries).

## Use cases

### Standard textual queries

Text-based fields have a selection of matching operations. For example, you may wish to allow lookup on an author like so:
```
>>> Author.objects.filter(name__contains='Terry')
[<Author: Terry Gilliam>, <Author: Terry Jones>]
```
This is a very fragile solution as it requires the user to know an exact substring of the author's name. A better approach could be a case-insensitive match ([`icontains`](https://docs.djangoproject.com/en/4.0/ref/models/querysets/#std:fieldlookup-icontains)), but this is only marginally better.

### A database's more advanced comparison functions

If you're using PostgreSQL, Django provides [a selection of database specific tools]() <!-- future folder? (https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/search/) --> to allow you to leverage more complex querying options. Other databases have different selections of tools, possibly via plugins or user-defined functions. Django doesn't include any support for them at this time. We'll use some examples from PostgreSQL to demonstrate the kind of functionality databases may have.

<hr>

**mSearching in other databases**

All of the searching tools provided by [`django.contrib.postgres`](https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/#module-django.contrib.postgres) are constructed entirely on public APIs such as [custom lookups](https://docs.djangoproject.com/en/4.0/ref/models/lookups/) and [database functions](https://docs.djangoproject.com/en/4.0/ref/models/database-functions/). Depending on your database, you should be able to construct queries to allow similar APIs. If there are specific things which cannot be achieved this way, please [open a ticket on the DjangoProject website](https://code.djangoproject.com/).

<hr>

In the above example, we determined that a case insensitive lookup would be more useful. When dealing with non-English names, a further improvement is to use [`unaccent`ed comparison](https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/lookups/#unaccent).
```
>>> Author.objects.filter(name__unaccent__icontains='Helen')
[<Author: Hellen Mirren>, <Author: Helena Bonham Carter>, <Author: Hélène Joy>]
```
This shows another issue, where we are matching against a different spelling of the name. In this case, we have an asymmetry, though -- a search for `Helen` will pick up `Helena` or `Hélène`, but not the reverse. Another option would be to use a [`trigram_similar`](https://docs.djangoproject.com/en/4.0/ref/contrib/postgres/lookups/#std:fieldlookup-trigram_similar) comparison, which compares sequences of letters.

For example:
```
>>> Author.objects.filter(name__unaccent__lower__trigram_similar='Hélène')
[<Author: Helen Mirren>, <Author: Hélène Joy>]
```
Now we have a different problem -- the longer name of "Helena Bonham Carter" doesn't show up as it is much longer. Trigram searches consider all combinations of three letters, and compares how may appear in both search and source strings. For the longer name, there are more combinations that don't appear in the source string, so it is no longer considered a close match.

The correct choice of comparison functions here depends on your particular data set -- for example, the language(s) used and the type of text being searched. All of the examples we've seen are on short strings where the user is likely to enter something close (by varying definitions) to the source data.