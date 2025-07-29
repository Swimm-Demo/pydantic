---
title: The BaseConfig class
---
This document will cover the following aspects of <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken>:

1. What is <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken>
2. Variables and functions

# What is <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken>

<SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> is a class defined in <SwmPath>[pydantic/deprecated/config.py](pydantic/deprecated/config.py)</SwmPath> that is retained solely for backwards compatibility. It is marked as deprecated and users are advised to use <SwmToken path="pydantic/deprecated/config.py" pos="29:16:18" line-data="@deprecated(&#39;BaseConfig is deprecated. Use the `pydantic.ConfigDict` instead.&#39;, category=PydanticDeprecatedSince20)">`pydantic.ConfigDict`</SwmToken> instead. The class serves as a legacy configuration holder in the Pydantic library, warning users about its deprecation whenever its attributes or subclasses are accessed or created.

<SwmSnippet path="/pydantic/deprecated/config.py" line="19">

---

<SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> uses a metaclass called <SwmToken path="pydantic/deprecated/config.py" pos="19:2:2" line-data="class _ConfigMetaclass(type):">`_ConfigMetaclass`</SwmToken> which intercepts attribute access at the class level to provide default configuration values from <SwmToken path="pydantic/deprecated/config.py" pos="22:5:7" line-data="            obj = _config.config_defaults[item]">`_config.config_defaults`</SwmToken> and emits deprecation warnings.

```python
class _ConfigMetaclass(type):
    def __getattr__(self, item: str) -> Any:
        try:
            obj = _config.config_defaults[item]
            warnings.warn(_config.DEPRECATION_MESSAGE, DeprecationWarning)
            return obj
        except KeyError as exc:
            raise AttributeError(f"type object '{self.__name__}' has no attribute {exc}") from exc

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/config.py" line="37">

---

The **getattr** method in <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> is overridden to intercept instance attribute access. It tries to get the attribute normally, emits a deprecation warning, and returns the value. If the attribute is not found on the instance, it attempts to get it from the class, raising <SwmToken path="pydantic/deprecated/config.py" pos="42:3:3" line-data="        except AttributeError as exc:">`AttributeError`</SwmToken> if missing.

```python
    def __getattr__(self, item: str) -> Any:
        try:
            obj = super().__getattribute__(item)
            warnings.warn(_config.DEPRECATION_MESSAGE, DeprecationWarning)
            return obj
        except AttributeError as exc:
            try:
                return getattr(type(self), item)
            except AttributeError:
                # re-raising changes the displayed text to reflect that `self` is not a type
                raise AttributeError(str(exc)) from exc
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/config.py" line="49">

---

The **init_subclass** method is overridden to emit a deprecation warning whenever a subclass of <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> is created, ensuring that subclassing this deprecated class is also discouraged.

```python
    def __init_subclass__(cls, **kwargs: Any) -> None:
        warnings.warn(_config.DEPRECATION_MESSAGE, DeprecationWarning)
        return super().__init_subclass__(**kwargs)
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> Usage in Deprecated Config Module

<SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> is imported and exposed in the deprecated configuration module, suggesting it serves as a foundational configuration class that other configurations might extend or rely on. This module also defines a custom deprecation warning, which implies that <SwmToken path="pydantic/deprecated/config.py" pos="16:5:5" line-data="__all__ = &#39;BaseConfig&#39;, &#39;Extra&#39;">`BaseConfig`</SwmToken> might be part of legacy support or transitioning features.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
