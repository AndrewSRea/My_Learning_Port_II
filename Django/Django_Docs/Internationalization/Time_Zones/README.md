# Time zones

## Overview

When support for time zones is enabled, Django stores datetime information in UTC in the database, uses time-zone-aware datetime objects internally, and translates then to the end user's time zone in templates and forms.