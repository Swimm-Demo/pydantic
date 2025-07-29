---
title: The PydanticGenericMetadata class
---
This document covers the class <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken>. We'll address:

1. What <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken> is and its purpose
2. The variables defined in <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken>

<SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken> is a <SwmToken path="pydantic/_internal/_generics.py" pos="99:6:6" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`TypedDict`</SwmToken> defined in <SwmPath>[pydantic/\_internal/\_generics.py](pydantic/_internal/_generics.py)</SwmPath>. It is used to store metadata about generic Pydantic models, specifically capturing information analogous to Python's typing generics. This metadata is attached to generic model classes to track their origin, type arguments, and type parameters, enabling Pydantic to handle generic models' instantiation, validation, and schema generation in a type-safe and introspectable way.

<SwmSnippet path="/pydantic/_internal/_generics.py" line="99">

---

The variable <SwmToken path="pydantic/_internal/_generics.py" pos="100:1:1" line-data="    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__">`origin`</SwmToken> stores the original <SwmToken path="pydantic/_internal/_generics.py" pos="100:6:6" line-data="    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__">`BaseModel`</SwmToken> type from which a generic model is derived. It is analogous to the <SwmToken path="pydantic/_internal/_generics.py" pos="100:23:23" line-data="    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__">`__origin__`</SwmToken> attribute in Python's typing system, and can be None if not set.

```python
class PydanticGenericMetadata(typing_extensions.TypedDict):
    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__
    args: tuple[Any, ...]  # analogous to typing._GenericAlias.__args__
    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_generics.py" line="99">

---

The variable <SwmToken path="pydantic/_internal/_generics.py" pos="101:1:1" line-data="    args: tuple[Any, ...]  # analogous to typing._GenericAlias.__args__">`args`</SwmToken> holds a tuple of arguments that were used to parameterize the generic model. This is similar to the <SwmToken path="pydantic/_internal/_generics.py" pos="101:22:22" line-data="    args: tuple[Any, ...]  # analogous to typing._GenericAlias.__args__">`__args__`</SwmToken> attribute in typing generics, and allows Pydantic to know what concrete types have been substituted for the model's type variables.

```python
class PydanticGenericMetadata(typing_extensions.TypedDict):
    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__
    args: tuple[Any, ...]  # analogous to typing._GenericAlias.__args__
    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_generics.py" line="99">

---

The variable <SwmToken path="pydantic/_internal/_generics.py" pos="102:1:1" line-data="    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__">`parameters`</SwmToken> contains a tuple of <SwmToken path="pydantic/_internal/_generics.py" pos="102:6:6" line-data="    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__">`TypeVar`</SwmToken> instances representing the type parameters of the generic model. This is analogous to the <SwmToken path="pydantic/_internal/_generics.py" pos="102:22:22" line-data="    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__">`__parameters__`</SwmToken> attribute in <SwmToken path="pydantic/_internal/_generics.py" pos="102:18:20" line-data="    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__">`typing.Generic`</SwmToken>, and is used to track which type variables are available for substitution.

```python
class PydanticGenericMetadata(typing_extensions.TypedDict):
    origin: type[BaseModel] | None  # analogous to typing._GenericAlias.__origin__
    args: tuple[Any, ...]  # analogous to typing._GenericAlias.__args__
    parameters: tuple[TypeVar, ...]  # analogous to typing.Generic.__parameters__
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken> in Model Metadata

<SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken> is used as a class variable named <SwmToken path="pydantic/_internal/_generics.py" pos="197:1:1" line-data="    pydantic_generic_metadata: PydanticGenericMetadata | None = getattr(v, &#39;__pydantic_generic_metadata__&#39;, None)">`pydantic_generic_metadata`</SwmToken> in the main model class to store metadata about generic models. This metadata serves a similar purpose to the **args**, **origin**, and **parameters** attributes found in Python's typing generics, helping to manage generic type information within Pydantic models.

## <SwmToken path="pydantic/_internal/_generics.py" pos="99:2:2" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`PydanticGenericMetadata`</SwmToken> in Generic Type Utilities

The class is also utilized in internal generic utility functions such as <SwmToken path="pydantic/_internal/_generics.py" pos="191:5:5" line-data="        args = get_args(v)">`get_args`</SwmToken> and <SwmToken path="pydantic/_internal/_generics.py" pos="203:2:2" line-data="def get_origin(v: Any) -&gt; Any:">`get_origin`</SwmToken>. These functions attempt to retrieve generic type arguments and origins from an object by first checking if the object has <SwmToken path="pydantic/_internal/_generics.py" pos="197:1:1" line-data="    pydantic_generic_metadata: PydanticGenericMetadata | None = getattr(v, &#39;__pydantic_generic_metadata__&#39;, None)">`pydantic_generic_metadata`</SwmToken>. If present, they extract the relevant information from this metadata; otherwise, they fall back to using the standard <SwmToken path="pydantic/_internal/_generics.py" pos="99:4:4" line-data="class PydanticGenericMetadata(typing_extensions.TypedDict):">`typing_extensions`</SwmToken> methods. This approach allows Pydantic to support enhanced generic type introspection beyond what the standard typing module provides.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
