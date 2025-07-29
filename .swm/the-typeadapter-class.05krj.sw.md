---
title: The TypeAdapter class
---
This document covers the <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> class in detail, focusing on:

1. What <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken>, with code citations for each

# What is <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken>

<SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is a class in Pydantic that provides a flexible way to perform validation and serialization based on a Python type. It exposes some of the functionality from <SwmToken path="pydantic/type_adapter.py" pos="205:20:20" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`BaseModel`</SwmToken> instance methods for types that do not have such methods, such as dataclasses, primitive types, and more. <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is not itself a type and cannot be used as a type annotation for fields. It is designed to allow validation, serialization, and JSON schema generation for arbitrary Python types without requiring the creation of a <SwmToken path="pydantic/type_adapter.py" pos="205:20:20" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`BaseModel`</SwmToken>. This makes it especially useful for validating or serializing collections, TypedDicts, or other complex types that are not Pydantic models.

<SwmSnippet path="/pydantic/type_adapter.py" line="167">

---

The variable <SwmToken path="pydantic/type_adapter.py" pos="167:1:1" line-data="    core_schema: CoreSchema">`core_schema`</SwmToken> holds the core schema for the type being adapted. This schema is used internally for validation and serialization.

```python
    core_schema: CoreSchema
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="168">

---

The variable validator is the schema validator for the type. It is responsible for validating data against the adapted type.

```python
    validator: SchemaValidator | PluggableSchemaValidator
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="169">

---

The variable serializer is the schema serializer for the type. It handles serialization of instances of the adapted type.

```python
    serializer: SchemaSerializer
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="170">

---

The variable <SwmToken path="pydantic/type_adapter.py" pos="170:1:1" line-data="    pydantic_complete: bool">`pydantic_complete`</SwmToken> is a boolean indicating whether the core schema for the type has been successfully built.

```python
    pydantic_complete: bool
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="195">

---

The **init** function initializes a <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> instance. It sets up the type, configuration, and internal state, and initializes the core schema, validator, and serializer. It also handles namespace resolution for forward references.

```python
    def __init__(
        self,
        type: Any,
        *,
        config: ConfigDict | None = None,
        _parent_depth: int = 2,
        module: str | None = None,
    ) -> None:
        if _type_has_config(type) and config is not None:
            raise PydanticUserError(
                'Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.'
                ' These types can have their own config and setting the config via the `config`'
                ' parameter to TypeAdapter will not override it, thus the `config` you passed to'
                ' TypeAdapter becomes meaningless, which is probably not what you want.',
                code='type-adapter-config-unused',
            )

        self._type = type
        self._config = config
        self._parent_depth = _parent_depth
        self.pydantic_complete = False

        parent_frame = self._fetch_parent_frame()
        if parent_frame is not None:
            globalns = parent_frame.f_globals
            # Do not provide a local ns if the type adapter happens to be instantiated at the module level:
            localns = parent_frame.f_locals if parent_frame.f_locals is not globalns else {}
        else:
            globalns = {}
            localns = {}

        self._module_name = module or cast(str, globalns.get('__name__', ''))
        self._init_core_attrs(
            ns_resolver=_namespace_utils.NsResolver(
                namespaces_tuple=_namespace_utils.NamespacesTuple(locals=localns, globals=globalns),
                parent_namespace=localns,
            ),
            force=False,
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="235">

---

The <SwmToken path="pydantic/type_adapter.py" pos="235:3:3" line-data="    def _fetch_parent_frame(self) -&gt; FrameType | None:">`_fetch_parent_frame`</SwmToken> function retrieves the parent frame in the call stack, which is used for resolving forward references during schema building.

```python
    def _fetch_parent_frame(self) -> FrameType | None:
        frame = sys._getframe(self._parent_depth)
        if frame.f_globals.get('__name__') == 'typing':
            # Because `TypeAdapter` is generic, explicitly parametrizing the class results
            # in a `typing._GenericAlias` instance, which proxies instantiation calls to the
            # "real" `TypeAdapter` class and thus adding an extra frame to the call. To avoid
            # pulling anything from the `typing` module, use the correct frame (the one before):
            return frame.f_back

        return frame

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="246">

---

The <SwmToken path="pydantic/type_adapter.py" pos="246:3:3" line-data="    def _init_core_attrs(">`_init_core_attrs`</SwmToken> function initializes the core schema, validator, and serializer for the type. It handles both direct attribute access and schema generation if necessary.

```python
    def _init_core_attrs(
        self, ns_resolver: _namespace_utils.NsResolver, force: bool, raise_errors: bool = False
    ) -> bool:
        """Initialize the core schema, validator, and serializer for the type.

        Args:
            ns_resolver: The namespace resolver to use when building the core schema for the adapted type.
            force: Whether to force the construction of the core schema, validator, and serializer.
                If `force` is set to `False` and `_defer_build` is `True`, the core schema, validator, and serializer will be set to mocks.
            raise_errors: Whether to raise errors if initializing any of the core attrs fails.

        Returns:
            `True` if the core schema, validator, and serializer were successfully initialized, otherwise `False`.

        Raises:
            PydanticUndefinedAnnotation: If `PydanticUndefinedAnnotation` occurs in`__get_pydantic_core_schema__`
                and `raise_errors=True`.
        """
        if not force and self._defer_build:
            _mock_val_ser.set_type_adapter_mocks(self)
            self.pydantic_complete = False
            return False

        try:
            self.core_schema = _getattr_no_parents(self._type, '__pydantic_core_schema__')
            self.validator = _getattr_no_parents(self._type, '__pydantic_validator__')
            self.serializer = _getattr_no_parents(self._type, '__pydantic_serializer__')

            # TODO: we don't go through the rebuild logic here directly because we don't want
            # to repeat all of the namespace fetching logic that we've already done
            # so we simply skip to the block below that does the actual schema generation
            if (
                isinstance(self.core_schema, _mock_val_ser.MockCoreSchema)
                or isinstance(self.validator, _mock_val_ser.MockValSer)
                or isinstance(self.serializer, _mock_val_ser.MockValSer)
            ):
                raise AttributeError()
        except AttributeError:
            config_wrapper = _config.ConfigWrapper(self._config)

            schema_generator = _generate_schema.GenerateSchema(config_wrapper, ns_resolver=ns_resolver)

            try:
                core_schema = schema_generator.generate_schema(self._type)
            except PydanticUndefinedAnnotation:
                if raise_errors:
                    raise
                _mock_val_ser.set_type_adapter_mocks(self)
                return False

            try:
                self.core_schema = schema_generator.clean_schema(core_schema)
            except _generate_schema.InvalidSchemaError:
                _mock_val_ser.set_type_adapter_mocks(self)
                return False

            core_config = config_wrapper.core_config(None)

            self.validator = create_schema_validator(
                schema=self.core_schema,
                schema_type=self._type,
                schema_type_module=self._module_name,
                schema_type_name=str(self._type),
                schema_kind='TypeAdapter',
                config=core_config,
                plugin_settings=config_wrapper.plugin_settings,
            )
            self.serializer = SchemaSerializer(self.core_schema, core_config)

        self.pydantic_complete = True
        return True
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="319">

---

The <SwmToken path="pydantic/type_adapter.py" pos="319:3:3" line-data="    def _defer_build(self) -&gt; bool:">`_defer_build`</SwmToken> property checks if schema building should be deferred based on the configuration.

```python
    def _defer_build(self) -> bool:
        config = self._config if self._config is not None else self._model_config
        if config:
            return config.get('defer_build') is True
        return False
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="325">

---

The <SwmToken path="pydantic/type_adapter.py" pos="326:3:3" line-data="    def _model_config(self) -&gt; ConfigDict | None:">`_model_config`</SwmToken> property retrieves the configuration dictionary for the adapted type, if available.

```python
    @property
    def _model_config(self) -> ConfigDict | None:
        type_: Any = _typing_extra.annotated_type(self._type) or self._type  # Eg FastAPI heavily uses Annotated
        if _utils.lenient_issubclass(type_, BaseModel):
            return type_.model_config
        return getattr(type_, '__pydantic_config__', None)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="332">

---

The **repr** function returns a string representation of the <SwmToken path="pydantic/type_adapter.py" pos="333:5:5" line-data="        return f&#39;TypeAdapter({_repr.display_as_type(self._type)})&#39;">`TypeAdapter`</SwmToken> instance, displaying the adapted type.

```python
    def __repr__(self) -> str:
        return f'TypeAdapter({_repr.display_as_type(self._type)})'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="335">

---

The rebuild function attempts to rebuild the core schema for the adapted type. This is useful if forward references could not be resolved during initial schema creation.

```python
    def rebuild(
        self,
        *,
        force: bool = False,
        raise_errors: bool = True,
        _parent_namespace_depth: int = 2,
        _types_namespace: _namespace_utils.MappingNamespace | None = None,
    ) -> bool | None:
        """Try to rebuild the pydantic-core schema for the adapter's type.

        This may be necessary when one of the annotations is a ForwardRef which could not be resolved during
        the initial attempt to build the schema, and automatic rebuilding fails.

        Args:
            force: Whether to force the rebuilding of the type adapter's schema, defaults to `False`.
            raise_errors: Whether to raise errors, defaults to `True`.
            _parent_namespace_depth: Depth at which to search for the [parent frame][frame-objects]. This
                frame is used when resolving forward annotations during schema rebuilding, by looking for
                the locals of this frame. Defaults to 2, which will result in the frame where the method
                was called.
            _types_namespace: An explicit types namespace to use, instead of using the local namespace
                from the parent frame. Defaults to `None`.

        Returns:
            Returns `None` if the schema is already "complete" and rebuilding was not required.
            If rebuilding _was_ required, returns `True` if rebuilding was successful, otherwise `False`.
        """
        if not force and self.pydantic_complete:
            return None

        if _types_namespace is not None:
            rebuild_ns = _types_namespace
        elif _parent_namespace_depth > 0:
            rebuild_ns = _typing_extra.parent_frame_namespace(parent_depth=_parent_namespace_depth, force=True) or {}
        else:
            rebuild_ns = {}

        # we have to manually fetch globals here because there's no type on the stack of the NsResolver
        # and so we skip the globalns = get_module_ns_of(typ) call that would normally happen
        globalns = sys._getframe(max(_parent_namespace_depth - 1, 1)).f_globals
        ns_resolver = _namespace_utils.NsResolver(
            namespaces_tuple=_namespace_utils.NamespacesTuple(locals=rebuild_ns, globals=globalns),
            parent_namespace=rebuild_ns,
        )
        return self._init_core_attrs(ns_resolver=ns_resolver, force=True, raise_errors=raise_errors)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="381">

---

The <SwmToken path="pydantic/type_adapter.py" pos="381:3:3" line-data="    def validate_python(">`validate_python`</SwmToken> function validates a Python object against the adapted type, supporting strictness, attribute extraction, and partial validation.

```python
    def validate_python(
        self,
        object: Any,
        /,
        *,
        strict: bool | None = None,
        from_attributes: bool | None = None,
        context: dict[str, Any] | None = None,
        experimental_allow_partial: bool | Literal['off', 'on', 'trailing-strings'] = False,
        by_alias: bool | None = None,
        by_name: bool | None = None,
    ) -> T:
        """Validate a Python object against the model.

        Args:
            object: The Python object to validate against the model.
            strict: Whether to strictly check types.
            from_attributes: Whether to extract data from object attributes.
            context: Additional context to pass to the validator.
            experimental_allow_partial: **Experimental** whether to enable
                [partial validation](../concepts/experimental.md#partial-validation), e.g. to process streams.
                * False / 'off': Default behavior, no partial validation.
                * True / 'on': Enable partial validation.
                * 'trailing-strings': Enable partial validation and allow trailing strings in the input.
            by_alias: Whether to use the field's alias when validating against the provided input data.
            by_name: Whether to use the field's name when validating against the provided input data.

        !!! note
            When using `TypeAdapter` with a Pydantic `dataclass`, the use of the `from_attributes`
            argument is not supported.

        Returns:
            The validated object.
        """
        if by_alias is False and by_name is not True:
            raise PydanticUserError(
                'At least one of `by_alias` or `by_name` must be set to True.',
                code='validate-by-alias-and-name-false',
            )

        return self.validator.validate_python(
            object,
            strict=strict,
            from_attributes=from_attributes,
            context=context,
            allow_partial=experimental_allow_partial,
            by_alias=by_alias,
            by_name=by_name,
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="431">

---

The <SwmToken path="pydantic/type_adapter.py" pos="431:3:3" line-data="    def validate_json(">`validate_json`</SwmToken> function validates a JSON string or bytes against the adapted type, supporting strictness, context, and partial validation.

```python
    def validate_json(
        self,
        data: str | bytes | bytearray,
        /,
        *,
        strict: bool | None = None,
        context: dict[str, Any] | None = None,
        experimental_allow_partial: bool | Literal['off', 'on', 'trailing-strings'] = False,
        by_alias: bool | None = None,
        by_name: bool | None = None,
    ) -> T:
        """!!! abstract "Usage Documentation"
            [JSON Parsing](../concepts/json.md#json-parsing)

        Validate a JSON string or bytes against the model.

        Args:
            data: The JSON data to validate against the model.
            strict: Whether to strictly check types.
            context: Additional context to use during validation.
            experimental_allow_partial: **Experimental** whether to enable
                [partial validation](../concepts/experimental.md#partial-validation), e.g. to process streams.
                * False / 'off': Default behavior, no partial validation.
                * True / 'on': Enable partial validation.
                * 'trailing-strings': Enable partial validation and allow trailing strings in the input.
            by_alias: Whether to use the field's alias when validating against the provided input data.
            by_name: Whether to use the field's name when validating against the provided input data.

        Returns:
            The validated object.
        """
        if by_alias is False and by_name is not True:
            raise PydanticUserError(
                'At least one of `by_alias` or `by_name` must be set to True.',
                code='validate-by-alias-and-name-false',
            )

        return self.validator.validate_json(
            data,
            strict=strict,
            context=context,
            allow_partial=experimental_allow_partial,
            by_alias=by_alias,
            by_name=by_name,
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="477">

---

The <SwmToken path="pydantic/type_adapter.py" pos="477:3:3" line-data="    def validate_strings(">`validate_strings`</SwmToken> function validates objects containing string data against the adapted type, with support for strictness and partial validation.

```python
    def validate_strings(
        self,
        obj: Any,
        /,
        *,
        strict: bool | None = None,
        context: dict[str, Any] | None = None,
        experimental_allow_partial: bool | Literal['off', 'on', 'trailing-strings'] = False,
        by_alias: bool | None = None,
        by_name: bool | None = None,
    ) -> T:
        """Validate object contains string data against the model.

        Args:
            obj: The object contains string data to validate.
            strict: Whether to strictly check types.
            context: Additional context to use during validation.
            experimental_allow_partial: **Experimental** whether to enable
                [partial validation](../concepts/experimental.md#partial-validation), e.g. to process streams.
                * False / 'off': Default behavior, no partial validation.
                * True / 'on': Enable partial validation.
                * 'trailing-strings': Enable partial validation and allow trailing strings in the input.
            by_alias: Whether to use the field's alias when validating against the provided input data.
            by_name: Whether to use the field's name when validating against the provided input data.

        Returns:
            The validated object.
        """
        if by_alias is False and by_name is not True:
            raise PydanticUserError(
                'At least one of `by_alias` or `by_name` must be set to True.',
                code='validate-by-alias-and-name-false',
            )

        return self.validator.validate_strings(
            obj,
            strict=strict,
            context=context,
            allow_partial=experimental_allow_partial,
            by_alias=by_alias,
            by_name=by_name,
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="520">

---

The <SwmToken path="pydantic/type_adapter.py" pos="520:3:3" line-data="    def get_default_value(self, *, strict: bool | None = None, context: dict[str, Any] | None = None) -&gt; Some[T] | None:">`get_default_value`</SwmToken> function retrieves the default value for the adapted type, if one exists.

```python
    def get_default_value(self, *, strict: bool | None = None, context: dict[str, Any] | None = None) -> Some[T] | None:
        """Get the default value for the wrapped type.

        Args:
            strict: Whether to strictly check types.
            context: Additional context to pass to the validator.

        Returns:
            The default value wrapped in a `Some` if there is one or None if not.
        """
        return self.validator.get_default_value(strict=strict, context=context)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="532">

---

The <SwmToken path="pydantic/type_adapter.py" pos="532:3:3" line-data="    def dump_python(">`dump_python`</SwmToken> function serializes an instance of the adapted type to a Python object, with options for output format, inclusion/exclusion, and error handling.

```python
    def dump_python(
        self,
        instance: T,
        /,
        *,
        mode: Literal['json', 'python'] = 'python',
        include: IncEx | None = None,
        exclude: IncEx | None = None,
        by_alias: bool | None = None,
        exclude_unset: bool = False,
        exclude_defaults: bool = False,
        exclude_none: bool = False,
        round_trip: bool = False,
        warnings: bool | Literal['none', 'warn', 'error'] = True,
        fallback: Callable[[Any], Any] | None = None,
        serialize_as_any: bool = False,
        context: dict[str, Any] | None = None,
    ) -> Any:
        """Dump an instance of the adapted type to a Python object.

        Args:
            instance: The Python object to serialize.
            mode: The output format.
            include: Fields to include in the output.
            exclude: Fields to exclude from the output.
            by_alias: Whether to use alias names for field names.
            exclude_unset: Whether to exclude unset fields.
            exclude_defaults: Whether to exclude fields with default values.
            exclude_none: Whether to exclude fields with None values.
            round_trip: Whether to output the serialized data in a way that is compatible with deserialization.
            warnings: How to handle serialization errors. False/"none" ignores them, True/"warn" logs errors,
                "error" raises a [`PydanticSerializationError`][pydantic_core.PydanticSerializationError].
            fallback: A function to call when an unknown value is encountered. If not provided,
                a [`PydanticSerializationError`][pydantic_core.PydanticSerializationError] error is raised.
            serialize_as_any: Whether to serialize fields with duck-typing serialization behavior.
            context: Additional context to pass to the serializer.

        Returns:
            The serialized object.
        """
        return self.serializer.to_python(
            instance,
            mode=mode,
            by_alias=by_alias,
            include=include,
            exclude=exclude,
            exclude_unset=exclude_unset,
            exclude_defaults=exclude_defaults,
            exclude_none=exclude_none,
            round_trip=round_trip,
            warnings=warnings,
            fallback=fallback,
            serialize_as_any=serialize_as_any,
            context=context,
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="588">

---

The <SwmToken path="pydantic/type_adapter.py" pos="588:3:3" line-data="    def dump_json(">`dump_json`</SwmToken> function serializes an instance of the adapted type to JSON, returning a bytes object. This is different from <SwmToken path="pydantic/type_adapter.py" pos="205:20:20" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`BaseModel`</SwmToken>'s model_dump_json, which returns a string.

```python
    def dump_json(
        self,
        instance: T,
        /,
        *,
        indent: int | None = None,
        ensure_ascii: bool = False,
        include: IncEx | None = None,
        exclude: IncEx | None = None,
        by_alias: bool | None = None,
        exclude_unset: bool = False,
        exclude_defaults: bool = False,
        exclude_none: bool = False,
        round_trip: bool = False,
        warnings: bool | Literal['none', 'warn', 'error'] = True,
        fallback: Callable[[Any], Any] | None = None,
        serialize_as_any: bool = False,
        context: dict[str, Any] | None = None,
    ) -> bytes:
        """!!! abstract "Usage Documentation"
            [JSON Serialization](../concepts/json.md#json-serialization)

        Serialize an instance of the adapted type to JSON.

        Args:
            instance: The instance to be serialized.
            indent: Number of spaces for JSON indentation.
            ensure_ascii: If `True`, the output is guaranteed to have all incoming non-ASCII characters escaped.
                If `False` (the default), these characters will be output as-is.
            include: Fields to include.
            exclude: Fields to exclude.
            by_alias: Whether to use alias names for field names.
            exclude_unset: Whether to exclude unset fields.
            exclude_defaults: Whether to exclude fields with default values.
            exclude_none: Whether to exclude fields with a value of `None`.
            round_trip: Whether to serialize and deserialize the instance to ensure round-tripping.
            warnings: How to handle serialization errors. False/"none" ignores them, True/"warn" logs errors,
                "error" raises a [`PydanticSerializationError`][pydantic_core.PydanticSerializationError].
            fallback: A function to call when an unknown value is encountered. If not provided,
                a [`PydanticSerializationError`][pydantic_core.PydanticSerializationError] error is raised.
            serialize_as_any: Whether to serialize fields with duck-typing serialization behavior.
            context: Additional context to pass to the serializer.

        Returns:
            The JSON representation of the given instance as bytes.
        """
        return self.serializer.to_json(
            instance,
            indent=indent,
            include=include,
            exclude=exclude,
            by_alias=by_alias,
            exclude_unset=exclude_unset,
            exclude_defaults=exclude_defaults,
            exclude_none=exclude_none,
            round_trip=round_trip,
            warnings=warnings,
            fallback=fallback,
            serialize_as_any=serialize_as_any,
            context=context,
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="650">

---

The <SwmToken path="pydantic/type_adapter.py" pos="650:3:3" line-data="    def json_schema(">`json_schema`</SwmToken> function generates a JSON schema for the adapted type, supporting customization of alias usage, reference templates, and schema generation mode.

```python
    def json_schema(
        self,
        *,
        by_alias: bool = True,
        ref_template: str = DEFAULT_REF_TEMPLATE,
        schema_generator: type[GenerateJsonSchema] = GenerateJsonSchema,
        mode: JsonSchemaMode = 'validation',
    ) -> dict[str, Any]:
        """Generate a JSON schema for the adapted type.

        Args:
            by_alias: Whether to use alias names for field names.
            ref_template: The format string used for generating $ref strings.
            schema_generator: The generator class used for creating the schema.
            mode: The mode to use for schema generation.

        Returns:
            The JSON schema for the model as a dictionary.
        """
        schema_generator_instance = schema_generator(by_alias=by_alias, ref_template=ref_template)
        if isinstance(self.core_schema, _mock_val_ser.MockCoreSchema):
            self.core_schema.rebuild()
            assert not isinstance(self.core_schema, _mock_val_ser.MockCoreSchema), 'this is a bug! please report it'
        return schema_generator_instance.generate(self.core_schema, mode=mode)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/type_adapter.py" line="675">

---

The static method <SwmToken path="pydantic/type_adapter.py" pos="676:3:3" line-data="    def json_schemas(">`json_schemas`</SwmToken> generates JSON schemas including definitions from multiple <SwmToken path="pydantic/type_adapter.py" pos="677:14:14" line-data="        inputs: Iterable[tuple[JsonSchemaKeyT, JsonSchemaMode, TypeAdapter[Any]]],">`TypeAdapter`</SwmToken> instances, returning both a mapping of schemas and a definitions schema.

```python
    @staticmethod
    def json_schemas(
        inputs: Iterable[tuple[JsonSchemaKeyT, JsonSchemaMode, TypeAdapter[Any]]],
        /,
        *,
        by_alias: bool = True,
        title: str | None = None,
        description: str | None = None,
        ref_template: str = DEFAULT_REF_TEMPLATE,
        schema_generator: type[GenerateJsonSchema] = GenerateJsonSchema,
    ) -> tuple[dict[tuple[JsonSchemaKeyT, JsonSchemaMode], JsonSchemaValue], JsonSchemaValue]:
        """Generate a JSON schema including definitions from multiple type adapters.

        Args:
            inputs: Inputs to schema generation. The first two items will form the keys of the (first)
                output mapping; the type adapters will provide the core schemas that get converted into
                definitions in the output JSON schema.
            by_alias: Whether to use alias names.
            title: The title for the schema.
            description: The description for the schema.
            ref_template: The format string used for generating $ref strings.
            schema_generator: The generator class used for creating the schema.

        Returns:
            A tuple where:

                - The first element is a dictionary whose keys are tuples of JSON schema key type and JSON mode, and
                    whose values are the JSON schema corresponding to that pair of inputs. (These schemas may have
                    JsonRef references to definitions that are defined in the second returned element.)
                - The second element is a JSON schema containing all definitions referenced in the first returned
                    element, along with the optional title and description keys.

        """
        schema_generator_instance = schema_generator(by_alias=by_alias, ref_template=ref_template)

        inputs_ = []
        for key, mode, adapter in inputs:
            # This is the same pattern we follow for model json schemas - we attempt a core schema rebuild if we detect a mock
            if isinstance(adapter.core_schema, _mock_val_ser.MockCoreSchema):
                adapter.core_schema.rebuild()
                assert not isinstance(adapter.core_schema, _mock_val_ser.MockCoreSchema), (
                    'this is a bug! please report it'
                )
            inputs_.append((key, mode, adapter.core_schema))

        json_schemas_map, definitions = schema_generator_instance.generate_definitions(inputs_)

        json_schema: dict[str, Any] = {}
        if definitions:
            json_schema['$defs'] = definitions
        if title:
            json_schema['title'] = title
        if description:
            json_schema['description'] = description

        return json_schemas_map, json_schema
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> with <SwmToken path="pydantic/type_adapter.py" pos="205:27:27" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`TypedDict`</SwmToken> and Config

<SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is used to validate and transform data according to a specified type, such as a <SwmToken path="pydantic/type_adapter.py" pos="205:27:27" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`TypedDict`</SwmToken>. In the example, a <SwmToken path="pydantic/type_adapter.py" pos="205:27:27" line-data="                &#39;Cannot use `config` when the type is a BaseModel, dataclass or TypedDict.&#39;">`TypedDict`</SwmToken> with a configuration to convert strings to lowercase is wrapped by <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken>. When validating a dictionary with string values, <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> applies the configuration and returns the transformed data.

## <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> in Plugin and Schema Contexts

<SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is referenced in plugin-related code to define where a schema type was originally defined or where <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> was called. This helps in managing schema validation paths and associating schemas with their original types or models.

## <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> and Configuration Settings

Since version 2.10, configuration settings that affect models also apply to <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> instances. This allows consistent behavior across models, dataclasses, and <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> objects, especially when nested within other models or when manually defining type namespaces.

## <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> in Internal Utilities

Internally, <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is used to access core schemas for models or dataclasses, facilitating schema cleaning and pretty-printing. It also appears in namespace utilities to support schema generation for simple annotations, where <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> helps analyze types without requiring a full class context.

## <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> and Version Compatibility

<SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> is involved in warnings about mixing different model versions, indicating that constructs like <SwmToken path="pydantic/type_adapter.py" pos="207:7:7" line-data="                &#39; parameter to TypeAdapter will not override it, thus the `config` you passed to&#39;">`TypeAdapter`</SwmToken> should be used consistently with the same model version to avoid compatibility issues.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
