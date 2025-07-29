---
title: The EnvSettingsSource class
---
This document covers the <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> class in detail, including:

1. What <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> is and its role in settings management
2. All variables and functions defined in <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken>, with explanations and code citations.

# What is <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken>

<SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> is a class responsible for loading and parsing environment variables (including those from .env files) to provide configuration values for Pydantic settings models. It acts as one of the sources from which Pydantic's <SwmToken path="pydantic/v1/env_settings.py" pos="166:11:11" line-data="    def __call__(self, settings: BaseSettings) -&gt; Dict[str, Any]:  # noqa C901">`BaseSettings`</SwmToken> can retrieve values, supporting both direct environment variables and values loaded from dotenv files. This enables flexible and secure configuration management, especially for applications that follow 12-factor app principles or need to manage secrets and environment-specific settings.

<SwmSnippet path="/pydantic/v1/env_settings.py" line="152">

---

The class uses **slots** to define its instance attributes, which are <SwmToken path="pydantic/v1/env_settings.py" pos="152:7:7" line-data="    __slots__ = (&#39;env_file&#39;, &#39;env_file_encoding&#39;, &#39;env_nested_delimiter&#39;, &#39;env_prefix_len&#39;)">`env_file`</SwmToken>, <SwmToken path="pydantic/v1/env_settings.py" pos="152:12:12" line-data="    __slots__ = (&#39;env_file&#39;, &#39;env_file_encoding&#39;, &#39;env_nested_delimiter&#39;, &#39;env_prefix_len&#39;)">`env_file_encoding`</SwmToken>, <SwmToken path="pydantic/v1/env_settings.py" pos="152:17:17" line-data="    __slots__ = (&#39;env_file&#39;, &#39;env_file_encoding&#39;, &#39;env_nested_delimiter&#39;, &#39;env_prefix_len&#39;)">`env_nested_delimiter`</SwmToken>, and <SwmToken path="pydantic/v1/env_settings.py" pos="152:22:22" line-data="    __slots__ = (&#39;env_file&#39;, &#39;env_file_encoding&#39;, &#39;env_nested_delimiter&#39;, &#39;env_prefix_len&#39;)">`env_prefix_len`</SwmToken>. This restricts the class to only these attributes and can improve memory efficiency.

```python
    __slots__ = ('env_file', 'env_file_encoding', 'env_nested_delimiter', 'env_prefix_len')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="154">

---

The constructor (**init**) initializes the <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> instance with the environment file(s), encoding, nested delimiter, and prefix length. These parameters control how and where environment variables are loaded from, and how nested or prefixed variables are handled.

```python
    def __init__(
        self,
        env_file: Optional[DotenvType],
        env_file_encoding: Optional[str],
        env_nested_delimiter: Optional[str] = None,
        env_prefix_len: int = 0,
    ):
        self.env_file: Optional[DotenvType] = env_file
        self.env_file_encoding: Optional[str] = env_file_encoding
        self.env_nested_delimiter: Optional[str] = env_nested_delimiter
        self.env_prefix_len: int = env_prefix_len

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="166">

---

The **call** method is the core of <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken>. When invoked, it builds a dictionary of environment variables suitable for passing to a Pydantic model. It merges variables from the OS environment and any specified .env files, handles case sensitivity, resolves field names, and supports complex/nested fields and JSON parsing.

```python
    def __call__(self, settings: BaseSettings) -> Dict[str, Any]:  # noqa C901
        """
        Build environment variables suitable for passing to the Model.
        """
        d: Dict[str, Any] = {}

        if settings.__config__.case_sensitive:
            env_vars: Mapping[str, Optional[str]] = os.environ
        else:
            env_vars = {k.lower(): v for k, v in os.environ.items()}

        dotenv_vars = self._read_env_files(settings.__config__.case_sensitive)
        if dotenv_vars:
            env_vars = {**dotenv_vars, **env_vars}

        for field in settings.__fields__.values():
            env_val: Optional[str] = None
            for env_name in field.field_info.extra['env_names']:
                env_val = env_vars.get(env_name)
                if env_val is not None:
                    break

            is_complex, allow_parse_failure = self.field_is_complex(field)
            if is_complex:
                if env_val is None:
                    # field is complex but no value found so far, try explode_env_vars
                    env_val_built = self.explode_env_vars(field, env_vars)
                    if env_val_built:
                        d[field.alias] = env_val_built
                else:
                    # field is complex and there's a value, decode that as JSON, then add explode_env_vars
                    try:
                        env_val = settings.__config__.parse_env_var(field.name, env_val)
                    except ValueError as e:
                        if not allow_parse_failure:
                            raise SettingsError(f'error parsing env var "{env_name}"') from e

                    if isinstance(env_val, dict):
                        d[field.alias] = deep_update(env_val, self.explode_env_vars(field, env_vars))
                    else:
                        d[field.alias] = env_val
            elif env_val is not None:
                # simplest case, field is not complex, we only need to add the value if it was found
                d[field.alias] = env_val

        return d

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="213">

---

The <SwmToken path="pydantic/v1/env_settings.py" pos="213:3:3" line-data="    def _read_env_files(self, case_sensitive: bool) -&gt; Dict[str, Optional[str]]:">`_read_env_files`</SwmToken> function loads environment variables from one or more .env files, if specified. It expands user paths, checks for file existence, and uses the <SwmToken path="pydantic/v1/env_settings.py" pos="226:1:1" line-data="                    read_env_file(env_path, encoding=self.env_file_encoding, case_sensitive=case_sensitive)">`read_env_file`</SwmToken> utility to parse the files, returning a dictionary of variables.

```python
    def _read_env_files(self, case_sensitive: bool) -> Dict[str, Optional[str]]:
        env_files = self.env_file
        if env_files is None:
            return {}

        if isinstance(env_files, (str, os.PathLike)):
            env_files = [env_files]

        dotenv_vars = {}
        for env_file in env_files:
            env_path = Path(env_file).expanduser()
            if env_path.is_file():
                dotenv_vars.update(
                    read_env_file(env_path, encoding=self.env_file_encoding, case_sensitive=case_sensitive)
                )

        return dotenv_vars

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="231">

---

The <SwmToken path="pydantic/v1/env_settings.py" pos="231:3:3" line-data="    def field_is_complex(self, field: ModelField) -&gt; Tuple[bool, bool]:">`field_is_complex`</SwmToken> function determines if a given model field is complex (such as a nested model or a type requiring JSON parsing), and whether parsing errors should be ignored. This helps <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> decide how to process and validate environment values for that field.

```python
    def field_is_complex(self, field: ModelField) -> Tuple[bool, bool]:
        """
        Find out if a field is complex, and if so whether JSON errors should be ignored
        """
        if lenient_issubclass(field.annotation, JsonWrapper):
            return False, False

        if field.is_complex():
            allow_parse_failure = False
        elif is_union(get_origin(field.type_)) and field.sub_fields and any(f.is_complex() for f in field.sub_fields):
            allow_parse_failure = True
        else:
            return False, False

        return True, allow_parse_failure

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="247">

---

The <SwmToken path="pydantic/v1/env_settings.py" pos="247:3:3" line-data="    def explode_env_vars(self, field: ModelField, env_vars: Mapping[str, Optional[str]]) -&gt; Dict[str, Any]:">`explode_env_vars`</SwmToken> function processes environment variables that use a nested delimiter (such as '\_\_') to represent nested structures. It extracts and organizes these variables into nested dictionaries, supporting complex configuration schemas via environment variables.

```python
    def explode_env_vars(self, field: ModelField, env_vars: Mapping[str, Optional[str]]) -> Dict[str, Any]:
        """
        Process env_vars and extract the values of keys containing env_nested_delimiter into nested dictionaries.

        This is applied to a single field, hence filtering by env_var prefix.
        """
        prefixes = [f'{env_name}{self.env_nested_delimiter}' for env_name in field.field_info.extra['env_names']]
        result: Dict[str, Any] = {}
        for env_name, env_val in env_vars.items():
            if not any(env_name.startswith(prefix) for prefix in prefixes):
                continue
            # we remove the prefix before splitting in case the prefix has characters in common with the delimiter
            env_name_without_prefix = env_name[self.env_prefix_len :]
            _, *keys, last_key = env_name_without_prefix.split(self.env_nested_delimiter)
            env_var = result
            for key in keys:
                env_var = env_var.setdefault(key, {})
            env_var[last_key] = env_val

        return result

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/env_settings.py" line="268">

---

The **repr** function provides a string representation of the <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> instance, showing its configuration (<SwmToken path="pydantic/v1/env_settings.py" pos="270:5:5" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`env_file`</SwmToken>, <SwmToken path="pydantic/v1/env_settings.py" pos="270:16:16" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`env_file_encoding`</SwmToken>, <SwmToken path="pydantic/v1/env_settings.py" pos="271:3:3" line-data="            f&#39;env_nested_delimiter={self.env_nested_delimiter!r})&#39;">`env_nested_delimiter`</SwmToken>). This is useful for debugging and logging.

```python
    def __repr__(self) -> str:
        return (
            f'EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, '
            f'env_nested_delimiter={self.env_nested_delimiter!r})'
        )
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken>

<SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> is used to load environment variables as a source of settings. It is instantiated with parameters such as the path to an environment file and the encoding of that file. This allows the settings system to read configuration values from environment files, supporting flexible configuration management.

## Example Usage

An example usage of <SwmToken path="pydantic/v1/env_settings.py" pos="270:3:3" line-data="            f&#39;EnvSettingsSource(env_file={self.env_file!r}, env_file_encoding={self.env_file_encoding!r}, &#39;">`EnvSettingsSource`</SwmToken> involves creating an instance with an environment file path and encoding, which can be specified either directly or via configuration defaults. This instance is then used as part of a collection of settings sources to gather configuration data from environment variables and files.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
