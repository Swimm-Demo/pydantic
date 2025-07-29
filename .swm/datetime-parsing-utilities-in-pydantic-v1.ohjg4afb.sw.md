---
title: Datetime Parsing Utilities in Pydantic v1
---
# introduction

This document explains the main design decisions behind the datetime parsing utilities in pydantic <SwmToken path="pydantic/v1/datetime_parse.py" pos="21:4:4" line-data="from pydantic.v1 import errors">`v1`</SwmToken>. The goal is to provide robust, flexible parsing of date, time, datetime, and duration inputs from various formats, including strings, numbers, and bytes.

We will cover:

1. Why regex-based parsing was chosen over standard library functions.
2. How unix timestamps are handled and normalized.
3. How timezone offsets are parsed and applied.
4. The approach to parsing date, time, and datetime inputs.
5. How durations are parsed from multiple formats.

# why regex-based parsing

The parsing functions rely on regular expressions instead of Python's built-in <SwmToken path="pydantic/v1/datetime_parse.py" pos="4:14:16" line-data="We&#39;re using regular expressions rather than time.strptime because:">`time.strptime`</SwmToken>. This choice was made because regexes allow simultaneous validation and parsing, which is more flexible for the variety of datetime formats pydantic supports. Additionally, using the standard <SwmToken path="pydantic/v1/datetime_parse.py" pos="7:4:8" line-data="- The date/datetime/time constructors produce friendlier error messages.">`date/datetime/time`</SwmToken> constructors after regex extraction produces clearer error messages for invalid inputs.

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="1">

---

This rationale is summarized in the file docstring:

```python
"""
Functions to parse datetime objects.

We're using regular expressions rather than time.strptime because:
- They provide both validation and parsing.
- They're more flexible for datetimes.
- The date/datetime/time constructors produce friendlier error messages.
```

---

</SwmSnippet>

# handling unix timestamps and numeric inputs

The code supports parsing unix timestamps for dates and datetimes, which can be provided as int, float, str, or bytes. To handle this, the utility first tries to convert the input to a numeric value. If successful, it normalizes the value to seconds (handling milliseconds if the number is large) and converts it to a UTC datetime relative to the Unix epoch.

This normalization logic is encapsulated in <SwmToken path="pydantic/v1/datetime_parse.py" pos="66:2:2" line-data="def get_numeric(value: StrBytesIntFloat, native_expected_type: str) -&gt; Union[None, int, float]:">`get_numeric`</SwmToken> and <SwmToken path="pydantic/v1/datetime_parse.py" pos="77:2:2" line-data="def from_unix_seconds(seconds: Union[int, float]) -&gt; datetime:">`from_unix_seconds`</SwmToken>:

- <SwmToken path="pydantic/v1/datetime_parse.py" pos="66:2:2" line-data="def get_numeric(value: StrBytesIntFloat, native_expected_type: str) -&gt; Union[None, int, float]:">`get_numeric`</SwmToken> attempts to convert the input to float, raising <SwmToken path="pydantic/v1/datetime_parse.py" pos="73:3:3" line-data="    except TypeError:">`TypeError`</SwmToken> if the input type is invalid.
- <SwmToken path="pydantic/v1/datetime_parse.py" pos="77:2:2" line-data="def from_unix_seconds(seconds: Union[int, float]) -&gt; datetime:">`from_unix_seconds`</SwmToken> converts the numeric timestamp to a datetime, clamping extreme values to datetime.min/max and dividing by 1000 if the number is suspiciously large (indicating milliseconds).

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="66">

---

This approach ensures consistent handling of timestamps regardless of input type or scale.

```python
def get_numeric(value: StrBytesIntFloat, native_expected_type: str) -> Union[None, int, float]:
    if isinstance(value, (int, float)):
        return value
    try:
        return float(value)
    except ValueError:
        return None
    except TypeError:
        raise TypeError(f'invalid type; expected {native_expected_type}, string, bytes, int or float')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="77">

---

&nbsp;

```python
def from_unix_seconds(seconds: Union[int, float]) -> datetime:
    if seconds > MAX_NUMBER:
        return datetime.max
    elif seconds < -MAX_NUMBER:
        return datetime.min

    while abs(seconds) > MS_WATERSHED:
        seconds /= 1000
    dt = EPOCH + timedelta(seconds=seconds)
    return dt.replace(tzinfo=timezone.utc)
```

---

</SwmSnippet>

# parsing timezone offsets

Timezone parsing is handled by a dedicated helper that interprets strings like 'Z' (UTC) or offsets like '+05:30'. It extracts hour and minute offsets, applies the sign, and returns a fixed-offset timezone object. If the offset is invalid, it raises the appropriate error.

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="89">

---

This function is used by time and datetime parsers to attach timezone info to the resulting objects.

```python
def _parse_timezone(value: Optional[str], error: Type[Exception]) -> Union[None, int, timezone]:
    if value == 'Z':
        return timezone.utc
    elif value is not None:
        offset_mins = int(value[-2:]) if len(value) > 3 else 0
        offset = 60 * int(value[1:3]) + offset_mins
        if value[0] == '-':
            offset = -offset
        try:
            return timezone(timedelta(minutes=offset))
        except ValueError:
            raise error()
    else:
        return None
```

---

</SwmSnippet>

# parsing date, time, and datetime inputs

Each of these types has a dedicated parse function that:

- Returns the input immediately if it is already the correct type.
- Attempts numeric parsing as a unix timestamp.
- Decodes bytes to string if needed.
- Matches the input against a regex to extract components.
- Converts extracted components to integers.
- Applies timezone parsing for time and datetime.
- Constructs the corresponding datetime object, raising errors on invalid inputs.

For example, <SwmToken path="pydantic/v1/datetime_parse.py" pos="105:2:2" line-data="def parse_date(value: Union[date, StrBytesIntFloat]) -&gt; date:">`parse_date`</SwmToken>:

- Returns the date if input is already a date or datetime.
- Parses numeric unix timestamps to dates.
- Uses a regex to extract year, month, day.
- Constructs a date object or raises <SwmToken path="pydantic/v1/datetime_parse.py" pos="127:5:5" line-data="        raise errors.DateError()">`DateError`</SwmToken>.

<SwmToken path="pydantic/v1/datetime_parse.py" pos="137:2:2" line-data="def parse_time(value: Union[time, StrBytesIntFloat]) -&gt; time:">`parse_time`</SwmToken> and <SwmToken path="pydantic/v1/datetime_parse.py" pos="175:2:2" line-data="def parse_datetime(value: Union[datetime, StrBytesIntFloat]) -&gt; datetime:">`parse_datetime`</SwmToken> follow similar patterns but also handle microseconds and timezone offsets.

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="105">

---

This layered approach separates concerns: validation via regex, conversion via constructors, and timezone handling via a helper.

```python
def parse_date(value: Union[date, StrBytesIntFloat]) -> date:
    """
    Parse a date/int/float/string and return a datetime.date.

    Raise ValueError if the input is well formatted but not a valid date.
    Raise ValueError if the input isn't well formatted.
    """
    if isinstance(value, date):
        if isinstance(value, datetime):
            return value.date()
        else:
            return value

    number = get_numeric(value, 'date')
    if number is not None:
        return from_unix_seconds(number).date()

    if isinstance(value, bytes):
        value = value.decode()

    match = date_re.match(value)  # type: ignore
    if match is None:
        raise errors.DateError()

    kw = {k: int(v) for k, v in match.groupdict().items()}

    try:
        return date(**kw)
    except ValueError:
        raise errors.DateError()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="137">

---

&nbsp;

```python
def parse_time(value: Union[time, StrBytesIntFloat]) -> time:
    """
    Parse a time/string and return a datetime.time.

    Raise ValueError if the input is well formatted but not a valid time.
    Raise ValueError if the input isn't well formatted, in particular if it contains an offset.
    """
    if isinstance(value, time):
        return value

    number = get_numeric(value, 'time')
    if number is not None:
        if number >= 86400:
            # doesn't make sense since the time time loop back around to 0
            raise errors.TimeError()
        return (datetime.min + timedelta(seconds=number)).time()

    if isinstance(value, bytes):
        value = value.decode()

    match = time_re.match(value)  # type: ignore
    if match is None:
        raise errors.TimeError()

    kw = match.groupdict()
    if kw['microsecond']:
        kw['microsecond'] = kw['microsecond'].ljust(6, '0')

    tzinfo = _parse_timezone(kw.pop('tzinfo'), errors.TimeError)
    kw_: Dict[str, Union[None, int, timezone]] = {k: int(v) for k, v in kw.items() if v is not None}
    kw_['tzinfo'] = tzinfo

    try:
        return time(**kw_)  # type: ignore
    except ValueError:
        raise errors.TimeError()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="175">

---

&nbsp;

```python
def parse_datetime(value: Union[datetime, StrBytesIntFloat]) -> datetime:
    """
    Parse a datetime/int/float/string and return a datetime.datetime.

    This function supports time zone offsets. When the input contains one,
    the output uses a timezone with a fixed offset from UTC.

    Raise ValueError if the input is well formatted but not a valid datetime.
    Raise ValueError if the input isn't well formatted.
    """
    if isinstance(value, datetime):
        return value

    number = get_numeric(value, 'datetime')
    if number is not None:
        return from_unix_seconds(number)

    if isinstance(value, bytes):
        value = value.decode()

    match = datetime_re.match(value)  # type: ignore
    if match is None:
        raise errors.DateTimeError()

    kw = match.groupdict()
    if kw['microsecond']:
        kw['microsecond'] = kw['microsecond'].ljust(6, '0')

    tzinfo = _parse_timezone(kw.pop('tzinfo'), errors.DateTimeError)
    kw_: Dict[str, Union[None, int, timezone]] = {k: int(v) for k, v in kw.items() if v is not None}
    kw_['tzinfo'] = tzinfo

    try:
        return datetime(**kw_)  # type: ignore
    except ValueError:
        raise errors.DateTimeError()
```

---

</SwmSnippet>

# parsing durations

Duration parsing supports both a Django-style format ('%d %H:%M:%S.%f') and ISO 8601 durations. The parser:

- Returns the input if already a timedelta.
- Converts numeric inputs to strings for regex matching.
- Matches against either the standard or ISO 8601 regex.
- Extracts components, normalizes microseconds, and applies sign.
- Constructs a timedelta from the extracted float values.

<SwmSnippet path="/pydantic/v1/datetime_parse.py" line="213">

---

This dual-format support increases flexibility for duration inputs.

```python
def parse_duration(value: StrBytesIntFloat) -> timedelta:
    """
    Parse a duration int/float/string and return a datetime.timedelta.

    The preferred format for durations in Django is '%d %H:%M:%S.%f'.

    Also supports ISO 8601 representation.
    """
    if isinstance(value, timedelta):
        return value

    if isinstance(value, (int, float)):
        # below code requires a string
        value = f'{value:f}'
    elif isinstance(value, bytes):
        value = value.decode()

    try:
        match = standard_duration_re.match(value) or iso8601_duration_re.match(value)
    except TypeError:
        raise TypeError('invalid type; expected timedelta, string, bytes, int or float')

    if not match:
        raise errors.DurationError()

    kw = match.groupdict()
    sign = -1 if kw.pop('sign', '+') == '-' else 1
    if kw.get('microseconds'):
        kw['microseconds'] = kw['microseconds'].ljust(6, '0')

    if kw.get('seconds') and kw.get('microseconds') and kw['seconds'].startswith('-'):
        kw['microseconds'] = '-' + kw['microseconds']

    kw_ = {k: float(v) for k, v in kw.items() if v is not None}

    return sign * timedelta(**kw_)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
