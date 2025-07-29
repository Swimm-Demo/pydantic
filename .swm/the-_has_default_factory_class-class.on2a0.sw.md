---
title: The _HAS_DEFAULT_FACTORY_CLASS class
---
This document will cover the following aspects of <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>:

1. What is <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>
2. Variables and functions defined in <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>

<SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken> is a class defined in <SwmPath>[pydantic/\_internal/\_signature.py](pydantic/_internal/_signature.py)</SwmPath> that serves as a marker to indicate the presence of a default factory for a dataclass field. It is a minimal class with a custom **repr** method that returns the string '<factory>'. This class is copied from the standard library dataclasses module and is used internally within Pydantic to mimic dataclass behavior when handling default factories for fields.

<SwmSnippet path="/pydantic/_internal/_signature.py" line="17">

---

The class <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken> defines a single method **repr**, which returns the string '<factory>'. This method is used to provide a meaningful string representation of instances of this class, indicating that it represents a factory.

```python
class _HAS_DEFAULT_FACTORY_CLASS:
    def __repr__(self):
        return '<factory>'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_signature.py" line="22">

---

The module defines a constant <SwmToken path="pydantic/_internal/_signature.py" pos="22:0:0" line-data="_HAS_DEFAULT_FACTORY = _HAS_DEFAULT_FACTORY_CLASS()">`_HAS_DEFAULT_FACTORY`</SwmToken> which is an instance of <SwmToken path="pydantic/_internal/_signature.py" pos="22:4:4" line-data="_HAS_DEFAULT_FACTORY = _HAS_DEFAULT_FACTORY_CLASS()">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>. This instance is used as a sentinel value to indicate that a default factory exists for a dataclass field when generating function signatures.

```python
_HAS_DEFAULT_FACTORY = _HAS_DEFAULT_FACTORY_CLASS()

```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>

<SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken> is defined as a class with a custom **repr** method that returns the string '<factory>'. This class is instantiated once as <SwmToken path="pydantic/_internal/_signature.py" pos="22:0:0" line-data="_HAS_DEFAULT_FACTORY = _HAS_DEFAULT_FACTORY_CLASS()">`_HAS_DEFAULT_FACTORY`</SwmToken> and serves as a unique marker or sentinel value within the codebase.

## Usage of <SwmToken path="pydantic/_internal/_signature.py" pos="17:2:2" line-data="class _HAS_DEFAULT_FACTORY_CLASS:">`_HAS_DEFAULT_FACTORY_CLASS`</SwmToken>

The instance <SwmToken path="pydantic/_internal/_signature.py" pos="22:0:0" line-data="_HAS_DEFAULT_FACTORY = _HAS_DEFAULT_FACTORY_CLASS()">`_HAS_DEFAULT_FACTORY`</SwmToken> is used internally to represent a default factory in data validation or model signature processing. It acts as a distinct identifier to differentiate between fields that have a default factory function and those that do not.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
