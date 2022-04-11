# Sending email

Although Python provides a mail sending interface via the [`smtplib`](https://docs.python.org/3/library/smtplib.html#module-smtplib) module, Django provides a couple of light wrappers over it. These wrappers are provided to make sending email extra quick, to help test email sending during development, and to provide support for platforms that can't use SMTP.

The code lives in the `django.core.mail` module.

## Quick example

In two lines:
```
from django.core.mail import send_mail

send_mail(
    'Subject here',
    'Here is the message.',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)
```
Mail is sent using the SMTP host and port specified in the [`EMAIL_HOST`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST) and [`EMAIL_PORT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_PORT) settings. The [`EMAIL_HOST_USER`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_USER) and [`EMAIL_HOST_PASSWORD`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_PASSWORD) settings, if set, are used to authenticate to the SMTP server, and the [`EMAIL_USE_TLS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_USE_TLS) and [`EMAIL_USE_SSL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_USE_SSL) settings control whether a secure connection is used.

<hr>

**Note**: The character set of email sent with `django.core.mail` will be set to the value of your [`DEFAULT_CHARSET`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DEFAULT_CHARSET) setting.

<hr>

## `send_mail()`

##### [`send_mail(subject, message, from_email, recipient_list, fail_silently=False, auth_user=None, auth_password=None, connection=None, html_message=None)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/#send_mail)

In most cases, you can send email using `django.core.mail.send_mail()`.

The `subject`, `message`, `from_email`, and `recipient_list` parameters are required.

* `subject`: A string.
* `message`: A string.
* `from_email`: A string. If `None`, Django will use the value of the [`DEFAULT_FROM_EMAIL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DEFAULT_FROM_EMAIL) setting.
* `recipient_list`: A list of strings, each an email address. Each member of `recipient_list` will see the other recipients in the "To:" field of the email message.
* `fail_silently`: A Boolean. When it's `False`, `send_email()` will raise an [`smtplib.SMTPException`](https://docs.python.org/3/library/smtplib.html#smtplib.SMTPException) if an error occurs. See the [`smtplib`](https://docs.python.org/3/library/smtplib.html#module-smtplib) docs for a list of possible exceptions, all of which are subclasses of [`SMTPException`](https://docs.python.org/3/library/smtplib.html#smtplib.SMTPException).
* `auth_user`: The optional username to use to authenticate to the SMTP server. If this isn't provided, Django will use the value of the [`EMAIL_HOST_USER`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_USER) setting.
* `auth_password`: The optional password to use to authenticate to the SMTP server. If this isn't provided, Django will use the value of the [`EMAIL_HOST_PASSWORD`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_PASSWORD) setting.
* `connection`: The optional email backend to use to send the mail. If unspecified, an instance of the default backend will be used. See the documentation on [Email backends]() for more details. <!-- "Email backends" below -->
* `html_message`: If `html_message` is provided, the resulting email will be a *multipart/alternative* email with `message` as the *text/plain* content type and `html_message` as the *text/html* content type.

The return value will be the number of successfully delivered messages (which can be `0` or `1` since it can only send one message).

## `send_mass_mail()`

##### [`send_mass_mail(datatuple, fail_silently=False, auth_user=None, auth_password=None, connection=None)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/#send_mass_mail)

`django.core.mail.send_mass_mail()` is intended to handle mass emailing.

`datatuple` is a tuple in which each element is in this format:
```
(subject, message, from_email, recipient_list)
```
`fail_silently`, `auth_user`, and `auth_password` have the swame functions as in [`send_mail()`](). <!-- just above -->

Each separate element of `datatuple` results in a separate email message. As in `send_mail()`, recipients in the same `recipient_list` will all see the other addresses in the email messages' "To:" field.

For example, the following code would send two different messages to two different sets of recioients; however, only one connection to the mail server would be opened:
```
message1 = ('Subject here', 'Here is the message', 'from@example.com', ['first@example.com', 'other@example.com'])
message2 = ('Another subject', 'Here is another message', 'from@example.com', ['second@test.com'])
send_mass_mail((message1, message2), fail_silently=False)
```
The return value will be the number of successfully delivered messages.

### `send_mass_mail()` vs. `send_mail()`

The main difference between [`send_mass_mail()`]() and [`send_mail()`]() <!-- both above --> is that `send_mail()` opens a connection to the mail server each time it's executed, while `send_mass_mail()` uses a single connection for all of its messages. This makes `send_mass_mail()` slightly more efficient.

## `mail_admins()`

##### [`mail_admins(subject, message, fail_silently=False, connection=None, html_message=None)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/#mail_admins)

`django.core.mail.mail_admins()` is a shortcut for sending an email to the site admins, as defined in the [`ADMINS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-ADMINS) setting.

`mail_admins()` prefixes the subject with the value of the [`EMAIL_SUBJECT_PREFIX`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_SUBJECT_PREFIX) setting, which is `"[Django] "` by default.

The "From:" header of the email will be the value of the [`SERVER_EMAIL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SERVER_EMAIL) setting.

This method exists for convenience and readability.

If `html_message` is provided, the resulting eamil will be a *multipart/alternative* email with `message` as the *text/plain* content type and `html_message` as the *text/html* content type.

## `mail_managers()`

##### [`mail_managers(subject, message, fail_silently=False, connection=None, html_message=None)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/#mail_managers)

`django.core.mail.mail_managers()` is just like `mail_admins()`, except it sends an email to the site managers, as defined in the [`MANAGERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MANAGERS) setting.

## Examples

This sends a single email to john@example.com and jane@example.com, with them both appearing in the "To:":
```
send_email(
    'Subject',
    'Message.',
    'from@example.com',
    ['john@example.com', 'jane@example.com'],
)
```