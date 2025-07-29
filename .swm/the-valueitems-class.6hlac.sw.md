---
title: The ValueItems class
---
This document covers the <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> class in detail, including:

1. What <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken>, with explanations and code citations for each.

# What is <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken>

<SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> is a utility class designed to facilitate the calculation of excluded or included fields on values, such as when serializing or processing data structures. It provides a convenient interface for determining which items (fields or indexes) should be included or excluded, supporting both mapping and set semantics. <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> is especially useful when working with complex data structures that require fine-grained control over which elements are processed.

<SwmSnippet path="/pydantic/_internal/_utils.py" line="174">

---

The variable <SwmToken path="pydantic/_internal/_utils.py" pos="174:1:1" line-data="    __slots__ = (&#39;_items&#39;, &#39;_type&#39;)">`__slots__`</SwmToken> is used to restrict the attributes that instances of <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> can have, improving memory efficiency. It allows only <SwmToken path="pydantic/_internal/_utils.py" pos="174:7:7" line-data="    __slots__ = (&#39;_items&#39;, &#39;_type&#39;)">`_items`</SwmToken> and <SwmToken path="pydantic/_internal/_utils.py" pos="174:12:12" line-data="    __slots__ = (&#39;_items&#39;, &#39;_type&#39;)">`_type`</SwmToken> attributes.

```python
    __slots__ = ('_items', '_type')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="176">

---

The constructor <SwmToken path="pydantic/_internal/_utils.py" pos="176:3:3" line-data="    def __init__(self, value: Any, items: AbstractSetIntStr | MappingIntStrAny) -&gt; None:">`__init__`</SwmToken> initializes a <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> instance by coercing the input items into a mapping. If the value is a list or tuple, it normalizes the indexes to handle negative indices and the special '**all**' keyword. The processed items are stored in the <SwmToken path="pydantic/_internal/_utils.py" pos="182:3:3" line-data="        self._items: MappingIntStrAny = items  # type: ignore">`_items`</SwmToken> attribute.

```python
    def __init__(self, value: Any, items: AbstractSetIntStr | MappingIntStrAny) -> None:
        items = self._coerce_items(items)

        if isinstance(value, (list, tuple)):
            items = self._normalize_indexes(items, len(value))  # type: ignore

        self._items: MappingIntStrAny = items  # type: ignore
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="184">

---

The function <SwmToken path="pydantic/_internal/_utils.py" pos="184:3:3" line-data="    def is_excluded(self, item: Any) -&gt; bool:">`is_excluded`</SwmToken> checks if a given item (key or index) is fully excluded by evaluating if its value in <SwmToken path="pydantic/_internal/_utils.py" pos="189:9:9" line-data="        return self.is_true(self._items.get(item))">`_items`</SwmToken> is considered 'true' (either `True` or `...`).

```python
    def is_excluded(self, item: Any) -> bool:
        """Check if item is fully excluded.

        :param item: key or index of a value
        """
        return self.is_true(self._items.get(item))

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="191">

---

The function <SwmToken path="pydantic/_internal/_utils.py" pos="191:3:3" line-data="    def is_included(self, item: Any) -&gt; bool:">`is_included`</SwmToken> determines if a given item (key or index) is present in the <SwmToken path="pydantic/_internal/_utils.py" pos="192:18:18" line-data="        &quot;&quot;&quot;Check if value is contained in self._items.">`_items`</SwmToken> mapping, indicating it is included.

```python
    def is_included(self, item: Any) -> bool:
        """Check if value is contained in self._items.

        :param item: key or index of value
        """
        return item in self._items
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="198">

---

The function <SwmToken path="pydantic/_internal/_utils.py" pos="198:3:3" line-data="    def for_element(self, e: int | str) -&gt; AbstractSetIntStr | MappingIntStrAny | None:">`for_element`</SwmToken> retrieves the raw value for a specific element (by key or index) from <SwmToken path="pydantic/_internal/_utils.py" pos="200:17:17" line-data="        :return: raw values for element if self._items is dict and contain needed element">`_items`</SwmToken>, unless the value is considered 'true', in which case it returns None.

```python
    def for_element(self, e: int | str) -> AbstractSetIntStr | MappingIntStrAny | None:
        """:param e: key or index of element on value
        :return: raw values for element if self._items is dict and contain needed element
        """
        item = self._items.get(e)  # type: ignore
        return item if not self.is_true(item) else None
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="205">

---

The function <SwmToken path="pydantic/_internal/_utils.py" pos="205:3:3" line-data="    def _normalize_indexes(self, items: MappingIntStrAny, v_length: int) -&gt; dict[int | str, Any]:">`_normalize_indexes`</SwmToken> processes a mapping of indexes or keys, normalizing negative indexes and handling the '**all**' keyword for sequences. It ensures all indexes are valid and merges values as needed.

```python
    def _normalize_indexes(self, items: MappingIntStrAny, v_length: int) -> dict[int | str, Any]:
        """:param items: dict or set of indexes which will be normalized
        :param v_length: length of sequence indexes of which will be

        >>> self._normalize_indexes({0: True, -2: True, -1: True}, 4)
        {0: True, 2: True, 3: True}
        >>> self._normalize_indexes({'__all__': True}, 4)
        {0: True, 1: True, 2: True, 3: True}
        """
        normalized_items: dict[int | str, Any] = {}
        all_items = None
        for i, v in items.items():
            if not (isinstance(v, typing.Mapping) or isinstance(v, typing.AbstractSet) or self.is_true(v)):
                raise TypeError(f'Unexpected type of exclude value for index "{i}" {v.__class__}')
            if i == '__all__':
                all_items = self._coerce_value(v)
                continue
            if not isinstance(i, int):
                raise TypeError(
                    'Excluding fields from a sequence of sub-models or dicts must be performed index-wise: '
                    'expected integer keys or keyword "__all__"'
                )
            normalized_i = v_length + i if i < 0 else i
            normalized_items[normalized_i] = self.merge(v, normalized_items.get(normalized_i))

        if not all_items:
            return normalized_items
        if self.is_true(all_items):
            for i in range(v_length):
                normalized_items.setdefault(i, ...)
            return normalized_items
        for i in range(v_length):
            normalized_item = normalized_items.setdefault(i, {})
            if not self.is_true(normalized_item):
                normalized_items[i] = self.merge(all_items, normalized_item)
        return normalized_items
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="243">

---

The class method <SwmToken path="pydantic/_internal/_utils.py" pos="243:3:3" line-data="    def merge(cls, base: Any, override: Any, intersect: bool = False) -&gt; Any:">`merge`</SwmToken> combines two items (base and override), converting sets to dictionaries and merging recursively. It supports both union and intersection of keys, depending on the <SwmToken path="pydantic/_internal/_utils.py" pos="243:20:20" line-data="    def merge(cls, base: Any, override: Any, intersect: bool = False) -&gt; Any:">`intersect`</SwmToken> flag.

```python
    def merge(cls, base: Any, override: Any, intersect: bool = False) -> Any:
        """Merge a `base` item with an `override` item.

        Both `base` and `override` are converted to dictionaries if possible.
        Sets are converted to dictionaries with the sets entries as keys and
        Ellipsis as values.

        Each key-value pair existing in `base` is merged with `override`,
        while the rest of the key-value pairs are updated recursively with this function.

        Merging takes place based on the "union" of keys if `intersect` is
        set to `False` (default) and on the intersection of keys if
        `intersect` is set to `True`.
        """
        override = cls._coerce_value(override)
        base = cls._coerce_value(base)
        if override is None:
            return base
        if cls.is_true(base) or base is None:
            return override
        if cls.is_true(override):
            return base if intersect else override

        # intersection or union of keys while preserving ordering:
        if intersect:
            merge_keys = [k for k in base if k in override] + [k for k in override if k in base]
        else:
            merge_keys = list(base) + [k for k in override if k not in base]

        merged: dict[int | str, Any] = {}
        for k in merge_keys:
            merged_item = cls.merge(base.get(k), override.get(k), intersect=intersect)
            if merged_item is not None:
                merged[k] = merged_item

        return merged
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="281">

---

The static method <SwmToken path="pydantic/_internal/_utils.py" pos="281:3:3" line-data="    def _coerce_items(items: AbstractSetIntStr | MappingIntStrAny) -&gt; MappingIntStrAny:">`_coerce_items`</SwmToken> converts sets to dictionaries with ellipsis as values and validates the input type, raising a <SwmToken path="pydantic/_internal/_utils.py" pos="288:3:3" line-data="            raise TypeError(f&#39;Unexpected type of exclude value {class_name}&#39;)">`TypeError`</SwmToken> for unsupported types.

```python
    def _coerce_items(items: AbstractSetIntStr | MappingIntStrAny) -> MappingIntStrAny:
        if isinstance(items, typing.Mapping):
            pass
        elif isinstance(items, typing.AbstractSet):
            items = dict.fromkeys(items, ...)  # type: ignore
        else:
            class_name = getattr(items, '__class__', '???')
            raise TypeError(f'Unexpected type of exclude value {class_name}')
        return items  # type: ignore
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="292">

---

The class method <SwmToken path="pydantic/_internal/_utils.py" pos="292:3:3" line-data="    def _coerce_value(cls, value: Any) -&gt; Any:">`_coerce_value`</SwmToken> returns the value if it is None or considered 'true'; otherwise, it coerces the value using <SwmToken path="pydantic/_internal/_utils.py" pos="295:5:5" line-data="        return cls._coerce_items(value)">`_coerce_items`</SwmToken>.

```python
    def _coerce_value(cls, value: Any) -> Any:
        if value is None or cls.is_true(value):
            return value
        return cls._coerce_items(value)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="298">

---

The static method <SwmToken path="pydantic/_internal/_utils.py" pos="298:3:3" line-data="    def is_true(v: Any) -&gt; bool:">`is_true`</SwmToken> determines if a value is considered 'true' for the purposes of inclusion/exclusion logic, specifically if it is `True` or `...`.

```python
    def is_true(v: Any) -> bool:
        return v is True or v is ...
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_utils.py" line="301">

---

The function <SwmToken path="pydantic/_internal/_utils.py" pos="301:3:3" line-data="    def __repr_args__(self) -&gt; _repr.ReprArgs:">`__repr_args__`</SwmToken> customizes the representation of <SwmToken path="pydantic/_internal/_utils.py" pos="171:2:2" line-data="class ValueItems(_repr.Representation):">`ValueItems`</SwmToken> instances by returning the internal <SwmToken path="pydantic/_internal/_utils.py" pos="302:10:10" line-data="        return [(None, self._items)]">`_items`</SwmToken> mapping for display.

```python
    def __repr_args__(self) -> _repr.ReprArgs:
        return [(None, self._items)]
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
