---
title: The Dataclass class
---
# What is Dataclass

# Variables and functions

This document covers the following aspects of the <SwmToken path="pydantic/v1/dataclasses.py" pos="77:10:10" line-data="        __pydantic_validate_values__: ClassVar[Callable[[&#39;Dataclass&#39;], None]]">`Dataclass`</SwmToken> class in Pydantic:

1. What the <SwmToken path="pydantic/v1/dataclasses.py" pos="77:10:10" line-data="        __pydantic_validate_values__: ClassVar[Callable[[&#39;Dataclass&#39;], None]]">`Dataclass`</SwmToken> class is and its purpose.
2. All variables and functions defined in the <SwmToken path="pydantic/v1/dataclasses.py" pos="77:10:10" line-data="        __pydantic_validate_values__: ClassVar[Callable[[&#39;Dataclass&#39;], None]]">`Dataclass`</SwmToken> class, with explanations and code citations.

# What is Dataclass

The <SwmToken path="pydantic/v1/dataclasses.py" pos="77:10:10" line-data="        __pydantic_validate_values__: ClassVar[Callable[[&#39;Dataclass&#39;], None]]">`Dataclass`</SwmToken> class in Pydantic is a foundational class that underpins Pydantic's support for data validation on Python dataclasses. It acts as a base for dataclasses that are enhanced with Pydantic's validation logic, enabling type-checked and validated data structures while maintaining compatibility with the standard library's dataclasses. This class is not intended for direct use by end users, but rather serves as an internal mechanism to attach validation and configuration attributes to dataclasses decorated with Pydantic's `@`<SwmToken path="pydantic/v1/dataclasses.py" pos="3:4:4" line-data="A pydantic dataclass can be generated from scratch or from a stdlib one.">`dataclass`</SwmToken> decorator.

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="68">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="68:1:1" line-data="        __dataclass_fields__: ClassVar[Dict[str, Any]]">`__dataclass_fields__`</SwmToken> is a class variable that holds a dictionary mapping field names to their definitions. This mirrors the standard library dataclass attribute and is used to introspect the fields defined on the dataclass.

```python
        __dataclass_fields__: ClassVar[Dict[str, Any]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="69">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="69:1:1" line-data="        __dataclass_params__: ClassVar[Any]  # in reality `dataclasses._DataclassParams`">`__dataclass_params__`</SwmToken> is a class variable that stores the parameters used to configure the dataclass, such as whether it is frozen or has an <SwmToken path="pydantic/v1/dataclasses.py" pos="80:3:3" line-data="        def __init__(self, *args: object, **kwargs: object) -&gt; None:">`__init__`</SwmToken> method. This is also a standard attribute from the Python dataclasses module.

```python
        __dataclass_params__: ClassVar[Any]  # in reality `dataclasses._DataclassParams`
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="70">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="70:1:1" line-data="        __post_init__: ClassVar[Callable[..., None]]">`__post_init__`</SwmToken> is a class variable referencing the post-initialization method for the dataclass. This method is called after the dataclass's <SwmToken path="pydantic/v1/dataclasses.py" pos="80:3:3" line-data="        def __init__(self, *args: object, **kwargs: object) -&gt; None:">`__init__`</SwmToken> method, allowing for additional setup or validation logic.

```python
        __post_init__: ClassVar[Callable[..., None]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="73">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="73:1:1" line-data="        __pydantic_run_validation__: ClassVar[bool]">`__pydantic_run_validation__`</SwmToken> is a class variable added by Pydantic to indicate whether validation should be run on initialization. It controls the execution of Pydantic's validation logic when creating instances of the dataclass.

```python
        __pydantic_run_validation__: ClassVar[bool]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="74">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="74:1:1" line-data="        __post_init_post_parse__: ClassVar[Callable[..., None]]">`__post_init_post_parse__`</SwmToken> is a class variable that references a method to be called after Pydantic's parsing and validation logic has run. This allows for custom logic to be executed after validation is complete.

```python
        __post_init_post_parse__: ClassVar[Callable[..., None]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="75">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="75:1:1" line-data="        __pydantic_initialised__: ClassVar[bool]">`__pydantic_initialised__`</SwmToken> is a class variable indicating whether the dataclass instance has been fully initialized and validated by Pydantic. This helps prevent redundant validation or initialization steps.

```python
        __pydantic_initialised__: ClassVar[bool]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="76">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="76:1:1" line-data="        __pydantic_model__: ClassVar[Type[BaseModel]]">`__pydantic_model__`</SwmToken> is a class variable that holds a reference to the Pydantic <SwmToken path="pydantic/v1/dataclasses.py" pos="76:8:8" line-data="        __pydantic_model__: ClassVar[Type[BaseModel]]">`BaseModel`</SwmToken> generated from the dataclass. This model is used internally for validation and schema generation.

```python
        __pydantic_model__: ClassVar[Type[BaseModel]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="77">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="77:1:1" line-data="        __pydantic_validate_values__: ClassVar[Callable[[&#39;Dataclass&#39;], None]]">`__pydantic_validate_values__`</SwmToken> is a class variable referencing a callable that performs value validation on the dataclass instance. This function is responsible for running Pydantic's validation logic on the dataclass fields.

```python
        __pydantic_validate_values__: ClassVar[Callable[['Dataclass'], None]]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="78">

---

<SwmToken path="pydantic/v1/dataclasses.py" pos="78:1:1" line-data="        __pydantic_has_field_info_default__: ClassVar[bool]  # whether a `pydantic.Field` is used as default value">`__pydantic_has_field_info_default__`</SwmToken> is a class variable indicating whether any field in the dataclass uses a <SwmToken path="pydantic/v1/dataclasses.py" pos="78:16:18" line-data="        __pydantic_has_field_info_default__: ClassVar[bool]  # whether a `pydantic.Field` is used as default value">`pydantic.Field`</SwmToken> as its default value. This affects how defaults are handled during validation.

```python
        __pydantic_has_field_info_default__: ClassVar[bool]  # whether a `pydantic.Field` is used as default value
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="80">

---

The <SwmToken path="pydantic/v1/dataclasses.py" pos="80:3:3" line-data="        def __init__(self, *args: object, **kwargs: object) -&gt; None:">`__init__`</SwmToken> method is defined to accept arbitrary positional and keyword arguments, but its implementation is a placeholder (`pass`). The actual initialization logic is injected dynamically by Pydantic when the dataclass is created.

```python
        def __init__(self, *args: object, **kwargs: object) -> None:
            pass
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="83">

---

The <SwmToken path="pydantic/v1/dataclasses.py" pos="84:3:3" line-data="        def __get_validators__(cls: Type[&#39;Dataclass&#39;]) -&gt; &#39;CallableGenerator&#39;:">`__get_validators__`</SwmToken> class method is a placeholder that is intended to yield validator functions for the dataclass. This method is used by Pydantic's validation system to retrieve the appropriate validation logic for the dataclass.

```python
        @classmethod
        def __get_validators__(cls: Type['Dataclass']) -> 'CallableGenerator':
            pass
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/dataclasses.py" line="87">

---

The <SwmToken path="pydantic/v1/dataclasses.py" pos="88:3:3" line-data="        def __validate__(cls: Type[&#39;DataclassT&#39;], v: Any) -&gt; &#39;DataclassT&#39;:">`__validate__`</SwmToken> class method is a placeholder for the main validation entry point. It is called to validate input data and construct an instance of the dataclass, raising validation errors if the input does not conform to the expected types and constraints.

```python
        @classmethod
        def __validate__(cls: Type['DataclassT'], v: Any) -> 'DataclassT':
            pass
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
