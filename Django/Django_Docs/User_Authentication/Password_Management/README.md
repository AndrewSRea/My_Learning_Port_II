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

### Using `scrypt` with Django

[scrypt](https://en.wikipedia.org/wiki/Scrypt) is similar to PBKDF2 and bcrypt in utilizing a set number of iterations to slow down brute-force attacks. However, because PBKDF2 and bcrypt do not require a lot of memory, attackers with sufficient resources can launch large-scale parallel attacks in order to speed up the attacking process. scrypt is specifically designed to use more memory compared to other password-based key derivation functions in order to limit the amount or parallelism an attacker can use. See [RFC 7914](https://datatracker.ietf.org/doc/html/rfc7914.html) for more details.

To use scrypt as your default storage algorithm, do the following:

1. Modify [`PASSWORD_HASHERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD_HASHERS) to list `ScryptPasswordHasher` first. That is, in your settings file:

    ```
    PASSWORD_HASHERS = [
        'django.contrib.auth.hashers.ScryptPasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.Argon2PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    ]
    ```

    Keep and/or add any entries in this list if you need Django to [upgrade passwords](). <!-- "Password upgrading" below -->

<hr>

**Note**: `scrypt` requires OpenSSL 1.1+.

<hr>

### Increasing the salt entropy

Most password hashes include a salt along with their password hash in order to protect against rainbow table attacks. The salt itself is a random value which increases the size and thus the cost of the rainbow table and is currently set at 128 bits with the `salt_entropy` value in the `BasePasswordHasher`. As computing and storage costs decrease, this value should be raised. When implementing your own password hasher, you are free to override this value in order to use a desired entropy level for your password hashes. `salt_entropy` is measured in bits.

<hr>

**Implemented detail**

Due to the method in which salt values are stored, the `salt_entropy` value is effectively a minimum value. For instance, a value of 128 would provide a salt which would actually contain 131 bits of entropy.

<hr>

### Increasing the work factor

#### PBKDF2 and bcrypt

The PBKDF2 and bcrypt algorithms use a number of iterations or rounds of hashing. This deliberately slows down attackers, making attacks against hashed passwords harder. However, as computing power increases, the number of iterations needs to be increased. We've chosen a reasonable default (and will increase it with each release of Django), but you may wish to tune it up or down, depending on your security needs and available processing power. To do so, you'll subclass the appropriate algorithm and override the `iterations` parameter (use the `rounds` parameter when subclassing a bcrypt hasher). For example, to increase the number of iterations used by the default PBKDF2 algorithm:

1. Create a subclass of `django.contrib.auth.hashers.PBKDF2PasswordHasher`:

    ```
    from django.contrib.auth.hashers import PBKDF2PasswordHasher

    class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):
        """
        A subclass of PBKDF2PasswordHasher that uses 100 times more iterations.
        """
        iterations = PBKDF2PasswordHasher.iterations * 100
    ```

    Save this somewhere in your project. For example, you might put this in a file like `myproject/hashers.py`.

2. Add your new hasher as the first entry in [`PASSWORD_HASHERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-PASSWORD_HASHERS):

    ```
    PASSWORD_HASHERS = [
        'myproject.hashers.MyPBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.Argon2PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.ScryptPasswordHasher',
    ]

That's it -- now your Django install will use more iterations when it stores passwords using PBKDF2.

<hr>

**Note**: bycrypt `rounds` is a logarithmic work factor, e.g. 12 rounds means `2 ** 12` iterations.

<hr>

#### Argon2

Argon2 has three attributes that can be customized:

1. `time_cost` controls the number of iterations within the hash.
2. `memory_cost` controls the size of memory that must be used during the computation of the hash.
3. `parallelism` controls how many CPUs the computation of the hash can be parallelized on.

The default values of these attributes are probably fione for you. If you determine that the password hash is too fast or too slow, you can tweak it as follows:

1. Choose `parallelism` to be the number of threads you can spare computing the hash.
2. Choose `memory_cost` to be the KiB of memory you can spare.
3. Adjust `time_cost` and measure the time hashing a password takes. Pick a `time_cost` that takes an acceptable time for you. If `time_cost` set to 1 is unacceptably slow, lower `memory_cost`.

<hr>

**`memory_cost` interpretation**

The argon2 command-line utility and some other libraries interpret the `memory_cost` parameter differently from the value that Django uses. The conversion is given by `memory_cost == 2 ** memory_cost_commandline`.

<hr>

#### `scrypt`

[scrypt](https://en.wikipedia.org/wiki/Scrypt) has four attributes that can be customized:

1. `work_factor` controls the number of iterations within the hash.
2. `block_size` 
3. `parallelism` controls how many threads will run in parallel.
4. `maxmem` limits the maximum size of memory that can be used during the computation of the hash. Defaults to `0`, which means the default limitation from the OpenSSL library.

We've chosen reasonable defaults, but you may wish to tune it up or down, depending on your security needs and available processing power.

<hr>

**Estimating memory usage**

The minimum memory requirement of [scrypt](https://en.wikipedia.org/wiki/Scrypt) is:
```
work_factor * 2 block_size * 64
```
...so you may need to tweak `maxmem` when changing the `work_factor` or `block_size` values.

<hr>