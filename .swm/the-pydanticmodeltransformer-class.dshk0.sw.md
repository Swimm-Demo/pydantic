---
title: The PydanticModelTransformer class
---
This document covers the <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> class. We'll address:

1. What <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken>, with code citations for each.

# What is <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken>

<SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> is a class responsible for transforming subclasses of <SwmToken path="pydantic/mypy.py" pos="454:8:8" line-data="        &quot;&quot;&quot;Configures the BaseModel subclass according to the plugin settings.">`BaseModel`</SwmToken> according to the plugin settings in the Pydantic mypy plugin. It analyzes and configures Pydantic models during static type checking, handling model configuration, field collection, method signature generation, and class freezing. This class is central to ensuring that Pydantic models are correctly interpreted and type-checked by mypy, based on both explicit and inherited configuration.

<SwmSnippet path="/pydantic/mypy.py" line="429">

---

The class variable <SwmToken path="pydantic/mypy.py" pos="429:1:1" line-data="    tracked_config_fields: set[str] = {">`tracked_config_fields`</SwmToken> is a set of configuration field names that the plugin tracks for value changes. These include options like 'extra', 'frozen', <SwmToken path="pydantic/mypy.py" pos="432:2:2" line-data="        &#39;from_attributes&#39;,">`from_attributes`</SwmToken>, and others.

```python
    tracked_config_fields: set[str] = {
        'extra',
        'frozen',
        'from_attributes',
        'populate_by_name',
        'validate_by_alias',
        'validate_by_name',
        'alias_generator',
        'strict',
    }
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="440">

---

The constructor (<SwmToken path="pydantic/mypy.py" pos="440:3:3" line-data="    def __init__(">`__init__`</SwmToken>) initializes the transformer with the class definition, the reason for transformation, the plugin API, and the plugin configuration. It stores these for use in transformation logic.

```python
    def __init__(
        self,
        cls: ClassDef,
        reason: Expression | Statement,
        api: SemanticAnalyzerPluginInterface,
        plugin_config: PydanticPluginConfig,
    ) -> None:
        self._cls = cls
        self._reason = reason
        self._api = api

        self.plugin_config = plugin_config

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="453">

---

The <SwmToken path="pydantic/mypy.py" pos="453:3:3" line-data="    def transform(self) -&gt; bool:">`transform`</SwmToken> method is the main entry point. It configures a <SwmToken path="pydantic/mypy.py" pos="454:8:8" line-data="        &quot;&quot;&quot;Configures the BaseModel subclass according to the plugin settings.">`BaseModel`</SwmToken> subclass according to plugin settings by collecting config and fields, generating method signatures, freezing the class if needed, and storing metadata for subclasses. It returns True if transformation is complete, or False if another pass is needed.

```python
    def transform(self) -> bool:
        """Configures the BaseModel subclass according to the plugin settings.

        In particular:

        * determines the model config and fields,
        * adds a fields-aware signature for the initializer and construct methods
        * freezes the class if frozen = True
        * stores the fields, config, and if the class is settings in the mypy metadata for access by subclasses
        """
        info = self._cls.info
        is_a_root_model = is_root_model(info)
        config = self.collect_config()
        fields, class_vars = self.collect_fields_and_class_vars(config, is_a_root_model)
        if fields is None or class_vars is None:
            # Some definitions are not ready. We need another pass.
            return False
        for field in fields:
            if field.type is None:
                return False

        is_settings = info.has_base(BASESETTINGS_FULLNAME)
        self.add_initializer(fields, config, is_settings, is_a_root_model)
        self.add_model_construct_method(fields, config, is_settings, is_a_root_model)
        self.set_frozen(fields, self._api, frozen=config.frozen is True)

        self.adjust_decorator_signatures()

        info.metadata[METADATA_KEY] = {
            'fields': {field.name: field.serialize() for field in fields},
            'class_vars': {class_var.name: class_var.serialize() for class_var in class_vars},
            'config': config.get_values_dict(),
        }

        return True

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="489">

---

The <SwmToken path="pydantic/mypy.py" pos="489:3:3" line-data="    def adjust_decorator_signatures(self) -&gt; None:">`adjust_decorator_signatures`</SwmToken> function updates method signatures for functions decorated with Pydantic's validator, <SwmToken path="pydantic/mypy.py" pos="490:32:32" line-data="        &quot;&quot;&quot;When we decorate a function `f` with `pydantic.validator(...)`, `pydantic.field_validator`">`field_validator`</SwmToken>, or serializer decorators. It marks these as classmethods in mypy's type system when appropriate.

```python
    def adjust_decorator_signatures(self) -> None:
        """When we decorate a function `f` with `pydantic.validator(...)`, `pydantic.field_validator`
        or `pydantic.serializer(...)`, mypy sees `f` as a regular method taking a `self` instance,
        even though pydantic internally wraps `f` with `classmethod` if necessary.

        Teach mypy this by marking any function whose outermost decorator is a `validator()`,
        `field_validator()` or `serializer()` call as a `classmethod`.
        """
        for sym in self._cls.info.names.values():
            if isinstance(sym.node, Decorator):
                first_dec = sym.node.original_decorators[0]
                if (
                    isinstance(first_dec, CallExpr)
                    and isinstance(first_dec.callee, NameExpr)
                    and first_dec.callee.fullname in IMPLICIT_CLASSMETHOD_DECORATOR_FULLNAMES
                    # @model_validator(mode="after") is an exception, it expects a regular method
                    and not (
                        first_dec.callee.fullname == MODEL_VALIDATOR_FULLNAME
                        and any(
                            first_dec.arg_names[i] == 'mode' and isinstance(arg, StrExpr) and arg.value == 'after'
                            for i, arg in enumerate(first_dec.args)
                        )
                    )
                ):
                    # TODO: Only do this if the first argument of the decorated function is `cls`
                    sym.node.func.is_class = True

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="516">

---

The <SwmToken path="pydantic/mypy.py" pos="516:3:3" line-data="    def collect_config(self) -&gt; ModelConfigData:  # noqa: C901 (ignore complexity)">`collect_config`</SwmToken> function gathers configuration values used by the plugin, considering both direct class attributes and inherited settings. It handles config specified via class keywords, <SwmToken path="pydantic/mypy.py" pos="531:6:6" line-data="        # Handle `model_config`">`model_config`</SwmToken> assignments, or deprecated Config classes.

```python
    def collect_config(self) -> ModelConfigData:  # noqa: C901 (ignore complexity)
        """Collects the values of the config attributes that are used by the plugin, accounting for parent classes."""
        cls = self._cls
        config = ModelConfigData()

        has_config_kwargs = False
        has_config_from_namespace = False

        # Handle `class MyModel(BaseModel, <name>=<expr>, ...):`
        for name, expr in cls.keywords.items():
            config_data = self.get_config_update(name, expr)
            if config_data:
                has_config_kwargs = True
                config.update(config_data)

        # Handle `model_config`
        stmt: Statement | None = None
        for stmt in cls.defs.body:
            if not isinstance(stmt, (AssignmentStmt, ClassDef)):
                continue

            if isinstance(stmt, AssignmentStmt):
                lhs = stmt.lvalues[0]
                if not isinstance(lhs, NameExpr) or lhs.name != 'model_config':
                    continue

                if isinstance(stmt.rvalue, CallExpr):  # calls to `dict` or `ConfigDict`
                    for arg_name, arg in zip(stmt.rvalue.arg_names, stmt.rvalue.args):
                        if arg_name is None:
                            continue
                        config.update(self.get_config_update(arg_name, arg, lax_extra=True))
                elif isinstance(stmt.rvalue, DictExpr):  # dict literals
                    for key_expr, value_expr in stmt.rvalue.items:
                        if not isinstance(key_expr, StrExpr):
                            continue
                        config.update(self.get_config_update(key_expr.value, value_expr))

            elif isinstance(stmt, ClassDef):
                if stmt.name != 'Config':  # 'deprecated' Config-class
                    continue
                for substmt in stmt.defs.body:
                    if not isinstance(substmt, AssignmentStmt):
                        continue
                    lhs = substmt.lvalues[0]
                    if not isinstance(lhs, NameExpr):
                        continue
                    config.update(self.get_config_update(lhs.name, substmt.rvalue))

            if has_config_kwargs:
                self._api.fail(
                    'Specifying config in two places is ambiguous, use either Config attribute or class kwargs',
                    cls,
                )
                break

            has_config_from_namespace = True

        if has_config_kwargs or has_config_from_namespace:
            if (
                stmt
                and config.has_alias_generator
                and not (config.validate_by_name or config.populate_by_name)
                and self.plugin_config.warn_required_dynamic_aliases
            ):
                error_required_dynamic_aliases(self._api, stmt)

        for info in cls.info.mro[1:]:  # 0 is the current class
            if METADATA_KEY not in info.metadata:
                continue

            # Each class depends on the set of fields in its ancestors
            self._api.add_plugin_dependency(make_wildcard_trigger(info.fullname))
            for name, value in info.metadata[METADATA_KEY]['config'].items():
                config.setdefault(name, value)
        return config

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="592">

---

The <SwmToken path="pydantic/mypy.py" pos="592:3:3" line-data="    def collect_fields_and_class_vars(">`collect_fields_and_class_vars`</SwmToken> function collects all fields and class variables for the model, including those inherited from parent classes. It ensures correct ordering and overrides, and validates field definitions.

```python
    def collect_fields_and_class_vars(
        self, model_config: ModelConfigData, is_root_model: bool
    ) -> tuple[list[PydanticModelField] | None, list[PydanticModelClassVar] | None]:
        """Collects the fields for the model, accounting for parent classes."""
        cls = self._cls

        # First, collect fields and ClassVars belonging to any class in the MRO, ignoring duplicates.
        #
        # We iterate through the MRO in reverse because attrs defined in the parent must appear
        # earlier in the attributes list than attrs defined in the child. See:
        # https://docs.python.org/3/library/dataclasses.html#inheritance
        #
        # However, we also want fields defined in the subtype to override ones defined
        # in the parent. We can implement this via a dict without disrupting the attr order
        # because dicts preserve insertion order in Python 3.7+.
        found_fields: dict[str, PydanticModelField] = {}
        found_class_vars: dict[str, PydanticModelClassVar] = {}
        for info in reversed(cls.info.mro[1:-1]):  # 0 is the current class, -2 is BaseModel, -1 is object
            # if BASEMODEL_METADATA_TAG_KEY in info.metadata and BASEMODEL_METADATA_KEY not in info.metadata:
            #     # We haven't processed the base class yet. Need another pass.
            #     return None, None
            if METADATA_KEY not in info.metadata:
                continue

            # Each class depends on the set of attributes in its dataclass ancestors.
            self._api.add_plugin_dependency(make_wildcard_trigger(info.fullname))

            for name, data in info.metadata[METADATA_KEY]['fields'].items():
                field = PydanticModelField.deserialize(info, data, self._api)
                # (The following comment comes directly from the dataclasses plugin)
                # TODO: We shouldn't be performing type operations during the main
                #       semantic analysis pass, since some TypeInfo attributes might
                #       still be in flux. This should be performed in a later phase.
                field.expand_typevar_from_subtype(cls.info, self._api)
                found_fields[name] = field

                sym_node = cls.info.names.get(name)
                if sym_node and sym_node.node and not isinstance(sym_node.node, Var):
                    self._api.fail(
                        'BaseModel field may only be overridden by another field',
                        sym_node.node,
                    )
            # Collect ClassVars
            for name, data in info.metadata[METADATA_KEY]['class_vars'].items():
                found_class_vars[name] = PydanticModelClassVar.deserialize(data)

        # Second, collect fields and ClassVars belonging to the current class.
        current_field_names: set[str] = set()
        current_class_vars_names: set[str] = set()
        for stmt in self._get_assignment_statements_from_block(cls.defs):
            maybe_field = self.collect_field_or_class_var_from_stmt(stmt, model_config, found_class_vars)
            if maybe_field is None:
                continue

            lhs = stmt.lvalues[0]
            assert isinstance(lhs, NameExpr)  # collect_field_or_class_var_from_stmt guarantees this
            if isinstance(maybe_field, PydanticModelField):
                if is_root_model and lhs.name != 'root':
                    error_extra_fields_on_root_model(self._api, stmt)
                else:
                    current_field_names.add(lhs.name)
                    found_fields[lhs.name] = maybe_field
            elif isinstance(maybe_field, PydanticModelClassVar):
                current_class_vars_names.add(lhs.name)
                found_class_vars[lhs.name] = maybe_field

        return list(found_fields.values()), list(found_class_vars.values())

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="660">

---

The <SwmToken path="pydantic/mypy.py" pos="660:3:3" line-data="    def _get_assignment_statements_from_if_statement(self, stmt: IfStmt) -&gt; Iterator[AssignmentStmt]:">`_get_assignment_statements_from_if_statement`</SwmToken> function yields assignment statements from the branches of an if statement, recursively handling nested blocks.

```python
    def _get_assignment_statements_from_if_statement(self, stmt: IfStmt) -> Iterator[AssignmentStmt]:
        for body in stmt.body:
            if not body.is_unreachable:
                yield from self._get_assignment_statements_from_block(body)
        if stmt.else_body is not None and not stmt.else_body.is_unreachable:
            yield from self._get_assignment_statements_from_block(stmt.else_body)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="667">

---

The <SwmToken path="pydantic/mypy.py" pos="667:3:3" line-data="    def _get_assignment_statements_from_block(self, block: Block) -&gt; Iterator[AssignmentStmt]:">`_get_assignment_statements_from_block`</SwmToken> function yields assignment statements from a block, including those nested within if statements.

```python
    def _get_assignment_statements_from_block(self, block: Block) -> Iterator[AssignmentStmt]:
        for stmt in block.body:
            if isinstance(stmt, AssignmentStmt):
                yield stmt
            elif isinstance(stmt, IfStmt):
                yield from self._get_assignment_statements_from_if_statement(stmt)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="674">

---

The <SwmToken path="pydantic/mypy.py" pos="674:3:3" line-data="    def collect_field_or_class_var_from_stmt(  # noqa C901">`collect_field_or_class_var_from_stmt`</SwmToken> function analyzes an assignment statement to determine if it defines a Pydantic model field or class variable, returning the appropriate object or None.

```python
    def collect_field_or_class_var_from_stmt(  # noqa C901
        self, stmt: AssignmentStmt, model_config: ModelConfigData, class_vars: dict[str, PydanticModelClassVar]
    ) -> PydanticModelField | PydanticModelClassVar | None:
        """Get pydantic model field from statement.

        Args:
            stmt: The statement.
            model_config: Configuration settings for the model.
            class_vars: ClassVars already known to be defined on the model.

        Returns:
            A pydantic model field if it could find the field in statement. Otherwise, `None`.
        """
        cls = self._cls

        lhs = stmt.lvalues[0]
        if not isinstance(lhs, NameExpr) or not _fields.is_valid_field_name(lhs.name) or lhs.name == 'model_config':
            return None

        if not stmt.new_syntax:
            if (
                isinstance(stmt.rvalue, CallExpr)
                and isinstance(stmt.rvalue.callee, CallExpr)
                and isinstance(stmt.rvalue.callee.callee, NameExpr)
                and stmt.rvalue.callee.callee.fullname in DECORATOR_FULLNAMES
            ):
                # This is a (possibly-reused) validator or serializer, not a field
                # In particular, it looks something like: my_validator = validator('my_field')(f)
                # Eventually, we may want to attempt to respect model_config['ignored_types']
                return None

            if lhs.name in class_vars:
                # Class vars are not fields and are not required to be annotated
                return None

            # The assignment does not have an annotation, and it's not anything else we recognize
            error_untyped_fields(self._api, stmt)
            return None

        lhs = stmt.lvalues[0]
        if not isinstance(lhs, NameExpr):
            return None

        if not _fields.is_valid_field_name(lhs.name) or lhs.name == 'model_config':
            return None

        sym = cls.info.names.get(lhs.name)
        if sym is None:  # pragma: no cover
            # This is likely due to a star import (see the dataclasses plugin for a more detailed explanation)
            # This is the same logic used in the dataclasses plugin
            return None

        node = sym.node
        if isinstance(node, PlaceholderNode):  # pragma: no cover
            # See the PlaceholderNode docstring for more detail about how this can occur
            # Basically, it is an edge case when dealing with complex import logic

            # The dataclasses plugin now asserts this cannot happen, but I'd rather not error if it does..
            return None

        if isinstance(node, TypeAlias):
            self._api.fail(
                'Type aliases inside BaseModel definitions are not supported at runtime',
                node,
            )
            # Skip processing this node. This doesn't match the runtime behaviour,
            # but the only alternative would be to modify the SymbolTable,
            # and it's a little hairy to do that in a plugin.
            return None

        if not isinstance(node, Var):  # pragma: no cover
            # Don't know if this edge case still happens with the `is_valid_field` check above
            # but better safe than sorry

            # The dataclasses plugin now asserts this cannot happen, but I'd rather not error if it does..
            return None

        # x: ClassVar[int] is not a field
        if node.is_classvar:
            return PydanticModelClassVar(lhs.name)

        # x: InitVar[int] is not supported in BaseModel
        node_type = get_proper_type(node.type)
        if isinstance(node_type, Instance) and node_type.type.fullname == 'dataclasses.InitVar':
            self._api.fail(
                'InitVar is not supported in BaseModel',
                node,
            )

        has_default = self.get_has_default(stmt)
        strict = self.get_strict(stmt)

        if sym.type is None and node.is_final and node.is_inferred:
            # This follows the logic from the dataclasses plugin. The following comment is taken verbatim:
            #
            # This is a special case, assignment like x: Final = 42 is classified
            # annotated above, but mypy strips the `Final` turning it into x = 42.
            # We do not support inferred types in dataclasses, so we can try inferring
            # type for simple literals, and otherwise require an explicit type
            # argument for Final[...].
            typ = self._api.analyze_simple_literal_type(stmt.rvalue, is_final=True)
            if typ:
                node.type = typ
            else:
                self._api.fail(
                    'Need type argument for Final[...] with non-literal default in BaseModel',
                    stmt,
                )
                node.type = AnyType(TypeOfAny.from_error)

        if node.is_final and has_default:
            # TODO this path should be removed (see https://github.com/pydantic/pydantic/issues/11119)
            return PydanticModelClassVar(lhs.name)

        alias, has_dynamic_alias = self.get_alias_info(stmt)
        if (
            has_dynamic_alias
            and not (model_config.validate_by_name or model_config.populate_by_name)
            and self.plugin_config.warn_required_dynamic_aliases
        ):
            error_required_dynamic_aliases(self._api, stmt)
        is_frozen = self.is_field_frozen(stmt)

        init_type = self._infer_dataclass_attr_init_type(sym, lhs.name, stmt)
        return PydanticModelField(
            name=lhs.name,
            has_dynamic_alias=has_dynamic_alias,
            has_default=has_default,
            strict=strict,
            alias=alias,
            is_frozen=is_frozen,
            line=stmt.line,
            column=stmt.column,
            type=init_type,
            info=cls.info,
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="811">

---

The <SwmToken path="pydantic/mypy.py" pos="811:3:3" line-data="    def _infer_dataclass_attr_init_type(self, sym: SymbolTableNode, name: str, context: Context) -&gt; Type | None:">`_infer_dataclass_attr_init_type`</SwmToken> function infers the type of an attribute for use in the generated **init** method, possibly using the signature of a **set** method if present.

```python
    def _infer_dataclass_attr_init_type(self, sym: SymbolTableNode, name: str, context: Context) -> Type | None:
        """Infer __init__ argument type for an attribute.

        In particular, possibly use the signature of __set__.
        """
        default = sym.type
        if sym.implicit:
            return default
        t = get_proper_type(sym.type)

        # Perform a simple-minded inference from the signature of __set__, if present.
        # We can't use mypy.checkmember here, since this plugin runs before type checking.
        # We only support some basic scanerios here, which is hopefully sufficient for
        # the vast majority of use cases.
        if not isinstance(t, Instance):
            return default
        setter = t.type.get('__set__')
        if setter:
            if isinstance(setter.node, FuncDef):
                super_info = t.type.get_containing_type_info('__set__')
                assert super_info
                if setter.type:
                    setter_type = get_proper_type(map_type_from_supertype(setter.type, t.type, super_info))
                else:
                    return AnyType(TypeOfAny.unannotated)
                if isinstance(setter_type, CallableType) and setter_type.arg_kinds == [
                    ARG_POS,
                    ARG_POS,
                    ARG_POS,
                ]:
                    return expand_type_by_instance(setter_type.arg_types[2], t)
                else:
                    self._api.fail(f'Unsupported signature for "__set__" in "{t.type.name}"', context)
            else:
                self._api.fail(f'Unsupported "__set__" in "{t.type.name}"', context)

        return default

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="849">

---

The <SwmToken path="pydantic/mypy.py" pos="849:3:3" line-data="    def add_initializer(">`add_initializer`</SwmToken> function generates and adds a <SwmToken path="pydantic/mypy.py" pos="852:8:10" line-data="        &quot;&quot;&quot;Adds a fields-aware `__init__` method to the class.">`fields-aware`</SwmToken> **init** method to the model class, with type annotations determined by plugin settings and model configuration.

```python
    def add_initializer(
        self, fields: list[PydanticModelField], config: ModelConfigData, is_settings: bool, is_root_model: bool
    ) -> None:
        """Adds a fields-aware `__init__` method to the class.

        The added `__init__` will be annotated with types vs. all `Any` depending on the plugin settings.
        """
        if '__init__' in self._cls.info.names and not self._cls.info.names['__init__'].plugin_generated:
            return  # Don't generate an __init__ if one already exists

        typed = self.plugin_config.init_typed
        model_strict = bool(config.strict)
        use_alias = not (config.validate_by_name or config.populate_by_name) and config.validate_by_alias is not False
        requires_dynamic_aliases = bool(config.has_alias_generator and not config.validate_by_name)
        args = self.get_field_arguments(
            fields,
            typed=typed,
            model_strict=model_strict,
            requires_dynamic_aliases=requires_dynamic_aliases,
            use_alias=use_alias,
            is_settings=is_settings,
            is_root_model=is_root_model,
            force_typevars_invariant=True,
        )

        if is_settings:
            base_settings_node = self._api.lookup_fully_qualified(BASESETTINGS_FULLNAME).node
            assert isinstance(base_settings_node, TypeInfo)
            if '__init__' in base_settings_node.names:
                base_settings_init_node = base_settings_node.names['__init__'].node
                assert isinstance(base_settings_init_node, FuncDef)
                if base_settings_init_node is not None and base_settings_init_node.type is not None:
                    func_type = base_settings_init_node.type
                    assert isinstance(func_type, CallableType)
                    for arg_idx, arg_name in enumerate(func_type.arg_names):
                        if arg_name is None or arg_name.startswith('__') or not arg_name.startswith('_'):
                            continue
                        analyzed_variable_type = self._api.anal_type(func_type.arg_types[arg_idx])
                        if analyzed_variable_type is not None and arg_name == '_cli_settings_source':
                            # _cli_settings_source is defined as CliSettingsSource[Any], and as such
                            # the Any causes issues with --disallow-any-explicit. As a workaround, change
                            # the Any type (as if CliSettingsSource was left unparameterized):
                            analyzed_variable_type = analyzed_variable_type.accept(
                                ChangeExplicitTypeOfAny(TypeOfAny.from_omitted_generics)
                            )
                        variable = Var(arg_name, analyzed_variable_type)
                        args.append(Argument(variable, analyzed_variable_type, None, ARG_OPT))

        if not self.should_init_forbid_extra(fields, config):
            var = Var('kwargs')
            args.append(Argument(var, AnyType(TypeOfAny.explicit), None, ARG_STAR2))

        add_method(self._api, self._cls, '__init__', args=args, return_type=NoneType())

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="903">

---

The <SwmToken path="pydantic/mypy.py" pos="903:3:3" line-data="    def add_model_construct_method(">`add_model_construct_method`</SwmToken> function generates and adds a fully typed <SwmToken path="pydantic/mypy.py" pos="910:13:13" line-data="        &quot;&quot;&quot;Adds a fully typed `model_construct` classmethod to the class.">`model_construct`</SwmToken> classmethod to the model class, using field names and handling root models and settings appropriately.

```python
    def add_model_construct_method(
        self,
        fields: list[PydanticModelField],
        config: ModelConfigData,
        is_settings: bool,
        is_root_model: bool,
    ) -> None:
        """Adds a fully typed `model_construct` classmethod to the class.

        Similar to the fields-aware __init__ method, but always uses the field names (not aliases),
        and does not treat settings fields as optional.
        """
        set_str = self._api.named_type(f'{BUILTINS_NAME}.set', [self._api.named_type(f'{BUILTINS_NAME}.str')])
        optional_set_str = UnionType([set_str, NoneType()])
        fields_set_argument = Argument(Var('_fields_set', optional_set_str), optional_set_str, None, ARG_OPT)
        with state.strict_optional_set(self._api.options.strict_optional):
            args = self.get_field_arguments(
                fields,
                typed=True,
                model_strict=bool(config.strict),
                requires_dynamic_aliases=False,
                use_alias=False,
                is_settings=is_settings,
                is_root_model=is_root_model,
            )
        if not self.should_init_forbid_extra(fields, config):
            var = Var('kwargs')
            args.append(Argument(var, AnyType(TypeOfAny.explicit), None, ARG_STAR2))

        args = args + [fields_set_argument] if is_root_model else [fields_set_argument] + args

        add_method(
            self._api,
            self._cls,
            'model_construct',
            args=args,
            return_type=fill_typevars(self._cls.info),
            is_classmethod=True,
        )

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="943">

---

The <SwmToken path="pydantic/mypy.py" pos="943:3:3" line-data="    def set_frozen(self, fields: list[PydanticModelField], api: SemanticAnalyzerPluginInterface, frozen: bool) -&gt; None:">`set_frozen`</SwmToken> function marks all fields as properties if the model or field is frozen, so that attempts to set them trigger mypy errors, following the approach used by dataclasses and attrs plugins.

```python
    def set_frozen(self, fields: list[PydanticModelField], api: SemanticAnalyzerPluginInterface, frozen: bool) -> None:
        """Marks all fields as properties so that attempts to set them trigger mypy errors.

        This is the same approach used by the attrs and dataclasses plugins.
        """
        info = self._cls.info
        for field in fields:
            sym_node = info.names.get(field.name)
            if sym_node is not None:
                var = sym_node.node
                if isinstance(var, Var):
                    var.is_property = frozen or field.is_frozen
                elif isinstance(var, PlaceholderNode) and not self._api.final_iteration:
                    # See https://github.com/pydantic/pydantic/issues/5191 to hit this branch for test coverage
                    self._api.defer()
                else:  # pragma: no cover
                    # I don't know whether it's possible to hit this branch, but I've added it for safety
                    try:
                        var_str = str(var)
                    except TypeError:
                        # This happens for PlaceholderNode; perhaps it will happen for other types in the future..
                        var_str = repr(var)
                    detail = f'sym_node.node: {var_str} (of type {var.__class__})'
                    error_unexpected_behavior(detail, self._api, self._cls)
            else:
                var = field.to_var(info, api, use_alias=False)
                var.info = info
                var.is_property = frozen
                var._fullname = info.fullname + '.' + var.name
                info.names[var.name] = SymbolTableNode(MDEF, var)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="974">

---

The <SwmToken path="pydantic/mypy.py" pos="974:3:3" line-data="    def get_config_update(self, name: str, arg: Expression, lax_extra: bool = False) -&gt; ModelConfigData | None:">`get_config_update`</SwmToken> function determines the configuration update for a single keyword argument in a <SwmToken path="pydantic/mypy.py" pos="975:26:26" line-data="        &quot;&quot;&quot;Determines the config update due to a single kwarg in the ConfigDict definition.">`ConfigDict`</SwmToken> definition, returning a <SwmToken path="pydantic/mypy.py" pos="974:32:32" line-data="    def get_config_update(self, name: str, arg: Expression, lax_extra: bool = False) -&gt; ModelConfigData | None:">`ModelConfigData`</SwmToken> object or None.

```python
    def get_config_update(self, name: str, arg: Expression, lax_extra: bool = False) -> ModelConfigData | None:
        """Determines the config update due to a single kwarg in the ConfigDict definition.

        Warns if a tracked config attribute is set to a value the plugin doesn't know how to interpret (e.g., an int)
        """
        if name not in self.tracked_config_fields:
            return None
        if name == 'extra':
            if isinstance(arg, StrExpr):
                forbid_extra = arg.value == 'forbid'
            elif isinstance(arg, MemberExpr):
                forbid_extra = arg.name == 'forbid'
            else:
                if not lax_extra:
                    # Only emit an error for other types of `arg` (e.g., `NameExpr`, `ConditionalExpr`, etc.) when
                    # reading from a config class, etc. If a ConfigDict is used, then we don't want to emit an error
                    # because you'll get type checking from the ConfigDict itself.
                    #
                    # It would be nice if we could introspect the types better otherwise, but I don't know what the API
                    # is to evaluate an expr into its type and then check if that type is compatible with the expected
                    # type. Note that you can still get proper type checking via: `model_config = ConfigDict(...)`, just
                    # if you don't use an explicit string, the plugin won't be able to infer whether extra is forbidden.
                    error_invalid_config_value(name, self._api, arg)
                return None
            return ModelConfigData(forbid_extra=forbid_extra)
        if name == 'alias_generator':
            has_alias_generator = True
            if isinstance(arg, NameExpr) and arg.fullname == 'builtins.None':
                has_alias_generator = False
            return ModelConfigData(has_alias_generator=has_alias_generator)
        if isinstance(arg, NameExpr) and arg.fullname in ('builtins.True', 'builtins.False'):
            return ModelConfigData(**{name: arg.fullname == 'builtins.True'})
        error_invalid_config_value(name, self._api, arg)
        return None

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1010">

---

The static method <SwmToken path="pydantic/mypy.py" pos="1010:3:3" line-data="    def get_has_default(stmt: AssignmentStmt) -&gt; bool:">`get_has_default`</SwmToken> checks if a field defined in an assignment statement has a default value.

```python
    def get_has_default(stmt: AssignmentStmt) -> bool:
        """Returns a boolean indicating whether the field defined in `stmt` is a required field."""
        expr = stmt.rvalue
        if isinstance(expr, TempNode):
            # TempNode means annotation-only, so has no default
            return False
        if isinstance(expr, CallExpr) and isinstance(expr.callee, RefExpr) and expr.callee.fullname == FIELD_FULLNAME:
            # The "default value" is a call to `Field`; at this point, the field has a default if and only if:
            # * there is a positional argument that is not `...`
            # * there is a keyword argument named "default" that is not `...`
            # * there is a "default_factory" that is not `None`
            for arg, name in zip(expr.args, expr.arg_names):
                # If name is None, then this arg is the default because it is the only positional argument.
                if name is None or name == 'default':
                    return arg.__class__ is not EllipsisExpr
                if name == 'default_factory':
                    return not (isinstance(arg, NameExpr) and arg.fullname == 'builtins.None')
            return False
        # Has no default if the "default value" is Ellipsis (i.e., `field_name: Annotation = ...`)
        return not isinstance(expr, EllipsisExpr)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1032">

---

The static method <SwmToken path="pydantic/mypy.py" pos="1032:3:3" line-data="    def get_strict(stmt: AssignmentStmt) -&gt; bool | None:">`get_strict`</SwmToken> checks if a field defined in an assignment statement has the strict flag set.

```python
    def get_strict(stmt: AssignmentStmt) -> bool | None:
        """Returns a the `strict` value of a field if defined, otherwise `None`."""
        expr = stmt.rvalue
        if isinstance(expr, CallExpr) and isinstance(expr.callee, RefExpr) and expr.callee.fullname == FIELD_FULLNAME:
            for arg, name in zip(expr.args, expr.arg_names):
                if name != 'strict':
                    continue
                if isinstance(arg, NameExpr):
                    if arg.fullname == 'builtins.True':
                        return True
                    elif arg.fullname == 'builtins.False':
                        return False
                return None
        return None

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1048">

---

The static method <SwmToken path="pydantic/mypy.py" pos="1048:3:3" line-data="    def get_alias_info(stmt: AssignmentStmt) -&gt; tuple[str | None, bool]:">`get_alias_info`</SwmToken> extracts alias information from a field assignment, returning the alias and whether it is dynamic.

```python
    def get_alias_info(stmt: AssignmentStmt) -> tuple[str | None, bool]:
        """Returns a pair (alias, has_dynamic_alias), extracted from the declaration of the field defined in `stmt`.

        `has_dynamic_alias` is True if and only if an alias is provided, but not as a string literal.
        If `has_dynamic_alias` is True, `alias` will be None.
        """
        expr = stmt.rvalue
        if isinstance(expr, TempNode):
            # TempNode means annotation-only
            return None, False

        if not (
            isinstance(expr, CallExpr) and isinstance(expr.callee, RefExpr) and expr.callee.fullname == FIELD_FULLNAME
        ):
            # Assigned value is not a call to pydantic.fields.Field
            return None, False

        if 'validation_alias' in expr.arg_names:
            arg = expr.args[expr.arg_names.index('validation_alias')]
        elif 'alias' in expr.arg_names:
            arg = expr.args[expr.arg_names.index('alias')]
        else:
            return None, False

        if isinstance(arg, StrExpr):
            return arg.value, False
        else:
            return None, True

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1078">

---

The static method <SwmToken path="pydantic/mypy.py" pos="1078:3:3" line-data="    def is_field_frozen(stmt: AssignmentStmt) -&gt; bool:">`is_field_frozen`</SwmToken> determines if a field is declared as frozen in its assignment statement.

```python
    def is_field_frozen(stmt: AssignmentStmt) -> bool:
        """Returns whether the field is frozen, extracted from the declaration of the field defined in `stmt`.

        Note that this is only whether the field was declared to be frozen in a `<field_name> = Field(frozen=True)`
        sense; this does not determine whether the field is frozen because the entire model is frozen; that is
        handled separately.
        """
        expr = stmt.rvalue
        if isinstance(expr, TempNode):
            # TempNode means annotation-only
            return False

        if not (
            isinstance(expr, CallExpr) and isinstance(expr.callee, RefExpr) and expr.callee.fullname == FIELD_FULLNAME
        ):
            # Assigned value is not a call to pydantic.fields.Field
            return False

        for i, arg_name in enumerate(expr.arg_names):
            if arg_name == 'frozen':
                arg = expr.args[i]
                return isinstance(arg, NameExpr) and arg.fullname == 'builtins.True'
        return False

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1102">

---

The <SwmToken path="pydantic/mypy.py" pos="1102:3:3" line-data="    def get_field_arguments(">`get_field_arguments`</SwmToken> function constructs a list of mypy Argument instances for use in generated method signatures, based on the model's fields and configuration.

```python
    def get_field_arguments(
        self,
        fields: list[PydanticModelField],
        typed: bool,
        model_strict: bool,
        use_alias: bool,
        requires_dynamic_aliases: bool,
        is_settings: bool,
        is_root_model: bool,
        force_typevars_invariant: bool = False,
    ) -> list[Argument]:
        """Helper function used during the construction of the `__init__` and `model_construct` method signatures.

        Returns a list of mypy Argument instances for use in the generated signatures.
        """
        info = self._cls.info
        arguments = [
            field.to_argument(
                info,
                typed=typed,
                model_strict=model_strict,
                force_optional=requires_dynamic_aliases or is_settings,
                use_alias=use_alias,
                api=self._api,
                force_typevars_invariant=force_typevars_invariant,
                is_root_model_root=is_root_model and field.name == 'root',
            )
            for field in fields
            if not (use_alias and field.has_dynamic_alias)
        ]
        return arguments

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1134">

---

The <SwmToken path="pydantic/mypy.py" pos="1134:3:3" line-data="    def should_init_forbid_extra(self, fields: list[PydanticModelField], config: ModelConfigData) -&gt; bool:">`should_init_forbid_extra`</SwmToken> function determines whether the generated **init** method should include a \*\*kwargs parameter, based on config and field analysis.

```python
    def should_init_forbid_extra(self, fields: list[PydanticModelField], config: ModelConfigData) -> bool:
        """Indicates whether the generated `__init__` should get a `**kwargs` at the end of its signature.

        We disallow arbitrary kwargs if the extra config setting is "forbid", or if the plugin config says to,
        *unless* a required dynamic alias is present (since then we can't determine a valid signature).
        """
        if not (config.validate_by_name or config.populate_by_name):
            if self.is_dynamic_alias_present(fields, bool(config.has_alias_generator)):
                return False
        if config.forbid_extra:
            return True
        return self.plugin_config.init_forbid_extra

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/mypy.py" line="1148">

---

The static method <SwmToken path="pydantic/mypy.py" pos="1148:3:3" line-data="    def is_dynamic_alias_present(fields: list[PydanticModelField], has_alias_generator: bool) -&gt; bool:">`is_dynamic_alias_present`</SwmToken> checks if any fields on the model have a dynamic alias, which affects signature generation.

```python
    def is_dynamic_alias_present(fields: list[PydanticModelField], has_alias_generator: bool) -> bool:
        """Returns whether any fields on the model have a "dynamic alias", i.e., an alias that cannot be
        determined during static analysis.
        """
        for field in fields:
            if field.has_dynamic_alias:
                return True
        if has_alias_generator:
            for field in fields:
                if field.alias is None:
                    return True
        return False
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> Usage

<SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> is instantiated within the mypy plugin's class maker callback function. This callback receives a context object representing a class definition, and the transformer is created with this class along with additional context such as the reason for transformation, the mypy API, and plugin configuration.

After instantiation, the transform method of <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> is called to apply the necessary modifications or validations to the Pydantic model class. This process integrates Pydantic's data validation features with mypy's static type checking.

This usage pattern shows how <SwmToken path="pydantic/mypy.py" pos="161:5:5" line-data="        transformer = PydanticModelTransformer(ctx.cls, ctx.reason, ctx.api, self.plugin_config)">`PydanticModelTransformer`</SwmToken> acts as a bridge between Pydantic models and the mypy type system, enabling enhanced type validation and model transformation during static analysis.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
