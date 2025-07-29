---
title: Internal Schema Representation in Validation and Serialization
---
# Internal Schema Representation Overview

Internal schema is a structured representation of Python types, models, and their interrelationships used throughout the validation and serialization processes. It serves as a foundational abstraction that enables the library to accurately describe and process complex Python constructs such as models, unions, and annotated types.

# Purpose and Features of Internal Schema

The internal schema system supports advanced features including references, definitions, and discriminators. These features are critical for managing recursive types, shared definitions, and tagged unions effectively. By leveraging these mechanisms, the schema system ensures efficient handling of complex data structures and maintains correctness during validation and serialization.

# Schema Generation and Management

During schema generation, the system tracks and resolves references and definitions to prevent duplication and to support capabilities like circular references and deferred application of discriminators. Specialized logic traverses the schema to decide when to inline or preserve definitions based on their usage context, metadata, and serialization needs. This process guarantees that the resulting schema is both accurate and optimized for downstream tasks such as validation and JSON schema generation.

# Usage in the Codebase

Internal schema is generated dynamically during model and type processing phases. It is manipulated by various internal functions that traverse and modify the schema structure. For instance, the <SwmToken path="pydantic/_internal/_generate_schema.py" pos="683:3:3" line-data="    def clean_schema(self, schema: CoreSchema) -&gt; CoreSchema:">`clean_schema`</SwmToken> method within the <SwmToken path="pydantic/_internal/_generate_schema.py" pos="330:2:2" line-data="class GenerateSchema:">`GenerateSchema`</SwmToken> class finalizes the schema by resolving references and applying any deferred discriminators, preparing it for subsequent validation and serialization steps.

<SwmSnippet path="/pydantic/_internal/_generate_schema.py" line="683">

---

A concrete example of internal schema usage is the <SwmToken path="pydantic/_internal/_generate_schema.py" pos="683:3:3" line-data="    def clean_schema(self, schema: CoreSchema) -&gt; CoreSchema:">`clean_schema`</SwmToken> function, which is invoked during model construction and type adapter initialization. This function systematically traverses the internal schema, resolves all references, inlines definitions where appropriate, and applies deferred discriminators. This ensures that the schema is fully prepared and consistent for validation and serialization workflows.

```python
    def clean_schema(self, schema: CoreSchema) -> CoreSchema:
        return self.defs.finalize_schema(schema)
```

---

</SwmSnippet>

# Integration Points

The internal schema is integral to multiple stages of the library's operation, including model construction, dataclass completion, and type adapter initialization. Functions such as <SwmToken path="pydantic/_internal/_generate_schema.py" pos="683:3:3" line-data="    def clean_schema(self, schema: CoreSchema) -&gt; CoreSchema:">`clean_schema`</SwmToken>, `traverse_schema`, and <SwmToken path="pydantic/_internal/_generate_schema.py" pos="740:7:7" line-data="        with self.defs.get_schema_or_ref(cls) as (model_ref, maybe_schema):">`get_schema_or_ref`</SwmToken> interact with the internal schema to maintain its correctness and efficiency, ensuring seamless validation and serialization.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
