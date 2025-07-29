---
title: The Discriminator class
---
This document covers the Discriminator class in Pydantic, focusing on:

1. What Discriminator is and its purpose
2. Variables and functions defined in Discriminator

# What is Discriminator

Discriminator is a class in Pydantic that enables advanced data validation for union types by allowing the use of either a field name or a custom callable to determine which member of a union a value belongs to. This approach provides more flexibility than traditional field-based discrimination, as it does not require a shared field across all union choices. Discriminator is especially useful for handling unions of models and primitive types, and it supports custom error handling and improved performance for discriminated unions.

<SwmSnippet path="/pydantic/types.py" line="3048">

---

The variable <SwmToken path="pydantic/types.py" pos="3048:1:1" line-data="    discriminator: str | Callable[[Any], Hashable]">`discriminator`</SwmToken> stores either a string (the field name to discriminate on) or a callable (a function that extracts the discriminator value from the input). This is the core mechanism by which the Discriminator class determines how to select the correct union member.

```python
    discriminator: str | Callable[[Any], Hashable]
    """The callable or field name for discriminating the type in a tagged union.

    A `Callable` discriminator must extract the value of the discriminator from the input.
    A `str` discriminator must be the name of a field to discriminate against.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/types.py" line="3054">

---

The variable <SwmToken path="pydantic/types.py" pos="3054:1:1" line-data="    custom_error_type: str | None = None">`custom_error_type`</SwmToken> allows specifying a custom error type to be used in place of the standard discriminated union validation errors. This enables more descriptive or domain-specific error reporting when validation fails.

```python
    custom_error_type: str | None = None
    """Type to use in [custom errors](../errors/errors.md) replacing the standard discriminated union
    validation errors.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/types.py" line="3058">

---

The variable <SwmToken path="pydantic/types.py" pos="3058:1:1" line-data="    custom_error_message: str | None = None">`custom_error_message`</SwmToken> is used to provide a custom error message for validation errors related to the discriminated union. This message will be shown instead of the default error message.

```python
    custom_error_message: str | None = None
    """Message to use in custom errors."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/types.py" line="3060">

---

The variable <SwmToken path="pydantic/types.py" pos="3060:1:1" line-data="    custom_error_context: dict[str, int | str | float] | None = None">`custom_error_context`</SwmToken> holds additional context for custom errors, allowing you to pass extra information (such as values or types) that can be used in error formatting or reporting.

```python
    custom_error_context: dict[str, int | str | float] | None = None
    """Context to use in custom errors."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/types.py" line="3063">

---

The function <SwmToken path="pydantic/types.py" pos="3063:3:3" line-data="    def __get_pydantic_core_schema__(self, source_type: Any, handler: GetCoreSchemaHandler) -&gt; CoreSchema:">`__get_pydantic_core_schema__`</SwmToken> generates the core schema for the discriminated union. It checks if the source type is a union, and then either applies a field-based discriminator or uses the custom callable to build the schema. If a callable is used, it delegates to the <SwmToken path="pydantic/types.py" pos="3073:5:5" line-data="            return self._convert_schema(original_schema, handler)">`_convert_schema`</SwmToken> method to construct a tagged union schema.

```python
    def __get_pydantic_core_schema__(self, source_type: Any, handler: GetCoreSchemaHandler) -> CoreSchema:
        if not is_union_origin(get_origin(source_type)):
            raise TypeError(f'{type(self).__name__} must be used with a Union type, not {source_type}')

        if isinstance(self.discriminator, str):
            from pydantic import Field

            return handler(Annotated[source_type, Field(discriminator=self.discriminator)])
        else:
            original_schema = handler(source_type)
            return self._convert_schema(original_schema, handler)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/types.py" line="3075">

---

The function <SwmToken path="pydantic/types.py" pos="3075:3:3" line-data="    def _convert_schema(">`_convert_schema`</SwmToken> is responsible for transforming a union schema into a tagged union schema using the provided discriminator. It ensures that each union choice is properly tagged (raising an error if a tag is missing), and incorporates any custom error type, message, or context. The resulting schema enables efficient and accurate validation of discriminated unions.

```python
    def _convert_schema(
        self, original_schema: core_schema.CoreSchema, handler: GetCoreSchemaHandler | None = None
    ) -> core_schema.TaggedUnionSchema:
        if original_schema['type'] != 'union':
            # This likely indicates that the schema was a single-item union that was simplified.
            # In this case, we do the same thing we do in
            # `pydantic._internal._discriminated_union._ApplyInferredDiscriminator._apply_to_root`, namely,
            # package the generated schema back into a single-item union.
            original_schema = core_schema.union_schema([original_schema])

        tagged_union_choices = {}
        for choice in original_schema['choices']:
            tag = None
            if isinstance(choice, tuple):
                choice, tag = choice
            metadata = cast('CoreMetadata | None', choice.get('metadata'))
            if metadata is not None:
                tag = metadata.get('pydantic_internal_union_tag_key') or tag
            if tag is None:
                # `handler` is None when this method is called from `apply_discriminator()` (deferred discriminators)
                if handler is not None and choice['type'] == 'definition-ref':
                    # If choice was built from a PEP 695 type alias, try to resolve the def:
                    try:
                        choice = handler.resolve_ref_schema(choice)
                    except LookupError:
                        pass
                    else:
                        metadata = cast('CoreMetadata | None', choice.get('metadata'))
                        if metadata is not None:
                            tag = metadata.get('pydantic_internal_union_tag_key')

                if tag is None:
                    raise PydanticUserError(
                        f'`Tag` not provided for choice {choice} used with `Discriminator`',
                        code='callable-discriminator-no-tag',
                    )
            tagged_union_choices[tag] = choice

        # Have to do these verbose checks to ensure falsy values ('' and {}) don't get ignored
        custom_error_type = self.custom_error_type
        if custom_error_type is None:
            custom_error_type = original_schema.get('custom_error_type')

        custom_error_message = self.custom_error_message
        if custom_error_message is None:
            custom_error_message = original_schema.get('custom_error_message')

        custom_error_context = self.custom_error_context
        if custom_error_context is None:
            custom_error_context = original_schema.get('custom_error_context')

        custom_error_type = original_schema.get('custom_error_type') if custom_error_type is None else custom_error_type
        return core_schema.tagged_union_schema(
            tagged_union_choices,
            self.discriminator,
            custom_error_type=custom_error_type,
            custom_error_message=custom_error_message,
            custom_error_context=custom_error_context,
            strict=original_schema.get('strict'),
            ref=original_schema.get('ref'),
            metadata=original_schema.get('metadata'),
            serialization=original_schema.get('serialization'),
        )
```

---

</SwmSnippet>

# Usage

## Discriminator in Field Definitions

The Discriminator is used as a type for the 'discriminator' attribute in field definitions, allowing fields to specify a discriminator key or object. This is useful for distinguishing between types in tagged unions, enabling more precise validation and serialization behavior.

## Discriminator in Tagged Unions

Discriminator is also used in type annotations for tagged unions, where it helps determine which subtype a given data instance corresponds to. For example, it can be used alongside Annotated types to specify a function or value that extracts the discriminator value, facilitating polymorphic model validation.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
