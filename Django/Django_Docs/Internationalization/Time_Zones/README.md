# Time zones

## Overview

When support for time zones is enabled, Django stores datetime information in UTC in the database, uses time-zone-aware datetime objects internally, and translates then to the end user's time zone in templates and forms.

This is handy if your users live in more than one time zone and you want to display datetime information according to each user's wall clock.

Even if your website is available in only one time zone, it's still good practice to store data in UTC in your database. The main reason is daylight saving time (DST). Many countries have a system of DST, where clocks are moved forward in spring and backward in autumn. If you're working in local time, you're likely to encounter errors twice a year, when the transitions happen. This probably doesn't matter for your blog, but it's a problem if you over bill or under bill your customers by one hour, twice a year, every year. The solution to this problem is to use UTC in the code and use local time only when interacting with end users.

Time zone support is disabled by default. To enable it, set [`USE_TZ = True`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_TZ) in your settings file.

<hr>

**Note**: In Django 5.0, time zone support will be enabled by default.

<hr>

Time zone support uses [`zoneinfo`](https://docs.python.org/3/library/zoneinfo.html#module-zoneinfo), which is part of the Python standard library from Python 3.9. The `backports.zoneinfo` package is automatically installed alongside Django if you are using Python 3.8.

<hr>

**Note**: The default `settings.py` file created by [`django-admin startproject`](https://docs.djangoproject.com/en/4.0/ref/django-admin/#django-admin-startproject) includes [`USE_TZ = True`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_TZ) for convenience.

<hr>

If you're wrestling with a particular problem, start with the [time zone FAQ](https://docs.djangoproject.com/en/4.0/topics/i18n/timezones/#time-zones-faq).

## Concepts

### Naive and aware datetime objects

Python's [`datetime.datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) objects have a `tzinfo` attribute that can be used to store time zone information, represented as an instance of a subclass of [`datetime.tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo). When this attribute is set and describes an offset, a datetime object is **aware**. Otherwise, it's **naive**.

You can use [`is_aware()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.timezone.is_aware) and [`is_naive()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.timezone.is_naive) to determine whether datetimes are aware or naive.

When time zone support is disabled, Django uses naive datetime objects in local time. This is sufficient for many use cases. In this mode, to obtain the current time, you would write:
```
import datetime

now = datetime.datetime.now()
```
When time zone support is enabled ([`USE_TZ=True`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_TZ)), Django uses time-zone-aware datetime objects. If your code creates datetime objects, they should be aware, too. In this mode, the example above becomes:
```
from django.utils import timezone

now = timezone.now()
```

<hr>

:warning: **Warning**: Dealing with aware datetime objects isn't always intuitive. For instance, the `tzinfo` argument of the standard datetime constructor doesn't work reliably for time zones with DST. Using UTC is generally safe; if you're using other time zones, you should review the [`zoneinfo`](https://docs.python.org/3/library/zoneinfo.html#module-zoneinfo) documentation carefully.

<hr>

**Note**: Python's [`datetime.time`](https://docs.python.org/3/library/datetime.html#datetime.time) objects also feature a `tzinfo` attribute, and PostgreSQL has a matching `time with time zone` type. However, as PostgreSQL's docs put it, this type "exhibits properties which lead to questionable usefulness".

Djkango only supports naive time objects and will raise an exception if you attempt to save an aware time object, as a timezone for a time with no associated date does not make sense.

<hr>

### Interpretation of naive datetime objects

When [`USE_TZ`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_TZ) is `True`, Django still accepts naive datetime objects, in order to preserve backwards-compatibility. When the database layer receives one, it attempts to make it aware by interpreting it in the [default time zone]() and raises a warning. <!-- "Default time zone and..." below -->

Unfortunately, during DST transitions, some datetimes don't exist or are ambiguous. That's why you should always create aware datetime objects when time zone support is enabled. (See the [Using `ZoneInfo` section of the `zoneinfo` docs](https://docs.python.org/3/library/zoneinfo.html#module-zoneinfo) for examples using the `fold` attribute to specify the offset that should apply to a datetime during a DST transition.)

In practice, this is rarely an issue. Django gives you aware datetime objects in the models and forms, and most often, new datetime objects are created from existing ones through [`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta) arithmetic. The only datetime that's often created in application code is the current time, and [`timezone.now()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.timezone.now) automatically does the right thing.

### Default time zone and current time zone

The **default time zone** is the time zone defined by the [`TIME_ZONE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TIME_ZONE) setting.

The **current time zone** is the time zone that's used for rendering.

You should set the current time zone to the end user's actual time zone with [`activate()`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.timezone.activate). Otherwise, the default time zone is used.

<hr>

**Note**: As explained in the documentation of [`TIME_ZONE`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-TIME_ZONE), Django sets environment variables so that its process runs in the default time zone. This happens regardless of the value of [`USE_TZ`](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-USE_TZ) and of the current time zone.

When `USE_TZ` is `True`, this is useful to preserve backwards-compatibility with applications that still rely on local time. However, [as explained above](), <!-- "Interpretation of naive datetime..." above --> this isn't entirely reliable, and you should always work with aware datetimes in UTC in your own code. For instance, use [`fromtimestamp()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.fromtimestamp) and set the `tz` parameter to [`utc`](https://docs.djangoproject.com/en/4.0/ref/utils/#django.utils.timezone.utc).

<hr>

### Selecting the current time zone

The current time zone is the equivalent of the current [locale](https://github.com/AndrewSRea/My_Learning_Port_II/tree/main/Django/Django_Docs/Internationalization#locale-name) for translations. However, there's no equivalent of the `Accept-Language` HTTP header that Django could use to determine the user's time zone automatically. Instead, Django provides [time zone selection functions](https://docs.djangoproject.com/en/4.0/ref/utils/#time-zone-selection-functions). Use them to build the time zone selection logic that makes sene for you.