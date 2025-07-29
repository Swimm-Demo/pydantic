---
title: The DecoratorInfos class
---
# What is <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken>

This document covers the <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> class in <SwmPath>[pydantic/\_internal/\_decorators.py](pydantic/_internal/_decorators.py)</SwmPath>. It will address:

1. What <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is and its purpose
2. The variables defined in <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken>
3. The functions implemented in <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken>

<SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is a class that acts as a mapping between names in a class's namespace and their associated decorator metadata. It is designed to collect and organize information about various decorators (such as validators, serializers, and computed fields) that are defined on a Pydantic model or dataclass. This mapping is keyed by the function or attribute name in the class, not by the field name. <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is used internally to gather, manage, and update decorator-related metadata, supporting features like inheritance and configuration updates.

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="421">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="421:1:1" line-data="    validators: dict[str, Decorator[ValidatorDecoratorInfo]] = field(default_factory=dict)">`validators`</SwmToken> is a dictionary that maps function or attribute names to their corresponding Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="421:11:11" line-data="    validators: dict[str, Decorator[ValidatorDecoratorInfo]] = field(default_factory=dict)">`ValidatorDecoratorInfo`</SwmToken>. This is used to store information about all <SwmToken path="pydantic/_internal/_decorators.py" pos="36:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@validator&#39;.">`@validator`</SwmToken> decorators found in the class.

```python
    validators: dict[str, Decorator[ValidatorDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="422">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="422:1:1" line-data="    field_validators: dict[str, Decorator[FieldValidatorDecoratorInfo]] = field(default_factory=dict)">`field_validators`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="422:11:11" line-data="    field_validators: dict[str, Decorator[FieldValidatorDecoratorInfo]] = field(default_factory=dict)">`FieldValidatorDecoratorInfo`</SwmToken>. It holds metadata for all <SwmToken path="pydantic/_internal/_decorators.py" pos="60:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@field_validator&#39;.">`@field_validator`</SwmToken> decorators defined in the class.

```python
    field_validators: dict[str, Decorator[FieldValidatorDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="423">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="423:1:1" line-data="    root_validators: dict[str, Decorator[RootValidatorDecoratorInfo]] = field(default_factory=dict)">`root_validators`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="423:11:11" line-data="    root_validators: dict[str, Decorator[RootValidatorDecoratorInfo]] = field(default_factory=dict)">`RootValidatorDecoratorInfo`</SwmToken>. This stores metadata for all <SwmToken path="pydantic/_internal/_decorators.py" pos="83:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@root_validator&#39;.">`@root_validator`</SwmToken> decorators in the class.

```python
    root_validators: dict[str, Decorator[RootValidatorDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="424">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="424:1:1" line-data="    field_serializers: dict[str, Decorator[FieldSerializerDecoratorInfo]] = field(default_factory=dict)">`field_serializers`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="424:11:11" line-data="    field_serializers: dict[str, Decorator[FieldSerializerDecoratorInfo]] = field(default_factory=dict)">`FieldSerializerDecoratorInfo`</SwmToken>. It is used to keep track of all <SwmToken path="pydantic/_internal/_decorators.py" pos="97:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@field_serializer&#39;.">`@field_serializer`</SwmToken> decorators present in the class.

```python
    field_serializers: dict[str, Decorator[FieldSerializerDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="425">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="425:1:1" line-data="    model_serializers: dict[str, Decorator[ModelSerializerDecoratorInfo]] = field(default_factory=dict)">`model_serializers`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="425:11:11" line-data="    model_serializers: dict[str, Decorator[ModelSerializerDecoratorInfo]] = field(default_factory=dict)">`ModelSerializerDecoratorInfo`</SwmToken>. It stores metadata for all <SwmToken path="pydantic/_internal/_decorators.py" pos="120:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@model_serializer&#39;.">`@model_serializer`</SwmToken> decorators defined in the class.

```python
    model_serializers: dict[str, Decorator[ModelSerializerDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="426">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="426:1:1" line-data="    model_validators: dict[str, Decorator[ModelValidatorDecoratorInfo]] = field(default_factory=dict)">`model_validators`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="426:11:11" line-data="    model_validators: dict[str, Decorator[ModelValidatorDecoratorInfo]] = field(default_factory=dict)">`ModelValidatorDecoratorInfo`</SwmToken>. This holds information about all <SwmToken path="pydantic/_internal/_decorators.py" pos="139:20:21" line-data="        decorator_repr: A class variable representing the decorator string, &#39;@model_validator&#39;.">`@model_validator`</SwmToken> decorators in the class.

```python
    model_validators: dict[str, Decorator[ModelValidatorDecoratorInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="427">

---

The variable <SwmToken path="pydantic/_internal/_decorators.py" pos="427:1:1" line-data="    computed_fields: dict[str, Decorator[ComputedFieldInfo]] = field(default_factory=dict)">`computed_fields`</SwmToken> is a dictionary mapping names to Decorator objects containing <SwmToken path="pydantic/_internal/_decorators.py" pos="427:11:11" line-data="    computed_fields: dict[str, Decorator[ComputedFieldInfo]] = field(default_factory=dict)">`ComputedFieldInfo`</SwmToken>. It is used to store metadata for computed fields defined in the class.

```python
    computed_fields: dict[str, Decorator[ComputedFieldInfo]] = field(default_factory=dict)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="429">

---

The static method <SwmToken path="pydantic/_internal/_decorators.py" pos="430:3:3" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`build`</SwmToken> is responsible for collecting all decorator information from a given model or dataclass. It traverses the class hierarchy from the oldest ancestor to the leaf class, gathering all decorator instances and handling overrides to mimic Python's method resolution order. It also ensures that each decorator is properly bound to the class and manages replacement of descriptor wrappers with their wrapped functions.

```python
    @staticmethod
    def build(model_dc: type[Any]) -> DecoratorInfos:  # noqa: C901 (ignore complexity)
        """We want to collect all DecFunc instances that exist as
        attributes in the namespace of the class (a BaseModel or dataclass)
        that called us
        But we want to collect these in the order of the bases
        So instead of getting them all from the leaf class (the class that called us),
        we traverse the bases from root (the oldest ancestor class) to leaf
        and collect all of the instances as we go, taking care to replace
        any duplicate ones with the last one we see to mimic how function overriding
        works with inheritance.
        If we do replace any functions we put the replacement into the position
        the replaced function was in; that is, we maintain the order.
        """
        # reminder: dicts are ordered and replacement does not alter the order
        res = DecoratorInfos()
        for base in reversed(mro(model_dc)[1:]):
            existing: DecoratorInfos | None = base.__dict__.get('__pydantic_decorators__')
            if existing is None:
                existing = DecoratorInfos.build(base)
            res.validators.update({k: v.bind_to_cls(model_dc) for k, v in existing.validators.items()})
            res.field_validators.update({k: v.bind_to_cls(model_dc) for k, v in existing.field_validators.items()})
            res.root_validators.update({k: v.bind_to_cls(model_dc) for k, v in existing.root_validators.items()})
            res.field_serializers.update({k: v.bind_to_cls(model_dc) for k, v in existing.field_serializers.items()})
            res.model_serializers.update({k: v.bind_to_cls(model_dc) for k, v in existing.model_serializers.items()})
            res.model_validators.update({k: v.bind_to_cls(model_dc) for k, v in existing.model_validators.items()})
            res.computed_fields.update({k: v.bind_to_cls(model_dc) for k, v in existing.computed_fields.items()})

        to_replace: list[tuple[str, Any]] = []

        for var_name, var_value in vars(model_dc).items():
            if isinstance(var_value, PydanticDescriptorProxy):
                info = var_value.decorator_info
                if isinstance(info, ValidatorDecoratorInfo):
                    res.validators[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                elif isinstance(info, FieldValidatorDecoratorInfo):
                    res.field_validators[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                elif isinstance(info, RootValidatorDecoratorInfo):
                    res.root_validators[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                elif isinstance(info, FieldSerializerDecoratorInfo):
                    # check whether a serializer function is already registered for fields
                    for field_serializer_decorator in res.field_serializers.values():
                        # check that each field has at most one serializer function.
                        # serializer functions for the same field in subclasses are allowed,
                        # and are treated as overrides
                        if field_serializer_decorator.cls_var_name == var_name:
                            continue
                        for f in info.fields:
                            if f in field_serializer_decorator.info.fields:
                                raise PydanticUserError(
                                    'Multiple field serializer functions were defined '
                                    f'for field {f!r}, this is not allowed.',
                                    code='multiple-field-serializers',
                                )
                    res.field_serializers[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                elif isinstance(info, ModelValidatorDecoratorInfo):
                    res.model_validators[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                elif isinstance(info, ModelSerializerDecoratorInfo):
                    res.model_serializers[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=var_value.shim, info=info
                    )
                else:
                    from ..fields import ComputedFieldInfo

                    isinstance(var_value, ComputedFieldInfo)
                    res.computed_fields[var_name] = Decorator.build(
                        model_dc, cls_var_name=var_name, shim=None, info=info
                    )
                to_replace.append((var_name, var_value.wrapped))
        if to_replace:
            # If we can save `__pydantic_decorators__` on the class we'll be able to check for it above
            # so then we don't need to re-process the type, which means we can discard our descriptor wrappers
            # and replace them with the thing they are wrapping (see the other setattr call below)
            # which allows validator class methods to also function as regular class methods
            model_dc.__pydantic_decorators__ = res
            for name, value in to_replace:
                setattr(model_dc, name, value)
        return res

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_decorators.py" line="518">

---

The method <SwmToken path="pydantic/_internal/_decorators.py" pos="518:3:3" line-data="    def update_from_config(self, config_wrapper: ConfigWrapper) -&gt; None:">`update_from_config`</SwmToken> updates the decorator metadata based on the configuration of the class they are attached to. It specifically updates computed field decorators with configuration-driven changes, such as alias or title generators.

```python
    def update_from_config(self, config_wrapper: ConfigWrapper) -> None:
        """Update the decorator infos from the configuration of the class they are attached to."""
        for name, computed_field_dec in self.computed_fields.items():
            computed_field_dec.info._update_from_config(config_wrapper, name)
```

---

</SwmSnippet>

# Usage

## Usage in Model Metadata

<SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is assigned to the class variable **pydantic_decorators** in model classes to hold metadata about decorators defined on the model. This metadata replaces the older **validators** and <SwmToken path="pydantic/_internal/_decorators.py" pos="423:1:1" line-data="    root_validators: dict[str, Decorator[RootValidatorDecoratorInfo]] = field(default_factory=dict)">`root_validators`</SwmToken> attributes from Pydantic <SwmToken path="pydantic/_internal/_decorators.py" pos="175:14:14" line-data="        shim: A wrapper function to wrap V1 style function.">`V1`</SwmToken>, enabling the new schema generation and validation mechanisms to work properly.

## Usage in Dataclasses

When working with dataclasses, <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is built from the class using a build method and then updated from the configuration wrapper. This allows the dataclass to maintain decorator metadata consistent with the model's configuration, ensuring that validation and serialization behaviors are correctly applied.

## Usage in Internal Dataclass Structures

Within internal dataclass implementations, <SwmToken path="pydantic/_internal/_decorators.py" pos="430:16:16" line-data="    def build(model_dc: type[Any]) -&gt; DecoratorInfos:  # noqa: C901 (ignore complexity)">`DecoratorInfos`</SwmToken> is stored as a class variable **pydantic_decorators** alongside other class variables like **pydantic_config** and **pydantic_fields**. This placement indicates its role as part of the core metadata that supports the dataclass's validation and serialization features.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
