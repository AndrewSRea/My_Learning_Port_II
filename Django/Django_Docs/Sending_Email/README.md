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
`fail_silently`, `auth_user`, and `auth_password` have the swame functions as in [`send_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mailsubject-message-from_email-recipient_list-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone-html_messagenone).

Each separate element of `datatuple` results in a separate email message. As in `send_mail()`, recipients in the same `recipient_list` will all see the other addresses in the email messages' "To:" field.

For example, the following code would send two different messages to two different sets of recioients; however, only one connection to the mail server would be opened:
```
message1 = ('Subject here', 'Here is the message', 'from@example.com', ['first@example.com', 'other@example.com'])
message2 = ('Another subject', 'Here is another message', 'from@example.com', ['second@test.com'])
send_mass_mail((message1, message2), fail_silently=False)
```
The return value will be the number of successfully delivered messages.

### `send_mass_mail()` vs. `send_mail()`

The main difference between [`send_mass_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mass_maildatatuple-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone) and [`send_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mailsubject-message-from_email-recipient_list-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone-html_messagenone) is that `send_mail()` opens a connection to the mail server each time it's executed, while `send_mass_mail()` uses a single connection for all of its messages. This makes `send_mass_mail()` slightly more efficient.

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
This sends a message to john@example.com and jane@example.com, with them both receiving a separate email:
```
datatuple = (
    ('Subject', 'Message', 'from@example.com', ['john@example.com']),
    ('Subject', 'Message', 'from@example.com', ['john@example.com']),
)
send_mass_mail(datatuple)
```

## Preventing header injection

[Header injection](http://www.nyphp.org/phundamentals/8_Preventing-Email-Header-Injection.html) is a security exploit in which an attacker inserts extra email headers to control the "To:" and "From:" in email messages that your scripts generate.

The Django email functions outlined above all protect against header injection by forbidding newlines in header values. If any `subject`, `from_email`, or `recipient_list` contains a newline (in either Unix, Windows, or Mac style), the email function (e.g. [`send_email()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mailsubject-message-from_email-recipient_list-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone-html_messagenone)) will raise `django.core.mail.BadHeaderError` (a subclass of `ValueError`) and, hence, will not send the email. It's your responsibility to validate all data before passing it to the email functions.

If a `message` contains headers at the start of the string, the headers will be printed as the first bit of the email message.

Here's an example view that takes a `subject`, `message`, and `from_email` from the request's `POST` data, sends that to admin@example.com and redirects to "/contact/thanks/" when it's done:
```
from django.core.mail import BadHeaderError, send_mail
from django.http import HttpResponse, HttpResponseRedirect

def send_email(request):
    subject = request.POST.get('subject', '')
    message = request.POST.get('message', '')
    from_email = request.POST.get('from_email', '')
    if subject and message and from_email:
        try:
            send_mail(subject, message, from_email, ['admin@example.com'])
        except BadHeaderError:
            return HttpResponse('Invalid header found.')
        return HttpResponseRedirect('/contact/thanks/')
    else:
        # In reality we'd use a form class
        # to get proper validation errors.
        return HttpResponse('Make sure all fields are entered and valid.')
```

## The `EmailMessage` class

Django's [`send_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mailsubject-message-from_email-recipient_list-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone-html_messagenone) and [`send_mass_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mass_maildatatuple-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone) functions are actually thin wrappers that make use of the [`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) class.

Not all features of the `EmailMessage` class are available through the `send_mail()` and related wrapper functions. If you wish to use advanced features, such as BCC'ed recipients, file attachments, or multi-part email, you'll need to create `EmailMessage` instances directly.

<hr>

**Note**: This is a design feature. [`send_mail()`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#send_mailsubject-message-from_email-recipient_list-fail_silentlyfalse-auth_usernone-auth_passwordnone-connectionnone-html_messagenone) and related functions were originally the only interface Django provided. However, the list of parameters they accepted was slowly growing over time. It made sense to move to a more object-oriented design for email messages and retain the original functions only for backwards compatibility.

<hr>

[`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) <!-- below --> is responsible for creating the email message itself. The [email backend]() <!-- below --> is then responsible for sending the email.

For convenience, `EmailMessage` provides a `send()` method for sending a single email. If you need to send multiple messages, the email backend API [provides an alternative](). <!-- "Sending multiple emails" below -->

### `EmailMessage` objects

##### [`class EmailMessage`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/message/#EmailMessage)

The `EmailMessage` class is initialized with the following parameters (in the given order, if positional arguments are used). All parameters are optional and can be set at any time prior to calling the `send()` method.

* `subject`: The subject line of the email.
* `body`: The body text. This should be a plain text message.
* `from_email`: The sender's address. Both `fred@example.com` and `"Fred" <fred@example.com>` forms are legal. If omitted, the [`DEFAULT_FROM_EMAIL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-DEFAULT_FROM_EMAIL) setting. is used.
* `to`: A list or tuple of recipient addresses.
* `bcc`: A list or tuple of addresses used in the "Bcc" header when sending the email.
* `connection`: An email backend instance. Use this parameter if you want to use the same connection for multiple messages. If omitted, a new connection is created when `send()` is called.
* `attachments`: A list of attachments to put on the message. These can be either [`MIMEBase`](https://docs.python.org/3/library/email.mime.html#email.mime.base.MIMEBase) instances, or `(filename, content, mimetype)` triples.
* `headers`: A dictionary of extra headers to put on the message. The keys are the header name, values are the header values. It's up to the caller to ensure header names and values are in the correct format for an email message. The corresponding attribute is `extra_headers`.
* `cc`: A list or tuple of recipient addresses used in the "Cc" header when sending the email.
* `reply_to`: A list or tuple of recipient addresses used in the "Reply-To" header when sending the email.

For example:
```
from django.core.mail import EmailMessage

email = EmailMessage(
    'Hello',
    'Body goes here',
    'from@example.com',
    ['to1@example.com', 'to2@example.com'],
    ['bcc@example.com'],
    reply_to=['another@example.com'],
    headers={'Message-ID: 'foo'},
)
```
The class has the following methods:

* `send(fail_silently=False)` sends the message. If a connection was specified when the email was constructed, that connection will be used. Otherwise, an instance of the default backend will be instantiated and used. If the keyword argument `fail_silently` is `True`, exceptions raised while sending the message will be quashed. An empty list of recipients will not raise an exception. It will return `1` if the message was sent successfully, otherwise `0`.
* `message()` constructs a `django.core.mail.SafeMIMEText` object (a subclass of Python's [`MIMEText`](https://docs.python.org/3/library/email.mime.html#email.mime.text.MIMEText) class) or a `django.core.mail.SafeMIMEMultipart` object holding the message to be sent. If you ever need to extend the [`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) class, you'll probably want to override this method to put the content you want into the MIME object.
* `recipients()` returns a list of all the recipients of the message, whether they're recorded in the `to`, `cc`, or `bcc` attributes. This is another method you might need to override when subclassing, because the SMTP server needs to be told the full list of recipients when the message is sent. If you add another way to specify recipients in your class, they need to be returned from this method as well.
* `attach()` creates a new file attachment and adds it to the message. There are two ways to call `attach()`:
    - You can pass it a single argument that is a [`MIMEBase`](https://docs.python.org/3/library/email.mime.html#email.mime.base.MIMEBase) instance. This will be inserted directly into the resulting message.
    - Alternatively, you can pass `attach()` three arguments: `filename`, `content`, and `mimetype`. `filename` is the name of the file attachment as it will appear in the email, `content` is the data that will be contained inside the attachment, and `mimetype` is the optional MIME type for the attachment. If you omit `mimetype`, the MIME content type will be guessed from the filename of the attachment.

    For example:

    ```
    message.attach('design.png', img_data, 'image/png')
    ```

    If you specify a `mimetype` of *message/rfc822*, it will also accept [`django.core.mail.EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) and [`email.message.Message`](https://docs.python.org/3/library/email.compat32-message.html#email.message.Message).

    For a `mimetype` starting with *text/*, content is expected to be a string. Binary data will be decoded using UTF-8, and if that fails, the MIME type will be changed to *application/octet-stream* and the data will be attached unchanged.

    In addition, *message/rfc822* attachments will no longer be base64-encoded in violation of [RFC 2046#section-5.2.1](), which can cause issues with displaying the attachments in [Evolution]() and [Thunderbird]().

* `attach_file()` creates a new attachment using a file from your filesystem. Call it with the path of the file to attach and, optionally, the MIME type to use for the attachment. If the MIME type is omitted, it will be guessed from the filename. You can use it like this:

    ```
    message.attach_file('/images/weather_map.png')
    ```

    For MIME types starting with *text/*, binary data is handled as in `attach()`.

#### Sending alternative content types

It can be useful to include multiple versions of the content in an email; the classic example is to send both text and HTML versions of a message. With Django's email library, you can do this using the `EmailMultiAlternatives` class. This subclass of [`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) has an `attach_alternative()` method for including extra versions of the message body in the email. All the other methods (including class initialization) are inherited directly from `EmailMessage`.

To send a text and HTML combination, you could write:
```
from django.core.mail import EmailMultiAlternatives

subject, from_email, to = 'hello', 'from@example.com', 'to@example.com'
text_content = 'This is an important message.'
html_content = '<p>This is an <strong>important</strong> message.</p>'
msg = EmailMultiAlternatives(subject, text_content, from_email, [to])
msg.attach_alternative(html_content, "text/html")
msg.send()
```
By default, the MIME type of the `body` parameter in an [`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) is `"text/plain"`. It is good practice to leave this alone, because it guarantees that any recipient will be able to read the email, regardless of their mail client. However, if you are confident that your recipients can handle an alternative content type, you can use the `content_subtype` attribute on the `EmailMessage` class to change the main content type. The major type will always be `"text"`, but you can change the subtype. For example:
```
msg = EmailMessage(subject, html_content, from_email, [to])
msg.content_subtype = "html"   # Main content is now text/html
msg.send()
```

## Email backends

The actual sending of an email is handled by the email backend.

The email backend class has the following methods:

* `open()` instantiates a long-lived email-sending connection.
* `close()` closes the current email-sending connection.
* `send_messages(email_messages)` sends a list of [`EmailMessage`](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Sending_Email#class-emailmessage) objects. If the connection is not open, this call will implicitly open the connection, and close the connection afterward. If the connection is already open, it will be left open after mail has been sent.

It can also be used as a context manager, which will automatically call `open()` and `close()` as needed:
```
from django.core import mail

with mail.get_connection() as connection:
    mail.EmailMessage(
        subject1, body1, from1, [to1],
        connection=connection,
    ).send()
    mail.EmailMessage(
        subject2, body2, from2, [to2],
        connection=connection,
    ).send()
```

### Obtaining an instance of an email backend

The `get_connection()` function in `django.core.mail` returns an instance of the email backend that you can use.

##### [`get_connection(backend=None, fail_silently=False, *args, **kwargs)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/mail/#get_connection)

By default, a call to `get_connection()` will return an instance of the email backend specified in [`EMAIL_BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_BACKEND). If you specify the `backend` argument, an instance of that backend will be instantiated.

The `fail_silently` argument controls how the backend should handle errors. If `fail_silently` is `True`, exceptions during the email sending process will be silently ignored.

All other arguments are pased directly to the constructor of the email backend.

Django ships with several email sending backends. With the exception of the SMTP backend (which is the default), these backends are only useful during testing and development. If you have special email sending requirements, you can [write your own email backend](). <!-- "Defining a custom email..." below -->

#### SMTP backend

##### `class backends.smtp.EmailBackend(host=None, port=None, username=None, password=None, use_tls=None, fail_silently=False, use_ssl=None, timeout=None, ssl_keyfile=None, ssl_certfile=None, **kwargs)`

This is the default backend. Email will be sent through an SMTP server.

The value for each argument is retrieved from the matching setting if the argument is `None`:

* `host`: [`EMAIL_HOST`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST)
* `port`: [`EMAIL_PORT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_PORT)
* `username`: [`EMAIL_HOST_USER`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_USER)
* `password`: [`EMAIL_HOST_PASSWORD`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_HOST_PASSWORD)
* `use_tls`: [`EMAIL_USE_TLS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_USE_TLS)
* `use_ssl`: [`EMAIL_USE_SSL`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_USE_SSL)
* `timeout`: [`EMAIL_TIMEOUT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_TIMEOUT)
* `ssl_keyfile`: [`EMAIL_SSL_KEYFILE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_SSL_KEYFILE)
* `ssl_certfile`: [`EMAIL_SSL_CERTFILE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-EMAIL_SSL_CERTFILE)

The SMTP backend is the default configuration inherited by Django. If you want to specify it explicitly, put the following in your settings:
```
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
```
If unspecified, the default `timeout` will be the one provided by [`socket.getdefaulttimeout()`](https://docs.python.org/3/library/socket.html#socket.getdefaulttimeout), which defaults to `None` (no timeout).