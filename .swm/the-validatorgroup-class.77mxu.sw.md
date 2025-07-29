---
title: The ValidatorGroup class
---
This document covers the <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> class in detail, including:

1. What <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken>

# What is <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken>

<SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> is a class in <SwmPath>[pydantic/v1/class_validators.py](pydantic/v1/class_validators.py)</SwmPath> that manages and organizes field validators for Pydantic models. It is responsible for storing, retrieving, and tracking the usage of validators associated with model fields. <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> helps ensure that validators are correctly applied to fields and provides mechanisms to detect unused or misconfigured validators, supporting robust data validation workflows in Pydantic.

<SwmSnippet path="/pydantic/v1/class_validators.py" line="162">

---

The variable <SwmToken path="pydantic/v1/class_validators.py" pos="162:8:8" line-data="    def __init__(self, validators: &#39;ValidatorListDict&#39;) -&gt; None:">`validators`</SwmToken> stores a dictionary mapping field names to lists of Validator instances. This structure allows <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> to efficiently retrieve all validators associated with a specific field.

```python
    def __init__(self, validators: 'ValidatorListDict') -> None:
        self.validators = validators
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/class_validators.py" line="164">

---

The variable <SwmToken path="pydantic/v1/class_validators.py" pos="164:3:3" line-data="        self.used_validators = {&#39;*&#39;}">`used_validators`</SwmToken> is a set that tracks which validator names have been accessed or used. It is initialized with the wildcard '\*' to account for validators that apply to all fields.

```python
        self.used_validators = {'*'}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/class_validators.py" line="162">

---

The function <SwmToken path="pydantic/v1/class_validators.py" pos="162:3:3" line-data="    def __init__(self, validators: &#39;ValidatorListDict&#39;) -&gt; None:">`__init__`</SwmToken> initializes a <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> instance by assigning the provided validators dictionary to the instance and setting up the <SwmToken path="pydantic/v1/class_validators.py" pos="164:3:3" line-data="        self.used_validators = {&#39;*&#39;}">`used_validators`</SwmToken> set.

```python
    def __init__(self, validators: 'ValidatorListDict') -> None:
        self.validators = validators
        self.used_validators = {'*'}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/class_validators.py" line="166">

---

The function <SwmToken path="pydantic/v1/class_validators.py" pos="166:3:3" line-data="    def get_validators(self, name: str) -&gt; Optional[Dict[str, Validator]]:">`get_validators`</SwmToken> retrieves all validators associated with a given field name. It marks the field as used, fetches validators for the specific field, and also includes any wildcard ('\*') validators if the field is not the root. The function returns a dictionary mapping validator function names to Validator instances, or None if no validators are found.

```python
    def get_validators(self, name: str) -> Optional[Dict[str, Validator]]:
        self.used_validators.add(name)
        validators = self.validators.get(name, [])
        if name != ROOT_KEY:
            validators += self.validators.get('*', [])
        if validators:
            return {getattr(v.func, '__name__', f'<No __name__: id:{id(v.func)}>'): v for v in validators}
        else:
            return None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/class_validators.py" line="176">

---

The function <SwmToken path="pydantic/v1/class_validators.py" pos="176:3:3" line-data="    def check_for_unused(self) -&gt; None:">`check_for_unused`</SwmToken> checks for any validators that have been defined for fields but have not been used. It collects the names of such unused validators (where <SwmToken path="pydantic/v1/class_validators.py" pos="182:5:5" line-data="                    if v.check_fields">`check_fields`</SwmToken> is True) and raises a <SwmToken path="pydantic/v1/class_validators.py" pos="189:3:3" line-data="            raise ConfigError(">`ConfigError`</SwmToken> if any are found, helping developers catch misconfigurations or typos in field names.

```python
    def check_for_unused(self) -> None:
        unused_validators = set(
            chain.from_iterable(
                (
                    getattr(v.func, '__name__', f'<No __name__: id:{id(v.func)}>')
                    for v in self.validators[f]
                    if v.check_fields
                )
                for f in (self.validators.keys() - self.used_validators)
            )
        )
        if unused_validators:
            fn = ', '.join(unused_validators)
            raise ConfigError(
                f"Validators defined with incorrect fields: {fn} "  # noqa: Q000
                f"(use check_fields=False if you're inheriting from the model and intended this)"
            )
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken>

<SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> is used to group multiple validators together, allowing them to be managed as a single unit. This is particularly useful when applying validation logic to data models.

An example of its usage is found in the main module where validators are first extracted and inherited from a namespace, then passed to <SwmToken path="pydantic/v1/class_validators.py" pos="161:2:2" line-data="class ValidatorGroup:">`ValidatorGroup`</SwmToken> to create a grouped validator instance.

This grouped validator instance can then be used to apply validation rules consistently across fields or data structures, ensuring that all relevant validators are considered during data validation.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
