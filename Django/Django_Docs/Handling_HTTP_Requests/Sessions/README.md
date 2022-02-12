# How to use sessions

Django provides full support for anonymous sessions. The session framework lets you store and retrieve arbitrary data on a per-site-visitor basis. It stores data on the server side and abstracts the sending and receiving of cookies. Cookies contain a session ID -- not the data itself (unless you're using the [cookie based backend]()). <!-- below -->

## Enabling sessions

Sessions are implemented via a piece of [middleware](https://docs.djangoproject.com/en/4.0/ref/middleware/).

To enable session functionality, do the following:

* Edit the [`MIDDLEWARE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIDDLEWARE) setting and make sure it contains `'django.contrib.sessions.middleware.SessionMiddleware'`. The default `settings.py` created by `django-admin startproject` has `SessionMiddleware` activated.

If you don't want to use sessions, you might as well remove the `SessionMiddleware` line from `MIDDLEWARE` and `'django.contrib.sessions'` from your [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS). It'll save you a small bit of overhead.

## Configuring the session engine

By default, Django stores sessions in your database (using the model `django.contrib.sessions.models.Session`). Though this is convenient, in some setups it's faster to store session data elsewhere, so Django can be configured to store session data on your filesystem or in your cache.

### Using database-backed sessions

If you want to use a database-backed session, you need to add `'django.contrib.sessions'` to your [`INSTALLED_APPS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS) setting.

Once you have configured your installation, run `manage.py migrate` to install the single database table that stores session data.

### Using cached sessions

For better performance, you may want to use a cache-based session backend.

To store session data using Django's cache system, you'll first need to make sure you've configured your cache; see the [cache documentation]() for details. <!-- possible future folder? (https://docs.djangoproject.com/en/4.0/topics/cache/) -->

<hr>

:warning: **Warning**: You should only use cache-based sessions if you're using the Memcached or Redis cache backend. The local-memory cache backend doesn't retain data long enough to be a good choice, and it'll be faster to use file or database sessions directly instead of sending everything through the file or database cache backends. Additionally, the local-memory cache backend is NOT multi-process safe, therefore probably not a good choice for production environments.

<hr>

If you have multiple caches defined in [`CACHES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES), Django will use the default cache. To use another cache, set [`SESSION_CACHE_ALIAS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_CACHE_ALIAS) to the name of that cache.

Once your cache is configured, you've got two choices for how to store data in the cache:

* Set [`SESSION_ENGINE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_ENGINE) to `"django.contrib.sessions.backends.cache"` for a simple caching session store. Session data will be stored directly ion your cache. However, session data may not be persistent: cached data can be evicted if the cache fills up or if the cache server is restarted.
* For persistent cached data, set `SESSION_ENGINE` to `"django.contrib.sessions.backends.cached_db"`. This uses a write-through cache -- every write to the cache will also be written to the database. Session reads only use the database if the data is not already in the cache.

Both session stores are quite fast, but the simple cache is faster because it disregards persistence. In most cases, the `cached_db` backend will be fast enough, but if you need that last bit of performance, and are willing to let session data be expunged from time to time, the `cache` backend is for you.

If you use the `cached_db` session backend, you also need to follow the configuration instructions for the [using database-backed sessions](). <!-- above -->

### Using file-based sessions

To use file-based sessions, set the [`SESSION_ENGINE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_ENGINE) setting to `"django.contrib.sessions.backends.file"`.

You might also want to set the [`SESSION_FILE_PATH`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_FILE_PATH) setting (which defaults to output from `tempfile.gettempdir()`, most likely `/tmp`) to control where Django stores session files. Be sure to check that your web server has permissions to read and write to this location.

### Using cookie-based sessions

To use cookies-based sessions, set the [`SESSION_ENGINE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_ENGINE) setting to `"django.contrib.sessions.backends.signed_cookies"`. The session data will be stored using Django's tools for [cryptographic signing]() <!-- possible future folder? (https://docs.djangoproject.com/en/4.0/topics/signing/) --> and the [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY) setting.

<hr>

**Note**: It's recommended to leave the [`SESSION_COOKIE_HTTPONLY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_COOKIE_HTTPONLY) setting on `True` to prevent access to the stored data from JavaScript.

<hr>

:warning: **Warning**: **If the `SECRET_KEY` is not kept secret and you are using the [`PickleSerializer`](), this can lead to arbitrary remote code execution.** <!-- below -->

An attacker in possession of the [`SECRET_KEY`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SECRET_KEY) can not only generate falsified session data, which your site will trust, but also remotely execute arbitrary code, as the data is serialized using pickle.

If you use cookie-based sessions, pay extra care that your secret key is always kept completely secret, for any system which might be remotely accessible.

**The session data is signed but not encrypted.**

When using the cookies backend, the session data can be read by the client.

A MAC (Message Authentication Code) is used to protect the data against changes by the client, so that the session data will be invalidated when being tampered with. The same invalidation happens if the client storing the cookie (e.g. your user's browser) can't store all of the session cookie and drops data. Even though Django compresses the data, it's still entirely possible to exceed the **[common limit of 4906 bytes](https://datatracker.ietf.org/doc/html/rfc2965.html#section-5.3)** per cookie.