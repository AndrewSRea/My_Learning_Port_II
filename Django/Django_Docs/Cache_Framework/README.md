# Django's cache framework

A fundamental trade-off in dynamic websites is, well, they're dynamic. Each time a user requests a page, the web server makes all sorts of calculations -- from database queries to template rendering to business logic -- to create the page that your site's visitor sees. This is a lot more expensive, from a processing-overhead perspective, than your standard read-a-file-off-the-filesystem server arrangement.

For most web applications, this overhead isn't a big deal. Most web applications aren't `washingtonpost.com` or `slashdot.org`; they're small- to medium-sized sites with so-so-traffic. But for medium- to high-traffic sites, it's essential to cut as much overhead as possible.

That's where caching comes in.

To cache something is to save the result of an expensive calculation so that you don't have to perform the calculation next time. Here's some psuedocode explaining how this would work for a dynamically generated web page:
```
given a URL, try finding that page in the cache
if the page is in the cache:
    return the cached page
else:
    generate the page
    save the generated page in the cache (for next time)
    return the generated page
```
Django comes with a robust cache system that lets you save dynamic pages so they don't have to be calculated for each request. For convenience, Django offers different levels of cache granularity: You can cache the output of specific views, you can cache only the pieces that are difficult to produce, or you can cache your entire site.

Django also works well with "downstream" caches, such as [Squid](http://www.squid-cache.org/) and browser-based caches. These are the types of caches that you don't directly control but to which you can provide hints (via HTTP headers) about which parts of your site should be cached, and how.

<hr>

**See also**

The [Cache Framework design philosophy](https://docs.djangoproject.com/en/4.0/misc/design-philosophies/#cache-design-philosophy) explains a few of the design decisions of the framework.

<hr>

## Setting up the cache

The cache system requires a small amount of setup. Namely, you have to tell it where your cached data should live -- whether in a database, on the filesystem or directly in memory. This is an important decision that affects your cache's performance; yes, some cache types are faster than others.

Your cache preference goes in the [`CACHES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES) setting in your settings file. Here's an explanation of all available values for `CACHES`.

### Memcached

[Memcached](https://memcached.org/) is an entirely memory-based cache server, originally developed to handle high loads at LiveJournal.com and susequently open-sourced by Danga Interactive. It is used by sites such as Facebook and Wikipedia to reduce database access and dramatically increase site performance.

Memcached runs as a daemon and is allotted a specified amount of RAM. All it does is provide a fast interface for adding, retrieving, and deleting data in the cache. All data is stored directly in memory, so there's no overhead of database or filesystem usage.

After installing Memcached itself, you'll need to install a Memcached binding. There are several Python Memcached bindings available; the two supported by Django are [pylibmc](https://pypi.org/project/pylibmc/) and [pymemcache](https://pypi.org/project/pymemcache/).

To use Memcached with Django:

* Set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) to `django.core.cache.backends.memcached.PyMemcacheCache` or `django.core.cache.backends.memcached.PyLibMCCache` (depending on your chosen memcached binding).
* Set [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) to `ip:port` values, where `ip` is the IP address of the Memcached daemon and `port` is the port on which Memcached is running, or to a `unix:path` value, where `path` is the path to a Memcached Unix socket file.

In this example, Memcached is running on localhost (127.0.0.1) port 11211, using the `pymemcache` binding:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
```
In this example, Memcached is available through a local Unix socket file `/tmp/memcached.sock` using the `pymemcache` binding:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': 'unix:/tmp/memcached.sock',
    }
}
```
One excellent feature of Memcached is its ability to share a cache over multiple servers. This means you can run Memcached daemons on multiple machines, and the program will treat the group of machines as a *single* cache, without the need to duplicate cache values on each machine. To take advantage of this feature, include all server addresses in [`LOCATION`](), either as a semicolon or comma delimited string, or as a list.

In this example, the cache is shared over Memcached instances running on IP address 172.19.26.240 and 172.19.26.242, both on port 11211:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}
```
In the following example, the cache is shared over Memcached instances running on the IP addresses 172.19.26.240 (port 11211), 172.19.26.242 (port 11212), and 172.19.26.244 (port 11213):
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11212',
            '172.19.26.244:11213',
        ]
    }
}
```
A final point about Memcached is that memory-based caching has a disadvantage: because the cached data is stored in memory, the data will be lost if your server crashes. Clearly, memory isn't intended for permanent data storage, so don't rely on memory-based caching as your only data storage. Without a doubt, *none* of the Django caching backends should be used for permanent storage -- they're all intended to be solutions for caching, not storage -- but we point this out here because memory-based caching is particularly temporary.

<hr>

**Deprecated since version 3.2**:

The `MemcachedCache` backend is deprecated as `python-memcached` has some problems and seems to be unmaintained. Use `PyMemcacheCache` or `PyLibMCCache` instead.

<hr>

### Redis

[Redis](https://redis.io/) is an in-memory database that can be used for caching. To begin, you'll need a Redis server running either locally or on a remote machine.

After setting up the Redis server, you'll need to install Python bindings for Redis. [redis.py](https://pypi.org/project/redis/) is the binding supported natively by Django. Installing the additional [hiredis-py](https://pypi.org/project/hiredis/) package is also recommended.

To use Redis as your cache backend with Django:

* Set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) to `django.core.cache.backends.redis.RedisCache`.
* Set [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) to the URL pointing to your Redis instance, using the appropriate scheme. See the `redis-py` docs for [details on the available schemes](https://redis-py.readthedocs.io/en/stable/connections.html#redis.connection.ConnectionPool.from_url).

For example, if Redis is running on localhost (127.0.0.1) port 6379:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379',
    }
}
```
Often Redis servers are protected with authentication. In order to supply a username and password, add them in the `LOCATION` along with the URL:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://username:password@127.0.0.1:6379',
    }
}
```
If you have multiple Redis servers set up in the replication mode, you can specify the servers either as a semicolon or comma delimited string, or as a list. While using multiple servers, write operations are performed on the first server (leader). Read operations are performed on the other servers (replicas) chosen at random:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': [
            'redis://127.0.0.1:6379',   # leader
            'redis://127.0.0.1:6378',   # read-replica 1
            'redis://127.0.0.1:6377',   # read-replica 2
        ],
    }
}
```

### Database caching

Django can store its cached data in your database. This works best if you've got a fast, well-indexed database server.

To use a database table as your cache backend:

* Set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) to `django.core.cache.backends.db.DatabaseCache`.
* Set [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) to `tablename`, the name of the database table. This name can be whatever you want, as long as it's a valid table name that's not already being used in your database.

In this example, the cache table's name is `my_cache_table`:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
```
Unlike other cache backends, the database cache does not support automatic culling of expired entries at the database level. Instead, expired cache entries are culled each time `add()`, `set()`, or `touch()` is called.

#### Creating the cache table

Before using the database cache, you must create the cache table with this command:
```
python manage.py createcachetable
```
This creates a table in your database that is in the proper format that Django's database-cache system expects. The name of the table is taken from [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION).

If you are using multiple database caches, [`createcachetable`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-createcachetable) creates one table for each cache.

If you are using multiple databases, `createcachetable` observes the `allow_migrate()` method of your database routers (see below).

Like [`migrate`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-migrate), `createcachetable` won't touch an existing table. It will only create missing tables.

To print the SQL that would be run, rather than run it, use the [`createcachetable --dry-run`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#cmdoption-createcachetable-dry-run) option.

#### Multiple databases

If you use database caching with multiple databases, you'll also need to set up routing instructions for your database cache table. For the purposes of routing, the database cache table appears as a model named `CacheEntry`, in an application named `django_cache`. This model won't appear in the models cache, but the model details can be used for routing purposes.

For example, the following router would direct all cache read operations to `cache_replica`, and all write operations to `cache_primary`. The cache table will only be synchronized onto `cache_primary`:
```
class CacheRouter:
    """A router to control all database cache operations"""

    def db_for_read(self, model, **hints):
        "All cache read operations go to the replica"
        if model._meta.app_label == 'django_cache':
            return 'cache_replica'
        return None

    def db_for_write(self, model, **hints):
        "All cache write operations go to primary"
        if model._meta.app_label == 'django_cache':
            return 'cache_primary'
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        "Only install the cache model on primary"
        if app_label == 'django_cache':
            return db == 'cache_primary'
        return None
```
If you don't specify routing directions for the database cache model, the cache backend will use the `default` database.

And if you don't use the database cache backend, you don't need to worry about providing routing instructions for the database cache model.

### Filesystem caching

The file-based backend serializes and stores each cache value as a separate file. To use this backend, set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) to `"django.core.cache.backends.filebased.FileBasedCache"` and [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) to a suitable directory. For example, to store cached data in `/var/tmp/django_cache`, use this setting:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
    }
}
```
If you're on Windows, put the drive letter at the beginning of the path, like this:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': 'c:/foo/bar',
    }
}
```
The directory path should be absolute -- that is, it should start at the root of your filesystem. It doesn't matter whether you put a slash at the end of the setting.

Make sure the directory pointed-to by this setting either exists and is readable and writable, or that it can be created by the system user under which your web server runs. Continuing the above example, if your server runs as the user `apache`, make sure the directory `/var/tmp/django_cache` exists and is readable and writable by the user `apache`, or that it can be created by the user `apache`.

<hr>

**Warning**

When the cache [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) is contained within [`MEDIA_ROOT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MEDIA_ROOT), [`STATIC_ROOT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-STATIC_ROOT), or [`STATICFILES_FINDERS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-STATICFILES_FINDERS), sensitive data may be exposed.

An attacker who gains access to the cache file can not only falsify HTML content, which your site will trust, but also remotely execute arbitrary code, as the data is serialized using [`pickle`](https://docs.python.org/3/library/pickle.html#module-pickle).

<hr>

### Local-memory caching

This is the default cache if another is not specified in your settings file. If you want the speed advantages of in-memory caching but don't have the capability of running Memcached, consider the local-memory cache backend. This cache is per-process (see below) and thread-safe. To use it, set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) to `"django.core.cache.backends.locmem.LocMemCache"`. For example:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}
```
The cache [`LOCATION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-LOCATION) is used to identify individual memory stores. If you only have one `locmem` cache, you can omit the `LOCATION`; however, if you have more than one local memory cache, you will need to assign a name to, at least, one of them in order to keep them separate.

The cache uses a least-recently-used (LRU) culling strategy.

Note that each process will have its own private cache instance, which means no cross-process caching is possible. This also means the local memory cache isn't particularly memory-efficient, so it's probably not a good choice for production environments. It's nice for development.

### Dummy caching (for development)

Finally, Django comes with a "dummy" cache that doesn't actually cache -- it just implements the cache interface without doing anything.

This is useful if you have a production site that uses heavy-duty caching in various places but a development/test environment where you don't want to cache and don't want to have to change your code to special-case the latter. To activate dummy caching, set [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) like so:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}
```

### Using a custom cache backend

While Django includes support for a number of cache backends out-of-the-box, sometimes you might want to use a customized cache backend. To use an external cache backend with Django, use the Python import path as the [`BACKEND`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-BACKEND) of the [`CACHES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES) setting, like so:
```
CACHES = {
    'default': {
        'BACKEND': 'path.to.backend',
    }
}
```
If you're building your own backend, you can use the standard cache backends as reference implementations. You'll find the code in the `django/core/cache/backends/` directory of the Django source.

Note: Without a really compelling reason, such as a host that doesn't support them, you should stick to the cache backends included with Django. They've been well-tested and are well-documented.

### Cache arguments

Each cache backend can be given additional arguments to control caching behavior. These arguments are provided as additional keys in the [`CACHES`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES) setting. Valid arguments are as follows:

* [`TIMEOUT`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-TIMEOUT): The default timeout, in seconds, to use for the cache. This argument defaults to `300` seconds (5 minutes). You can set `TIMEOUT` to `None` so that, by default, cache keys never expire. A value of `0` causes keys to immediately expire (effectively "don't cache").
* [`OPTIONS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-OPTIONS): Any options that should be passed to the cache backend. The list of valid options will vary with each backend, and cache backends backed by a third-party library will pass their options directly to the underlying cache library.

    Cache backends that implement their own culling strategy (i.e., the `locmem`, `filesystem`, and `database` backends) will honor the following options:

    * `MAX_ENTRIES`: The maximum number of entries allowed in the cache before old values are deleted. This argument defaults to `300`.
    * `CULL_FREQUENCY`: The fraction of entries that are culled when `MAX_ENTRIES` is reached. The actual ratio if `1 / CULL_FREQUENCY`, so set `CULL_FREQUENCY` to `2` to cull half the entries when `MAX_ENTRIES` is reached. This argument should be an integer and defaults to `3`.

        A value of `0` for `CULL_FREQUENCY` means that the entire cache will be dumped when `MAX_ENTRIES` is reached. On some backends (`database` in particular), this makes culling *much* faster at the expense of more cache misses.

    The Memcached and Redis backends pass the contents of [`OPTIONS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-OPTIONS) as keyword arguments to the client constructors, allowing for more advanced control of client behavior. For example usage, see below.

* [`KEY_PREFIX`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-KEY_PREFIX): A string that will be automatically included (prepended by default) to all cache keys used by the Django server.

    See the [cache documentation]() for more information. <!-- "Cache key prefixing" below -->

* [`VERSION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-VERSION): The default version number for cache keys generated by the Django server.

    See the [cache documentation]() for more information. <!-- "Cache versioning" below -->

* [`KEY_FUNCTION`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHES-KEY_FUNCTION): The default version number for cache keys generated by the Django server.

    See the [cache documentation]() for more information. <!-- "Cache key transformation" below -->

In this example, a filesystem backend is being configured with a timeout of 60 seconds, and a maximum capacity of 1000 items:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        'TIMEOUT': 60,
        'OPTIONS': {
            'MAX_ENTRIES': 1000
        }
    }
}
```
Here's an example configuration for a `pylibmc` based backend that enables the binary protocol, SASL authentication, and the `ketama` behavior mode:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
        'OPTIONS': {
            'binary': True,
            'username': 'user',
            'password': 'pass',
            'behaviors': {
                'ketama': True,
            }
        }
    }
}
```
Here's an example configuration for a `pymemcache` based backend that enables client pooling (which may improve performance by keeping clients connected), treats memcache/network errors as cache misses, and sets the `TCP_NODELAY` flag on the connection's socket:
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
        'OPTIONS': {
            'no_delay': True,
            'ignore_exc': True,
            'max_pool_size': 4,
            'use_pooling': True,
        }
    }
}
```
Here's an example configuration for a `redis` based backend that selects database `10` (by default, Redis ships with 16 logical databases), specifies a [parser class](https://github.com/redis/redis-py#parsers) (`redis.connection.HiredisParser` will be used by default if the `hiredis-py` package is installed), and sets a custom [connection pool class](https://github.com/redis/redis-py#connection-pools) (`redis.ConnectionPool` is used by default):
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379',
        'OPTIONS': {
            'db': '10',
            'parser_class': 'redis.connection.PythonParser',
            'pool_class': 'redis.BlockingConnectionPool',
        }
    }
}
```

## The per-site cache

Once the cache is set up, the simplest way to use caching is to cache your entire site. You'll need to add `'django.middleware.cache.UpdateCacheMiddleware'` and `'django.middleware.cache.FetchFromCacheMiddleware'` to your [`MIDDLEWARE`]() setting, as in this example:
```
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',
]
```

<hr>

**Note**: No, that's not a typo: the "update" middleware must be first in the list, and the "fetch" middleware must be last. The details are a bit obscure, but see [Order of `MIDDLEWARE`]() below if you'd like the full story.

<hr>

Then, add the following required settings to your Django settings file:

* [`CACHE_MIDDLEWARE_ALIAS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHE_MIDDLEWARE_ALIAS) - The cache alias to use for storage.
* [`CACHE_MIDDLEWARE_SECONDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHE_MIDDLEWARE_SECONDS) - The number of seconds each page should be cached.
* [`CACHE_MIDDLEWARE_KEY_PREFIX`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHE_MIDDLEWARE_KEY_PREFIX) - If the cache is shared across multiple sites using the same Django installation, set this to the name of the site, or some other string that is unique to this Django instance, to prevent key collisions. Use an empty string if you don't care.

`FetchFromCachedMiddleware` caches `GET` and `HEAD` responses with status 200, where the request and response headers allow. Responses to requests for the same URL with different query parameters are considered to be unique pages and are cached separately. This middleware expects that a `HEAD` request is answered with the same response headers as the corresponding `GET` request; in which case it can return a cached `GET` response for `HEAD` request.

Additionally, `UpdateCacheMiddleware` automatically sets a few headers in each [`HttpResponse`](https://docs.djangoproject.com/en/4.0/ref/request-response/#django.http.HttpResponse) which affect [downstream caches](): <!-- "Downstream caches" below -->

* Sets the `Expires` header to the current date/time plus the defined [`CACHE_MIDDLEWARE_SECONDS`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-CACHE_MIDDLEWARE_SECONDS).
* Sets the `Cache-Control` header to give a max age for the page -- again, from the `CACHE_MIDDLEWARE_SECONDS` setting.

See [Middleware](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Handling_HTTP_Requests/Middleware#middleware) for more on middleware.

If a view sets its own cache expiry time (i.e. it has a `max-age` section in its `Cache-Control` header), then the page will be cached until the expiry time, rather than `CACHE_MIDDLEWARE_SECONDS`. Using the decorators in `django.views.decorators.cache`, you can easily set a view's expiry time (using the [`cache_control()`](https://docs.djangoproject.com/en/4.0/topics/http/decorators/#django.views.decorators.cache.cache_control) decorator) or disable caching for a view (using the [`never_cache()`](https://docs.djangoproject.com/en/4.0/topics/http/decorators/#django.views.decorators.cache.never_cache) decorator). See the [using other headers]() section for more on these decorators.