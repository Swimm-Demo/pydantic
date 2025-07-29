---
title: Known Annotated Metadata Utilities and Constraints
---
# Introduction

This document explains the main ideas behind the implementation of known annotated metadata utilities and constraints in <SwmPath>[pydantic/\_internal/\_known_annotated_metadata.py](pydantic/_internal/_known_annotated_metadata.py)</SwmPath>.

The code is designed to:

1. Define which constraints are valid for which schema types, and why this mapping is explicit.
2. Provide utilities to expand, collect, and apply metadata from type annotations, and why these steps are separated.
3. Handle the application of constraints to schemas, including edge cases and error handling.
4. Validate that constraints are only applied to compatible schema types.

# Constraint-to-schema mapping

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="18">

---

The code explicitly defines which constraints are allowed for each schema type. This avoids accidental or unsupported constraint application and makes it easy to see and update the allowed pairings.

```python
STRICT = {'strict'}
FAIL_FAST = {'fail_fast'}
LENGTH_CONSTRAINTS = {'min_length', 'max_length'}
INEQUALITY = {'le', 'ge', 'lt', 'gt'}
NUMERIC_CONSTRAINTS = {'multiple_of', *INEQUALITY}
ALLOW_INF_NAN = {'allow_inf_nan'}

STR_CONSTRAINTS = {
    *LENGTH_CONSTRAINTS,
    *STRICT,
    'strip_whitespace',
    'to_lower',
    'to_upper',
    'pattern',
    'coerce_numbers_to_str',
}
BYTES_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="36">

---

The mapping is extended for all relevant types, including collections, numbers, and special types like enums and URLs.

```python
LIST_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT, *FAIL_FAST}
TUPLE_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT, *FAIL_FAST}
SET_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT, *FAIL_FAST}
DICT_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT}
GENERATOR_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *STRICT}
SEQUENCE_CONSTRAINTS = {*LENGTH_CONSTRAINTS, *FAIL_FAST}

FLOAT_CONSTRAINTS = {*NUMERIC_CONSTRAINTS, *ALLOW_INF_NAN, *STRICT}
DECIMAL_CONSTRAINTS = {'max_digits', 'decimal_places', *FLOAT_CONSTRAINTS}
INT_CONSTRAINTS = {*NUMERIC_CONSTRAINTS, *ALLOW_INF_NAN, *STRICT}
BOOL_CONSTRAINTS = STRICT
UUID_CONSTRAINTS = STRICT
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="66">

---

The actual mapping from constraint names to allowed schema types is built up in a single dictionary, which is then used for validation and application logic.

```python
TEXT_SCHEMA_TYPES = ('str', 'bytes', 'url', 'multi-host-url')
SEQUENCE_SCHEMA_TYPES = ('list', 'tuple', 'set', 'frozenset', 'generator', *TEXT_SCHEMA_TYPES)
NUMERIC_SCHEMA_TYPES = ('float', 'int', 'date', 'time', 'timedelta', 'datetime')

CONSTRAINTS_TO_ALLOWED_SCHEMAS: dict[str, set[str]] = defaultdict(set)

constraint_schema_pairings: list[tuple[set[str], tuple[str, ...]]] = [
    (STR_CONSTRAINTS, TEXT_SCHEMA_TYPES),
    (BYTES_CONSTRAINTS, ('bytes',)),
    (LIST_CONSTRAINTS, ('list',)),
    (TUPLE_CONSTRAINTS, ('tuple',)),
    (SET_CONSTRAINTS, ('set', 'frozenset')),
    (DICT_CONSTRAINTS, ('dict',)),
    (GENERATOR_CONSTRAINTS, ('generator',)),
    (FLOAT_CONSTRAINTS, ('float',)),
    (INT_CONSTRAINTS, ('int',)),
    (DATE_TIME_CONSTRAINTS, ('date', 'time', 'datetime', 'timedelta')),
    # TODO: this is a bit redundant, we could probably avoid some of these
    (STRICT, (*TEXT_SCHEMA_TYPES, *SEQUENCE_SCHEMA_TYPES, *NUMERIC_SCHEMA_TYPES, 'typed-dict', 'model')),
    (UNION_CONSTRAINTS, ('union',)),
    (URL_CONSTRAINTS, ('url', 'multi-host-url')),
    (BOOL_CONSTRAINTS, ('bool',)),
    (UUID_CONSTRAINTS, ('uuid',)),
    (LAX_OR_STRICT_CONSTRAINTS, ('lax-or-strict',)),
    (ENUM_CONSTRAINTS, ('enum',)),
    (DECIMAL_CONSTRAINTS, ('decimal',)),
    (COMPLEX_CONSTRAINTS, ('complex',)),
]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="95">

---

The mapping is populated at import time, so constraint checks are fast and consistent.

```python
for constraints, schemas in constraint_schema_pairings:
    for c in constraints:
        CONSTRAINTS_TO_ALLOWED_SCHEMAS[c].update(schemas)


def as_jsonable_value(v: Any) -> Any:
    if type(v) not in (int, str, float, bytes, bool, type(None)):
        return to_jsonable_python(v)
    return v
```

---

</SwmSnippet>

# Expanding and collecting metadata

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="106">

---

Annotations can be grouped or nested, so the code provides a utility to flatten and normalize them. This ensures that all metadata is considered, regardless of how it was provided.

```python
def expand_grouped_metadata(annotations: Iterable[Any]) -> Iterable[Any]:
    """Expand the annotations.

    Args:
        annotations: An iterable of annotations.

    Returns:
        An iterable of expanded annotations.
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="129">

---

The function walks through the annotations, expanding grouped metadata and handling special cases like <SwmToken path="pydantic/_internal/_known_annotated_metadata.py" pos="132:8:8" line-data="        elif isinstance(annotation, FieldInfo):">`FieldInfo`</SwmToken>. This is important for supporting both standard and custom metadata in a uniform way.

```python
    for annotation in annotations:
        if isinstance(annotation, at.GroupedMetadata):
            yield from annotation
        elif isinstance(annotation, FieldInfo):
            yield from annotation.metadata
            # this is a bit problematic in that it results in duplicate metadata
            # all of our "consumers" can handle it, but it is not ideal
            # we probably should split up FieldInfo into:
            # - annotated types metadata
            # - individual metadata known only to Pydantic
            annotation = copy(annotation)
            annotation.metadata = []
            yield annotation
        else:
            yield annotation
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="332">

---

To split out known constraints from unknown or custom metadata, another utility is provided. This makes it easy to process only the constraints that Pydantic understands, while passing through or ignoring the rest.

```python
def collect_known_metadata(annotations: Iterable[Any]) -> tuple[dict[str, Any], list[Any]]:
    """Split `annotations` into known metadata and unknown annotations.

    Args:
        annotations: An iterable of annotations.

    Returns:
        A tuple contains a dict of known metadata and a list of unknown annotations.
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="356">

---

The function checks each annotation, extracting known constraints and collecting the rest. This separation is important for extensibility and for clear error reporting.

```python
    for annotation in annotations:
        # isinstance(annotation, PydanticMetadata) also covers ._fields:_PydanticGeneralMetadata
        if isinstance(annotation, PydanticMetadata):
            res.update(annotation.__dict__)
        # we don't use dataclasses.asdict because that recursively calls asdict on the field values
        elif (annotation_type := type(annotation)) in (at_to_constraint_map := _get_at_to_constraint_map()):
            constraint = at_to_constraint_map[annotation_type]
            res[constraint] = getattr(annotation, constraint)
        elif isinstance(annotation, type) and issubclass(annotation, PydanticMetadata):
            # also support PydanticMetadata classes being used without initialisation,
            # e.g. `Annotated[int, Strict]` as well as `Annotated[int, Strict()]`
            res.update({k: v for k, v in vars(annotation).items() if not k.startswith('_')})
        else:
            remaining.append(annotation)
    # Nones can sneak in but pydantic-core will reject them
    # it'd be nice to clean things up so we don't put in None (we probably don't _need_ to, it was just easier)
    # but this is simple enough to kick that can down the road
    res = {k: v for k, v in res.items() if v is not None}
    return res, remaining
```

---

</SwmSnippet>

# Applying known metadata to schemas

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="168">

---

The core logic for applying constraints to schemas is handled in a single function. This function is responsible for updating the schema in-place, wrapping it with validators if necessary, and handling edge cases.

```python
def apply_known_metadata(annotation: Any, schema: CoreSchema) -> CoreSchema | None:  # noqa: C901
    """Apply `annotation` to `schema` if it is an annotation we know about (Gt, Le, etc.).
    Otherwise return `None`.

    This does not handle all known annotations. If / when it does, it can always
    return a CoreSchema and return the unmodified schema if the annotation should be ignored.

    Assumes that GroupedMetadata has already been expanded via `expand_grouped_metadata`.
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="204">

---

The function first collects all known metadata and determines the schema type. It then checks if each constraint is allowed for the schema type, raising an error if not.

```python
    for constraint, value in schema_update.items():
        if constraint not in CONSTRAINTS_TO_ALLOWED_SCHEMAS:
            raise ValueError(f'Unknown constraint {constraint}')
        allowed_schemas = CONSTRAINTS_TO_ALLOWED_SCHEMAS[constraint]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="246">

---

Some constraints require wrapping the schema in a function validator, especially when the constraint can't be directly represented in the schema. This is handled with a chain of validators.

```python
            chain_schema_steps.append(
                cs.no_info_wrap_validator_function(
                    _apply_constraint_with_incompatibility_info, cs.str_schema(**{constraint: value})
                )
            )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="266">

---

For numeric and length constraints, the function updates both the runtime validator and the JSON schema metadata, ensuring that both validation and schema generation are correct.

```python
            schema = cs.no_info_after_validator_function(
                partial(NUMERIC_VALIDATOR_LOOKUP[constraint], **{constraint: value}), schema
            )
            metadata = schema.get('metadata', {})
            if (existing_json_schema_updates := metadata.get('pydantic_js_updates')) is not None:
                metadata['pydantic_js_updates'] = {
                    **existing_json_schema_updates,
                    **{js_constraint_key: as_jsonable_value(value)},
                }
            else:
                metadata['pydantic_js_updates'] = {js_constraint_key: as_jsonable_value(value)}
            schema['metadata'] = metadata
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="278">

---

If a constraint is not recognized or can't be applied, the function raises a runtime error. This prevents silent failures and makes debugging easier.

```python
        elif constraint == 'allow_inf_nan' and value is False:
            schema = cs.no_info_after_validator_function(
                forbid_inf_nan_check,
                schema,
            )
        else:
            # It's rare that we'd get here, but it's possible if we add a new constraint and forget to handle it
            # Most constraint errors are caught at runtime during attempted application
            raise RuntimeError(f"Unable to apply constraint '{constraint}' to schema of type '{schema_type}'")
```

---

</SwmSnippet>

# Predicate and custom constraint handling

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="301">

---

The code also supports predicate-based constraints, which are functions that must return True for the value to be valid. These are wrapped in validators that raise custom errors if the predicate fails.

```python
            def val_func(v: Any) -> Any:
                predicate_satisfied = annotation.func(v)  # noqa: B023

                # annotation.func may also raise an exception, let it pass through
                if isinstance(annotation, at.Predicate):  # noqa: B023
                    if not predicate_satisfied:
                        raise PydanticCustomError(
                            'predicate_failed',
                            f'Predicate {predicate_name} failed',  # type: ignore  # noqa: B023
                        )
                else:
                    if predicate_satisfied:
                        raise PydanticCustomError(
                            'not_operation_failed',
                            f'Not of {predicate_name} failed',  # type: ignore  # noqa: B023
                        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="318">

---

The validator is attached to the schema, so it runs automatically during validation.

```python
                return v

            schema = cs.no_info_after_validator_function(val_func, schema)
```

---

</SwmSnippet>

# Metadata validation utility

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="377">

---

Finally, a utility is provided to check that only allowed constraints are applied to a given schema type. This is used internally to provide consistent error messages and prevent misconfiguration.

```python
def check_metadata(metadata: dict[str, Any], allowed: Iterable[str], source_type: Any) -> None:
    """A small utility function to validate that the given metadata can be applied to the target.
    More than saving lines of code, this gives us a consistent error message for all of our internal implementations.

    Args:
        metadata: A dict of metadata.
        allowed: An iterable of allowed metadata.
        source_type: The source type.
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_known_annotated_metadata.py" line="386">

---

If any unknown constraints are found, a clear error is raised.

```python
    Raises:
        TypeError: If there is metadatas that can't be applied on source type.
    """
    unknown = metadata.keys() - set(allowed)
    if unknown:
        raise TypeError(
            f'The following constraints cannot be applied to {source_type!r}: {", ".join([f"{k!r}" for k in unknown])}'
        )
```

---

</SwmSnippet>

This approach ensures that constraint application is explicit, predictable, and easy to debug.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
