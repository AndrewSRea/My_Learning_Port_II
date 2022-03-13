# Using the Django authentication system

This document explains the usage of Django's authentication system in its default configuration. This configuration has evolved to serve the most common project needs, handling a reasonably wide range of tasks, and has a careful implementation of passwords and permissions. For projects where authentication needs differ from the default, Django supports extensive [extension and customization]() of authentication.

Django authentication provides both authentication and authorization together and is generally referred to as the authentication system, as these features are somewhat coupled.

## `User` objects

[`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) objects are the core of the authentication system. They typically represent the people interacting with your site and are used to enable things like restricting access, registering user profiles, associating content with creators, etc. Only one class of user exists in Django's authentication framework, i.e. [`'superusers'`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.is_superuser) or admin [`'staff'`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.is_staff) users, are just user objects with special attrbiutes set, not different classes of user objects.

The primary attributes of the default user are:

* [`username`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.username)
* [`password`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.password)
* [`email`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.email)
* [`first_name`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.first_name)
* [`last_name`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.last_name)

See the [full API documentation](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) for full reference. The documentation that follows is more task oriented.

### Creating users

The most direct way to create users is to use the included [`create_user()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.UserManager.create_user) helper function:
```
>>> from django.contrib.auth.models import User
>>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')

# At this point, user is a User object that has already been saved
# to the database. You can continue to change its attributes
# if you want to change other fields.
>>> user.last_name = 'Lennon'
>>> user.save()
```
If you have the Django admin installed, you can also [create users interactively](). <!-- below -->

### Creating superusers

Create superusers using the [`createsuperuser`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-createsuperuser) command:
```
$ python manage.py createsuperuser --username=joe --email=joe@example.com
```
You will be prompted for a password. After you enter one, the user will be created immediately. If you leave off the [`--username`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-createsuperuser-username) or [`--email`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-createsuperuser-email) options, it will prompt you for those values.

### Changing passwords

Django does not store raw (clear text) passwords on the user model, but only a hash (see [documentation of how passwords are managed]() <!-- next page --> for full details). Because of this, do not attempt to manipulate the password attribute of the user directly. This is why a helper function is used when creating a user.

To change a user's password, you have several options:

[`manage.py changepassword *username*`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-changepassword) offers a method of changing a user's password from the command line. It prompts you to change the password of a given user which you must enter twice. If they both match, the new password will be changed immediately. If you do not supply a user, the command will attempt to change the password whose username matches the current system user.

You can also change a password programmatically, using [`set_password()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.set_password):
```
>>> from django.contrib.auth.models import User
>>> u = User.objects.get(username='john')
>>> u.set_password('new password')
>>> u.save()
```
If you have the Django admin installed, you can also change user's passwords on the [authentication system's admin pages](). <!-- below -->

Django also provides [views]() and [forms]() that may be used to allow users to change their own passwords. <!-- both below -->

Changing a user's password will log out all their sessions. See [Session invalidation on password change]() for details. <!-- below -->