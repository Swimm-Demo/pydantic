---
title: The GenerateJsonSchema class
---
This document covers the <SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken> class in detail, focusing on:

1. What <SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken> is and its purpose
2. Variables and functions defined in <SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken>, with explanations and code references

# What is <SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken>

<SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken> is a class in <SwmPath>[pydantic/json_schema.py](pydantic/json_schema.py)</SwmPath> responsible for generating JSON schemas from Pydantic core schemas. It is highly configurable, allowing for customization of schema generation, including how references are handled, which warnings are emitted, and whether field aliases are used. The class is designed to support both validation and serialization modes, and it manages the mapping between internal schema references and the resulting JSON Schema output. <SwmToken path="pydantic/json_schema.py" pos="96:4:4" line-data="See [`GenerateJsonSchema.render_warning_message`][pydantic.json_schema.GenerateJsonSchema.render_warning_message]">`GenerateJsonSchema`</SwmToken> is the core engine behind Pydantic's ability to produce standards-compliant JSON Schema representations for models and types.

<SwmSnippet path="/pydantic/json_schema.py" line="250">

---

The class variable <SwmToken path="pydantic/json_schema.py" pos="250:1:1" line-data="    schema_dialect = &#39;https://json-schema.org/draft/2020-12/schema&#39;">`schema_dialect`</SwmToken> defines the JSON Schema dialect used for generation, defaulting to '<https://json-schema.org/draft/2020-12/schema>'.

```python
    schema_dialect = 'https://json-schema.org/draft/2020-12/schema'

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="254">

---

The class variable <SwmToken path="pydantic/json_schema.py" pos="254:1:1" line-data="    ignored_warning_kinds: set[JsonSchemaWarningKind] = {&#39;skipped-choice&#39;}">`ignored_warning_kinds`</SwmToken> is a set of warning kinds that should be ignored during schema generation. By default, it contains <SwmToken path="pydantic/json_schema.py" pos="254:13:15" line-data="    ignored_warning_kinds: set[JsonSchemaWarningKind] = {&#39;skipped-choice&#39;}">`skipped-choice`</SwmToken>.

```python
    ignored_warning_kinds: set[JsonSchemaWarningKind] = {'skipped-choice'}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="256">

---

The constructor (<SwmToken path="pydantic/json_schema.py" pos="256:3:3" line-data="    def __init__(self, by_alias: bool = True, ref_template: str = DEFAULT_REF_TEMPLATE):">`__init__`</SwmToken>) initializes configuration options such as <SwmToken path="pydantic/json_schema.py" pos="256:8:8" line-data="    def __init__(self, by_alias: bool = True, ref_template: str = DEFAULT_REF_TEMPLATE):">`by_alias`</SwmToken> and <SwmToken path="pydantic/json_schema.py" pos="256:18:18" line-data="    def __init__(self, by_alias: bool = True, ref_template: str = DEFAULT_REF_TEMPLATE):">`ref_template`</SwmToken>, and sets up internal mappings for references, definitions, and configuration stacks. It also prepares the schema type-to-method mapping and tracks usage to prevent accidental reuse.

```python
    def __init__(self, by_alias: bool = True, ref_template: str = DEFAULT_REF_TEMPLATE):
        self.by_alias = by_alias
        self.ref_template = ref_template

        self.core_to_json_refs: dict[CoreModeRef, JsonRef] = {}
        self.core_to_defs_refs: dict[CoreModeRef, DefsRef] = {}
        self.defs_to_core_refs: dict[DefsRef, CoreModeRef] = {}
        self.json_to_defs_refs: dict[JsonRef, DefsRef] = {}

        self.definitions: dict[DefsRef, JsonSchemaValue] = {}
        self._config_wrapper_stack = _config.ConfigWrapperStack(_config.ConfigWrapper({}))

        self._mode: JsonSchemaMode = 'validation'

        # The following includes a mapping of a fully-unique defs ref choice to a list of preferred
        # alternatives, which are generally simpler, such as only including the class name.
        # At the end of schema generation, we use these to produce a JSON schema with more human-readable
        # definitions, which would also work better in a generated OpenAPI client, etc.
        self._prioritized_defsref_choices: dict[DefsRef, list[DefsRef]] = {}
        self._collision_counter: dict[str, int] = defaultdict(int)
        self._collision_index: dict[str, int] = {}

        self._schema_type_to_method = self.build_schema_type_to_method()

        # When we encounter definitions we need to try to build them immediately
        # so that they are available schemas that reference them
        # But it's possible that CoreSchema was never going to be used
        # (e.g. because the CoreSchema that references short circuits is JSON schema generation without needing
        #  the reference) so instead of failing altogether if we can't build a definition we
        # store the error raised and re-throw it if we end up needing that def
        self._core_defs_invalid_for_json_schema: dict[DefsRef, PydanticInvalidForJsonSchema] = {}

        # This changes to True after generating a schema, to prevent issues caused by accidental reuse
        # of a single instance of a schema generator
        self._used = False

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="292">

---

The property <SwmToken path="pydantic/json_schema.py" pos="293:3:3" line-data="    def _config(self) -&gt; _config.ConfigWrapper:">`_config`</SwmToken> returns the current configuration wrapper from the stack, allowing access to configuration options relevant to schema generation.

```python
    @property
    def _config(self) -> _config.ConfigWrapper:
        return self._config_wrapper_stack.tail

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="296">

---

The property <SwmToken path="pydantic/json_schema.py" pos="297:3:3" line-data="    def mode(self) -&gt; JsonSchemaMode:">`mode`</SwmToken> returns the current schema generation mode ('validation' or 'serialization'), considering any overrides from the configuration.

```python
    @property
    def mode(self) -> JsonSchemaMode:
        if self._config.json_schema_mode_override is not None:
            return self._config.json_schema_mode_override
        else:
            return self._mode

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="303">

---

The function <SwmToken path="pydantic/json_schema.py" pos="303:3:3" line-data="    def build_schema_type_to_method(">`build_schema_type_to_method`</SwmToken> constructs a mapping from core schema types to their corresponding handler methods for JSON schema generation. It ensures that each schema type has an associated method, raising an error if not.

```python
    def build_schema_type_to_method(
        self,
    ) -> dict[CoreSchemaOrFieldType, Callable[[CoreSchemaOrField], JsonSchemaValue]]:
        """Builds a dictionary mapping fields to methods for generating JSON schemas.

        Returns:
            A dictionary containing the mapping of `CoreSchemaOrFieldType` to a handler method.

        Raises:
            TypeError: If no method has been defined for generating a JSON schema for a given pydantic core schema type.
        """
        mapping: dict[CoreSchemaOrFieldType, Callable[[CoreSchemaOrField], JsonSchemaValue]] = {}
        core_schema_types: list[CoreSchemaOrFieldType] = list(get_literal_values(CoreSchemaOrFieldType))
        for key in core_schema_types:
            method_name = f'{key.replace("-", "_")}_schema'
            try:
                mapping[key] = getattr(self, method_name)
            except AttributeError as e:  # pragma: no cover
                if os.getenv('PYDANTIC_PRIVATE_ALLOW_UNHANDLED_SCHEMA_TYPES'):
                    continue
                raise TypeError(
                    f'No method for generating JsonSchema for core_schema.type={key!r} '
                    f'(expected: {type(self).__name__}.{method_name})'
                ) from e
        return mapping

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="329">

---

The function <SwmToken path="pydantic/json_schema.py" pos="329:3:3" line-data="    def generate_definitions(">`generate_definitions`</SwmToken> generates JSON schema definitions for a sequence of core schemas, returning a mapping from input keys to schema references and the actual definitions. It prevents reuse of the same generator instance and remaps references for output.

```python
    def generate_definitions(
        self, inputs: Sequence[tuple[JsonSchemaKeyT, JsonSchemaMode, core_schema.CoreSchema]]
    ) -> tuple[dict[tuple[JsonSchemaKeyT, JsonSchemaMode], JsonSchemaValue], dict[DefsRef, JsonSchemaValue]]:
        """Generates JSON schema definitions from a list of core schemas, pairing the generated definitions with a
        mapping that links the input keys to the definition references.

        Args:
            inputs: A sequence of tuples, where:

                - The first element is a JSON schema key type.
                - The second element is the JSON mode: either 'validation' or 'serialization'.
                - The third element is a core schema.

        Returns:
            A tuple where:

                - The first element is a dictionary whose keys are tuples of JSON schema key type and JSON mode, and
                    whose values are the JSON schema corresponding to that pair of inputs. (These schemas may have
                    JsonRef references to definitions that are defined in the second returned element.)
                - The second element is a dictionary whose keys are definition references for the JSON schemas
                    from the first returned element, and whose values are the actual JSON schema definitions.

        Raises:
            PydanticUserError: Raised if the JSON schema generator has already been used to generate a JSON schema.
        """
        if self._used:
            raise PydanticUserError(
                'This JSON schema generator has already been used to generate a JSON schema. '
                f'You must create a new instance of {type(self).__name__} to generate a new JSON schema.',
                code='json-schema-already-used',
            )

        for _, mode, schema in inputs:
            self._mode = mode
            self.generate_inner(schema)

        definitions_remapping = self._build_definitions_remapping()

        json_schemas_map: dict[tuple[JsonSchemaKeyT, JsonSchemaMode], DefsRef] = {}
        for key, mode, schema in inputs:
            self._mode = mode
            json_schema = self.generate_inner(schema)
            json_schemas_map[(key, mode)] = definitions_remapping.remap_json_schema(json_schema)

        json_schema = {'$defs': self.definitions}
        json_schema = definitions_remapping.remap_json_schema(json_schema)
        self._used = True
        return json_schemas_map, self.sort(json_schema['$defs'])  # type: ignore

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="378">

---

The function <SwmToken path="pydantic/json_schema.py" pos="378:3:3" line-data="    def generate(self, schema: CoreSchema, mode: JsonSchemaMode = &#39;validation&#39;) -&gt; JsonSchemaValue:">`generate`</SwmToken> produces a JSON schema for a single core schema in a specified mode. It manages reference unpacking, garbage collection of unused definitions, and remapping of references. It also prevents reuse of the generator instance.

```python
    def generate(self, schema: CoreSchema, mode: JsonSchemaMode = 'validation') -> JsonSchemaValue:
        """Generates a JSON schema for a specified schema in a specified mode.

        Args:
            schema: A Pydantic model.
            mode: The mode in which to generate the schema. Defaults to 'validation'.

        Returns:
            A JSON schema representing the specified schema.

        Raises:
            PydanticUserError: If the JSON schema generator has already been used to generate a JSON schema.
        """
        self._mode = mode
        if self._used:
            raise PydanticUserError(
                'This JSON schema generator has already been used to generate a JSON schema. '
                f'You must create a new instance of {type(self).__name__} to generate a new JSON schema.',
                code='json-schema-already-used',
            )

        json_schema: JsonSchemaValue = self.generate_inner(schema)
        json_ref_counts = self.get_json_ref_counts(json_schema)

        ref = cast(JsonRef, json_schema.get('$ref'))
        while ref is not None:  # may need to unpack multiple levels
            ref_json_schema = self.get_schema_from_definitions(ref)
            if json_ref_counts[ref] == 1 and ref_json_schema is not None and len(json_schema) == 1:
                # "Unpack" the ref since this is the only reference and there are no sibling keys
                json_schema = ref_json_schema.copy()  # copy to prevent recursive dict reference
                json_ref_counts[ref] -= 1
                ref = cast(JsonRef, json_schema.get('$ref'))
            ref = None

        self._garbage_collect_definitions(json_schema)
        definitions_remapping = self._build_definitions_remapping()

        if self.definitions:
            json_schema['$defs'] = self.definitions

        json_schema = definitions_remapping.remap_json_schema(json_schema)

        # For now, we will not set the $schema key. However, if desired, this can be easily added by overriding
        # this method and adding the following line after a call to super().generate(schema):
        # json_schema['$schema'] = self.schema_dialect

        self._used = True
        return self.sort(json_schema)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="427">

---

The function <SwmToken path="pydantic/json_schema.py" pos="427:3:3" line-data="    def generate_inner(self, schema: CoreSchemaOrField) -&gt; JsonSchemaValue:  # noqa: C901">`generate_inner`</SwmToken> is the core recursive engine for generating JSON schema from a core schema or field. It handles references, metadata, and applies any schema modifications specified in the metadata.

```python
    def generate_inner(self, schema: CoreSchemaOrField) -> JsonSchemaValue:  # noqa: C901
        """Generates a JSON schema for a given core schema.

        Args:
            schema: The given core schema.

        Returns:
            The generated JSON schema.

        TODO: the nested function definitions here seem like bad practice, I'd like to unpack these
        in a future PR. It'd be great if we could shorten the call stack a bit for JSON schema generation,
        and I think there's potential for that here.
        """
        # If a schema with the same CoreRef has been handled, just return a reference to it
        # Note that this assumes that it will _never_ be the case that the same CoreRef is used
        # on types that should have different JSON schemas
        if 'ref' in schema:
            core_ref = CoreRef(schema['ref'])  # type: ignore[typeddict-item]
            core_mode_ref = (core_ref, self.mode)
            if core_mode_ref in self.core_to_defs_refs and self.core_to_defs_refs[core_mode_ref] in self.definitions:
                return {'$ref': self.core_to_json_refs[core_mode_ref]}

        def populate_defs(core_schema: CoreSchema, json_schema: JsonSchemaValue) -> JsonSchemaValue:
            if 'ref' in core_schema:
                core_ref = CoreRef(core_schema['ref'])  # type: ignore[typeddict-item]
                defs_ref, ref_json_schema = self.get_cache_defs_ref_schema(core_ref)
                json_ref = JsonRef(ref_json_schema['$ref'])
                # Replace the schema if it's not a reference to itself
                # What we want to avoid is having the def be just a ref to itself
                # which is what would happen if we blindly assigned any
                if json_schema.get('$ref', None) != json_ref:
                    self.definitions[defs_ref] = json_schema
                    self._core_defs_invalid_for_json_schema.pop(defs_ref, None)
                json_schema = ref_json_schema
            return json_schema

        def handler_func(schema_or_field: CoreSchemaOrField) -> JsonSchemaValue:
            """Generate a JSON schema based on the input schema.

            Args:
                schema_or_field: The core schema to generate a JSON schema from.

            Returns:
                The generated JSON schema.

            Raises:
                TypeError: If an unexpected schema type is encountered.
            """
            # Generate the core-schema-type-specific bits of the schema generation:
            json_schema: JsonSchemaValue | None = None
            if self.mode == 'serialization' and 'serialization' in schema_or_field:
                # In this case, we skip the JSON Schema generation of the schema
                # and use the `'serialization'` schema instead (canonical example:
                # `Annotated[int, PlainSerializer(str)]`).
                ser_schema = schema_or_field['serialization']  # type: ignore
                json_schema = self.ser_schema(ser_schema)

                # It might be that the 'serialization'` is skipped depending on `when_used`.
                # This is only relevant for `nullable` schemas though, so we special case here.
                if (
                    json_schema is not None
                    and ser_schema.get('when_used') in ('unless-none', 'json-unless-none')
                    and schema_or_field['type'] == 'nullable'
                ):
                    json_schema = self.get_flattened_anyof([{'type': 'null'}, json_schema])
            if json_schema is None:
                if _core_utils.is_core_schema(schema_or_field) or _core_utils.is_core_schema_field(schema_or_field):
                    generate_for_schema_type = self._schema_type_to_method[schema_or_field['type']]
                    json_schema = generate_for_schema_type(schema_or_field)
                else:
                    raise TypeError(f'Unexpected schema type: schema={schema_or_field}')

            return json_schema

        current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, handler_func)

        metadata = cast(_core_metadata.CoreMetadata, schema.get('metadata', {}))

        # TODO: I dislike that we have to wrap these basic dict updates in callables, is there any way around this?

        if js_updates := metadata.get('pydantic_js_updates'):

            def js_updates_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
            ) -> JsonSchemaValue:
                json_schema = {**current_handler(schema_or_field), **js_updates}
                return json_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, js_updates_handler_func)

        if js_extra := metadata.get('pydantic_js_extra'):

            def js_extra_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
            ) -> JsonSchemaValue:
                json_schema = current_handler(schema_or_field)
                if isinstance(js_extra, dict):
                    json_schema.update(to_jsonable_python(js_extra))
                elif callable(js_extra):
                    # similar to typing issue in _update_class_schema when we're working with callable js extra
                    js_extra(json_schema)  # type: ignore
                return json_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, js_extra_handler_func)

        for js_modify_function in metadata.get('pydantic_js_functions', ()):

            def new_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
                js_modify_function: GetJsonSchemaFunction = js_modify_function,
            ) -> JsonSchemaValue:
                json_schema = js_modify_function(schema_or_field, current_handler)
                if _core_utils.is_core_schema(schema_or_field):
                    json_schema = populate_defs(schema_or_field, json_schema)
                original_schema = current_handler.resolve_ref_schema(json_schema)
                ref = json_schema.pop('$ref', None)
                if ref and json_schema:
                    original_schema.update(json_schema)
                return original_schema

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)

        for js_modify_function in metadata.get('pydantic_js_annotation_functions', ()):

            def new_handler_func(
                schema_or_field: CoreSchemaOrField,
                current_handler: GetJsonSchemaHandler = current_handler,
                js_modify_function: GetJsonSchemaFunction = js_modify_function,
            ) -> JsonSchemaValue:
                return js_modify_function(schema_or_field, current_handler)

            current_handler = _schema_generation_shared.GenerateJsonSchemaHandler(self, new_handler_func)

        json_schema = current_handler(schema)
        if _core_utils.is_core_schema(schema):
            json_schema = populate_defs(schema, json_schema)
        return json_schema

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="568">

---

The function <SwmToken path="pydantic/json_schema.py" pos="568:3:3" line-data="    def sort(self, value: JsonSchemaValue, parent_key: str | None = None) -&gt; JsonSchemaValue:">`sort`</SwmToken> recursively sorts the keys of the generated JSON schema, except for certain keys like 'properties' and 'default', to preserve field order where necessary.

```python
    def sort(self, value: JsonSchemaValue, parent_key: str | None = None) -> JsonSchemaValue:
        """Override this method to customize the sorting of the JSON schema (e.g., don't sort at all, sort all keys unconditionally, etc.)

        By default, alphabetically sort the keys in the JSON schema, skipping the 'properties' and 'default' keys to preserve field definition order.
        This sort is recursive, so it will sort all nested dictionaries as well.
        """
        sorted_dict: dict[str, JsonSchemaValue] = {}
        keys = value.keys()
        if parent_key not in ('properties', 'default'):
            keys = sorted(keys)
        for key in keys:
            sorted_dict[key] = self._sort_recursive(value[key], parent_key=key)
        return sorted_dict

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="2032">

---

The function <SwmToken path="pydantic/json_schema.py" pos="2032:3:3" line-data="    def ser_schema(">`ser_schema`</SwmToken> generates a JSON schema for serialization-specific schemas, handling different serializer types and delegating to the appropriate handler.

```python
    def ser_schema(
        self, schema: core_schema.SerSchema | core_schema.IncExSeqSerSchema | core_schema.IncExDictSerSchema
    ) -> JsonSchemaValue | None:
        """Generates a JSON schema that matches a schema that defines a serialized object.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        schema_type = schema['type']
        if schema_type == 'function-plain' or schema_type == 'function-wrap':
            # PlainSerializerFunctionSerSchema or WrapSerializerFunctionSerSchema
            return_schema = schema.get('return_schema')
            if return_schema is not None:
                return self.generate_inner(return_schema)
        elif schema_type == 'format' or schema_type == 'to-string':
            # FormatSerSchema or ToStringSerSchema
            return self.str_schema(core_schema.str_schema())
        elif schema['type'] == 'model':
            # ModelSerSchema
            return self.generate_inner(schema['schema'])
        return None

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="1266">

---

The function <SwmToken path="pydantic/json_schema.py" pos="1266:3:3" line-data="    def tagged_union_schema(self, schema: core_schema.TaggedUnionSchema) -&gt; JsonSchemaValue:">`tagged_union_schema`</SwmToken> generates a JSON schema for tagged unions, supporting discriminators and mapping each choice to its schema. It also emits warnings for skipped choices.

```python
    def tagged_union_schema(self, schema: core_schema.TaggedUnionSchema) -> JsonSchemaValue:
        """Generates a JSON schema that matches a schema that allows values matching any of the given schemas, where
        the schemas are tagged with a discriminator field that indicates which schema should be used to validate
        the value.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        generated: dict[str, JsonSchemaValue] = {}
        for k, v in schema['choices'].items():
            if isinstance(k, Enum):
                k = k.value
            try:
                # Use str(k) since keys must be strings for json; while not technically correct,
                # it's the closest that can be represented in valid JSON
                generated[str(k)] = self.generate_inner(v).copy()
            except PydanticOmit:
                continue
            except PydanticInvalidForJsonSchema as exc:
                self.emit_warning('skipped-choice', exc.message)

        one_of_choices = _deduplicate_schemas(generated.values())
        json_schema: JsonSchemaValue = {'oneOf': one_of_choices}

        # This reflects the v1 behavior; TODO: we should make it possible to exclude OpenAPI stuff from the JSON schema
        openapi_discriminator = self._extract_discriminator(schema, one_of_choices)
        if openapi_discriminator is not None:
            json_schema['discriminator'] = {
                'propertyName': openapi_discriminator,
                'mapping': {k: v.get('$ref', v) for k, v in generated.items()},
            }

        return json_schema

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="1348">

---

The function <SwmToken path="pydantic/json_schema.py" pos="1348:3:3" line-data="    def chain_schema(self, schema: core_schema.ChainSchema) -&gt; JsonSchemaValue:">`chain_schema`</SwmToken> generates a JSON schema for chain schemas, selecting the appropriate step based on the current mode (validation or serialization).

```python
    def chain_schema(self, schema: core_schema.ChainSchema) -> JsonSchemaValue:
        """Generates a JSON schema that matches a core_schema.ChainSchema.

        When generating a schema for validation, we return the validation JSON schema for the first step in the chain.
        For serialization, we return the serialization JSON schema for the last step in the chain.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        step_index = 0 if self.mode == 'validation' else -1  # use first step for validation, last for serialization
        return self.generate_inner(schema['steps'][step_index])

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="1788">

---

The function <SwmToken path="pydantic/json_schema.py" pos="1788:3:3" line-data="    def kw_arguments_schema(">`kw_arguments_schema`</SwmToken> generates a JSON schema for function keyword arguments, building the properties and required fields, and handling additional properties if present.

```python
    def kw_arguments_schema(
        self, arguments: list[core_schema.ArgumentsParameter], var_kwargs_schema: CoreSchema | None
    ) -> JsonSchemaValue:
        """Generates a JSON schema that matches a schema that defines a function's keyword arguments.

        Args:
            arguments: The core schema.

        Returns:
            The generated JSON schema.
        """
        properties: dict[str, JsonSchemaValue] = {}
        required: list[str] = []
        for argument in arguments:
            name = self.get_argument_name(argument)
            argument_schema = self.generate_inner(argument['schema']).copy()
            argument_schema['title'] = self.get_title_from_name(name)
            properties[name] = argument_schema

            if argument['schema']['type'] != 'default':
                # This assumes that if the argument has a default value,
                # the inner schema must be of type WithDefaultSchema.
                # I believe this is true, but I am not 100% sure
                required.append(name)

        json_schema: JsonSchemaValue = {'type': 'object', 'properties': properties}
        if required:
            json_schema['required'] = required

        if var_kwargs_schema:
            additional_properties_schema = self.generate_inner(var_kwargs_schema)
            if additional_properties_schema:
                json_schema['additionalProperties'] = additional_properties_schema
        else:
            json_schema['additionalProperties'] = False
        return json_schema
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="1825">

---

The function <SwmToken path="pydantic/json_schema.py" pos="1825:3:3" line-data="    def p_arguments_schema(">`p_arguments_schema`</SwmToken> generates a JSON schema for function positional arguments, building prefix items and handling minimum and maximum items as needed.

```python
    def p_arguments_schema(
        self, arguments: list[core_schema.ArgumentsParameter], var_args_schema: CoreSchema | None
    ) -> JsonSchemaValue:
        """Generates a JSON schema that matches a schema that defines a function's positional arguments.

        Args:
            arguments: The core schema.

        Returns:
            The generated JSON schema.
        """
        prefix_items: list[JsonSchemaValue] = []
        min_items = 0

        for argument in arguments:
            name = self.get_argument_name(argument)

            argument_schema = self.generate_inner(argument['schema']).copy()
            argument_schema['title'] = self.get_title_from_name(name)
            prefix_items.append(argument_schema)

            if argument['schema']['type'] != 'default':
                # This assumes that if the argument has a default value,
                # the inner schema must be of type WithDefaultSchema.
                # I believe this is true, but I am not 100% sure
                min_items += 1

        json_schema: JsonSchemaValue = {'type': 'array'}
        if prefix_items:
            json_schema['prefixItems'] = prefix_items
        if min_items:
            json_schema['minItems'] = min_items

        if var_args_schema:
            items_schema = self.generate_inner(var_args_schema)
            if items_schema:
                json_schema['items'] = items_schema
        else:
            json_schema['maxItems'] = len(prefix_items)

        return json_schema
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/json_schema.py" line="607">

---

The function <SwmToken path="pydantic/json_schema.py" pos="607:3:3" line-data="    def any_schema(self, schema: core_schema.AnySchema) -&gt; JsonSchemaValue:">`any_schema`</SwmToken> generates a JSON schema that matches any value, returning an empty schema object.

```python
    def any_schema(self, schema: core_schema.AnySchema) -> JsonSchemaValue:
        """Generates a JSON schema that matches any value.

        Args:
            schema: The core schema.

        Returns:
            The generated JSON schema.
        """
        return {}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
