---
title: The MissingDefinitionError class
---
# Intro

This document covers the <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> class. We will address:

1. What <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> is and its purpose
2. The variables and functions defined in <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken>

# What is <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken>

<SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> is a custom exception class defined in <SwmPath>[pydantic/\_internal/\_schema_gather.py](pydantic/_internal/_schema_gather.py)</SwmPath>. It is used to signal that a reference within a schema points to a core schema definition that does not exist. This exception is raised during schema traversal when a <SwmToken path="pydantic/_internal/_schema_gather.py" pos="77:7:9" line-data="        # The `&#39;definition-ref&#39;` schema was only encountered once, make it">`definition-ref`</SwmToken> schema is encountered that cannot be resolved to an actual definition. Its main role is to ensure schema integrity by preventing unresolved references from passing silently.

<SwmSnippet path="/pydantic/_internal/_schema_gather.py" line="31">

---

The class defines the variable <SwmToken path="pydantic/_internal/_schema_gather.py" pos="31:8:8" line-data="    def __init__(self, schema_reference: str, /) -&gt; None:">`schema_reference`</SwmToken>, which stores the string reference that could not be resolved to a core schema definition.

```python
    def __init__(self, schema_reference: str, /) -> None:
        self.schema_reference = schema_reference
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_schema_gather.py" line="31">

---

The <SwmToken path="pydantic/_internal/_schema_gather.py" pos="31:3:3" line-data="    def __init__(self, schema_reference: str, /) -&gt; None:">`__init__`</SwmToken> function initializes the <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> instance by accepting the unresolved schema reference as an argument and assigning it to the <SwmToken path="pydantic/_internal/_schema_gather.py" pos="31:8:8" line-data="    def __init__(self, schema_reference: str, /) -&gt; None:">`schema_reference`</SwmToken> variable.

```python
    def __init__(self, schema_reference: str, /) -> None:
        self.schema_reference = schema_reference
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken>

<SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> is raised when a schema reference does not correspond to any existing definition during the schema gathering process. This ensures that all references in the schema are valid and prevents unresolved references from propagating.

During the traversal of core schemas, any <SwmToken path="pydantic/_internal/_schema_gather.py" pos="77:7:9" line-data="        # The `&#39;definition-ref&#39;` schema was only encountered once, make it">`definition-ref`</SwmToken> schema must point to an existing definition. If the reference is missing, <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> is triggered to signal this inconsistency.

Additionally, when a schema reference is encountered, the system checks if it has been collected before. If not found in the collected references and no definition exists for it, <SwmToken path="pydantic/_internal/_schema_gather.py" pos="28:2:2" line-data="class MissingDefinitionError(LookupError):">`MissingDefinitionError`</SwmToken> is raised immediately to halt processing and highlight the missing definition.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
