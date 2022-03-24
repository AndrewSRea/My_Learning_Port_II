# Password management in Django

Password management is something that should generally not be reinvented unnecessarily, and Django endeavors to provide a secure and flexible set of tools for managing user passwords. This document describes how Django stores passwords, how the storage hashing can be configured, and some utilities to work with hashed passwords.

<hr>

**See also**

Even though users may use strong passwords, attackers might be able to eavesdrop on their connections. Use [HTTPS](https://docs.djangoproject.com/en/4.0/topics/security/#security-recommendation-ssl) to avoid sending passwords (or any other sensitive data) over plain HTTP connections because they will be vulnerable to password sniffing.

<hr>

## How Django stores passwords

Django provides a flexible password storage system and uses PBKDF2 by default.

The [`password`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User.password) attribute of a [`User`](https://docs.djangoproject.com/en/4.0/ref/contrib/auth/#django.contrib.auth.models.User) object is a string in this format:
```
<algorithm>$<iterations>$<salt>$<hash>
```
Those are the components used for storing a User's password, separated by the dollar-sign character and consist of: the hashing algorithm, the number of algorithm iterations (work factor), the random salt, and the resulting password hash. The algorithm is one of a number of one-way hashing or password storage algorithms Django can use; see below. Iterations describe the number of times the algorithm is run over the hash. Salt is the random seed used and the hash is the result of the one-way function.

By default, Django uses the [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2) algorithm with a SHA256 hash, a password stretching mechanism recommended by [NIST](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf). This should be sufficient for most users: it's quite secure, requiring massive amounts of computing time to break.

However, depending on your requirements, you may choose a different algorithm, or even use a custom algorithm to match your specific security situation. Again, most users shouldn't need to do this -- if you're not sure, you probably don't. If you do, please read on:

Django chooses the algorithm to use by consulting the [`PASSWORD_HASHERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD_HASHERS) setting. This is a list of hashing algorithm classes that this Django installation supports. The first entry in this list (that is, `settings.PASSWORD_HASHERS[0]`) will be used to store passwords, and all the other entries are valid hashers that can be used to check existing passwords. This means that if you want to use a different algorithm, you'll need to modify `PASSWORD_HASHERS` to list your preferred algorithm first in the list.

The default for `PASSWORD_HASHERS` is:
```
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.ScryptPasswordHasher',
]
```
This means that Django will use [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2) to store all passwords but will support checking passwords stored with PBKDF2SHA1, [argon2](https://en.wikipedia.org/wiki/Argon2), and [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

The next few sections describe a couple of common ways advanced users may want to modify this setting.

### Using Argon2 with Django

[Argon2](https://en.wikipedia.org/wiki/Argon2) is the winner of the 2015 [Password Hashing Competition](https://www.password-hashing.net/), a community organized open competition to select a next generation hashing algorithm. It's designed no to be easier to compute on custom hardware than it is to compute on an ordinary CPU.

Argon2 is not the default for Django because it requires a third-party library. The Password Hashing Competition panel, however, recommends immediate use of Argon2 rather than the other algorithms supported by Django.

To use Argon2 as your default storage algorithm, do the following:

1. Install the [argon-cffi library](https://pypi.org/project/argon2-cffi/). This can be done by running `python -m pip install django[argon2]`, which is equivalent to `python -m pip install argon2-cffi` (along with any version requirement from Django's `setup.cfg`).
2. Modify [`PASSWORD_HASHERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD_HASHERS) to list `Argon2PasswordHasher` first. That is, in your settings file, you'd put:
    
    ```
    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.Argon2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.ScryptPasswordHasher',
    ]
    ```

    Keep and/or add any entries in this list if you need Django to [upgrade passwords](). <!-- "Password upgrading" below -->

### Using `bcrypt` with Django

[Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) is a popular password storage algorithm that's specifically designed for long-term password storage. It's not the default used by Django since it requires the use of third-party librarie but since many people may want to use it, Django supports bcrypt with minimal effort.

To use Bcrypt as your default storage algorithm, do the following:

1. Install the [bcrypt library](https://pypi.org/project/bcrypt/). This can be done by running `python -m pip install django[bcrypt]`, which is equivalent to `python -m pip install bcrypt` (alopng with any version requirement from Django's `setup.cfg`).
2. Modify [`PASSWORD_HASHERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD_HASHERS) to list `BCryptSHA256PasswordHasher` first. That is, in your settings file, you'd put:

    ```
    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.Argon2PasswordHasher',
        'django.contrib.auth.hashers.ScryptPasswordHasher',
    ]
    ```

    Keep and/or add any entries in this list if you need Django to [upgrade passwords](). <!-- "Password upgrading" below -->

That's it -- now your Django install will use Bcrypt as the default storage algorithm.