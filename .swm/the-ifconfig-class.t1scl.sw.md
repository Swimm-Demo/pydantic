---
title: The IfConfig class
---
This document covers the <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> class in detail, focusing on:

1. What <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> is and its purpose
2. The variables and functions defined in <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken>, with code citations for each

# What is <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken>

<SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> is a utility class in <SwmPath>[pydantic/v1/validators.py](pydantic/v1/validators.py)</SwmPath> designed to conditionally apply validators based on configuration attributes. It is used to determine whether a specific validator should be executed, depending on the values of certain configuration options in a Pydantic model's config class. This allows for flexible and dynamic validation logic that adapts to user-defined settings.

<SwmSnippet path="/pydantic/v1/validators.py" line="639">

---

The variable <SwmToken path="pydantic/v1/validators.py" pos="639:8:8" line-data="    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -&gt; None:">`validator`</SwmToken> stores a reference to the validator function that may be conditionally applied. This function is provided when an <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> instance is created.

```python
    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -> None:
        self.validator = validator
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/validators.py" line="639">

---

The variable <SwmToken path="pydantic/v1/validators.py" pos="639:15:15" line-data="    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -&gt; None:">`config_attr_names`</SwmToken> holds the names of configuration attributes that <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> will check to decide if the validator should be used. These are passed as positional arguments during initialization.

```python
    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -> None:
        self.validator = validator
        self.config_attr_names = config_attr_names
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/validators.py" line="639">

---

The variable <SwmToken path="pydantic/v1/validators.py" pos="639:21:21" line-data="    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -&gt; None:">`ignored_value`</SwmToken> specifies a value that, if matched by a configuration attribute, will cause <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> to ignore that attribute. By default, this is set to False.

```python
    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -> None:
        self.validator = validator
        self.config_attr_names = config_attr_names
        self.ignored_value = ignored_value
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/validators.py" line="639">

---

The <SwmToken path="pydantic/v1/validators.py" pos="639:3:3" line-data="    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -&gt; None:">`__init__`</SwmToken> function initializes an <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> instance by storing the validator, the configuration attribute names to check, and the ignored value.

```python
    def __init__(self, validator: AnyCallable, *config_attr_names: str, ignored_value: Any = False) -> None:
        self.validator = validator
        self.config_attr_names = config_attr_names
        self.ignored_value = ignored_value
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/validators.py" line="644">

---

The <SwmToken path="pydantic/v1/validators.py" pos="644:3:3" line-data="    def check(self, config: Type[&#39;BaseConfig&#39;]) -&gt; bool:">`check`</SwmToken> function determines whether any of the specified configuration attributes on a given config class are set to a value other than None or the ignored value. If so, it returns True, indicating that the associated validator should be applied.

```python
    def check(self, config: Type['BaseConfig']) -> bool:
        return any(getattr(config, name) not in {None, self.ignored_value} for name in self.config_attr_names)
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> Usage in Validators

<SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> is used to apply specific validation functions conditionally based on configuration flags. For example, it wraps validators like <SwmToken path="pydantic/v1/validators.py" pos="230:2:2" line-data="def anystr_strip_whitespace(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_strip_whitespace`</SwmToken>, <SwmToken path="pydantic/v1/validators.py" pos="234:2:2" line-data="def anystr_upper(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_upper`</SwmToken>, <SwmToken path="pydantic/v1/validators.py" pos="238:2:2" line-data="def anystr_lower(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_lower`</SwmToken>, and <SwmToken path="pydantic/v1/validators.py" pos="216:2:2" line-data="def anystr_length_validator(v: &#39;StrBytes&#39;, config: &#39;BaseConfig&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_length_validator`</SwmToken> with corresponding configuration keys such as <SwmToken path="pydantic/v1/validators.py" pos="230:2:2" line-data="def anystr_strip_whitespace(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_strip_whitespace`</SwmToken> or <SwmToken path="pydantic/v1/validators.py" pos="219:7:7" line-data="    min_length = config.min_anystr_length">`min_anystr_length`</SwmToken>. This allows the validation logic to be enabled or disabled dynamically depending on the presence or value of these configuration options.

## Example of <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> with String Validators

In the <SwmPath>[pydantic/v1/validators.py](pydantic/v1/validators.py)</SwmPath> file, <SwmToken path="pydantic/v1/validators.py" pos="638:2:2" line-data="class IfConfig:">`IfConfig`</SwmToken> is used to conditionally apply string-related validators. For instance, <SwmToken path="pydantic/v1/validators.py" pos="230:2:2" line-data="def anystr_strip_whitespace(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_strip_whitespace`</SwmToken> is applied only if the <SwmToken path="pydantic/v1/validators.py" pos="230:2:2" line-data="def anystr_strip_whitespace(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_strip_whitespace`</SwmToken> configuration is set. Similarly, <SwmToken path="pydantic/v1/validators.py" pos="234:2:2" line-data="def anystr_upper(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_upper`</SwmToken> and <SwmToken path="pydantic/v1/validators.py" pos="238:2:2" line-data="def anystr_lower(v: &#39;StrBytes&#39;) -&gt; &#39;StrBytes&#39;:">`anystr_lower`</SwmToken> validators are applied based on their respective configuration flags. This pattern enables flexible and configurable validation behavior for string fields.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
