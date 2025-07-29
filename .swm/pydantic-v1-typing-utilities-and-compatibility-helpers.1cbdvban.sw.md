---
title: Pydantic v1 Typing Utilities and Compatibility Helpers
---
# Introduction

This document explains the main ideas behind the implementation of typing utilities and compatibility helpers in <SwmPath>[pydantic/v1/typing.py](pydantic/v1/typing.py)</SwmPath>. The file is designed to abstract over Python version differences and provide a consistent interface for type inspection, manipulation, and compatibility with Pydantic <SwmToken path="pydantic/v1/typing.py" pos="268:5:5" line-data="    from pydantic.v1.fields import ModelField">`v1`</SwmToken> models.

We will cover:

1. Why and how forward references are handled differently across Python versions.
2. The approach to extracting type hints and handling <SwmToken path="pydantic/v1/typing.py" pos="79:18:18" line-data="    # Ensure we always get all the whole `Annotated` hint, not just the annotated type.">`Annotated`</SwmToken> types.
3. How generic types and their arguments are normalized across Python versions.
4. The logic for identifying special typing constructs like <SwmToken path="pydantic/v1/typing.py" pos="332:0:0" line-data="NoneType = None.__class__">`NoneType`</SwmToken>, <SwmToken path="pydantic/v1/typing.py" pos="335:23:23" line-data="NONE_TYPES: Tuple[Any, Any, Any] = (None, NoneType, Literal[None])">`Literal`</SwmToken>, <SwmToken path="pydantic/v1/typing.py" pos="496:9:9" line-data="    return v.__class__ == ClassVar.__class__ and getattr(v, &#39;_name&#39;, None) == &#39;ClassVar&#39;">`ClassVar`</SwmToken>, and <SwmToken path="pydantic/v1/typing.py" pos="501:18:18" line-data="    Check if a given type is a `typing.Final` type.">`Final`</SwmToken>.
5. How forward references are resolved and updated in Pydantic fields and models.

# Forward reference evaluation across Python versions

<SwmSnippet path="/pydantic/v1/typing.py" line="58">

---

Forward references (<SwmToken path="pydantic/v1/typing.py" pos="58:8:8" line-data="    def evaluate_forwardref(type_: ForwardRef, globalns: Any, localns: Any) -&gt; Any:">`ForwardRef`</SwmToken>) are used to refer to types not yet defined. Their evaluation changed across Python versions, so the code provides version-specific implementations to ensure consistent behavior. For example, Python <SwmToken path="pydantic/v1/typing.py" pos="66:9:13" line-data="        # Python 3.13/3.12.4+ made `recursive_guard` a kwarg, so name it explicitly to avoid:">`3.12.4`</SwmToken>+ introduced a <SwmToken path="pydantic/v1/typing.py" pos="66:19:19" line-data="        # Python 3.13/3.12.4+ made `recursive_guard` a kwarg, so name it explicitly to avoid:">`recursive_guard`</SwmToken> keyword argument, and Python <SwmToken path="pydantic/v1/typing.py" pos="66:5:7" line-data="        # Python 3.13/3.12.4+ made `recursive_guard` a kwarg, so name it explicitly to avoid:">`3.13`</SwmToken>+ added <SwmToken path="pydantic/v1/typing.py" pos="73:27:27" line-data="        # Pydantic 1.x will not support PEP 695 syntax, but provide `type_params` to avoid">`type_params`</SwmToken>. The code adapts to these changes to avoid runtime errors and warnings.

```python
    def evaluate_forwardref(type_: ForwardRef, globalns: Any, localns: Any) -> Any:
        return type_._evaluate(globalns, localns)

elif sys.version_info < (3, 12, 4):

    def evaluate_forwardref(type_: ForwardRef, globalns: Any, localns: Any) -> Any:
        # Even though it is the right signature for python 3.9, mypy complains with
        # `error: Too many arguments for "_evaluate" of "ForwardRef"` hence the cast...
        # Python 3.13/3.12.4+ made `recursive_guard` a kwarg, so name it explicitly to avoid:
        # TypeError: ForwardRef._evaluate() missing 1 required keyword-only argument: 'recursive_guard'
        return cast(Any, type_)._evaluate(globalns, localns, recursive_guard=set())

else:

    def evaluate_forwardref(type_: ForwardRef, globalns: Any, localns: Any) -> Any:
        # Pydantic 1.x will not support PEP 695 syntax, but provide `type_params` to avoid
        # warnings:
        return cast(Any, type_)._evaluate(globalns, localns, type_params=(), recursive_guard=set())
```

---

</SwmSnippet>

# Extracting all type hints, including <SwmToken path="pydantic/v1/typing.py" pos="79:18:18" line-data="    # Ensure we always get all the whole `Annotated` hint, not just the annotated type.">`Annotated`</SwmToken>

<SwmSnippet path="/pydantic/v1/typing.py" line="78">

---

Type hints can include extra information, such as with <SwmToken path="pydantic/v1/typing.py" pos="79:18:18" line-data="    # Ensure we always get all the whole `Annotated` hint, not just the annotated type.">`Annotated`</SwmToken>. Python <SwmToken path="pydantic/v1/typing.py" pos="64:21:23" line-data="        # Even though it is the right signature for python 3.9, mypy complains with">`3.9`</SwmToken>+ supports the <SwmToken path="pydantic/v1/typing.py" pos="87:14:14" line-data="        return get_type_hints(obj, globalns, localns, include_extras=True)">`include_extras`</SwmToken> argument in <SwmToken path="pydantic/v1/typing.py" pos="80:17:17" line-data="    # For 3.7 to 3.8, `get_type_hints` doesn&#39;t recognize `typing_extensions.Annotated`,">`get_type_hints`</SwmToken>, but older versions do not. The code provides a fallback to ensure all annotation metadata is available, regardless of Python version.

```python
if sys.version_info < (3, 9):
    # Ensure we always get all the whole `Annotated` hint, not just the annotated type.
    # For 3.7 to 3.8, `get_type_hints` doesn't recognize `typing_extensions.Annotated`,
    # so it already returns the full annotation
    get_all_type_hints = get_type_hints

else:

    def get_all_type_hints(obj: Any, globalns: Any = None, localns: Any = None) -> Any:
        return get_type_hints(obj, globalns, localns, include_extras=True)
```

---

</SwmSnippet>

# Normalizing generic types and arguments

<SwmSnippet path="/pydantic/v1/typing.py" line="109">

---

Python's typing internals changed significantly between versions, especially with the introduction of <SwmToken path="pydantic/v1/typing.py" pos="226:18:18" line-data="        # recursively replace `str` instances inside of `GenericAlias` with `ForwardRef(arg)`">`GenericAlias`</SwmToken> and changes to how generic arguments are stored. The code provides custom <SwmToken path="pydantic/v1/typing.py" pos="111:3:3" line-data="    def get_origin(t: Type[Any]) -&gt; Optional[Type[Any]]:">`get_origin`</SwmToken> and <SwmToken path="pydantic/v1/typing.py" pos="135:3:3" line-data="    def get_args(t: Type[Any]) -&gt; Tuple[Any, ...]:">`get_args`</SwmToken> functions to extract the base type and arguments from generic types, handling edge cases like <SwmToken path="pydantic/v1/typing.py" pos="114:11:11" line-data="            return cast(Type[Any], Annotated)">`Annotated`</SwmToken> and empty generics.

```python
if sys.version_info < (3, 8):

    def get_origin(t: Type[Any]) -> Optional[Type[Any]]:
        if type(t).__name__ in AnnotatedTypeNames:
            # weirdly this is a runtime requirement, as well as for mypy
            return cast(Type[Any], Annotated)
        return getattr(t, '__origin__', None)

else:
    from typing import get_origin as _typing_get_origin

    def get_origin(tp: Type[Any]) -> Optional[Type[Any]]:
        """
        We can't directly use `typing.get_origin` since we need a fallback to support
        custom generic classes like `ConstrainedList`
        It should be useless once https://github.com/cython/cython/issues/3537 is
        solved and https://github.com/pydantic/pydantic/pull/1753 is merged.
        """
        if type(tp).__name__ in AnnotatedTypeNames:
            return cast(Type[Any], Annotated)  # mypy complains about _SpecialForm
        return _typing_get_origin(tp) or getattr(tp, '__origin__', None)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="132">

---

&nbsp;

```python
if sys.version_info < (3, 8):
    from typing import _GenericAlias

    def get_args(t: Type[Any]) -> Tuple[Any, ...]:
        """Compatibility version of get_args for python 3.7.

        Mostly compatible with the python 3.8 `typing` module version
        and able to handle almost all use cases.
        """
        if type(t).__name__ in AnnotatedTypeNames:
            return t.__args__ + t.__metadata__
        if isinstance(t, _GenericAlias):
            res = t.__args__
            if t.__origin__ is Callable and res and res[0] is not Ellipsis:
                res = (list(res[:-1]), res[-1])
            return res
        return getattr(t, '__args__', ())

else:
    from typing import get_args as _typing_get_args

    def _generic_get_args(tp: Type[Any]) -> Tuple[Any, ...]:
        """
        In python 3.9, `typing.Dict`, `typing.List`, ...
        do have an empty `__args__` by default (instead of the generic ~T for example).
        In order to still support `Dict` for example and consider it as `Dict[Any, Any]`,
        we retrieve the `_nparams` value that tells us how many parameters it needs.
        """
        if hasattr(tp, '_nparams'):
            return (Any,) * tp._nparams
        # Special case for `tuple[()]`, which used to return ((),) with `typing.Tuple`
        # in python 3.10- but now returns () for `tuple` and `Tuple`.
        # This will probably be clarified in pydantic v2
        try:
            if tp == Tuple[()] or sys.version_info >= (3, 9) and tp == tuple[()]:  # type: ignore[misc]
                return ((),)
        # there is a TypeError when compiled with cython
        except TypeError:  # pragma: no cover
            pass
        return ()

    def get_args(tp: Type[Any]) -> Tuple[Any, ...]:
        """Get type arguments with all substitutions performed.

        For unions, basic simplifications used by Union constructor are performed.
        Examples::
            get_args(Dict[str, int]) == (str, int)
            get_args(int) == ()
            get_args(Union[int, Union[T, int], str][int]) == (int, str)
            get_args(Union[int, Tuple[T, int]][str]) == (int, Tuple[str, int])
            get_args(Callable[[], T][int]) == ([], int)
        """
        if type(tp).__name__ in AnnotatedTypeNames:
            return tp.__args__ + tp.__metadata__
        # the fallback is needed for the same reasons as `get_origin` (see above)
        return _typing_get_args(tp) or getattr(tp, '__args__', ()) or _generic_get_args(tp)
```

---

</SwmSnippet>

# Handling generics with string type hints

<SwmSnippet path="/pydantic/v1/typing.py" line="201">

---

Python <SwmToken path="pydantic/v1/typing.py" pos="64:21:23" line-data="        # Even though it is the right signature for python 3.9, mypy complains with">`3.9`</SwmToken>+ allows generics like <SwmToken path="pydantic/v1/typing.py" pos="211:3:8" line-data="            convert_generics(list[&#39;Hero&#39;]) == list[ForwardRef(&#39;Hero&#39;)]">`list['Hero']`</SwmToken>, but these need to be converted to <SwmToken path="pydantic/v1/typing.py" pos="208:23:23" line-data="        Recursively searches for `str` type hints and replaces them with ForwardRef.">`ForwardRef`</SwmToken> for consistency. The code recursively replaces string arguments with <SwmToken path="pydantic/v1/typing.py" pos="208:23:23" line-data="        Recursively searches for `str` type hints and replaces them with ForwardRef.">`ForwardRef`</SwmToken> objects, ensuring that type resolution works the same way across all supported Python versions.

```python
else:
    from typing import _UnionGenericAlias  # type: ignore

    from typing_extensions import _AnnotatedAlias

    def convert_generics(tp: Type[Any]) -> Type[Any]:
        """
        Recursively searches for `str` type hints and replaces them with ForwardRef.

        Examples::
            convert_generics(list['Hero']) == list[ForwardRef('Hero')]
            convert_generics(dict['Hero', 'Team']) == dict[ForwardRef('Hero'), ForwardRef('Team')]
            convert_generics(typing.Dict['Hero', 'Team']) == typing.Dict[ForwardRef('Hero'), ForwardRef('Team')]
            convert_generics(list[str | 'Hero'] | int) == list[str | ForwardRef('Hero')] | int
        """
        origin = get_origin(tp)
        if not origin or not hasattr(tp, '__args__'):
            return tp

        args = get_args(tp)

        # typing.Annotated needs special treatment
        if origin is Annotated:
            return _AnnotatedAlias(convert_generics(args[0]), args[1:])

        # recursively replace `str` instances inside of `GenericAlias` with `ForwardRef(arg)`
        converted = tuple(
            ForwardRef(arg) if isinstance(arg, str) and isinstance(tp, TypingGenericAlias) else convert_generics(arg)
            for arg in args
        )

        if converted == args:
            return tp
        elif isinstance(tp, TypingGenericAlias):
            return TypingGenericAlias(origin, converted)
        elif isinstance(tp, TypesUnionType):
            # recreate types.UnionType (PEP604, Python >= 3.10)
            return _UnionGenericAlias(origin, converted)
        else:
            try:
                setattr(tp, '__args__', converted)
            except AttributeError:
                pass
            return tp
```

---

</SwmSnippet>

# Identifying special typing constructs

<SwmSnippet path="/pydantic/v1/typing.py" line="332">

---

The file provides helpers to identify types like <SwmToken path="pydantic/v1/typing.py" pos="332:0:0" line-data="NoneType = None.__class__">`NoneType`</SwmToken>, <SwmToken path="pydantic/v1/typing.py" pos="335:23:23" line-data="NONE_TYPES: Tuple[Any, Any, Any] = (None, NoneType, Literal[None])">`Literal`</SwmToken>, <SwmToken path="pydantic/v1/typing.py" pos="496:9:9" line-data="    return v.__class__ == ClassVar.__class__ and getattr(v, &#39;_name&#39;, None) == &#39;ClassVar&#39;">`ClassVar`</SwmToken>, and <SwmToken path="pydantic/v1/typing.py" pos="501:18:18" line-data="    Check if a given type is a `typing.Final` type.">`Final`</SwmToken>, which are important for Pydantic's data validation logic. These helpers account for subtle differences in how these constructs are represented in different Python versions and typing backends.

```python
NoneType = None.__class__


NONE_TYPES: Tuple[Any, Any, Any] = (None, NoneType, Literal[None])


if sys.version_info < (3, 8):
    # Even though this implementation is slower, we need it for python 3.7:
    # In python 3.7 "Literal" is not a builtin type and uses a different
    # mechanism.
    # for this reason `Literal[None] is Literal[None]` evaluates to `False`,
    # breaking the faster implementation used for the other python versions.

    def is_none_type(type_: Any) -> bool:
        return type_ in NONE_TYPES

elif sys.version_info[:2] == (3, 8):

    def is_none_type(type_: Any) -> bool:
        for none_type in NONE_TYPES:
            if type_ is none_type:
                return True
        # With python 3.8, specifically 3.8.10, Literal "is" check sare very flakey
        # can change on very subtle changes like use of types in other modules,
        # hopefully this check avoids that issue.
        if is_literal_type(type_):  # pragma: no cover
            return all_literal_values(type_) == (None,)
        return False

else:

    def is_none_type(type_: Any) -> bool:
        return type_ in NONE_TYPES


def display_as_type(v: Type[Any]) -> str:
    if not isinstance(v, typing_base) and not isinstance(v, WithArgsTypes) and not isinstance(v, type):
        v = v.__class__
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="496">

---

&nbsp;

```python
    return v.__class__ == ClassVar.__class__ and getattr(v, '_name', None) == 'ClassVar'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="499">

---

&nbsp;

```python
def _check_finalvar(v: Optional[Type[Any]]) -> bool:
    """
    Check if a given type is a `typing.Final` type.
    """
    if v is None:
        return False

    return v.__class__ == Final.__class__ and (sys.version_info < (3, 8) or getattr(v, '_name', None) == 'Final')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="509">

---

&nbsp;

```python
def is_classvar(ann_type: Type[Any]) -> bool:
    if _check_classvar(ann_type) or _check_classvar(get_origin(ann_type)):
        return True

    # this is an ugly workaround for class vars that contain forward references and are therefore themselves
    # forward references, see #3679
    if ann_type.__class__ == ForwardRef and ann_type.__forward_arg__.startswith('ClassVar['):
        return True
```

---

</SwmSnippet>

# Resolving and updating forward references in fields and models

<SwmSnippet path="/pydantic/v1/typing.py" line="385">

---

Pydantic models often use forward references, especially for recursive or mutually-referential models. The code provides utilities to resolve these references at runtime, updating fields and model configurations as needed. This ensures that type information is always accurate, even when models are defined out of order.

```python
def resolve_annotations(raw_annotations: Dict[str, Type[Any]], module_name: Optional[str]) -> Dict[str, Type[Any]]:
    """
    Partially taken from typing.get_type_hints.

    Resolve string or ForwardRef annotations into type objects if possible.
    """
    base_globals: Optional[Dict[str, Any]] = None
    if module_name:
        try:
            module = sys.modules[module_name]
        except KeyError:
            # happens occasionally, see https://github.com/pydantic/pydantic/issues/2363
            pass
        else:
            base_globals = module.__dict__

    annotations = {}
    for name, value in raw_annotations.items():
        if isinstance(value, str):
            if (3, 10) > sys.version_info >= (3, 9, 8) or sys.version_info >= (3, 10, 1):
                value = ForwardRef(value, is_argument=False, is_class=True)
            else:
                value = ForwardRef(value, is_argument=False)
        try:
            if sys.version_info >= (3, 13):
                value = _eval_type(value, base_globals, None, type_params=())
            else:
                value = _eval_type(value, base_globals, None)
        except NameError:
            # this is ok, it can be fixed with update_forward_refs
            pass
        annotations[name] = value
    return annotations
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="521">

---

&nbsp;

```python
def is_finalvar(ann_type: Type[Any]) -> bool:
    return _check_finalvar(ann_type) or _check_finalvar(get_origin(ann_type))


def update_field_forward_refs(field: 'ModelField', globalns: Any, localns: Any) -> None:
    """
    Try to update ForwardRefs on fields based on this ModelField, globalns and localns.
    """
    prepare = False
    if field.type_.__class__ == ForwardRef:
        prepare = True
        field.type_ = evaluate_forwardref(field.type_, globalns, localns or None)
    if field.outer_type_.__class__ == ForwardRef:
        prepare = True
        field.outer_type_ = evaluate_forwardref(field.outer_type_, globalns, localns or None)
    if prepare:
        field.prepare()

    if field.sub_fields:
        for sub_f in field.sub_fields:
            update_field_forward_refs(sub_f, globalns=globalns, localns=localns)

    if field.discriminator_key is not None:
        field.prepare_discriminated_union_sub_fields()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/typing.py" line="547">

---

&nbsp;

```python
def update_model_forward_refs(
    model: Type[Any],
    fields: Iterable['ModelField'],
    json_encoders: Dict[Union[Type[Any], str, ForwardRef], AnyCallable],
    localns: 'DictStrAny',
    exc_to_suppress: Tuple[Type[BaseException], ...] = (),
) -> None:
    """
    Try to update model fields ForwardRefs based on model and localns.
    """
    if model.__module__ in sys.modules:
        globalns = sys.modules[model.__module__].__dict__.copy()
    else:
        globalns = {}

    globalns.setdefault(model.__name__, model)

    for f in fields:
        try:
            update_field_forward_refs(f, globalns=globalns, localns=localns)
        except exc_to_suppress:
            pass

    for key in set(json_encoders.keys()):
        if isinstance(key, str):
            fr: ForwardRef = ForwardRef(key)
        elif isinstance(key, ForwardRef):
            fr = key
        else:
            continue

        try:
            new_key = evaluate_forwardref(fr, globalns, localns or None)
        except exc_to_suppress:  # pragma: no cover
            continue

        json_encoders[new_key] = json_encoders.pop(key)
```

---

</SwmSnippet>

# Extracting subtypes from complex type hints

<SwmSnippet path="/pydantic/v1/typing.py" line="604">

---

For validation and schema generation, it's often necessary to extract all possible subtypes from a type hint, including those nested in <SwmToken path="pydantic/v1/typing.py" pos="607:12:12" line-data="    `tp` can be a `Union` of allowed types or an `Annotated` type">`Union`</SwmToken> or <SwmToken path="pydantic/v1/typing.py" pos="607:26:26" line-data="    `tp` can be a `Union` of allowed types or an `Annotated` type">`Annotated`</SwmToken>. The code provides a recursive utility to flatten these structures.

```python
def get_sub_types(tp: Any) -> List[Any]:
    """
    Return all the types that are allowed by type `tp`
    `tp` can be a `Union` of allowed types or an `Annotated` type
    """
    origin = get_origin(tp)
    if origin is Annotated:
        return get_sub_types(get_args(tp)[0])
    elif is_union(origin):
        return [x for t in get_args(tp) for x in get_sub_types(t)]
    else:
        return [tp]
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
