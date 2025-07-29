---
title: The PydanticDataclass class
---
This document covers the <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken> class in detail:

1. What <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken>

<SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken> is a protocol that defines the attributes available on a class after it has been decorated as a Pydantic dataclass. It is used internally to ensure that dataclasses enhanced by Pydantic have a consistent set of metadata and validation-related attributes. This protocol is not meant to be instantiated directly, but rather serves as a type contract for Pydantic-decorated dataclasses, enabling features such as validation, serialization, and configuration management.

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="54">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="54:1:1" line-data="        __pydantic_config__: ClassVar[ConfigDict]">`__pydantic_config__`</SwmToken> holds the <SwmToken path="pydantic/_internal/_dataclasses.py" pos="45:4:6" line-data="            __pydantic_config__: Pydantic-specific configuration settings for the dataclass.">`Pydantic-specific`</SwmToken> configuration settings for the dataclass. This configuration dictates how the dataclass behaves in terms of validation and serialization.

```python
        __pydantic_config__: ClassVar[ConfigDict]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="55">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="55:1:1" line-data="        __pydantic_complete__: ClassVar[bool]">`__pydantic_complete__`</SwmToken> indicates whether the dataclass building process is finished or if there are still undefined fields. This helps track the readiness of the dataclass for use.

```python
        __pydantic_complete__: ClassVar[bool]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="56">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="56:1:1" line-data="        __pydantic_core_schema__: ClassVar[core_schema.CoreSchema]">`__pydantic_core_schema__`</SwmToken> stores the core schema used by Pydantic to build the <SwmToken path="pydantic/_internal/_dataclasses.py" pos="60:6:6" line-data="        __pydantic_validator__: ClassVar[SchemaValidator | PluggableSchemaValidator]">`SchemaValidator`</SwmToken> and <SwmToken path="pydantic/_internal/_dataclasses.py" pos="59:6:6" line-data="        __pydantic_serializer__: ClassVar[SchemaSerializer]">`SchemaSerializer`</SwmToken> for the dataclass. This schema defines the structure and validation rules for the dataclass.

```python
        __pydantic_core_schema__: ClassVar[core_schema.CoreSchema]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="57">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="57:1:1" line-data="        __pydantic_decorators__: ClassVar[_decorators.DecoratorInfos]">`__pydantic_decorators__`</SwmToken> contains metadata about the decorators that have been applied to the dataclass. This allows Pydantic to track and utilize any custom behaviors introduced via decorators.

```python
        __pydantic_decorators__: ClassVar[_decorators.DecoratorInfos]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="58">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="58:1:1" line-data="        __pydantic_fields__: ClassVar[dict[str, FieldInfo]]">`__pydantic_fields__`</SwmToken> holds metadata about the fields defined on the dataclass. This includes information such as field types, defaults, and validation constraints.

```python
        __pydantic_fields__: ClassVar[dict[str, FieldInfo]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="59">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="59:1:1" line-data="        __pydantic_serializer__: ClassVar[SchemaSerializer]">`__pydantic_serializer__`</SwmToken> is the <SwmToken path="pydantic/_internal/_dataclasses.py" pos="59:6:6" line-data="        __pydantic_serializer__: ClassVar[SchemaSerializer]">`SchemaSerializer`</SwmToken> instance used to serialize (dump) instances of the dataclass according to the schema.

```python
        __pydantic_serializer__: ClassVar[SchemaSerializer]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="60">

---

The variable <SwmToken path="pydantic/_internal/_dataclasses.py" pos="60:1:1" line-data="        __pydantic_validator__: ClassVar[SchemaValidator | PluggableSchemaValidator]">`__pydantic_validator__`</SwmToken> is the <SwmToken path="pydantic/_internal/_dataclasses.py" pos="60:6:6" line-data="        __pydantic_validator__: ClassVar[SchemaValidator | PluggableSchemaValidator]">`SchemaValidator`</SwmToken> (or <SwmToken path="pydantic/_internal/_dataclasses.py" pos="60:10:10" line-data="        __pydantic_validator__: ClassVar[SchemaValidator | PluggableSchemaValidator]">`PluggableSchemaValidator`</SwmToken>) instance used to validate instances of the dataclass. This enables automatic validation of input data when creating or updating dataclass instances.

```python
        __pydantic_validator__: ClassVar[SchemaValidator | PluggableSchemaValidator]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_dataclasses.py" line="62">

---

The class method <SwmToken path="pydantic/_internal/_dataclasses.py" pos="63:3:3" line-data="        def __pydantic_fields_complete__(cls) -&gt; bool: ...">`__pydantic_fields_complete__`</SwmToken> returns a boolean indicating whether all fields of the dataclass have been fully defined and processed. This is useful for checking the completeness of the dataclass setup.

```python
        @classmethod
        def __pydantic_fields_complete__(cls) -> bool: ...
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken>

The <SwmToken path="pydantic/_internal/_dataclasses.py" pos="41:3:3" line-data="    class PydanticDataclass(StandardDataclass, typing.Protocol):">`PydanticDataclass`</SwmToken> class overrides the **init** method of a dataclass to perform validation using Pydantic's validator before completing the instance initialization. This ensures that the input arguments to the dataclass are validated according to the defined Pydantic model rules.

During initialization, the **init** method calls the **pydantic_validator** attribute's <SwmToken path="pydantic/_internal/_dataclasses.py" pos="127:5:5" line-data="        s.__pydantic_validator__.validate_python(ArgsKwargs(args, kwargs), self_instance=s)">`validate_python`</SwmToken> method, passing the constructor arguments wrapped in an <SwmToken path="pydantic/_internal/_dataclasses.py" pos="16:1:1" line-data="    ArgsKwargs,">`ArgsKwargs`</SwmToken> object along with the instance itself. This validation step enforces type checks and other constraints defined in the dataclass.

This approach allows developers to use dataclasses with Pydantic's powerful validation features seamlessly, combining the simplicity of dataclasses with robust data validation.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
