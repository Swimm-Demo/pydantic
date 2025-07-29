---
title: Deprecated Utility Functions for Type Validation and Schema Generation
---
# Introduction

This document explains why certain utility functions for type validation and JSON schema generation were implemented as deprecated in pydantic. We will cover:

1. Why these utilities were deprecated and what replaced them.
2. How the deprecation warnings are structured to guide users.
3. The role of <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> in the new approach to validation and schema generation.

# rationale for deprecating utility functions

The functions <SwmToken path="pydantic/deprecated/tools.py" pos="27:3:3" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`parse_obj_as`</SwmToken>, <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken>, and <SwmToken path="pydantic/deprecated/tools.py" pos="82:3:3" line-data="    &#39;`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_json_of`</SwmToken> were originally convenience utilities for validating data against types and generating JSON schemas. However, they have been deprecated in favor of a more unified and flexible interface provided by the <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> class. This change consolidates functionality and reduces redundancy.

The deprecation is explicitly communicated through decorators and warnings that point users to <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> methods instead. This ensures users migrate to the new API without breaking existing code abruptly.

# deprecation warnings and user guidance

Each deprecated function is wrapped with the <SwmToken path="pydantic/deprecated/tools.py" pos="26:0:1" line-data="@deprecated(">`@deprecated`</SwmToken> decorator from <SwmToken path="pydantic/deprecated/tools.py" pos="7:2:2" line-data="from typing_extensions import deprecated">`typing_extensions`</SwmToken>, which marks the function as deprecated at the code level. Additionally, inside the function body, a <SwmToken path="pydantic/deprecated/tools.py" pos="31:1:3" line-data="    warnings.warn(">`warnings.warn`</SwmToken> call issues a runtime warning of category <SwmToken path="pydantic/deprecated/tools.py" pos="33:3:3" line-data="        category=PydanticDeprecatedSince20,">`PydanticDeprecatedSince20`</SwmToken>. This category is a custom subclass of <SwmToken path="pydantic/deprecated/tools.py" pos="39:1:1" line-data="            DeprecationWarning,">`DeprecationWarning`</SwmToken> tailored for pydantic's version 2.0 deprecations.

For example, <SwmToken path="pydantic/deprecated/tools.py" pos="27:3:3" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`parse_obj_as`</SwmToken> warns users to use <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:18" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter.validate_python`</SwmToken> instead. Similarly, <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken> and <SwmToken path="pydantic/deprecated/tools.py" pos="82:3:3" line-data="    &#39;`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_json_of`</SwmToken> direct users to <SwmToken path="pydantic/deprecated/tools.py" pos="46:16:18" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`TypeAdapter.json_schema`</SwmToken>. This dual-layer warning system ensures both static analysis tools and runtime environments notify developers about the preferred usage.

# using <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> for validation and schema generation

The new recommended approach centers on the <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> class. Instead of standalone functions, you instantiate a <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken> with the target type and then call its methods:

- <SwmToken path="pydantic/deprecated/tools.py" pos="42:8:11" line-data="    return TypeAdapter(type_).validate_python(obj)">`validate_python(obj)`</SwmToken> replaces <SwmToken path="pydantic/deprecated/tools.py" pos="27:3:3" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`parse_obj_as`</SwmToken> for validating Python objects against a type.
- <SwmToken path="pydantic/deprecated/tools.py" pos="46:18:18" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`json_schema`</SwmToken>`(...)` replaces both <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken> and <SwmToken path="pydantic/deprecated/tools.py" pos="82:3:3" line-data="    &#39;`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_json_of`</SwmToken> for generating JSON schemas, either as a dictionary or serialized JSON string.

This design encapsulates type adaptation logic in one place, improving maintainability and extensibility. It also removes legacy parameters like <SwmToken path="pydantic/deprecated/tools.py" pos="30:19:19" line-data="def parse_obj_as(type_: type[T], obj: Any, type_name: NameFactory | None = None) -&gt; T:">`type_name`</SwmToken> in <SwmToken path="pydantic/deprecated/tools.py" pos="27:3:3" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`parse_obj_as`</SwmToken> and callable <SwmToken path="pydantic/deprecated/tools.py" pos="52:1:1" line-data="    title: NameFactory | None = None,">`title`</SwmToken> in <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken>, which are now deprecated and warned against.

# handling schema title and JSON serialization

In <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken>, the optional <SwmToken path="pydantic/deprecated/tools.py" pos="52:1:1" line-data="    title: NameFactory | None = None,">`title`</SwmToken> parameter can be a string or callable. Passing a callable is deprecated and triggers a warning because the new API expects a direct string title. The function calls <SwmToken path="pydantic/deprecated/tools.py" pos="46:16:18" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`TypeAdapter.json_schema`</SwmToken> with parameters like <SwmToken path="pydantic/deprecated/tools.py" pos="53:1:1" line-data="    by_alias: bool = True,">`by_alias`</SwmToken>, <SwmToken path="pydantic/deprecated/tools.py" pos="54:1:1" line-data="    ref_template: str = DEFAULT_REF_TEMPLATE,">`ref_template`</SwmToken>, and a customizable <SwmToken path="pydantic/deprecated/tools.py" pos="55:1:1" line-data="    schema_generator: type[GenerateJsonSchema] = GenerateJsonSchema,">`schema_generator`</SwmToken> class, allowing flexible schema generation.

<SwmToken path="pydantic/deprecated/tools.py" pos="82:3:3" line-data="    &#39;`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_json_of`</SwmToken> builds on <SwmToken path="pydantic/deprecated/tools.py" pos="46:3:3" line-data="    &#39;`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.&#39;,">`schema_of`</SwmToken> by serializing the resulting schema dictionary to a JSON string using <SwmToken path="pydantic/deprecated/tools.py" pos="100:3:5" line-data="    return json.dumps(">`json.dumps`</SwmToken>. It accepts additional keyword arguments for JSON serialization, forwarding them transparently.

<SwmSnippet path="/pydantic/deprecated/tools.py" line="26">

---

This layered approach keeps schema generation logic DRY and consistent, while guiding users toward the new <SwmToken path="pydantic/deprecated/tools.py" pos="27:16:16" line-data="    &#39;`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.&#39;,">`TypeAdapter`</SwmToken>-based workflow.

```python
@deprecated(
    '`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.',
    category=None,
)
def parse_obj_as(type_: type[T], obj: Any, type_name: NameFactory | None = None) -> T:
    warnings.warn(
        '`parse_obj_as` is deprecated. Use `pydantic.TypeAdapter.validate_python` instead.',
        category=PydanticDeprecatedSince20,
        stacklevel=2,
    )
    if type_name is not None:  # pragma: no cover
        warnings.warn(
            'The type_name parameter is deprecated. parse_obj_as no longer creates temporary models',
            DeprecationWarning,
            stacklevel=2,
        )
    return TypeAdapter(type_).validate_python(obj)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/tools.py" line="45">

---

&nbsp;

```python
@deprecated(
    '`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.',
    category=None,
)
def schema_of(
    type_: Any,
    *,
    title: NameFactory | None = None,
    by_alias: bool = True,
    ref_template: str = DEFAULT_REF_TEMPLATE,
    schema_generator: type[GenerateJsonSchema] = GenerateJsonSchema,
) -> dict[str, Any]:
    """Generate a JSON schema (as dict) for the passed model or dynamically generated one."""
    warnings.warn(
        '`schema_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.',
        category=PydanticDeprecatedSince20,
        stacklevel=2,
    )
    res = TypeAdapter(type_).json_schema(
        by_alias=by_alias,
        schema_generator=schema_generator,
        ref_template=ref_template,
    )
    if title is not None:
        if isinstance(title, str):
            res['title'] = title
        else:
            warnings.warn(
                'Passing a callable for the `title` parameter is deprecated and no longer supported',
                DeprecationWarning,
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/tools.py" line="81">

---

&nbsp;

```python
@deprecated(
    '`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.',
    category=None,
)
def schema_json_of(
    type_: Any,
    *,
    title: NameFactory | None = None,
    by_alias: bool = True,
    ref_template: str = DEFAULT_REF_TEMPLATE,
    schema_generator: type[GenerateJsonSchema] = GenerateJsonSchema,
    **dumps_kwargs: Any,
) -> str:
    """Generate a JSON schema (as JSON) for the passed model or dynamically generated one."""
    warnings.warn(
        '`schema_json_of` is deprecated. Use `pydantic.TypeAdapter.json_schema` instead.',
        category=PydanticDeprecatedSince20,
        stacklevel=2,
    )
    return json.dumps(
        schema_of(type_, title=title, by_alias=by_alias, ref_template=ref_template, schema_generator=schema_generator),
        **dumps_kwargs,
    )
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
