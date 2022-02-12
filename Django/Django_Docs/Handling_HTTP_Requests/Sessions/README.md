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

**The session data is signed but not encrypted**

When using the cookies backend, the session data can be read by the client.

A MAC (Message Authentication Code) is used to protect the data against changes by the client, so that the session data will be invalidated when being tampered with. The same invalidation happens if the client storing the cookie (e.g. your user's browser) can't store all of the session cookie and drops data. Even though Django compresses the data, it's still entirely possible to exceed the **[common limit of 4906 bytes](https://datatracker.ietf.org/doc/html/rfc2965.html#section-5.3)** per cookie.

**No freshness guarantee**

Note also that while the MAC can guarantee the authenticity of the data (that it was generated by your site, and not someone else), and the integrity of the data (that it is all there and correct), it cannot guarantee freshness, i.e. that you are being sent back the last thing you sent to the client. This means that for some uses of session data, the cookie backend might open you up to [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). Unlike other session backends which keep a server-side record of each session and invalidate it when a user logs out, cookie-based sessions are not invalidated when a user logs out. Thus if an attacker steals a user's cookie, they can use that cookie to login as that user even if the user logs out. Cookie will only be detected as "stale" if they are older than your [`SESSION_COOKIE_AGE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_COOKIE_AGE).

**Performance**

Finally, the size of a cookie can have an impact on the speed of your site.

<hr>

## Using sessions in views

When `SessionMiddleware` is activated , each [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpRequest) object -- the first argument to any Django view function -- will have a `session` attribute, which is a dictionary-like object.

You can read it and write to `request.session` at any point in your view. You can edit it multiple times.

#### `*class* backends.base.SessionBase`

This is the base class for all session objects. It has the following standard dictionary methods:

#### `__getitem__(key)`

Example: `fav_color = request.session['fav_color']`

#### `__setitem__(key, value)`

Example: `request.session['fav_color'] = 'blue'`

#### `__delitem__(key)`

Example: `del request.session['fav_color']`. This raises `KeyError` if the given `key` isn't already in the session.

#### `__contains__(key)`

Example: `'fav_color' in request.session`

#### `get(key, default=None)`

Example: `fav_color = request.session.get('fav_color', 'red')`

#### `pop(key, default=None)`

Example: `fav_color = request.session.pop('fav_color', 'blue')`

#### `keys()`

#### `items()`

#### `setdefault()`

#### `clear()`

It also has these methods:

#### `flush()`

Deletes the current session data from the session and deletes the session cookie. This is used if you want to ensure that the previous session data can't be accessed again from the user's browser (for example, the [`django.contrib.auth.logout()`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.logout) function calls it).

#### `set_test_cookie()`

Sets a test cookie to determine whether the user's browser supports cookies. Due to the way cookies work, you won't be able to test this until the user's next page request. See [Setting test cookies]() below for more information.

#### `test_cookie_worked()`

Returns either `True` or `False`, depending on whether the user's browser accepted the test cookie. Due to the way cookies work, you'll have to call `set_test_cookie()` on a previous, separate page request. See [Setting test cookies]() below for more information.

#### `delete_test_cookie()`

Deletes the test cookie. Use this to clean up after yourself.

#### `get_session_cookie_age()`

Returns the age of session cookies, in seconds. Defaults to [`SESSION_COOKIE_AGE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_COOKIE_AGE).

#### `set_expiry(value)`

Sets the expiration time for the session. You can pass a number of different values:

* If `value` is an integer, the session will expire after the expressed `value` of seconds of inactivity. For example, calling `request.session.set_expiry(300)` would make the session expire in 5 minutes.
* If `value` is a `datetime` or `timedelta` object, the session will expire at that specific date/time. Note that `datetime` and `timedelta` values are only serializable if you are using the [`PickleSerializer`](). <!-- below -->
* If `value` is `0`, the user's session cookie will expire when the user's web browser is closed.
* If `value` is `None`, the session reverts to using the global session expiry policy.

Reading a session is not considered activity for expiration purposes. Session expiration is computed from the last time the session was *modified*.

#### `get_expiry_age()`

Returns the number of seconds until this session expires. For sessions with no custom expiration (or those set to expire at browser close), this will equal [`SESSION_COOKIE_AGE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_COOKIE_AGE).

This function accepts two optional keyword arguments:

* `modification`: last modification of the session, as a [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) object. Defaults to the current time.
* `expiry`: expiry information for the session, as a `datatime` object, an [`int`](https://docs.python.org/3/library/functions.html#int) (in seconds), or `None`. Defaults to the value stored in the session by [`set_expiry()`](), <!-- above --> if there is one, or `None`.

#### `get_expiry_date()`

Returns the date this session will expire. For sessions with no custom expiration (or those set to expire at browser close), this will equal the date [`SESSION_COOKIE_AGE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-SESSION_COOKIE_AGE) seconds from now.

This function accepts the same keyword arguments as [`get_expiry_age()`](). <!-- above -->

#### `get_expire_at_browser_close()`

Returns either `True` or `False`, depending on whether the user's session cookie will expire when the user's web browser is closed.

#### `clear_expired()`

Removes expired sessions from the session store. This class method is called by [`clearsessions`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-clearsessions).

#### `cycle_key()`

Creates a new session key while retaining the current session data. [`django.contrib.auth.login()`](https://docs.djangoproject.com/en/4.0/topics/auth/default/#django.contrib.auth.login) calls this method to mitigate against session fixation.