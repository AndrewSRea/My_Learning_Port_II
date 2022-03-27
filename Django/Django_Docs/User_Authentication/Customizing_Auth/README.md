# Customizing authentication in Django

The authentication that comes with Django is good enough for most common cases, but you may have needs not met by the out-of-the-box defaults. Customizing authentication in your projects requires understanding what points of the provided system are extensible or replaceable. This document provides details about how the auth system can be customized.

[Authentication backends]() <!-- "Other authentication sources" below --> provide an extensible system for when a username and password stored with the user model need to be authenticated against a different service than Django's default.

You can give your models [custom permissions]() <!-- same title below --> that can be checked through Django's authorization system.

You can [extend]() <!-- "Extending the existing..." below --> the default `User` model, or [substitute]() <!-- "Substituting a custom..." below --> a completely customized model.

## Other authentication sources

There may be times you have the need to hook into another authentication source -- that is, another source of usernames and passwords or authentication methods.

For example, your company may already have an LDAP setup that stores a username and password for every employee. It'd be a hassle for both the network administrator and the users themselves if users had separate accounts in LDAP and the Django-based applications.

So, to handle situations like this, the Django authentication system lets you plug in other authentication sources. You can override Django's default database-based scheme, or you can use the default system in tandem with other systems.

See the [authentication backend reference](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#authentication-backends-reference) for information on the authentication backends included with Django.

### Specifying authentication backends

Behind the scenes, Django maintains a list of "authentication backends" that it checks for authentication. When somebody calls [`django.contrib.auth.authenticate()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#authenticaterequestnone-credentials) -- as described in [How to log a user in](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#how-to-log-a-user-in) -- Django tries authenticating across all of its authentication backends. If the first authentication method fails, Django tries the second one, and so on, until all backends have been attempted.

The list of authentication backends to use is specified in the [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS) setting. This should be a list of Python path names that point to Python classes that know how to authenticate. These classes can be anywhere on your Python path.

By default, `AUTHENTICATION_BACKENDS` is set to:
```
['django.contrib.auth.backends.ModelBackend']
```
That's the basic authentication backend that checks the Django users database and queries the built-in permissions. It does not provide protection against brute force attacks via any rate limiting mechanism. You may either implement your own rate limiting mechanism in a custom auth backend, or use the mechanisms provided by most web servers.

The order of [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS) matters, so if the same username and password is valid in multiple backends, Django will stop processing at the first positive match.

If a backend raises a [`PermissionDenied`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.PermissionDenied) exception, authentication will immediately fail. Django won't check the backends that follow.

<hr>

**Note**: Once a user has authenticated, Django stores which backend was used to authenticate the user in the user's session, and re-uses the same backend for the duration of that session whenever access to the currently authenticated user is needed. This effectively means that authentication sources are cached on a per-session basis, so if you change [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS), you'll need to clear out session data if you need to force users to re-authenticate using different methods. A simple way to do that is to execute `Session.objects.all().delete()`.

<hr>

### Writing an authentication backend

An authentication backend is a class that implements two required methods: `get_user(user_id)` and `authenticate(request, **credentials)`, as well as a set of optional permission related [authorization methods](). <!-- "Handling authorization..." below -->

The `get_user` method takes a `user_id` -- which could be a username, database ID or whatever, but has to be the primary key of your user object -- and returns a user object or `None`.

The `authenticate` method takes a `request` argument and credentials as keyword arguments. Most of the time, it'll look like this:
```
from django.contrib.auth.backends import BaseBackend

class MyBackend(BaseBackend):
    def authenticate(self, request, username=None, password=None):
        # Check the username/password and return a user.
        ...
```
But it could also authenticate a token like so:
```
from django.contrib.auth.backends import BaseBackend

class MyBackend(BaseBackend):
    def authenticate(self, request, token=None):
        # Check the token and return a user.
        ...
```
Either way, `authenticate()` should check the credentials it gets and return a user object that matches those credentials if the credentials are valid. If they're not valid, it should return `None`.

`request` is an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) and may be `None` if it wasn't provided to [`authenticate()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#authenticaterequestnone-credentials) (which passes it on to the backend).

The Django admin is tightly coupled to the Django [User object](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#user-objects). The best way to deal with this is to create a Django `User` object for each user that exists for your backend (e.g., in your LDAP directory, your external SQL database, etc.) You can either write a script to do this in advance, or your `authenticate` method can do it the first time a user logs in.

Here's an example backend that authenticates against a username and password variable defined in your `settings.py` file and creates a Django `User` object the first time a user authenticates:
```
from django.conf import settings
from django.contrib.auth.backends import BaseBackend
from django.contrib.auth.hashers import check_password
from django.contrib,auth.models import User

class SettingsBackend(BaseBackend):
    """
    Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.

    Use the login name and a hash of the password. For example:

    ADMIN_LOGIN = 'admin'
    ADMIN_PASSWORD = 'pbkdf2_sha256$30000$Vo0VlMnkR4Bk$qEvtdyZRWTcOsCnI/oQ7fVOu1XAURIZYoOZ3iq8Dr4M='
    """

    def authenticate(self, request, username=None, password=None):
        login_valid = (settings.ADMIN_LOGIN == username)
        pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
        if login_valid and pwd_valid:
            try:
                user = User.objects.get(username=username)
            except User.DoesNotExist:
                # Create a new user. There's no need to set a password
                # because only the password from settings.py is checked.
                user = User(username=username)
                user.is_staff = True
                user.is_superuser = True
                user.save()
            return user
        return None

    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None
```

### Handling authorization in custom backends

Custom auth backends can provide their own permissions.

The user model and its manager will delegate permission lookup functions ([`get_user_permissions()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.get_user_permissions), [`get_group_permissions()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.get_group_permissions), [`get_all_permissions()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.get_all_permissions), [`has_perm()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.has_perm), [`has_module_perms()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.has_module_perms), and [`with_perm()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.UserManager.with_perm)) to any authentication backend that implements these functions.

The permissions given to the user will be the superset of all permissions returned by all backends. That is, Django grants a permission to a user that any one backend grants.

If a backend raises a [`PermissionDenied`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.PermissionDenied) exception in `has_perm()` or `has_module_perms()`, the authorization will immediately fail and Django won't check the backends that follow.

A backend could implement permissions for the magic admin like this:
```
from django.contrib.auth.backends import BaseBackend

class MagicAdminBackend(BaseBackend):
    def has_perm(self, user_obj, perm, obj=None):
        return user_obj.username == settings.ADMIN_LOGIN
```
This gives full permissions to the user granted access in the above example. Notice that in addition to the same arguments given  to the associated [`django.contrib.auth.models.User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) functions, the backend auth functions all take the user object, which may be an anonymous user, as an argument.

A full authorization implementation can be found in the `ModelBackend` class in [django/contrib/auth/backends.py](https://github.com/django/django/blob/main/django/contrib/auth/backends.py), which is the default backend and queries the `auth_permission` table most of the time.

#### Authorization for anonymous users

An anonymous user is one that is not authenticated, i.e. they have provided no valid authentication details. However, that does not necessarily mean they are not authorized to do anything. At the most basic level, most websites authorize anonymous users to browse most of the site, and many allow anonymous posting of comments, etc.

Django's permission framework does not have a place to store permissions for anonymous users. However, the user object passed to an authentication backend may be a [`django.contrib.auth.models.AnonymousUser`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.AnonymousUser) object, allowing the backend to specify custom authorization behavior for anonymous users. This is especially useful for the authors of reusable apps, who can delegate all questions of authorization to the auth backend, rather than needing settings, for example, to control anonymous access.

#### Authorization for inactive users

An inactive user is one that has its [`is_active`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.is_active) field set to `False`. The [`ModelBackend`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.backends.ModelBackend) and [`RemoteUserBackend`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.backends.RemoteUserBackend) authentication backends prohibits these users from authenticating. If a custom user model doesn't have an `is_active` field, all users will be allowed to authenticate.

You can use [`AllowAllUsersModelBackend`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.backends.AllowAllUsersModelBackend) or [`AllowAllUsersRemoteUserBackend`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.backends.AllowAllUsersRemoteUserBackend) if you want to allow inactive users to authenticate.

The support for anonymous users in the permission system allows for a scenario where anonymous users have permissions to do something while inactive authenticated users do not.

Do not forget to test for the `is_active` attribute of the user in your own backend permission methods.

#### Handling object permissions

Django's permission framework has a foundation for object permissions, though there is no implementation for it in the core. That means that checking for object permissions will always return `False` or an empty list (depending on the check performed). An authentication backend will receive the keyword parameters `obj` and `user_obj` for each object related authorization method and can return the object level permission as appropriate.