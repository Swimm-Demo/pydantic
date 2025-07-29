---
title: The _BaseUrl class
---
# Inheritance diagram

This diagram shows the inheritance tree of the class:

```mermaid
graph TD;
  _BaseUrl:::currentBaseStyle
_BaseUrl --> AnyUrl
AnyUrl --> 1[...]

 classDef currentBaseStyle color:#000000,fill:#7CB9F4

%% Swimm:
%% graph TD;
%%   <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken>:::currentBaseStyle
%% <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> --> <SwmToken path="pydantic/networks.py" pos="42:2:2" line-data="    &#39;AnyUrl&#39;,">`AnyUrl`</SwmToken>
%% <SwmToken path="pydantic/networks.py" pos="42:2:2" line-data="    &#39;AnyUrl&#39;,">`AnyUrl`</SwmToken> --> 1[...]
%% 
%%  classDef currentBaseStyle color:#000000,fill:#7CB9F4
```

# What is <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken>

# Variables and functions

# What is <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken>

<SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> is an internal base class in the Pydantic codebase that provides a unified interface for representing and manipulating <SwmToken path="pydantic/networks.py" pos="535:12:12" line-data="    &quot;&quot;&quot;Base type for all URLs.">`URLs`</SwmToken>. It encapsulates the logic for parsing, validating, and accessing different components of a URL, such as the scheme, host, port, path, query, and fragment. This class is designed to be extended by more specific URL types and is not intended for direct use by end users.

<SwmSnippet path="/pydantic/networks.py" line="124">

---

The class variable <SwmToken path="pydantic/networks.py" pos="124:1:1" line-data="    _constraints: ClassVar[UrlConstraints] = UrlConstraints()">`_constraints`</SwmToken> holds an instance of <SwmToken path="pydantic/networks.py" pos="124:6:6" line-data="    _constraints: ClassVar[UrlConstraints] = UrlConstraints()">`UrlConstraints`</SwmToken>, which defines the validation rules and constraints for <SwmToken path="pydantic/networks.py" pos="535:12:12" line-data="    &quot;&quot;&quot;Base type for all URLs.">`URLs`</SwmToken> handled by this class.

```python
    _constraints: ClassVar[UrlConstraints] = UrlConstraints()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="125">

---

The instance variable <SwmToken path="pydantic/networks.py" pos="125:1:1" line-data="    _url: _CoreUrl">`_url`</SwmToken> stores the parsed URL as a <SwmToken path="pydantic/networks.py" pos="125:4:4" line-data="    _url: _CoreUrl">`_CoreUrl`</SwmToken> object, which is used internally to access the different parts of the URL.

```python
    _url: _CoreUrl
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="127">

---

The <SwmToken path="pydantic/networks.py" pos="127:3:3" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`__init__`</SwmToken> function initializes a <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instance by validating and parsing the input URL, which can be a string, a <SwmToken path="pydantic/networks.py" pos="127:15:15" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_CoreUrl`</SwmToken>, or another <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instance.

```python
    def __init__(self, url: str | _CoreUrl | _BaseUrl) -> None:
        self._url = _build_type_adapter(self.__class__).validate_python(url)._url

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="130">

---

The <SwmToken path="pydantic/networks.py" pos="131:3:3" line-data="    def scheme(self) -&gt; str:">`scheme`</SwmToken> property returns the scheme part of the URL, such as 'https' in '<https://user:pass@host>:<SwmToken path="pydantic/networks.py" pos="134:21:23" line-data="        e.g. `https` in `https://user:pass@host:port/path?query#fragment`">`port/path`</SwmToken>?query#fragment'.

```python
    @property
    def scheme(self) -> str:
        """The scheme part of the URL.

        e.g. `https` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.scheme

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="138">

---

The <SwmToken path="pydantic/networks.py" pos="139:3:3" line-data="    def username(self) -&gt; str | None:">`username`</SwmToken> property returns the username part of the URL, or `None` if not present.

```python
    @property
    def username(self) -> str | None:
        """The username part of the URL, or `None`.

        e.g. `user` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.username

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="146">

---

The <SwmToken path="pydantic/networks.py" pos="147:3:3" line-data="    def password(self) -&gt; str | None:">`password`</SwmToken> property returns the password part of the URL, or `None` if not present.

```python
    @property
    def password(self) -> str | None:
        """The password part of the URL, or `None`.

        e.g. `pass` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.password

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="154">

---

The <SwmToken path="pydantic/networks.py" pos="155:3:3" line-data="    def host(self) -&gt; str | None:">`host`</SwmToken> property returns the host part of the URL, or `None` if not present. If the host must be punycode encoded, this property returns the encoded version.

```python
    @property
    def host(self) -> str | None:
        """The host part of the URL, or `None`.

        If the URL must be punycode encoded, this is the encoded host, e.g if the input URL is `https://£££.com`,
        `host` will be `xn--9aaa.com`
        """
        return self._url.host
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="163">

---

The <SwmToken path="pydantic/networks.py" pos="163:3:3" line-data="    def unicode_host(self) -&gt; str | None:">`unicode_host`</SwmToken> function returns the host part of the URL as a unicode string, decoding punycode if necessary.

```python
    def unicode_host(self) -> str | None:
        """The host part of the URL as a unicode string, or `None`.

        e.g. `host` in `https://user:pass@host:port/path?query#fragment`

        If the URL must be punycode encoded, this is the decoded host, e.g if the input URL is `https://£££.com`,
        `unicode_host()` will be `£££.com`
        """
        return self._url.unicode_host()

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="173">

---

The <SwmToken path="pydantic/networks.py" pos="174:3:3" line-data="    def port(self) -&gt; int | None:">`port`</SwmToken> property returns the port part of the URL, or `None` if not present.

```python
    @property
    def port(self) -> int | None:
        """The port part of the URL, or `None`.

        e.g. `port` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.port

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="181">

---

The <SwmToken path="pydantic/networks.py" pos="182:3:3" line-data="    def path(self) -&gt; str | None:">`path`</SwmToken> property returns the path part of the URL, or `None` if not present.

```python
    @property
    def path(self) -> str | None:
        """The path part of the URL, or `None`.

        e.g. `/path` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.path

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="189">

---

The <SwmToken path="pydantic/networks.py" pos="190:3:3" line-data="    def query(self) -&gt; str | None:">`query`</SwmToken> property returns the query part of the URL, or `None` if not present.

```python
    @property
    def query(self) -> str | None:
        """The query part of the URL, or `None`.

        e.g. `query` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.query

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="197">

---

The <SwmToken path="pydantic/networks.py" pos="197:3:3" line-data="    def query_params(self) -&gt; list[tuple[str, str]]:">`query_params`</SwmToken> function returns the query part of the URL as a list of <SwmToken path="pydantic/networks.py" pos="198:24:26" line-data="        &quot;&quot;&quot;The query part of the URL as a list of key-value pairs.">`key-value`</SwmToken> pairs.

```python
    def query_params(self) -> list[tuple[str, str]]:
        """The query part of the URL as a list of key-value pairs.

        e.g. `[('foo', 'bar')]` in `https://user:pass@host:port/path?foo=bar#fragment`
        """
        return self._url.query_params()

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="204">

---

The <SwmToken path="pydantic/networks.py" pos="205:3:3" line-data="    def fragment(self) -&gt; str | None:">`fragment`</SwmToken> property returns the fragment part of the URL, or `None` if not present.

```python
    @property
    def fragment(self) -> str | None:
        """The fragment part of the URL, or `None`.

        e.g. `fragment` in `https://user:pass@host:port/path?query#fragment`
        """
        return self._url.fragment

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="212">

---

The <SwmToken path="pydantic/networks.py" pos="212:3:3" line-data="    def unicode_string(self) -&gt; str:">`unicode_string`</SwmToken> function returns the URL as a unicode string, without punycode encoding the host.

```python
    def unicode_string(self) -> str:
        """The URL as a unicode string, unlike `__str__()` this will not punycode encode the host.

        If the URL must be punycode encoded, this is the decoded string, e.g if the input URL is `https://£££.com`,
        `unicode_string()` will be `https://£££.com`
        """
        return self._url.unicode_string()

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="220">

---

The <SwmToken path="pydantic/networks.py" pos="220:3:3" line-data="    def encoded_string(self) -&gt; str:">`encoded_string`</SwmToken> function returns the URL's encoded string representation, punycode encoding the host if required.

```python
    def encoded_string(self) -> str:
        """The URL's encoded string representation via __str__().

        This returns the punycode-encoded host version of the URL as a string.
        """
        return str(self)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="227">

---

The <SwmToken path="pydantic/networks.py" pos="227:3:3" line-data="    def __str__(self) -&gt; str:">`__str__`</SwmToken> method returns the URL as a string, punycode encoding the host if necessary.

```python
    def __str__(self) -> str:
        """The URL as a string, this will punycode encode the host if required."""
        return str(self._url)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="231">

---

The <SwmToken path="pydantic/networks.py" pos="231:3:3" line-data="    def __repr__(self) -&gt; str:">`__repr__`</SwmToken> method returns a string representation of the <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instance, showing the class name and the URL.

```python
    def __repr__(self) -> str:
        return f'{self.__class__.__name__}({str(self._url)!r})'

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="234">

---

The <SwmToken path="pydantic/networks.py" pos="234:3:3" line-data="    def __deepcopy__(self, memo: dict) -&gt; Self:">`__deepcopy__`</SwmToken> method creates a deep copy of the <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instance.

```python
    def __deepcopy__(self, memo: dict) -> Self:
        return self.__class__(self._url)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="237">

---

The <SwmToken path="pydantic/networks.py" pos="237:3:3" line-data="    def __eq__(self, other: Any) -&gt; bool:">`__eq__`</SwmToken> method checks equality between two <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instances by comparing their classes and underlying <SwmToken path="pydantic/networks.py" pos="535:12:12" line-data="    &quot;&quot;&quot;Base type for all URLs.">`URLs`</SwmToken>.

```python
    def __eq__(self, other: Any) -> bool:
        return self.__class__ is other.__class__ and self._url == other._url

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="240">

---

The <SwmToken path="pydantic/networks.py" pos="240:3:3" line-data="    def __lt__(self, other: Any) -&gt; bool:">`__lt__`</SwmToken>, <SwmToken path="pydantic/networks.py" pos="243:3:3" line-data="    def __gt__(self, other: Any) -&gt; bool:">`__gt__`</SwmToken>, <SwmToken path="pydantic/networks.py" pos="246:3:3" line-data="    def __le__(self, other: Any) -&gt; bool:">`__le__`</SwmToken>, and <SwmToken path="pydantic/networks.py" pos="249:3:3" line-data="    def __ge__(self, other: Any) -&gt; bool:">`__ge__`</SwmToken> methods provide comparison operations between <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instances based on their underlying <SwmToken path="pydantic/networks.py" pos="535:12:12" line-data="    &quot;&quot;&quot;Base type for all URLs.">`URLs`</SwmToken>.

```python
    def __lt__(self, other: Any) -> bool:
        return self.__class__ is other.__class__ and self._url < other._url

    def __gt__(self, other: Any) -> bool:
        return self.__class__ is other.__class__ and self._url > other._url

    def __le__(self, other: Any) -> bool:
        return self.__class__ is other.__class__ and self._url <= other._url

    def __ge__(self, other: Any) -> bool:
        return self.__class__ is other.__class__ and self._url >= other._url

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="252">

---

The <SwmToken path="pydantic/networks.py" pos="252:3:3" line-data="    def __hash__(self) -&gt; int:">`__hash__`</SwmToken> method returns a hash of the underlying URL, making <SwmToken path="pydantic/networks.py" pos="127:19:19" line-data="    def __init__(self, url: str | _CoreUrl | _BaseUrl) -&gt; None:">`_BaseUrl`</SwmToken> instances usable as dictionary keys.

```python
    def __hash__(self) -> int:
        return hash(self._url)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="255">

---

The <SwmToken path="pydantic/networks.py" pos="255:3:3" line-data="    def __len__(self) -&gt; int:">`__len__`</SwmToken> method returns the length of the URL string.

```python
    def __len__(self) -> int:
        return len(str(self._url))

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="259">

---

The <SwmToken path="pydantic/networks.py" pos="259:3:3" line-data="    def build(">`build`</SwmToken> class method constructs a new URL instance from its component parts, such as scheme, username, password, host, port, path, query, and fragment.

```python
    def build(
        cls,
        *,
        scheme: str,
        username: str | None = None,
        password: str | None = None,
        host: str,
        port: int | None = None,
        path: str | None = None,
        query: str | None = None,
        fragment: str | None = None,
    ) -> Self:
        """Build a new `Url` instance from its component parts.

        Args:
            scheme: The scheme part of the URL.
            username: The username part of the URL, or omit for no username.
            password: The password part of the URL, or omit for no password.
            host: The host part of the URL.
            port: The port part of the URL, or omit for no port.
            path: The path part of the URL, or omit for no path.
            query: The query part of the URL, or omit for no query.
            fragment: The fragment part of the URL, or omit for no fragment.

        Returns:
            An instance of URL
        """
        return cls(
            _CoreUrl.build(
                scheme=scheme,
                username=username,
                password=password,
                host=host,
                port=port,
                path=path,
                query=query,
                fragment=fragment,
            )
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="299">

---

The <SwmToken path="pydantic/networks.py" pos="300:3:3" line-data="    def serialize_url(cls, url: Any, info: core_schema.SerializationInfo) -&gt; str | Self:">`serialize_url`</SwmToken> class method serializes a URL instance for use in different contexts, such as JSON serialization.

```python
    @classmethod
    def serialize_url(cls, url: Any, info: core_schema.SerializationInfo) -> str | Self:
        if not isinstance(url, cls):
            raise PydanticSerializationUnexpectedValue(
                f"Expected `{cls}` but got `{type(url)}` with value `'{url}'` - serialized value may not be as expected."
            )
        if info.mode == 'json':
            return str(url)
        return url

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="310">

---

The <SwmToken path="pydantic/networks.py" pos="310:3:3" line-data="    def __get_pydantic_core_schema__(">`__get_pydantic_core_schema__`</SwmToken> class method provides the core schema for Pydantic's validation system, enabling custom validation and serialization logic for URL types.

```python
    def __get_pydantic_core_schema__(
        cls, source: type[_BaseUrl], handler: GetCoreSchemaHandler
    ) -> core_schema.CoreSchema:
        def wrap_val(v, h):
            if isinstance(v, source):
                return v
            if isinstance(v, _BaseUrl):
                v = str(v)
            core_url = h(v)
            instance = source.__new__(source)
            instance._url = core_url
            return instance

        return core_schema.no_info_wrap_validator_function(
            wrap_val,
            schema=core_schema.url_schema(**cls._constraints.defined_constraints),
            serialization=core_schema.plain_serializer_function_ser_schema(
                cls.serialize_url, info_arg=True, when_used='always'
            ),
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/networks.py" line="332">

---

The <SwmToken path="pydantic/networks.py" pos="332:3:3" line-data="    def __get_pydantic_json_schema__(">`__get_pydantic_json_schema__`</SwmToken> class method generates the JSON schema for the URL type, integrating with Pydantic's schema generation.

```python
    def __get_pydantic_json_schema__(
        cls, core_schema: core_schema.CoreSchema, handler: _schema_generation_shared.GetJsonSchemaHandler
    ) -> JsonSchemaValue:
        # we use the url schema for json schema generation, but we might have to extract it from
        # the function-wrap schema we use as a tool for validation on initialization
        inner_schema = core_schema['schema'] if core_schema['type'] == 'function-wrap' else core_schema
        return handler(inner_schema)

```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
