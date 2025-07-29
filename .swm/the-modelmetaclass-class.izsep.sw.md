---
title: The ModelMetaclass class
---
# What is <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken>

This document covers:

1. What <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken> is and its role in Pydantic.
2. All variables and functions defined in <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken>, with explanations and code references.

# What is <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken>

<SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken> is a metaclass used for constructing Pydantic models. It is responsible for orchestrating the creation of model classes, handling class-level configuration, collecting fields, managing private attributes, and supporting generics. By customizing the class creation process, <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken> enables Pydantic's advanced features such as data validation, field management, and integration with Python's type system.

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="80">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="80:3:3" line-data="    def __new__(">`__new__`</SwmToken> function is the core of <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken>. It customizes the creation of new model classes by collecting annotations, merging configuration from base classes, handling private attributes, setting up decorators, managing generics, and initializing various class-level attributes required for Pydantic models.

```python
    def __new__(
        mcs,
        cls_name: str,
        bases: tuple[type[Any], ...],
        namespace: dict[str, Any],
        __pydantic_generic_metadata__: PydanticGenericMetadata | None = None,
        __pydantic_reset_parent_namespace__: bool = True,
        _create_model_module: str | None = None,
        **kwargs: Any,
    ) -> type:
        """Metaclass for creating Pydantic models.

        Args:
            cls_name: The name of the class to be created.
            bases: The base classes of the class to be created.
            namespace: The attribute dictionary of the class to be created.
            __pydantic_generic_metadata__: Metadata for generic models.
            __pydantic_reset_parent_namespace__: Reset parent namespace.
            _create_model_module: The module of the class to be created, if created by `create_model`.
            **kwargs: Catch-all for any other keyword arguments.

        Returns:
            The new class created by the metaclass.
        """
        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact
        # that `BaseModel` itself won't have any bases, but any subclass of it will, to determine whether the `__new__`
        # call we're in the middle of is for the `BaseModel` class.
        if bases:
            raw_annotations: dict[str, Any]
            if sys.version_info >= (3, 14):
                if (
                    '__annotations__' in namespace
                ):  # `from __future__ import annotations` was used in the model's module
                    raw_annotations = namespace['__annotations__']
                else:
                    # See https://docs.python.org/3.14/library/annotationlib.html#using-annotations-in-a-metaclass:
                    from annotationlib import Format, call_annotate_function, get_annotate_from_class_namespace

                    if annotate := get_annotate_from_class_namespace(namespace):
                        raw_annotations = call_annotate_function(annotate, format=Format.FORWARDREF)
                    else:
                        raw_annotations = {}
            else:
                raw_annotations = namespace.get('__annotations__', {})

            base_field_names, class_vars, base_private_attributes = mcs._collect_bases_data(bases)

            config_wrapper = ConfigWrapper.for_model(bases, namespace, raw_annotations, kwargs)
            namespace['model_config'] = config_wrapper.config_dict
            private_attributes = inspect_namespace(
                namespace, raw_annotations, config_wrapper.ignored_types, class_vars, base_field_names
            )
            if private_attributes or base_private_attributes:
                original_model_post_init = get_model_post_init(namespace, bases)
                if original_model_post_init is not None:
                    # if there are private_attributes and a model_post_init function, we handle both

                    @wraps(original_model_post_init)
                    def wrapped_model_post_init(self: BaseModel, context: Any, /) -> None:
                        """We need to both initialize private attributes and call the user-defined model_post_init
                        method.
                        """
                        init_private_attributes(self, context)
                        original_model_post_init(self, context)

                    namespace['model_post_init'] = wrapped_model_post_init
                else:
                    namespace['model_post_init'] = init_private_attributes

            namespace['__class_vars__'] = class_vars
            namespace['__private_attributes__'] = {**base_private_attributes, **private_attributes}

            cls = cast('type[BaseModel]', super().__new__(mcs, cls_name, bases, namespace, **kwargs))
            BaseModel_ = import_cached_base_model()

            mro = cls.__mro__
            if Generic in mro and mro.index(Generic) < mro.index(BaseModel_):
                warnings.warn(
                    GenericBeforeBaseModelWarning(
                        'Classes should inherit from `BaseModel` before generic classes (e.g. `typing.Generic[T]`) '
                        'for pydantic generics to work properly.'
                    ),
                    stacklevel=2,
                )

            cls.__pydantic_custom_init__ = not getattr(cls.__init__, '__pydantic_base_init__', False)
            cls.__pydantic_post_init__ = (
                None if cls.model_post_init is BaseModel_.model_post_init else 'model_post_init'
            )

            cls.__pydantic_setattr_handlers__ = {}

            cls.__pydantic_decorators__ = DecoratorInfos.build(cls)
            cls.__pydantic_decorators__.update_from_config(config_wrapper)

            # Use the getattr below to grab the __parameters__ from the `typing.Generic` parent class
            if __pydantic_generic_metadata__:
                cls.__pydantic_generic_metadata__ = __pydantic_generic_metadata__
            else:
                parent_parameters = getattr(cls, '__pydantic_generic_metadata__', {}).get('parameters', ())
                parameters = getattr(cls, '__parameters__', None) or parent_parameters
                if parameters and parent_parameters and not all(x in parameters for x in parent_parameters):
                    from ..root_model import RootModelRootType

                    missing_parameters = tuple(x for x in parameters if x not in parent_parameters)
                    if RootModelRootType in parent_parameters and RootModelRootType not in parameters:
                        # This is a special case where the user has subclassed `RootModel`, but has not parametrized
                        # RootModel with the generic type identifiers being used. Ex:
                        # class MyModel(RootModel, Generic[T]):
                        #    root: T
                        # Should instead just be:
                        # class MyModel(RootModel[T]):
                        #   root: T
                        parameters_str = ', '.join([x.__name__ for x in missing_parameters])
                        error_message = (
                            f'{cls.__name__} is a subclass of `RootModel`, but does not include the generic type identifier(s) '
                            f'{parameters_str} in its parameters. '
                            f'You should parametrize RootModel directly, e.g., `class {cls.__name__}(RootModel[{parameters_str}]): ...`.'
                        )
                    else:
                        combined_parameters = parent_parameters + missing_parameters
                        parameters_str = ', '.join([str(x) for x in combined_parameters])
                        generic_type_label = f'typing.Generic[{parameters_str}]'
                        error_message = (
                            f'All parameters must be present on typing.Generic;'
                            f' you should inherit from {generic_type_label}.'
                        )
                        if Generic not in bases:  # pragma: no cover
                            # We raise an error here not because it is desirable, but because some cases are mishandled.
                            # It would be nice to remove this error and still have things behave as expected, it's just
                            # challenging because we are using a custom `__class_getitem__` to parametrize generic models,
                            # and not returning a typing._GenericAlias from it.
                            bases_str = ', '.join([x.__name__ for x in bases] + [generic_type_label])
                            error_message += (
                                f' Note: `typing.Generic` must go last: `class {cls.__name__}({bases_str}): ...`)'
                            )
                    raise TypeError(error_message)

                cls.__pydantic_generic_metadata__ = {
                    'origin': None,
                    'args': (),
                    'parameters': parameters,
                }

            cls.__pydantic_complete__ = False  # Ensure this specific class gets completed

            # preserve `__set_name__` protocol defined in https://peps.python.org/pep-0487
            # for attributes not in `new_namespace` (e.g. private attributes)
            for name, obj in private_attributes.items():
                obj.__set_name__(cls, name)

            if __pydantic_reset_parent_namespace__:
                cls.__pydantic_parent_namespace__ = build_lenient_weakvaluedict(parent_frame_namespace())
            parent_namespace: dict[str, Any] | None = getattr(cls, '__pydantic_parent_namespace__', None)
            if isinstance(parent_namespace, dict):
                parent_namespace = unpack_lenient_weakvaluedict(parent_namespace)

            ns_resolver = NsResolver(parent_namespace=parent_namespace)

            set_model_fields(cls, config_wrapper=config_wrapper, ns_resolver=ns_resolver)

            # This is also set in `complete_model_class()`, after schema gen because they are recreated.
            # We set them here as well for backwards compatibility:
            cls.__pydantic_computed_fields__ = {
                k: v.info for k, v in cls.__pydantic_decorators__.computed_fields.items()
            }

            if config_wrapper.defer_build:
                set_model_mocks(cls)
            else:
                # Any operation that requires accessing the field infos instances should be put inside
                # `complete_model_class()`:
                complete_model_class(
                    cls,
                    config_wrapper,
                    ns_resolver,
                    raise_errors=False,
                    create_model_module=_create_model_module,
                )

            if config_wrapper.frozen and '__hash__' not in namespace:
                set_default_hash_func(cls, bases)

            # using super(cls, cls) on the next line ensures we only call the parent class's __pydantic_init_subclass__
            # I believe the `type: ignore` is only necessary because mypy doesn't realize that this code branch is
            # only hit for _proper_ subclasses of BaseModel
            super(cls, cls).__pydantic_init_subclass__(**kwargs)  # type: ignore[misc]
            return cls
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="281">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="281:3:3" line-data="        def __getattr__(self, item: str) -&gt; Any:">`__getattr__`</SwmToken> function allows dynamic access to private attributes on the class. If the requested attribute is found in the class's <SwmToken path="pydantic/_internal/_model_construction.py" pos="283:12:12" line-data="            private_attributes = self.__dict__.get(&#39;__private_attributes__&#39;)">`__private_attributes__`</SwmToken> dictionary, it is returned; otherwise, an <SwmToken path="pydantic/_internal/_model_construction.py" pos="286:3:3" line-data="            raise AttributeError(item)">`AttributeError`</SwmToken> is raised.

```python
        def __getattr__(self, item: str) -> Any:
            """This is necessary to keep attribute access working for class attribute access."""
            private_attributes = self.__dict__.get('__private_attributes__')
            if private_attributes and item in private_attributes:
                return private_attributes[item]
            raise AttributeError(item)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="289">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="289:3:3" line-data="    def __prepare__(cls, *args: Any, **kwargs: Any) -&gt; dict[str, object]:">`__prepare__`</SwmToken> class method returns a custom dictionary (<SwmToken path="pydantic/_internal/_model_construction.py" pos="290:3:3" line-data="        return _ModelNamespaceDict()">`_ModelNamespaceDict`</SwmToken>) for the class namespace during class creation. This enables interception of attribute setting and can warn about decorator overrides.

```python
    def __prepare__(cls, *args: Any, **kwargs: Any) -> dict[str, object]:
        return _ModelNamespaceDict()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="292">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="292:3:3" line-data="    def __instancecheck__(self, instance: Any) -&gt; bool:">`__instancecheck__`</SwmToken> function customizes the behavior of <SwmToken path="pydantic/_internal/_model_construction.py" pos="234:3:3" line-data="            if isinstance(parent_namespace, dict):">`isinstance`</SwmToken> checks for classes using <SwmToken path="pydantic/_internal/_model_construction.py" pos="104:6:6" line-data="        # Note `ModelMetaclass` refers to `BaseModel`, but is also used to *create* `BaseModel`, so we rely on the fact">`ModelMetaclass`</SwmToken>. It ensures that only objects with the <SwmToken path="pydantic/_internal/_model_construction.py" pos="297:9:9" line-data="        return hasattr(instance, &#39;__pydantic_decorators__&#39;) and super().__instancecheck__(instance)">`__pydantic_decorators__`</SwmToken> attribute are considered instances, avoiding unnecessary calls to the ABC machinery.

```python
    def __instancecheck__(self, instance: Any) -> bool:
        """Avoid calling ABC _abc_instancecheck unless we're pretty sure.

        See #3829 and python/cpython#92810
        """
        return hasattr(instance, '__pydantic_decorators__') and super().__instancecheck__(instance)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="299">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="299:3:3" line-data="    def __subclasscheck__(self, subclass: type[Any]) -&gt; bool:">`__subclasscheck__`</SwmToken> function customizes the behavior of <SwmToken path="pydantic/_internal/_model_construction.py" pos="314:3:3" line-data="            if issubclass(base, BaseModel) and base is not BaseModel:">`issubclass`</SwmToken> checks. It only returns True for subclasses that have the <SwmToken path="pydantic/_internal/_model_construction.py" pos="304:9:9" line-data="        return hasattr(subclass, &#39;__pydantic_decorators__&#39;) and super().__subclasscheck__(subclass)">`__pydantic_decorators__`</SwmToken> attribute, similar to <SwmToken path="pydantic/_internal/_model_construction.py" pos="292:3:3" line-data="    def __instancecheck__(self, instance: Any) -&gt; bool:">`__instancecheck__`</SwmToken>.

```python
    def __subclasscheck__(self, subclass: type[Any]) -> bool:
        """Avoid calling ABC _abc_subclasscheck unless we're pretty sure.

        See #3829 and python/cpython#92810
        """
        return hasattr(subclass, '__pydantic_decorators__') and super().__subclasscheck__(subclass)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="307">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="307:3:3" line-data="    def _collect_bases_data(bases: tuple[type[Any], ...]) -&gt; tuple[set[str], set[str], dict[str, ModelPrivateAttr]]:">`_collect_bases_data`</SwmToken> static method gathers field names, class variables, and private attributes from all base classes that are subclasses of <SwmToken path="pydantic/_internal/_model_construction.py" pos="308:1:1" line-data="        BaseModel = import_cached_base_model()">`BaseModel`</SwmToken>. This is used during class creation to inherit and merge relevant information from parent models.

```python
    def _collect_bases_data(bases: tuple[type[Any], ...]) -> tuple[set[str], set[str], dict[str, ModelPrivateAttr]]:
        BaseModel = import_cached_base_model()

        field_names: set[str] = set()
        class_vars: set[str] = set()
        private_attributes: dict[str, ModelPrivateAttr] = {}
        for base in bases:
            if issubclass(base, BaseModel) and base is not BaseModel:
                # model_fields might not be defined yet in the case of generics, so we use getattr here:
                field_names.update(getattr(base, '__pydantic_fields__', {}).keys())
                class_vars.update(base.__class_vars__)
                private_attributes.update(base.__private_attributes__)
        return field_names, class_vars, private_attributes
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="323">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="323:3:3" line-data="    def __fields__(self) -&gt; dict[str, FieldInfo]:">`__fields__`</SwmToken> property is a deprecated way to access the model's fields. It emits a warning and returns the <SwmToken path="pydantic/_internal/_model_construction.py" pos="329:9:9" line-data="        return getattr(self, &#39;__pydantic_fields__&#39;, {})">`__pydantic_fields__`</SwmToken> attribute if present.

```python
    def __fields__(self) -> dict[str, FieldInfo]:
        warnings.warn(
            'The `__fields__` attribute is deprecated, use `model_fields` instead.',
            PydanticDeprecatedSince20,
            stacklevel=2,
        )
        return getattr(self, '__pydantic_fields__', {})
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="332">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="332:3:3" line-data="    def __pydantic_fields_complete__(self) -&gt; bool:">`__pydantic_fields_complete__`</SwmToken> property indicates whether all fields for the model have been successfully collected and their type hints resolved. It checks the completeness of all field information.

```python
    def __pydantic_fields_complete__(self) -> bool:
        """Whether the fields where successfully collected (i.e. type hints were successfully resolves).

        This is a private attribute, not meant to be used outside Pydantic.
        """
        if not hasattr(self, '__pydantic_fields__'):
            return False

        field_infos = cast('dict[str, FieldInfo]', self.__pydantic_fields__)  # pyright: ignore[reportAttributeAccessIssue]

        return all(field_info._complete for field_info in field_infos.values())
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_model_construction.py" line="344">

---

The <SwmToken path="pydantic/_internal/_model_construction.py" pos="344:3:3" line-data="    def __dir__(self) -&gt; list[str]:">`__dir__`</SwmToken> method customizes the list of attributes returned by the `dir()` function. It removes the deprecated <SwmToken path="pydantic/_internal/_model_construction.py" pos="346:4:4" line-data="        if &#39;__fields__&#39; in attributes:">`__fields__`</SwmToken> attribute from the list if present.

```python
    def __dir__(self) -> list[str]:
        attributes = list(super().__dir__())
        if '__fields__' in attributes:
            attributes.remove('__fields__')
        return attributes
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
