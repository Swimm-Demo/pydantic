---
title: Deprecated JSON Encoding Utilities in Pydantic
---
# Introduction

This document explains the rationale behind the deprecated JSON encoding utilities in <SwmPath>[pydantic/deprecated/json.py](pydantic/deprecated/json.py)</SwmPath>. We focus on:

1. Why these utilities were implemented and what problem they solve.
2. How the encoding of special types like <SwmToken path="pydantic/deprecated/json.py" pos="33:7:7" line-data="def decimal_encoder(dec_value: Decimal) -&gt; Union[int, float]:">`Decimal`</SwmToken> and <SwmToken path="pydantic/deprecated/json.py" pos="60:3:3" line-data="    datetime.timedelta: lambda td: td.total_seconds(),">`timedelta`</SwmToken> is handled.
3. The approach to extensible encoding via type-based dispatch.
4. The reason for deprecating these utilities and the suggested alternatives.

# encoding special types

The file provides custom encoding logic for types that the standard JSON encoder cannot handle directly. For example, <SwmToken path="pydantic/deprecated/json.py" pos="33:2:2" line-data="def decimal_encoder(dec_value: Decimal) -&gt; Union[int, float]:">`decimal_encoder`</SwmToken> converts <SwmToken path="pydantic/deprecated/json.py" pos="33:7:7" line-data="def decimal_encoder(dec_value: Decimal) -&gt; Union[int, float]:">`Decimal`</SwmToken> instances to either `int` or <SwmToken path="pydantic/deprecated/json.py" pos="33:17:17" line-data="def decimal_encoder(dec_value: Decimal) -&gt; Union[int, float]:">`float`</SwmToken> depending on their exponent. This avoids issues with <SwmToken path="pydantic/deprecated/json.py" pos="38:7:9" line-data="    results in failed round-tripping between encode and parse.">`round-tripping`</SwmToken> numeric values that are conceptually integers but typed as <SwmToken path="pydantic/deprecated/json.py" pos="33:7:7" line-data="def decimal_encoder(dec_value: Decimal) -&gt; Union[int, float]:">`Decimal`</SwmToken>. This is important because encoding all decimals as floats can cause precision loss or parsing errors later.

The <SwmToken path="pydantic/deprecated/json.py" pos="29:2:2" line-data="def isoformat(o: Union[datetime.date, datetime.time]) -&gt; str:">`isoformat`</SwmToken> function is a simple wrapper to call the standard <SwmToken path="pydantic/deprecated/json.py" pos="30:4:7" line-data="    return o.isoformat()">`.isoformat()`</SwmToken> method on date/time objects, ensuring consistent string serialization.

<SwmSnippet path="/pydantic/deprecated/json.py" line="29">

---

The <SwmToken path="pydantic/deprecated/json.py" pos="26:15:15" line-data="__all__ = &#39;pydantic_encoder&#39;, &#39;custom_pydantic_encoder&#39;, &#39;timedelta_isoformat&#39;">`timedelta_isoformat`</SwmToken> function encodes <SwmToken path="pydantic/deprecated/json.py" pos="60:3:3" line-data="    datetime.timedelta: lambda td: td.total_seconds(),">`timedelta`</SwmToken> objects into an ISO 8601 duration string. This is a custom format since <SwmToken path="pydantic/deprecated/json.py" pos="60:3:3" line-data="    datetime.timedelta: lambda td: td.total_seconds(),">`timedelta`</SwmToken> is not natively serializable to JSON. However, this function is deprecated, signaling a move away from custom string formats for timedeltas.

```python
def isoformat(o: Union[datetime.date, datetime.time]) -> str:
    return o.isoformat()


def decimal_encoder(dec_value: Decimal) -> Union[int, float]:
    """Encodes a Decimal as int of there's no exponent, otherwise float.

    This is useful when we use ConstrainedDecimal to represent Numeric(x,0)
    where a integer (but not int typed) is used. Encoding this as a float
    results in failed round-tripping between encode and parse.
    Our Id type is a prime example of this.

    >>> decimal_encoder(Decimal("1.0"))
    1.0

    >>> decimal_encoder(Decimal("1"))
    1
    """
    exponent = dec_value.as_tuple().exponent
    if isinstance(exponent, int) and exponent >= 0:
        return int(dec_value)
    else:
        return float(dec_value)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/json.py" line="135">

---

&nbsp;

```python
@deprecated('`timedelta_isoformat` is deprecated.', category=None)
def timedelta_isoformat(td: datetime.timedelta) -> str:
    """ISO 8601 encoding for Python timedelta object."""
    warnings.warn('`timedelta_isoformat` is deprecated.', category=PydanticDeprecatedSince20, stacklevel=2)
    minutes, seconds = divmod(td.seconds, 60)
    hours, minutes = divmod(minutes, 60)
    return f'{"-" if td.days < 0 else ""}P{abs(td.days)}DT{hours:d}H{minutes:d}M{seconds:d}.{td.microseconds:06d}S'
```

---

</SwmSnippet>

# type-based encoder dispatch

The core of the encoding logic is the <SwmToken path="pydantic/deprecated/json.py" pos="54:0:0" line-data="ENCODERS_BY_TYPE: dict[type[Any], Callable[[Any], Any]] = {">`ENCODERS_BY_TYPE`</SwmToken> dictionary. It maps Python types to functions that convert instances of those types into JSON-serializable forms. This dictionary covers many built-in and pydantic-specific types, such as <SwmToken path="pydantic/deprecated/json.py" pos="55:1:1" line-data="    bytes: lambda o: o.decode(),">`bytes`</SwmToken>, <SwmToken path="pydantic/deprecated/json.py" pos="56:1:1" line-data="    Color: str,">`Color`</SwmToken>, <SwmToken path="pydantic/deprecated/json.py" pos="62:1:1" line-data="    Enum: lambda o: o.value,">`Enum`</SwmToken>, IP address types, and secret types.

The <SwmToken path="pydantic/deprecated/json.py" pos="26:5:5" line-data="__all__ = &#39;pydantic_encoder&#39;, &#39;custom_pydantic_encoder&#39;, &#39;timedelta_isoformat&#39;">`pydantic_encoder`</SwmToken> function uses this dictionary to find the appropriate encoder for an object by checking its class and superclasses. It also handles <SwmToken path="pydantic/deprecated/json.py" pos="94:1:1" line-data="    BaseModel = import_cached_base_model()">`BaseModel`</SwmToken> instances by calling their <SwmToken path="pydantic/deprecated/json.py" pos="97:5:5" line-data="        return obj.model_dump()">`model_dump`</SwmToken> method, and dataclasses by converting them to dictionaries. If no encoder is found, it raises a <SwmToken path="pydantic/deprecated/json.py" pos="109:3:3" line-data="        raise TypeError(f&quot;Object of type &#39;{obj.__class__.__name__}&#39; is not JSON serializable&quot;)">`TypeError`</SwmToken>.

<SwmSnippet path="/pydantic/deprecated/json.py" line="54">

---

This design allows flexible and extensible encoding by type, centralizing the logic and avoiding scattered custom serialization code.

```python
ENCODERS_BY_TYPE: dict[type[Any], Callable[[Any], Any]] = {
    bytes: lambda o: o.decode(),
    Color: str,
    datetime.date: isoformat,
    datetime.datetime: isoformat,
    datetime.time: isoformat,
    datetime.timedelta: lambda td: td.total_seconds(),
    Decimal: decimal_encoder,
    Enum: lambda o: o.value,
    frozenset: list,
    deque: list,
    GeneratorType: list,
    IPv4Address: str,
    IPv4Interface: str,
    IPv4Network: str,
    IPv6Address: str,
    IPv6Interface: str,
    IPv6Network: str,
    NameEmail: str,
    Path: str,
    Pattern: lambda o: o.pattern,
    SecretBytes: str,
    SecretStr: str,
    set: list,
    UUID: str,
}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/json.py" line="82">

---

&nbsp;

```python
@deprecated(
    '`pydantic_encoder` is deprecated, use `pydantic_core.to_jsonable_python` instead.',
    category=None,
)
def pydantic_encoder(obj: Any) -> Any:
    warnings.warn(
        '`pydantic_encoder` is deprecated, use `pydantic_core.to_jsonable_python` instead.',
        category=PydanticDeprecatedSince20,
        stacklevel=2,
    )
    from dataclasses import asdict, is_dataclass

    BaseModel = import_cached_base_model()

    if isinstance(obj, BaseModel):
        return obj.model_dump()
    elif is_dataclass(obj):
        return asdict(obj)  # type: ignore

    # Check the class type and its superclasses for a matching encoder
    for base in obj.__class__.__mro__[:-1]:
        try:
            encoder = ENCODERS_BY_TYPE[base]
        except KeyError:
            continue
        return encoder(obj)
    else:  # We have exited the for loop without finding a suitable encoder
        raise TypeError(f"Object of type '{obj.__class__.__name__}' is not JSON serializable")
```

---

</SwmSnippet>

# deprecation and migration path

All the main encoding utilities here are marked as deprecated. The warnings direct users to use <SwmToken path="pydantic/deprecated/json.py" pos="83:14:16" line-data="    &#39;`pydantic_encoder` is deprecated, use `pydantic_core.to_jsonable_python` instead.&#39;,">`pydantic_core.to_jsonable_python`</SwmToken> or <SwmToken path="pydantic/deprecated/json.py" pos="114:14:16" line-data="    &#39;`custom_pydantic_encoder` is deprecated, use `BaseModel.model_dump` instead.&#39;,">`BaseModel.model_dump`</SwmToken> instead. This reflects a shift in the library towards a more core-driven and model-centric serialization approach, likely for better performance and maintainability.

The <SwmToken path="pydantic/deprecated/json.py" pos="26:10:10" line-data="__all__ = &#39;pydantic_encoder&#39;, &#39;custom_pydantic_encoder&#39;, &#39;timedelta_isoformat&#39;">`custom_pydantic_encoder`</SwmToken> function, which allowed passing custom type encoders, is also deprecated in favor of using <SwmToken path="pydantic/deprecated/json.py" pos="97:5:5" line-data="        return obj.model_dump()">`model_dump`</SwmToken>. This simplifies the API surface and encourages users to rely on model methods for serialization.

<SwmSnippet path="/pydantic/deprecated/json.py" line="112">

---

The deprecation warnings are implemented with a custom warning class <SwmToken path="pydantic/deprecated/json.py" pos="120:3:3" line-data="        category=PydanticDeprecatedSince20,">`PydanticDeprecatedSince20`</SwmToken> to clearly signal the version since when these utilities are deprecated.

```python
# TODO: Add a suggested migration path once there is a way to use custom encoders
@deprecated(
    '`custom_pydantic_encoder` is deprecated, use `BaseModel.model_dump` instead.',
    category=None,
)
def custom_pydantic_encoder(type_encoders: dict[Any, Callable[[type[Any]], Any]], obj: Any) -> Any:
    warnings.warn(
        '`custom_pydantic_encoder` is deprecated, use `BaseModel.model_dump` instead.',
        category=PydanticDeprecatedSince20,
        stacklevel=2,
    )
    # Check the class type and its superclasses for a matching encoder
    for base in obj.__class__.__mro__[:-1]:
        try:
            encoder = type_encoders[base]
        except KeyError:
            continue

        return encoder(obj)
    else:  # We have exited the for loop without finding a suitable encoder
        return pydantic_encoder(obj)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/json.py" line="21">

---

&nbsp;

```python
if not TYPE_CHECKING:
    # See PyCharm issues https://youtrack.jetbrains.com/issue/PY-21915
    # and https://youtrack.jetbrains.com/issue/PY-51428
    DeprecationWarning = PydanticDeprecatedSince20

__all__ = 'pydantic_encoder', 'custom_pydantic_encoder', 'timedelta_isoformat'
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
