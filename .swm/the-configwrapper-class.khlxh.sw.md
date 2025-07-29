---
title: The ConfigWrapper class
---
This document covers the <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> class in detail:

1. What <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is and its purpose
2. Variables and functions defined in <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken>

<SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is an internal utility class that acts as a wrapper for configuration in Pydantic. It exposes configuration options (from <SwmToken path="pydantic/_internal/_config.py" pos="39:4:4" line-data="    config_dict: ConfigDict">`ConfigDict`</SwmToken>) as attributes, making it easier to access and manage model configuration settings throughout the codebase. <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is primarily used to standardize and simplify how configuration is handled for models and dataclasses, ensuring consistent access to configuration values and supporting both legacy and new configuration styles.

<SwmSnippet path="/pydantic/_internal/_config.py" line="37">

---

The variable <SwmToken path="pydantic/_internal/_config.py" pos="37:1:1" line-data="    __slots__ = (&#39;config_dict&#39;,)">`__slots__`</SwmToken> is set to (<SwmToken path="pydantic/_internal/_config.py" pos="37:7:7" line-data="    __slots__ = (&#39;config_dict&#39;,)">`config_dict`</SwmToken>,), which restricts the instance to only have the <SwmToken path="pydantic/_internal/_config.py" pos="37:7:7" line-data="    __slots__ = (&#39;config_dict&#39;,)">`config_dict`</SwmToken> attribute, optimizing memory usage and preventing dynamic attribute creation.

```python
    __slots__ = ('config_dict',)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="39">

---

The variable <SwmToken path="pydantic/_internal/_config.py" pos="39:1:1" line-data="    config_dict: ConfigDict">`config_dict`</SwmToken> holds the actual configuration dictionary (<SwmToken path="pydantic/_internal/_config.py" pos="39:4:4" line-data="    config_dict: ConfigDict">`ConfigDict`</SwmToken>) for the model or dataclass. This is the central storage for all configuration options accessed by the wrapper.

```python
    config_dict: ConfigDict
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="43">

---

The class defines a large set of annotated attributes (such as <SwmToken path="pydantic/_internal/_config.py" pos="43:1:1" line-data="    title: str | None">`title`</SwmToken>, <SwmToken path="pydantic/_internal/_config.py" pos="44:1:1" line-data="    str_to_lower: bool">`str_to_lower`</SwmToken>, <SwmToken path="pydantic/_internal/_config.py" pos="49:1:1" line-data="    extra: ExtraValues | None">`extra`</SwmToken>, <SwmToken path="pydantic/_internal/_config.py" pos="50:1:1" line-data="    frozen: bool">`frozen`</SwmToken>, etc.) that mirror the keys in <SwmToken path="pydantic/_internal/_config.py" pos="39:4:4" line-data="    config_dict: ConfigDict">`ConfigDict`</SwmToken>. These annotations ensure that all configuration options are available as attributes on the wrapper, and they are kept in sync with <SwmToken path="pydantic/_internal/_config.py" pos="39:4:4" line-data="    config_dict: ConfigDict">`ConfigDict`</SwmToken>.

```python
    title: str | None
    str_to_lower: bool
    str_to_upper: bool
    str_strip_whitespace: bool
    str_min_length: int
    str_max_length: int | None
    extra: ExtraValues | None
    frozen: bool
    populate_by_name: bool
    use_enum_values: bool
    validate_assignment: bool
    arbitrary_types_allowed: bool
    from_attributes: bool
    # whether to use the actual key provided in the data (e.g. alias or first alias for "field required" errors) instead of field_names
    # to construct error `loc`s, default `True`
    loc_by_alias: bool
    alias_generator: Callable[[str], str] | AliasGenerator | None
    model_title_generator: Callable[[type], str] | None
    field_title_generator: Callable[[str, FieldInfo | ComputedFieldInfo], str] | None
    ignored_types: tuple[type, ...]
    allow_inf_nan: bool
    json_schema_extra: JsonDict | JsonSchemaExtraCallable | None
    json_encoders: dict[type[object], JsonEncoder] | None

    # new in V2
    strict: bool
    # whether instances of models and dataclasses (including subclass instances) should re-validate, default 'never'
    revalidate_instances: Literal['always', 'never', 'subclass-instances']
    ser_json_timedelta: Literal['iso8601', 'float']
    ser_json_bytes: Literal['utf8', 'base64', 'hex']
    val_json_bytes: Literal['utf8', 'base64', 'hex']
    ser_json_inf_nan: Literal['null', 'constants', 'strings']
    # whether to validate default values during validation, default False
    validate_default: bool
    validate_return: bool
    protected_namespaces: tuple[str | Pattern[str], ...]
    hide_input_in_errors: bool
    defer_build: bool
    plugin_settings: dict[str, object] | None
    schema_generator: type[GenerateSchema] | None
    json_schema_serialization_defaults_required: bool
    json_schema_mode_override: Literal['validation', 'serialization', None]
    coerce_numbers_to_str: bool
    regex_engine: Literal['rust-regex', 'python-re']
    validation_error_cause: bool
    use_attribute_docstrings: bool
    cache_strings: bool | Literal['all', 'keys', 'none']
    validate_by_alias: bool
    validate_by_name: bool
    serialize_by_alias: bool

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="94">

---

The <SwmToken path="pydantic/_internal/_config.py" pos="94:3:3" line-data="    def __init__(self, config: ConfigDict | dict[str, Any] | type[Any] | None, *, check: bool = True):">`__init__`</SwmToken> function initializes the <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> instance. It takes a configuration input (which can be a <SwmToken path="pydantic/_internal/_config.py" pos="94:11:11" line-data="    def __init__(self, config: ConfigDict | dict[str, Any] | type[Any] | None, *, check: bool = True):">`ConfigDict`</SwmToken>, a dictionary, a type, or None) and processes it using the <SwmToken path="pydantic/_internal/_config.py" pos="96:7:7" line-data="            self.config_dict = prepare_config(config)">`prepare_config`</SwmToken> function if <SwmToken path="pydantic/_internal/_config.py" pos="94:37:37" line-data="    def __init__(self, config: ConfigDict | dict[str, Any] | type[Any] | None, *, check: bool = True):">`check`</SwmToken> is True. Otherwise, it casts the input directly to <SwmToken path="pydantic/_internal/_config.py" pos="94:11:11" line-data="    def __init__(self, config: ConfigDict | dict[str, Any] | type[Any] | None, *, check: bool = True):">`ConfigDict`</SwmToken>.

```python
    def __init__(self, config: ConfigDict | dict[str, Any] | type[Any] | None, *, check: bool = True):
        if check:
            self.config_dict = prepare_config(config)
        else:
            self.config_dict = cast(ConfigDict, config)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="101">

---

The class method <SwmToken path="pydantic/_internal/_config.py" pos="101:3:3" line-data="    def for_model(">`for_model`</SwmToken> constructs a <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> for a <SwmToken path="pydantic/_internal/_config.py" pos="108:21:21" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`BaseModel`</SwmToken> by merging configuration options from base classes, the class namespace, and any keyword arguments. It ensures that configuration is built in the correct order of precedence and validates against conflicting or invalid configuration sources.

```python
    def for_model(
        cls,
        bases: tuple[type[Any], ...],
        namespace: dict[str, Any],
        raw_annotations: dict[str, Any],
        kwargs: dict[str, Any],
    ) -> Self:
        """Build a new `ConfigWrapper` instance for a `BaseModel`.

        The config wrapper built based on (in descending order of priority):
        - options from `kwargs`
        - options from the `namespace`
        - options from the base classes (`bases`)

        Args:
            bases: A tuple of base classes.
            namespace: The namespace of the class being created.
            raw_annotations: The (non-evaluated) annotations of the model.
            kwargs: The kwargs passed to the class being created.

        Returns:
            A `ConfigWrapper` instance for `BaseModel`.
        """
        config_new = ConfigDict()
        for base in bases:
            config = getattr(base, 'model_config', None)
            if config:
                config_new.update(config.copy())

        config_class_from_namespace = namespace.get('Config')
        config_dict_from_namespace = namespace.get('model_config')

        if raw_annotations.get('model_config') and config_dict_from_namespace is None:
            raise PydanticUserError(
                '`model_config` cannot be used as a model field name. Use `model_config` for model configuration.',
                code='model-config-invalid-field-name',
            )

        if config_class_from_namespace and config_dict_from_namespace:
            raise PydanticUserError('"Config" and "model_config" cannot be used together', code='config-both')

        config_from_namespace = config_dict_from_namespace or prepare_config(config_class_from_namespace)

        config_new.update(config_from_namespace)

        for k in list(kwargs.keys()):
            if k in config_keys:
                config_new[k] = kwargs.pop(k)

        return cls(config_new)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="155">

---

The <SwmToken path="pydantic/_internal/_config.py" pos="155:3:3" line-data="        def __getattr__(self, name: str) -&gt; Any:">`__getattr__`</SwmToken> method provides dynamic attribute access to configuration options. If an attribute is not found in the instance's <SwmToken path="pydantic/_internal/_config.py" pos="157:5:5" line-data="                return self.config_dict[name]">`config_dict`</SwmToken>, it falls back to global <SwmToken path="pydantic/_internal/_config.py" pos="160:3:3" line-data="                    return config_defaults[name]">`config_defaults`</SwmToken>. If the attribute is missing from both, it raises an <SwmToken path="pydantic/_internal/_config.py" pos="162:3:3" line-data="                    raise AttributeError(f&#39;Config has no attribute {name!r}&#39;) from None">`AttributeError`</SwmToken>.

```python
        def __getattr__(self, name: str) -> Any:
            try:
                return self.config_dict[name]
            except KeyError:
                try:
                    return config_defaults[name]
                except KeyError:
                    raise AttributeError(f'Config has no attribute {name!r}') from None

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="164">

---

The <SwmToken path="pydantic/_internal/_config.py" pos="164:3:3" line-data="    def core_config(self, title: str | None) -&gt; core_schema.CoreConfig:">`core_config`</SwmToken> function generates a <SwmToken path="pydantic/_internal/_config.py" pos="164:22:22" line-data="    def core_config(self, title: str | None) -&gt; core_schema.CoreConfig:">`CoreConfig`</SwmToken> object for use by <SwmToken path="pydantic/_internal/_config.py" pos="165:8:10" line-data="        &quot;&quot;&quot;Create a pydantic-core config.">`pydantic-core`</SwmToken>, based on the current configuration. It handles deprecation warnings, backward compatibility patches, and ensures that required validation flags are set appropriately before constructing the <SwmToken path="pydantic/_internal/_config.py" pos="164:22:22" line-data="    def core_config(self, title: str | None) -&gt; core_schema.CoreConfig:">`CoreConfig`</SwmToken>.

```python
    def core_config(self, title: str | None) -> core_schema.CoreConfig:
        """Create a pydantic-core config.

        We don't use getattr here since we don't want to populate with defaults.

        Args:
            title: The title to use if not set in config.

        Returns:
            A `CoreConfig` object created from config.
        """
        config = self.config_dict

        if config.get('schema_generator') is not None:
            warnings.warn(
                'The `schema_generator` setting has been deprecated since v2.10. This setting no longer has any effect.',
                PydanticDeprecatedSince210,
                stacklevel=2,
            )

        if (populate_by_name := config.get('populate_by_name')) is not None:
            # We include this patch for backwards compatibility purposes, but this config setting will be deprecated in v3.0, and likely removed in v4.0.
            # Thus, the above warning and this patch can be removed then as well.
            if config.get('validate_by_name') is None:
                config['validate_by_alias'] = True
                config['validate_by_name'] = populate_by_name

        # We dynamically patch validate_by_name to be True if validate_by_alias is set to False
        # and validate_by_name is not explicitly set.
        if config.get('validate_by_alias') is False and config.get('validate_by_name') is None:
            config['validate_by_name'] = True

        if (not config.get('validate_by_alias', True)) and (not config.get('validate_by_name', False)):
            raise PydanticUserError(
                'At least one of `validate_by_alias` or `validate_by_name` must be set to True.',
                code='validate-by-alias-and-name-false',
            )

        return core_schema.CoreConfig(
            **{  # pyright: ignore[reportArgumentType]
                k: v
                for k, v in (
                    ('title', config.get('title') or title or None),
                    ('extra_fields_behavior', config.get('extra')),
                    ('allow_inf_nan', config.get('allow_inf_nan')),
                    ('str_strip_whitespace', config.get('str_strip_whitespace')),
                    ('str_to_lower', config.get('str_to_lower')),
                    ('str_to_upper', config.get('str_to_upper')),
                    ('strict', config.get('strict')),
                    ('ser_json_timedelta', config.get('ser_json_timedelta')),
                    ('ser_json_bytes', config.get('ser_json_bytes')),
                    ('val_json_bytes', config.get('val_json_bytes')),
                    ('ser_json_inf_nan', config.get('ser_json_inf_nan')),
                    ('from_attributes', config.get('from_attributes')),
                    ('loc_by_alias', config.get('loc_by_alias')),
                    ('revalidate_instances', config.get('revalidate_instances')),
                    ('validate_default', config.get('validate_default')),
                    ('str_max_length', config.get('str_max_length')),
                    ('str_min_length', config.get('str_min_length')),
                    ('hide_input_in_errors', config.get('hide_input_in_errors')),
                    ('coerce_numbers_to_str', config.get('coerce_numbers_to_str')),
                    ('regex_engine', config.get('regex_engine')),
                    ('validation_error_cause', config.get('validation_error_cause')),
                    ('cache_strings', config.get('cache_strings')),
                    ('validate_by_alias', config.get('validate_by_alias')),
                    ('validate_by_name', config.get('validate_by_name')),
                    ('serialize_by_alias', config.get('serialize_by_alias')),
                )
                if v is not None
            }
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_config.py" line="236">

---

The <SwmToken path="pydantic/_internal/_config.py" pos="236:3:3" line-data="    def __repr__(self):">`__repr__`</SwmToken> function provides a string representation of the <SwmToken path="pydantic/_internal/_config.py" pos="238:5:5" line-data="        return f&#39;ConfigWrapper({c})&#39;">`ConfigWrapper`</SwmToken> instance, listing all key-value pairs in the <SwmToken path="pydantic/_internal/_config.py" pos="237:36:36" line-data="        c = &#39;, &#39;.join(f&#39;{k}={v!r}&#39; for k, v in self.config_dict.items())">`config_dict`</SwmToken> for easier debugging and inspection.

```python
    def __repr__(self):
        c = ', '.join(f'{k}={v!r}' for k, v in self.config_dict.items())
        return f'ConfigWrapper({c})'
```

---

</SwmSnippet>

# Usage

## Usage in Config Management

<SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is used to encapsulate configuration dictionaries, as seen in its **repr** method which formats the stored configuration for display. This suggests it acts as a container for configuration settings.

## Usage in <SwmToken path="pydantic/_internal/_config.py" pos="241:2:2" line-data="class ConfigWrapperStack:">`ConfigWrapperStack`</SwmToken>

<SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> instances are managed in a stack structure by the <SwmToken path="pydantic/_internal/_config.py" pos="241:2:2" line-data="class ConfigWrapperStack:">`ConfigWrapperStack`</SwmToken> class, which initializes with a <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> instance and maintains a list of these wrappers. This implies <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is used to handle layered or hierarchical configurations.

## Usage in Dataclass Field Setting

In the \_dataclasses module, <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is passed as an argument to the set_dataclass_fields function, which collects and sets the **pydantic_fields** attribute on dataclass types. This indicates <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> provides configuration context necessary for defining dataclass fields.

## Usage in Dataclass Completion

Similarly, <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> is used in the complete_dataclass function to finalize dataclass setup, with options to raise errors and resolve namespaces. This shows <SwmToken path="pydantic/_internal/_config.py" pos="108:11:11" line-data="        &quot;&quot;&quot;Build a new `ConfigWrapper` instance for a `BaseModel`.">`ConfigWrapper`</SwmToken> plays a role in the finalization and validation of dataclass configurations.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
