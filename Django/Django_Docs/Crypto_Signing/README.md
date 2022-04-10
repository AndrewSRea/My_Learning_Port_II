# Cryptographic signing

The golden rule of web application security is to never trust data from untrusted sources. Sometimes it can be useful to pass data through an untrusted medium. Cryptographically signed values can be passed through an untrusted channel safe in the knowledge that any tampering will be detected.

Django provides both a low-level API for signing values and a high-level API for setting and reading signed cookies, one of the most common uses of signing in web applications.

You may also find signing useful for the following:

* Generating "recover my account" URLs for sending to users who have lost their password.
* Ensuring data stored in hidden from fields has not been tampered with.
* Generating one-time secret URLs for allowing temporary access to a protected resource, for example, a downloadable file that a user has paid for.

## Protecting the `SECRET_KEY`

When you create a new Django project using [`startproject`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startproject), the `settings.py` file is generated automatically and gets a random [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY) value. This value is the key to securing signed data -- it is vital you keep this secure, or attackers could use it to generate their own signed values.

## Using the low-level API

Django's signing methods live in the `django.core.signing` module. To sign a value, first instantiate a `Signer` instance:
```
>>> from django.core.signing import Signer
>>> signer = Signer()
>>> value = signer.sign('My string')
>>> value
'My string:GdMGD6HNQ_qdgxYP8uBZAdAIV1w'
```
The signature is appended to the end of the string, following the colon. You can retrieve the original value using the `unsign` method:
```
>>> original = signer.unsign(value)
>>> original
'My string'
```
If you pass a non-string value to `sign`, the value will be forced to string before being signed, and the `unsign` result will give you that string value:
```
>>> signed = signer.sign(2.5)
>>> original = signer.unsign(signed)
>>> original
'2.5'
```
If you wish to protect a list, tuple, or dictionary, you can do so using the `sign_object()` and `unsign_object()` methods:
```
>>> signed_obj = signer.sign_object({'message': 'Hello!'})
>>> signed_obj
'eyJtZXNzYWdlIjoiSGVsbG8hIn0:Xdc-mOFDjs22KsQAqfVfi8PQSPdo3ckWJxPWwQOFhR4'
>>> obj = signer.unsign_object(signed_obj)
>>> obj
{'message': 'Hello!'}
```
See [Protecting complex data structures]() for more details. <!-- below -->

If the signature or value have been altered in any way, a `django.core.signing.BadSignature` exception will be raised:
```
>>> from django.core import signing
>>> value += 'm'
>>> try:
...     original = signer.unsign(value)
... except signing.BadSignature:
...     print("Tampering detected!")
```
By default, the `Signer` class uses the [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY) setting to generate signatures. You can use a different secret by passing it to the `Signer` constructor:
```
>>> signer = Signer('my-other-secret')
>>> value = signer.sign('My string')
>>> value
'My string:EkfQJafvGyiofrdGnuthdxImIJw'
```

##### [`class Signer(key=None, sep=':', salt=None, algorithm=None)`](https://docs.djangoproject.com/en/4.0/_modules/django/core/signing/#Signer)

Returns a signer which uses `key` to generate signatures and `sep` to separate values. `sep` cannot be in the [`URL safe base64 alphabet](https://datatracker.ietf.org/doc/html/rfc4648.html#section-5). This alphabet contains alphanumeric characters, hyphens, and underscores. `algorithm` must be an algorithm supported by [`hashlib`](https://docs.python.org/3/library/hashlib.html#module-hashlib), it defaults to `'sha256'`.

### Using the `salt` argument

If you do not wish for every occurrence of a particular string to have the same signature hash, you can use the optional `salt` argument to the `Signer` class. Using a salt will seed the signing hash function with both the salt and your [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY):
```
>>> signer = Signer()
>>> signer.sign('My string')
'My string:GdMGD6HNQ_qdgxYP8yBZAdAIV1w'
>>> signer.sign_object({'message': 'Hello!'})
'eyJtZXNzYWdlIjoiSGVsbG8hIn0:Xdc-mOFDjs22KsQAqfVfi8PQSPdo3ckWJxPWwQOFhR4'
>>> signer = Signer(salt='extra')
>>> signer.sign('My string')
'My string:Ee7vGi-ING6n02gkcJ-QLHg6vFw'
>>> signer.unsign('My string:Ee7vGi-ING6n02gkcJ-QLHg6vFw')
'My string'
>>> signer.sign_object({'message': 'Hello!'})
'eyJtZXNzYWdlIjoiSGVsbG8hIn0:-UWSLCE-oUAHzhkHviYz3SOZYBjFKllEOyVZNuUtM-I'
>>> signer.unsign_object('eyJtZXNzYWdlIjoiSGVsbG8hIn0:-UWSLCE-oUAHzhkHviYz3SOZYBjFKllEOyVZNuUtM-I')
{'message': 'Hello!'}
```
Using salt in this way puts the different signatures into different namespaces. A signature that comes from one namespace (a particular salt value) cannot be used to validate the same plaintext string in a different namespace that is using a different salt setting. The result is to prevent an attacker from using a signal string generated in one place in the code as input to another piece of code that is generating (and verifying) signatures using a different salt.

Unlike your [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY), your salt argument does not need to stay secret.