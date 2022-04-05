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