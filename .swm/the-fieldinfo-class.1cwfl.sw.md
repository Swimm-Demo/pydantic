---
title: The FieldInfo class
---
This document covers the <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> class in detail, including:

1. What <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken>, with code citations for each.

# What is <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken>

<SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> is a class in <SwmPath>[pydantic/fields.py](pydantic/fields.py)</SwmPath> that encapsulates all metadata and configuration for a field in a Pydantic model. It is used internally for every field definition, regardless of whether the Field() function is explicitly used. <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> stores information such as type annotations, default values, aliases, validation constraints, and serialization options. Typically, developers do not instantiate <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> directly; instead, it is created via the Field() function or through model field definitions. Access to <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> is usually through the .model_fields attribute of a <SwmToken path="pydantic/fields.py" pos="284:7:7" line-data="            class MyModel(pydantic.BaseModel):">`BaseModel`</SwmToken> instance.

<SwmSnippet path="/pydantic/fields.py" line="138">

---

The variable <SwmToken path="pydantic/fields.py" pos="138:1:1" line-data="    annotation: type[Any] | None">`annotation`</SwmToken> holds the type annotation of the field, which is used for type validation and schema generation.

```python
    annotation: type[Any] | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="139">

---

The variable <SwmToken path="pydantic/fields.py" pos="139:1:1" line-data="    default: Any">`default`</SwmToken> stores the default value for the field, if provided.

```python
    default: Any
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="140">

---

The variable <SwmToken path="pydantic/fields.py" pos="140:1:1" line-data="    default_factory: Callable[[], Any] | Callable[[dict[str, Any]], Any] | None">`default_factory`</SwmToken> is a callable that generates a default value for the field, supporting both zero-argument and single-argument (validated data) callables.

```python
    default_factory: Callable[[], Any] | Callable[[dict[str, Any]], Any] | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="141">

---

The variable <SwmToken path="pydantic/fields.py" pos="141:1:1" line-data="    alias: str | None">`alias`</SwmToken> allows the field to be referenced by an alternative name during serialization and deserialization.

```python
    alias: str | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="142">

---

The variable <SwmToken path="pydantic/fields.py" pos="142:1:1" line-data="    alias_priority: int | None">`alias_priority`</SwmToken> determines the priority of the alias when multiple aliases are present.

```python
    alias_priority: int | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="143">

---

The variable <SwmToken path="pydantic/fields.py" pos="143:1:1" line-data="    validation_alias: str | AliasPath | AliasChoices | None">`validation_alias`</SwmToken> specifies an alternative name to use during validation.

```python
    validation_alias: str | AliasPath | AliasChoices | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="144">

---

The variable <SwmToken path="pydantic/fields.py" pos="144:1:1" line-data="    serialization_alias: str | None">`serialization_alias`</SwmToken> specifies an alternative name to use during serialization.

```python
    serialization_alias: str | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="145">

---

The variable <SwmToken path="pydantic/fields.py" pos="145:1:1" line-data="    title: str | None">`title`</SwmToken> provides a human-readable title for the field, often used in generated documentation.

```python
    title: str | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="146">

---

The variable <SwmToken path="pydantic/fields.py" pos="146:1:1" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`field_title_generator`</SwmToken> is a callable that generates a title for the field based on its name.

```python
    field_title_generator: Callable[[str, FieldInfo], str] | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="147">

---

The variable <SwmToken path="pydantic/fields.py" pos="147:1:1" line-data="    description: str | None">`description`</SwmToken> provides a description of the field, useful for documentation and schema generation.

```python
    description: str | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="148">

---

The variable <SwmToken path="pydantic/fields.py" pos="148:1:1" line-data="    examples: list[Any] | None">`examples`</SwmToken> holds a list of example values for the field.

```python
    examples: list[Any] | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="149">

---

The variable <SwmToken path="pydantic/fields.py" pos="149:1:1" line-data="    exclude: bool | None">`exclude`</SwmToken> determines whether the field should be excluded from model serialization.

```python
    exclude: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="150">

---

The variable <SwmToken path="pydantic/fields.py" pos="150:1:1" line-data="    discriminator: str | types.Discriminator | None">`discriminator`</SwmToken> is used for tagged unions, specifying the field name or discriminator object.

```python
    discriminator: str | types.Discriminator | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="151">

---

The variable <SwmToken path="pydantic/fields.py" pos="151:1:1" line-data="    deprecated: Deprecated | str | bool | None">`deprecated`</SwmToken> can be a message, a boolean, or a deprecated instance, indicating if the field is deprecated.

```python
    deprecated: Deprecated | str | bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="152">

---

The variable <SwmToken path="pydantic/fields.py" pos="152:1:1" line-data="    json_schema_extra: JsonDict | Callable[[JsonDict], None] | None">`json_schema_extra`</SwmToken> allows attaching extra properties to the generated JSON schema, either as a dict or a callable.

```python
    json_schema_extra: JsonDict | Callable[[JsonDict], None] | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="153">

---

The variable <SwmToken path="pydantic/fields.py" pos="153:1:1" line-data="    frozen: bool | None">`frozen`</SwmToken> indicates whether the field is immutable after model creation.

```python
    frozen: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="154">

---

The variable <SwmToken path="pydantic/fields.py" pos="154:1:1" line-data="    validate_default: bool | None">`validate_default`</SwmToken> determines whether the default value should be validated.

```python
    validate_default: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="155">

---

The variable <SwmToken path="pydantic/fields.py" pos="155:1:1" line-data="    repr: bool">`repr`</SwmToken> controls whether the field appears in the model's string representation.

```python
    repr: bool
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="156">

---

The variable <SwmToken path="pydantic/fields.py" pos="156:1:1" line-data="    init: bool | None">`init`</SwmToken> specifies if the field should be included in the dataclass constructor.

```python
    init: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="157">

---

The variable <SwmToken path="pydantic/fields.py" pos="157:1:1" line-data="    init_var: bool | None">`init_var`</SwmToken> determines if the field is only included in the constructor and not stored.

```python
    init_var: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="158">

---

The variable <SwmToken path="pydantic/fields.py" pos="158:1:1" line-data="    kw_only: bool | None">`kw_only`</SwmToken> indicates if the field should be a <SwmToken path="pydantic/fields.py" pos="134:16:18" line-data="        kw_only: Whether the field should be a keyword-only argument in the constructor of the dataclass.">`keyword-only`</SwmToken> argument in the dataclass constructor.

```python
    kw_only: bool | None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="159">

---

The variable <SwmToken path="pydantic/fields.py" pos="159:1:1" line-data="    metadata: list[Any]">`metadata`</SwmToken> holds a list of metadata constraints and annotations for the field.

```python
    metadata: list[Any]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="211">

---

The function <SwmToken path="pydantic/fields.py" pos="211:3:3" line-data="    def __init__(self, **kwargs: Unpack[_FieldInfoInputs]) -&gt; None:">`__init__`</SwmToken> initializes a <SwmToken path="pydantic/fields.py" pos="259:9:9" line-data="        # Used to rebuild FieldInfo instances:">`FieldInfo`</SwmToken> instance with all relevant field metadata, handling default values, factories, aliases, and metadata extraction.

```python
    def __init__(self, **kwargs: Unpack[_FieldInfoInputs]) -> None:
        """This class should generally not be initialized directly; instead, use the `pydantic.fields.Field` function
        or one of the constructor classmethods.

        See the signature of `pydantic.fields.Field` for more details about the expected arguments.
        """
        self._attributes_set = {k: v for k, v in kwargs.items() if v is not _Unset and k not in self.metadata_lookup}
        kwargs = {k: _DefaultValues.get(k) if v is _Unset else v for k, v in kwargs.items()}  # type: ignore
        self.annotation = kwargs.get('annotation')

        default = kwargs.pop('default', PydanticUndefined)
        if default is Ellipsis:
            self.default = PydanticUndefined
            self._attributes_set.pop('default', None)
        else:
            self.default = default

        self.default_factory = kwargs.pop('default_factory', None)

        if self.default is not PydanticUndefined and self.default_factory is not None:
            raise TypeError('cannot specify both default and default_factory')

        self.alias = kwargs.pop('alias', None)
        self.validation_alias = kwargs.pop('validation_alias', None)
        self.serialization_alias = kwargs.pop('serialization_alias', None)
        alias_is_set = any(alias is not None for alias in (self.alias, self.validation_alias, self.serialization_alias))
        self.alias_priority = kwargs.pop('alias_priority', None) or 2 if alias_is_set else None
        self.title = kwargs.pop('title', None)
        self.field_title_generator = kwargs.pop('field_title_generator', None)
        self.description = kwargs.pop('description', None)
        self.examples = kwargs.pop('examples', None)
        self.exclude = kwargs.pop('exclude', None)
        self.discriminator = kwargs.pop('discriminator', None)
        # For compatibility with FastAPI<=0.110.0, we preserve the existing value if it is not overridden
        self.deprecated = kwargs.pop('deprecated', getattr(self, 'deprecated', None))
        self.repr = kwargs.pop('repr', True)
        self.json_schema_extra = kwargs.pop('json_schema_extra', None)
        self.validate_default = kwargs.pop('validate_default', None)
        self.frozen = kwargs.pop('frozen', None)
        # currently only used on dataclasses
        self.init = kwargs.pop('init', None)
        self.init_var = kwargs.pop('init_var', None)
        self.kw_only = kwargs.pop('kw_only', None)

        self.metadata = self._collect_metadata(kwargs)  # type: ignore

        # Private attributes:
        self._qualifiers: set[Qualifier] = set()
        # Used to rebuild FieldInfo instances:
        self._complete = True
        self._original_annotation: Any = PydanticUndefined
        self._original_assignment: Any = PydanticUndefined

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="265">

---

The static method <SwmToken path="pydantic/fields.py" pos="265:3:3" line-data="    def from_field(default: Any = PydanticUndefined, **kwargs: Unpack[_FromFieldInfoInputs]) -&gt; FieldInfo:">`from_field`</SwmToken> creates a new <SwmToken path="pydantic/fields.py" pos="265:27:27" line-data="    def from_field(default: Any = PydanticUndefined, **kwargs: Unpack[_FromFieldInfoInputs]) -&gt; FieldInfo:">`FieldInfo`</SwmToken> object using the Field function, enforcing that 'annotation' is not passed as a keyword argument.

````python
    def from_field(default: Any = PydanticUndefined, **kwargs: Unpack[_FromFieldInfoInputs]) -> FieldInfo:
        """Create a new `FieldInfo` object with the `Field` function.

        Args:
            default: The default value for the field. Defaults to Undefined.
            **kwargs: Additional arguments dictionary.

        Raises:
            TypeError: If 'annotation' is passed as a keyword argument.

        Returns:
            A new FieldInfo object with the given parameters.

        Example:
            This is how you can create a field with default value like this:

            ```python
            import pydantic

            class MyModel(pydantic.BaseModel):
                foo: int = pydantic.Field(4)
            ```
        """
        if 'annotation' in kwargs:
            raise TypeError('"annotation" is not permitted as a Field keyword argument')
        return FieldInfo(default=default, **kwargs)

````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="293">

---

The static method <SwmToken path="pydantic/fields.py" pos="293:3:3" line-data="    def from_annotation(annotation: type[Any], *, _source: AnnotationSource = AnnotationSource.ANY) -&gt; FieldInfo:">`from_annotation`</SwmToken> constructs a <SwmToken path="pydantic/fields.py" pos="293:30:30" line-data="    def from_annotation(annotation: type[Any], *, _source: AnnotationSource = AnnotationSource.ANY) -&gt; FieldInfo:">`FieldInfo`</SwmToken> instance from a bare type annotation, handling Annotated types and extracting metadata.

````python
    def from_annotation(annotation: type[Any], *, _source: AnnotationSource = AnnotationSource.ANY) -> FieldInfo:
        """Creates a `FieldInfo` instance from a bare annotation.

        This function is used internally to create a `FieldInfo` from a bare annotation like this:

        ```python
        import pydantic

        class MyModel(pydantic.BaseModel):
            foo: int  # <-- like this
        ```

        We also account for the case where the annotation can be an instance of `Annotated` and where
        one of the (not first) arguments in `Annotated` is an instance of `FieldInfo`, e.g.:

        ```python
        from typing import Annotated

        import annotated_types

        import pydantic

        class MyModel(pydantic.BaseModel):
            foo: Annotated[int, annotated_types.Gt(42)]
            bar: Annotated[int, pydantic.Field(gt=42)]
        ```

        Args:
            annotation: An annotation object.

        Returns:
            An instance of the field metadata.
        """
        try:
            inspected_ann = inspect_annotation(
                annotation,
                annotation_source=_source,
                unpack_type_aliases='skip',
            )
        except ForbiddenQualifier as e:
            raise PydanticForbiddenQualifier(e.qualifier, annotation)

        # TODO check for classvar and error?

        # No assigned value, this happens when using a bare `Final` qualifier (also for other
        # qualifiers, but they shouldn't appear here). In this case we infer the type as `Any`
        # because we don't have any assigned value.
        type_expr: Any = Any if inspected_ann.type is UNKNOWN else inspected_ann.type
        final = 'final' in inspected_ann.qualifiers
        metadata = inspected_ann.metadata

        attr_overrides = {'annotation': type_expr}
        if final:
            attr_overrides['frozen'] = True
        field_info = FieldInfo._construct(metadata, **attr_overrides)
        field_info._qualifiers = inspected_ann.qualifiers
        return field_info

````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="352">

---

The static method <SwmToken path="pydantic/fields.py" pos="352:3:3" line-data="    def from_annotated_attribute(">`from_annotated_attribute`</SwmToken> creates a <SwmToken path="pydantic/fields.py" pos="354:5:5" line-data="    ) -&gt; FieldInfo:">`FieldInfo`</SwmToken> from an annotation and a default value, supporting Annotated types and merging metadata.

````python
    def from_annotated_attribute(
        annotation: type[Any], default: Any, *, _source: AnnotationSource = AnnotationSource.ANY
    ) -> FieldInfo:
        """Create `FieldInfo` from an annotation with a default value.

        This is used in cases like the following:

        ```python
        from typing import Annotated

        import annotated_types

        import pydantic

        class MyModel(pydantic.BaseModel):
            foo: int = 4  # <-- like this
            bar: Annotated[int, annotated_types.Gt(4)] = 4  # <-- or this
            spam: Annotated[int, pydantic.Field(gt=4)] = 4  # <-- or this
        ```

        Args:
            annotation: The type annotation of the field.
            default: The default value of the field.

        Returns:
            A field object with the passed values.
        """
        if annotation is not MISSING and annotation is default:
            raise PydanticUserError(
                'Error when building FieldInfo from annotated attribute. '
                "Make sure you don't have any field name clashing with a type annotation.",
                code='unevaluable-type-annotation',
            )

        try:
            inspected_ann = inspect_annotation(
                annotation,
                annotation_source=_source,
                unpack_type_aliases='skip',
            )
        except ForbiddenQualifier as e:
            raise PydanticForbiddenQualifier(e.qualifier, annotation)

        # TODO check for classvar and error?

        # TODO infer from the default, this can be done in v3 once we treat final fields with
        # a default as proper fields and not class variables:
        type_expr: Any = Any if inspected_ann.type is UNKNOWN else inspected_ann.type
        final = 'final' in inspected_ann.qualifiers
        metadata = inspected_ann.metadata

        # HACK 1: the order in which the metadata is merged is inconsistent; we need to prepend
        # metadata from the assignment at the beginning of the metadata. Changing this is only
        # possible in v3 (at least). See https://github.com/pydantic/pydantic/issues/10507
        prepend_metadata: list[Any] | None = None
        attr_overrides = {'annotation': type_expr}
        if final:
            attr_overrides['frozen'] = True

        # HACK 2: FastAPI is subclassing `FieldInfo` and historically expected the actual
        # instance's type to be preserved when constructing new models with its subclasses as assignments.
        # This code is never reached by Pydantic itself, and in an ideal world this shouldn't be necessary.
        if not metadata and isinstance(default, FieldInfo) and type(default) is not FieldInfo:
            field_info = default._copy()
            field_info._attributes_set.update(attr_overrides)
            for k, v in attr_overrides.items():
                setattr(field_info, k, v)
            return field_info

        if isinstance(default, FieldInfo):
            default_copy = default._copy()  # Copy unnecessary when we remove HACK 1.
            prepend_metadata = default_copy.metadata
            default_copy.metadata = []
            metadata = metadata + [default_copy]
        elif isinstance(default, dataclasses.Field):
            from_field = FieldInfo._from_dataclass_field(default)
            prepend_metadata = from_field.metadata  # Unnecessary when we remove HACK 1.
            from_field.metadata = []
            metadata = metadata + [from_field]
            if 'init_var' in inspected_ann.qualifiers:
                attr_overrides['init_var'] = True
            if (init := getattr(default, 'init', None)) is not None:
                attr_overrides['init'] = init
            if (kw_only := getattr(default, 'kw_only', None)) is not None:
                attr_overrides['kw_only'] = kw_only
        else:
            # `default` is the actual default value
            attr_overrides['default'] = default

        field_info = FieldInfo._construct(
            prepend_metadata + metadata if prepend_metadata is not None else metadata, **attr_overrides
        )
        field_info._qualifiers = inspected_ann.qualifiers
        return field_info

````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/fields.py" line="448">

---

The class method <SwmToken path="pydantic/fields.py" pos="448:3:3" line-data="    def _construct(cls, metadata: list[Any], **attr_overrides: Any) -&gt; Self:">`_construct`</SwmToken> merges metadata and attribute overrides to produce a final <SwmToken path="pydantic/fields.py" pos="449:11:11" line-data="        &quot;&quot;&quot;Construct the final `FieldInfo` instance, by merging the possibly existing `FieldInfo` instances from the metadata.">`FieldInfo`</SwmToken> instance, supporting complex Annotated field definitions.

````python
    def _construct(cls, metadata: list[Any], **attr_overrides: Any) -> Self:
        """Construct the final `FieldInfo` instance, by merging the possibly existing `FieldInfo` instances from the metadata.

        With the following example:

        ```python {test="skip" lint="skip"}
        class Model(BaseModel):
            f: Annotated[int, Gt(1), Field(description='desc', lt=2)]
        ```

        `metadata` refers to the metadata elements of the `Annotated` form. This metadata is iterated over from left to right:

        - If the element is a `Field()` function (which is itself a `FieldInfo` instance), the field attributes (such as
          `description`) are saved to be set on the final `FieldInfo` instance.
          On the other hand, some kwargs (such as `lt`) are stored as `metadata` (see `FieldInfo.__init__()`, calling
          `FieldInfo._collect_metadata()`). In this case, the final metadata list is extended with the one from this instance.
        - Else, the element is considered as a single metadata object, and is appended to the final metadata list.

        Args:
            metadata: The list of metadata elements to merge together. If the `FieldInfo` instance to be constructed is for
                a field with an assigned `Field()`, this `Field()` assignment should be added as the last element of the
                provided metadata.
            **attr_overrides: Extra attributes that should be set on the final merged `FieldInfo` instance.

        Returns:
            The final merged `FieldInfo` instance.
        """
        merged_metadata: list[Any] = []
        merged_kwargs: dict[str, Any] = {}

        for meta in metadata:
            if isinstance(meta, FieldInfo):
                merged_metadata.extend(meta.metadata)

                new_js_extra: JsonDict | None = None
                current_js_extra = meta.json_schema_extra
                if current_js_extra is not None and 'json_schema_extra' in merged_kwargs:
                    # We need to merge `json_schema_extra`'s:
                    existing_js_extra = merged_kwargs['json_schema_extra']
                    if isinstance(existing_js_extra, dict):
                        if isinstance(current_js_extra, dict):
                            new_js_extra = {
                                **existing_js_extra,
                                **current_js_extra,
                            }
                        elif callable(current_js_extra):
                            warn(
                                'Composing `dict` and `callable` type `json_schema_extra` is not supported. '
                                'The `callable` type is being ignored. '
                                "If you'd like support for this behavior, please open an issue on pydantic.",
                                UserWarning,
                            )
                    elif callable(existing_js_extra) and isinstance(current_js_extra, dict):
                        warn(
                            'Composing `dict` and `callable` type `json_schema_extra` is not supported. '
                            'The `callable` type is being ignored. '
                            "If you'd like support for this behavior, please open an issue on pydantic.",
                            UserWarning,
                        )

                merged_kwargs.update(meta._attributes_set)
                if new_js_extra is not None:
                    merged_kwargs['json_schema_extra'] = new_js_extra
            elif typing_objects.is_deprecated(meta):
                merged_kwargs['deprecated'] = meta
            else:
                merged_metadata.append(meta)

        merged_kwargs.update(attr_overrides)
        merged_field_info = cls(**merged_kwargs)
        merged_field_info.metadata = merged_metadata
        return merged_field_info

````

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> Usage Examples

<SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> is used to store metadata about fields such as validation aliases, serialization aliases, titles, descriptions, examples, and exclusion flags. These attributes help define how fields are validated, serialized, and documented.

For instance, the attribute <SwmToken path="pydantic/fields.py" pos="143:1:1" line-data="    validation_alias: str | AliasPath | AliasChoices | None">`validation_alias`</SwmToken> can hold a string or an alias path to specify alternative names for validation purposes, while <SwmToken path="pydantic/fields.py" pos="144:1:1" line-data="    serialization_alias: str | None">`serialization_alias`</SwmToken> is used to define alternative names during serialization.

The title and description attributes provide human-readable information about the field, which can be used in generated documentation or user interfaces.

The examples attribute holds a list of example values for the field, aiding in illustrating expected input or output data.

The exclude attribute is a boolean flag that indicates whether the field should be excluded from certain operations, such as serialization or validation.

Additionally, <SwmToken path="pydantic/fields.py" pos="146:1:1" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`field_title_generator`</SwmToken> is a callable that can generate a title dynamically based on the field name and <SwmToken path="pydantic/fields.py" pos="146:10:10" line-data="    field_title_generator: Callable[[str, FieldInfo], str] | None">`FieldInfo`</SwmToken> instance, allowing for customizable field titles.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
