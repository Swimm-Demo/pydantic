---
title: Generating Core Schemas for Callable Arguments
---
# introduction

This document explains how the core schema for validating callable arguments is generated in pydantic. The main questions answered here are:

1. Why do we need a dedicated function to generate argument schemas for callables?
2. How does the function handle different schema versions and parameter filtering?
3. How is the configuration and namespace resolution integrated into schema generation?

# purpose and interface of the schema generator

The function <SwmToken path="pydantic/experimental/arguments_schema.py" pos="14:2:2" line-data="def generate_arguments_schema(">`generate_arguments_schema`</SwmToken> exists to produce a core schema that validates the arguments of any callable. This is useful because it allows pydantic to introspect function signatures and create validation rules automatically, leveraging Python type hints.

The function signature exposes key parameters:

- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="15:1:1" line-data="    func: Callable[..., Any],">`func`</SwmToken>: the callable to inspect.
- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="16:1:1" line-data="    schema_type: Literal[&#39;arguments&#39;, &#39;arguments-v3&#39;] = &#39;arguments-v3&#39;,">`schema_type`</SwmToken>: controls which schema version to generate, supporting backward compatibility or new features.
- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="17:1:1" line-data="    parameters_callback: Callable[[int, str, Any], Literal[&#39;skip&#39;] | None] | None = None,">`parameters_callback`</SwmToken>: a hook to customize or skip parameters during schema creation.
- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="18:1:1" line-data="    config: ConfigDict | None = None,">`config`</SwmToken>: allows passing configuration options affecting schema generation.

<SwmSnippet path="/pydantic/experimental/arguments_schema.py" line="10">

---

This design gives flexibility to adapt schema generation to different use cases and evolving schema formats.

```python
from pydantic import ConfigDict
from pydantic._internal import _config, _generate_schema, _namespace_utils


def generate_arguments_schema(
    func: Callable[..., Any],
    schema_type: Literal['arguments', 'arguments-v3'] = 'arguments-v3',
    parameters_callback: Callable[[int, str, Any], Literal['skip'] | None] | None = None,
    config: ConfigDict | None = None,
) -> CoreSchema:
    """Generate the schema for the arguments of a function.

    Args:
        func: The function to generate the schema for.
        schema_type: The type of schema to generate.
        parameters_callback: A callable that will be invoked for each parameter. The callback
            should take three required arguments: the index, the name and the type annotation
            (or [`Parameter.empty`][inspect.Parameter.empty] if not annotated) of the parameter.
            The callback can optionally return `'skip'`, so that the parameter gets excluded
            from the resulting schema.
        config: The configuration to use.
```

---

</SwmSnippet>

# configuration and namespace resolution

Before generating the schema, the function wraps the provided config in an internal <SwmToken path="pydantic/experimental/arguments_schema.py" pos="36:3:3" line-data="        _config.ConfigWrapper(config),">`ConfigWrapper`</SwmToken>. This standardizes config access and ensures defaults are handled consistently.

It also sets up a namespace resolver using <SwmToken path="pydantic/experimental/arguments_schema.py" pos="37:3:5" line-data="        ns_resolver=_namespace_utils.NsResolver(namespaces_tuple=_namespace_utils.ns_for_function(func)),">`_namespace_utils.NsResolver`</SwmToken>. This resolver is initialized with namespaces relevant to the target function, enabling correct resolution of types and references during schema generation.

<SwmSnippet path="/pydantic/experimental/arguments_schema.py" line="32">

---

This step is crucial because schema generation depends on accurate type resolution and configuration context to produce valid and usable schemas.

```python
    Returns:
        The generated schema.
    """
    generate_schema = _generate_schema.GenerateSchema(
        _config.ConfigWrapper(config),
        ns_resolver=_namespace_utils.NsResolver(namespaces_tuple=_namespace_utils.ns_for_function(func)),
    )
```

---

</SwmSnippet>

# schema generation and version handling

The function supports two schema types: <SwmToken path="pydantic/experimental/arguments_schema.py" pos="16:7:7" line-data="    schema_type: Literal[&#39;arguments&#39;, &#39;arguments-v3&#39;] = &#39;arguments-v3&#39;,">`arguments`</SwmToken> and <SwmToken path="pydantic/experimental/arguments_schema.py" pos="16:12:14" line-data="    schema_type: Literal[&#39;arguments&#39;, &#39;arguments-v3&#39;] = &#39;arguments-v3&#39;,">`arguments-v3`</SwmToken>. Depending on the <SwmToken path="pydantic/experimental/arguments_schema.py" pos="16:1:1" line-data="    schema_type: Literal[&#39;arguments&#39;, &#39;arguments-v3&#39;] = &#39;arguments-v3&#39;,">`schema_type`</SwmToken> argument, it calls the corresponding internal method on the <SwmToken path="pydantic/experimental/arguments_schema.py" pos="35:7:7" line-data="    generate_schema = _generate_schema.GenerateSchema(">`GenerateSchema`</SwmToken> instance:

- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="41:7:7" line-data="        schema = generate_schema._arguments_schema(func, parameters_callback)  # pyright: ignore[reportArgumentType]">`_arguments_schema`</SwmToken> for the older schema format.
- <SwmToken path="pydantic/experimental/arguments_schema.py" pos="43:7:7" line-data="        schema = generate_schema._arguments_v3_schema(func, parameters_callback)  # pyright: ignore[reportArgumentType]">`_arguments_v3_schema`</SwmToken> for the newer format.

This conditional dispatch allows the codebase to maintain compatibility with older schema consumers while enabling improvements in the newer version.

After generating the raw schema, the function calls <SwmToken path="pydantic/experimental/arguments_schema.py" pos="44:5:5" line-data="    return generate_schema.clean_schema(schema)">`clean_schema`</SwmToken> to finalize it. This likely involves pruning, normalizing, or optimizing the schema structure before returning it.

<SwmSnippet path="/pydantic/experimental/arguments_schema.py" line="40">

---

This approach cleanly separates schema generation logic by version and ensures the output is consistent and ready for use.

```python
    if schema_type == 'arguments':
        schema = generate_schema._arguments_schema(func, parameters_callback)  # pyright: ignore[reportArgumentType]
    else:
        schema = generate_schema._arguments_v3_schema(func, parameters_callback)  # pyright: ignore[reportArgumentType]
    return generate_schema.clean_schema(schema)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
