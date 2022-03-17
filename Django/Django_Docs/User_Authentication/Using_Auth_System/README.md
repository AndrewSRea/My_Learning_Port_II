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

### Authenticating users

##### `authenticate(request=None, **credentials)`

Use `authenticate()` to verify a set of credentials. It takes credentials as keyword arguments, `username` and `password` for the default case, checks them against each [authentication backend](), <!-- "Other authentication sources" section of the "Customizing authentication in Django" page --> and returns a [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) object if the credentials are valid for a backend. If the credentials aren't valid for any backend or if a backend raises [`PermissionDenied`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.PermissionDenied), it returns `None`. For example:
```
from django.contrib.auth import authenticate
user = authenticate(username='john', password='secret')
if user is not None:
    # A backend authenticated the credentials
else:
    # No backend authenticated the credentials
```
`request` is an optional [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) which is passed on the `authenticate()` method of the authentication backends.

<hr>

**Note**: This is a low level way to authenticate a set of credentials; for example, it's used by the [`RemoteUserMiddleware`](https://docs.djangoproject.com/en/4.0/ref/middleware/#django.contrib.auth.middleware.RemoteUserMiddleware). Unless you are writing your own authentication system, you probably won't use this. Rather if you're looking for a way to login a user, use the [`LoginView`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.views.LoginView).

<hr>

## Permissions and authorization

Django comes with a built-in permission system. It provides a way to assign permissions to specific users and groups of users.

It's used by the Django admin site, but you're welcome to use it ihn your own code.

The Django admin site uses permissions as follows:

* Access to view objects is limited to users with the "view" or "change" permission for that type of object.
* Access to view the "add" form and add an object is limited to users with the "add" permission for that type of object.
* Access to view the change list, view the "change" form, and change an object is limited to users with the "change" permission for that type of object.
* Access to delete an object is limited to users with the "delete" permission for that type of object.

Permissions can be set not only per type of object, but also per specific object instance. By using the [`has_view_permission()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_view_permission), [`has_add_permission()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_add_permission), [`has_change_permission()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_change_permission), and [`has_delete_permission()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin.has_delete_permission) methods provided by the [`ModelAdmin`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin) class, it is possible to customize permissions for different object instances of the same type.

[`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) objects have two many-to-many fields: `groups` and `user_permissions`. `User` objects can access their related objects in the same way as any other [Django model](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Models_and_Databases/Models#models):
```
myuser.groups.set([group_list])
myuser.groups.add(group, group, ...)
myuser.groups.remove(group, group, ...)
myuser.groups.clear()
myuser.user_permissions.set([permission_list])
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()
```

### Default permissions

When `django.contrib.auth` is listed in your [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) setting, it will ensure that four default permissions -- add, change, delete, and view -- are created for each Django model defined in one of your installed applications.

These permissions will be created when you run [`manage.py migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate); the first time you run `migrate` after adding `django.contrib.auth` to `INSTALLED_APPS`, the default permissions will be created for all previously-installed models, as well as for any new models being installed at that time. Afterward, it will create default permissions for new models each time you run `manage.py migrate` (the function that creates permissions is connected to the [`post_migrate`](https://docs.djangoproject.com/en/4.0/ref/signals/#django.db.models.signals.post_migrate) signal).

Assuming you have an application with an [`app_label`](https://docs.djangoproject.com/en/4.0/ref/models/options/#django.db.models.Options.app_label) `foo` and a model named `Bar`, to test for basic permissions you should use:

* add: `user.has_perm('foo.add_bar')`
* change: `user.has_perm('foo.change_bar')`
* delete: `user.has_perm('foo.delete_bar')`
* view: `user.has_perm('foo.view_bar')`

The [`Permission`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.Permission) model is rarely accessed directly.

### Groups

[`django.contrib.auth.models.Group`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.Group) models are a generic way of categorizing users so you can apply permissions, or some other label, to those users. A user can belong to any number of groups.

A user in a group automatically has the permissions granted to that group. For example, if the group `Site editors`has the permission `can_edit_home_page`, any user in that group will have that permission.

Beyond permissions, groups are a convenient way to categorize users to give them some label, or extended functionality. For example, you could create a group `'Special users'`, and you could write code that could, say, give them access to a members-only portion of your site, or send them members-only email messages.

### Programmatically creating permissions

While [custom permissions]() <!-- "Custom permissions" section of the "Customizing authentication in Django" page --> can be defined within a model's `Meta` class, you can also create permissions directly. For example, you can create the `can_publish` permission for a `BlogPost` model in `myapp`:
```
from myapp.models import BlogPost
from django.contrib.auth.models import Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(BlogPost)
permission = Permission.objects.create(
    codename='can_publish',
    name='Can Publish Posts',
    content_type=content_type,
)
```
The permission can then be assigned to a [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) via its `user_permissions` attribute or to a [`Group`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.Group) via its `permissions` attribute.

<hr>

**Proxy models need their own content type**

If you want to create [permissions for a proxy model](), <!-- "Proxy models" below --> pass `for_concrete_model=False` to [`ContentTypeManager.get_for_model()`](https://docs.djangoproject.com/en/4.0/ref/contrib/contenttypes/#django.contrib.contenttypes.models.ContentTypeManager.get_for_model) to get the appropriate `ContentType`:
```
content_type = ContentType.objects.get_for_model(BlogPostProxy, for_concrete_model=False)
```

<hr>

### Permission caching

The [`ModelBackend`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.backends.ModelBackend) caches permissions on the user object after the first time they need to be fetched for a permissions check. This is typically fine for the request-response cycle since permissions aren't typically checked immediately after they are added (in the admin, for example). If you are adding permissions and checking them immediately afterward in a test or view for example, the easiest solution is to re-fetch the user from the database. For example:
```
from django.contrib.auth.models import Permission, User
from django.contrib.contenttypes.models import ContentType
from django.shortcuts import get_object_or_404

from myapp.models import BlogPost

def user_gains_perms(request, user_id):
    user = get_object_or_404(User, pk=user_id)
    # any permission check will cache the current set of permissions
    user.has_perm('myapp.change_blogpost')

    content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.get(
        codename='change_blogpost',
        content_type=content_type,
    )
    user.user_permissions.add(permission)

    # Checking the cached permission set
    user.has_perm('myapp.change_blogpost')   # False

    # Request new instance of User
    # Be aware that user.refresh_from_db() won't clear the cache.
    user = get_object_or_404(User, pk=user_id)

    # Permission cache is repopulated from the database
    user.has_perm('myapp.change_blogpost')   # True

    ...
```

### Proxy models

Proxy models work exactly the same way as concrete models. Permissions are created using the own content type of the proxy model. Proxy models don't inherit the permissions of the concrete model they subclass:
```
class Person(models.Model):
    class Meta:
        permissions = [('can_eat_pizzas', 'Can eat pizzas')]

class Student(Person):
    class Meta:
        proxy = True
        permissions = [('can_deliver_pizzas', 'Can deliver pizzas')]

>>> # Fetch the content type for the proxy model.
>>> content_type = ContentType.objects.get_for_model(Student, for_concrete_model=False)
>>> student_permissions = Permission.objects.filter(content_type=content_type)
>>> [p.codename for p in student_permissions]
['add_student', 'change_student', 'delete_student', 'view_student', 'can_deliver_pizzas']
>>> for permission in student_permissions:
...     user.user_permissions.add(permission)
>>> user.has_perm('app.add_person')
False
>>> user.has_perm('app.can_eat_pizzas')
False
>>> user.has_perms(('app.add_student', 'app.can_deliver_pizzas'))
True
```

## Authentication in web requests

Django uses [sessions](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Sessions#how-to-use-sessions) and middleware to hook the authentication system into [`request` objects](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest).

These provide a [`request.user`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.user) attribute on every request which represents the current user. If the current user has not logged in, this attribute will be set to an instance of [`AnonymousUser`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.AnonymousUser), otherwise it will be an instance of [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User).

You can tell them apart with [`is_authenticated`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.is_authenticated), like so:
```
if request.user.is_authenticated:
    # Do something for authenticated users.
    ...
else:
    # Do something for anonymous users.
    ...
```

### How to log a user in

If you have an authenticated user you want to attach to the current session -- this is done with a `login()` function.

##### `login(request, user, backend=None)`

To log a user in, from a view, use `login()`. It takes an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) object and a [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) object. `login()` saves the user's ID in the session, using Django's session framework.

Note that any data set during the anonymous session is retained in the session after a user logs in.

This example shows how you might use both [`authenticate()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#authenticaterequestnone-credentials) and `login()`:
```
from django.contrib.auth import authenticate, login

def my_view(request):
    username = request.POST['username']
    password = request.POST['password']
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
        # Redirect to a success page.
        ...
    else:
        # Return an 'invalid login' error message.
        ...
```

#### Selecting the authentication backend

When a user logs in, the user's ID and the backend that was used for authentication are saved in the user's session. This allows the same [authentication backend]() <!-- "Other authentication sources" section of the "Customizing authentication in Django" page --> to fetch the user's details on a future request. The authentication backend to save in the session is selected as follows:

1. Use the value of the optional `backend` argument, if provided.
2. Use the value of the `user.backend` attribute, if present. This allows pairing [`authenticate()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#authenticaterequestnone-credentials) and [`login()`](): `authenticate()` sets the `user.backend` attribute on the user object it returns. <!-- "login()" above -->
3. Use the `backend` in [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS), if there is only one.
4. Otherwise, raise an exception.

In cases 1 and 2, the value of the `backend` argument or the `user.backend` attribute should be a dotted import path string (like that found in `AUTHENTICATION_BACKENDS`), not the actual backend class.

### How to log a user out

##### `logout(request)`

To log out a user who has been logged in via [`django.contrib.auth.login()`](), use [`django.contrib.auth.logout()`]() <!-- both above --> within your view. It takes an [`HttpRequest`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) object and has no return value. Example:
```
from django.contrib.auth import logout

def logout_view(request):
    logout(request)
    # Redirect to a success page.
```
Note that `logout()` doesn't throw any errors if the user wasn't logged in.

When you call `logout()`, the session data for the current request is completely cleaned out. All existing data is removed. This is to prevent another person from using the same web browser to log in and have access to the previous user's session data. If you want to put anything into the session that will be available to the user immediately after logging out, do that *after* calling [`django.contrib.auth.logout()`](). <!-- above -->

### Limiting access to logged-in users

#### The raw way

The raw way to limit access to pages is to check [`request.user.is_authenticated`]() and either redirect to a login page:
```
from django.conf import settings
from django.shortcuts import redirect

def my_view(request):
    if not request.user.is_authenticated:
        return redirect('%s?next=%s' % (settings.LOGIN_URL, request.path))
    # ...
```
...or display an error message:
```
from django.shortcuts import render

def my_view(request):
    if not request.user.is_authenticated:
        return render(request, 'myapp/login_error.html')
    # ...
```

#### The `login_required` decorator

##### `login_required(redirect_field_name='next', login_url=None)`

As a shortcut, you can use the convenient `login_required()` decorator:
```
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
```
`login_required()` does the following:

* If the user isn't logged in, redirect to [`settings.LOGIN_URL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-LOGIN_URL), passing the current absolute path in the query string. Example: `/accounts/login/?next=/polls/3/`.
* If the user is logged in, execute the view normally. The view code is free to assume the user is logged in.

By default, the path that the user should be redirected to upon successful authentication is stored in a query string parameter called `"next"`. If you would prefer to use a different name for this parameter, [`login_required()`]() <!-- directly above --> takes an optional `redirect_field_name` parameter:
```
from django.contrib.auth.decorators import login_required

@login_required(redirect_field_name='my_redirect_field')
def my_view(request):
    ...
```
Note that if you provide a value to `redirect_field_name`, you will most likely need to customize your login template as well, since the template context variable which stores the redirect path will use the value of `redirect_field_name` as its key rather than `"next"` (the default).

[`login_required()`]() <!-- directly above --> also takes an optional `login_url` parameter. Example:
```
from django.contrib.auth.decorators import login_required

@login_required(login_url='/accounts/login/')
def my_view(request):
    ...
```
Note that if you don't specify the `login_url` parameter, you'll need to ensure that the [`settings.LOGIN_URL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-LOGIN_URL) and your login view are properly associated. For example, using the defaults, add the following lines to your URLconf:
```
from django.contrib.auth import views as auth_views

path('accounts/login/', auth_views.LoginView.as_view()),
```
The `settings.LOGIN_URL` also accepts view function names and [named URL patterns](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/URL_Dispatcher#naming-url-patterns). This allows you to freely remap your login view within your URLconf without having to update the setting.

<hr>

**Note**: The `login_required` decorator does NOT check the `is_active` flag on a user, but the default [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS) reject insactive users.

<hr>

**See also**

If you are writing custom views for Django's admin (or need the same authorization check that the built-in views use), you may find the [`django.contrib.admin.views.decorators.staff_member_required()`](https://docs.djangoproject.com/en/4.0/ref/contrib/admin/#django.contrib.admin.views.decorators.staff_member_required) decorator a useful alternative to `login_required()`.

<hr>

#### The `LoginRequired` mixin

When using [class-based views](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Class-based_Views#class-based-views), you can achieve the same behavior as with `login_required` by using the `LoginRequiredMixin`. This mixin should be at the leftmost position in the inheritance list.

##### `class LoginRequiredMixin`

If a view is using this mixin, all requests by non-authenticated users will be redirected to the login page or shown an HTTP 403 Forbidden error, depending on the [`raise_exception`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.mixins.AccessMixin.raise_exception) parameter.

You can set any of the parameters of [`AccessMixin`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.mixins.AccessMixin) to customize the handling of unauthorized users:
```
from django.contrib.auth.mixins import LoginRequiredMixin

class MyView(LoginRequiredMixin, View):
    login_url = '/login/'
    redirect_field_name = 'redirect_to'
```

<hr>

**Note**: Just as the `login_required` decorator, this mixin does NOT check the `is_active` flag on a user, but the default [`AUTHENTICATION_BACKENDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-AUTHENTICATION_BACKENDS) reject inactive users.

<hr>

#### Limiting access to logged-in users that pass a test

To limit access based on certain permissions or some other test, you'd do essentially the same thing as described in the previous section.

You can run your test on [`request.user`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest.user) in the view directly. For example, this view checks to make sure the user has an email in the desired domain and if not, redirects to the login page:
```
from django.shortcuts import redirect

def my_view(request):
    if not request.user.email.endswith('@example.com')
        return redirect('/login/?next=%s' % request.path)
    # ...
```

##### `user_passes_test(test_func, login_url=None, redirect_field_name='next')`

As a shortcut, you can use the convenient `user_passes_test` decorator which performs a redirect when the callable returns `False`:
```
from django.contrib.auth.decorators import user_passes_test

def email_check(user):
    return user.email.endwith('@example.com')

@user_passes_test(email_check)
def my_view(request):
    ...
```
`user_passes_test()` takes a required argument: a callable that takes a [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) object and returns `True` if the user is allowed to view the page. Note that `user_passes_test()` does not automatically check that the `User` is not anonymous.

`user_passes_test()` takes two optional arguments:

* `login_url`: Lets you specify the URL that users who don't pass the test will be redirected to. It may be a login page and defaults to [`settings.LOGIN_URL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-LOGIN_URL) if you don't specify one.
* `redirect_field_name`: Same as for [`login_required()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#login_requiredredirect_field_namenext-login_urlnone). Setting it to `None` removes it from the URL, which you may want to do if you are redirecting users that don't pass the test to a non-login page where there's no "next page".

For example:
```
@user_passes_test(email_check, login_url='/login/')
def my_view(request):
    ...
```

##### `class UserPassesTestMixin`

When using [class-based views](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Class-based_Views#class-based-views), you can use the `UserPassesTestMixin` to do this.

* `test_func()`: You have to override the `test_func()` method of the class to provide the test that is performed. Furthermore, you can set any of the parameters of [`AccessMixin`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.mixins.AccessMixin) to customize the handling of unauthorized users:

    ```
    from django.contrib.auth.mixins import UserPassesTestMixin

    class MyView(UserPassesTestMixin, View):

        def test_func(self):
            return self.request.user.email.endswith('@example.com')
    ```

* `get_test_func()`: You can also override the `get_test_func()` method to have the mixin use a differently named function for its checks (instead of `test_func()`).

<hr>

**Stacking `UserPassesTestMixin`**

Due to the way `UserPassesTestMixin` is implemented, you cannot stack them in your inheritance list. The following does NOT work:
```
class TestMixin1(UserPassesTestMixin):
    def test_func(self):
        return self.request.user.email.endswith('@example.com')

class TestMixin2(UserPassesTestMixin):
    def test_func(self):
        return self.request.user.username.startswith('django')

class MyView(TestMixin1, TestMixin2, View):
    ...
```
If `TestMixin1` would call `super()` and take that result into account, `TestMixin1` wouldn't work standalone anymore.

<hr>

#### The `permission_required` decorator

##### `permission_required(perm, login_url=None, raise_exception=False)`

It's a relatively common task to check whether a user has a particular permission. For that reason, Django provides a shortcut for that case: the `permission_required()` decorator:
```
from django.contrib.auth.decorators import permission_required

@permission_required('polls.add_choice')
def my_view(request):
    ...
```
Just like the [`has_perm()`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.has_perm) method, permission names take the form `"<app label>.<permission codename>"` (i.e., `polls.add_choice` for a permission on a model in the `polls` application).

The decorator may also take an iterable of permissions, in which case the user must have all of the permissions in order to access the view.

Note that `permission_required()` also takes an optional `login_url` parameter:
```
from django.contrib.auth.decorators import permission_required

@permission_required('polls.add_choice', login_url='/loginpage/')
def my_view(request):
    ...
```
As in the [`login_required()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#login_requiredredirect_field_namenext-login_urlnone) decorator, `login_url` defaults to [`settings.LOGIN_URL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-LOGIN_URL).

If the `raise_exception` parameter is given, the decorator will raise [`PermissionDenied`](https://docs.djangoproject.com/en/4.0/ref/exceptions/#django.core.exceptions.PermissionDenied), prompting [the 403 (HTTP Forbidden) view](https://docs.djangoproject.com/en/4.0/ref/views/#http-forbidden-view) instead of redirecting to the login page.

If you want to use `raise_exception` but also give your users a chance to login first, you can add the [`login_required()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/User_Authentication/Using_Auth_System#login_requiredredirect_field_namenext-login_urlnone) decorator:
```
from django.contrib.auth.decorators import login_required, permission_required

@login_required
@permission_required('polls.add_choice', raise_exception=True)
def my_view(request):
    ...
```
This also avoids a redirect loop when [`LoginView`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.views.LoginView)'s `redirect_authenticated_user=True` and the logged-in user doesn't have all of the required permissions.