---
title: Advanced Typing Utilities and Annotation Handling
---
# Introduction

This document explains the main ideas behind the implementation of advanced typing utilities and annotation handling in <SwmPath>[pydantic/\_internal/\_typing_extra.py](pydantic/_internal/_typing_extra.py)</SwmPath>. The file provides helpers for working with Python type annotations, especially to support features and edge cases across Python versions.

We will cover:

1. Why and how we detect special typing constructs like <SwmToken path="pydantic/_internal/_typing_extra.py" pos="41:6:6" line-data="_t_annotated = typing.Annotated">`Annotated`</SwmToken>, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="97:28:28" line-data="_classvar_re = re.compile(r&#39;((\w+\.)?Annotated\[)?(\w+\.)?ClassVar\[&#39;)">`ClassVar`</SwmToken>, and <SwmToken path="pydantic/_internal/_typing_extra.py" pos="138:6:6" line-data="_t_final = typing.Final">`Final`</SwmToken>.
2. How annotation evaluation is made robust across Python versions and error scenarios.
3. The approach to resolving namespaces and forward references for type hints.
4. How type hints are collected from classes and functions, including handling of edge cases and compatibility.

# Detecting special typing constructs

The code needs to reliably identify constructs like <SwmToken path="pydantic/_internal/_typing_extra.py" pos="41:6:6" line-data="_t_annotated = typing.Annotated">`Annotated`</SwmToken>, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="97:28:28" line-data="_classvar_re = re.compile(r&#39;((\w+\.)?Annotated\[)?(\w+\.)?ClassVar\[&#39;)">`ClassVar`</SwmToken>, and <SwmToken path="pydantic/_internal/_typing_extra.py" pos="138:6:6" line-data="_t_final = typing.Final">`Final`</SwmToken>, even when wrapped or coming from different modules (<SwmToken path="pydantic/_internal/_typing_extra.py" pos="41:4:4" line-data="_t_annotated = typing.Annotated">`typing`</SwmToken> vs <SwmToken path="pydantic/_internal/_typing_extra.py" pos="42:4:4" line-data="_te_annotated = typing_extensions.Annotated">`typing_extensions`</SwmToken>). This is necessary for correct model field detection and validation logic.

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="41">

---

The function <SwmToken path="pydantic/_internal/_typing_extra.py" pos="45:2:2" line-data="def is_annotated(tp: Any, /) -&gt; bool:">`is_annotated`</SwmToken> checks if a type is an <SwmToken path="pydantic/_internal/_typing_extra.py" pos="41:6:6" line-data="_t_annotated = typing.Annotated">`Annotated`</SwmToken> special form, supporting both <SwmToken path="pydantic/_internal/_typing_extra.py" pos="41:4:4" line-data="_t_annotated = typing.Annotated">`typing`</SwmToken> and <SwmToken path="pydantic/_internal/_typing_extra.py" pos="42:4:4" line-data="_te_annotated = typing_extensions.Annotated">`typing_extensions`</SwmToken> variants. This avoids subtle bugs when users mix imports.

````python
_t_annotated = typing.Annotated
_te_annotated = typing_extensions.Annotated


def is_annotated(tp: Any, /) -> bool:
    """Return whether the provided argument is a `Annotated` special form.

    ```python {test="skip" lint="skip"}
    is_annotated(Annotated[int, ...])
    #> True
    ```
    """
    origin = get_origin(tp)
    return origin is _t_annotated or origin is _te_annotated
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="97">

---

Similarly, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="100:2:2" line-data="def is_classvar_annotation(tp: Any, /) -&gt; bool:">`is_classvar_annotation`</SwmToken> is more involved. It detects <SwmToken path="pydantic/_internal/_typing_extra.py" pos="97:28:28" line-data="_classvar_re = re.compile(r&#39;((\w+\.)?Annotated\[)?(\w+\.)?ClassVar\[&#39;)">`ClassVar`</SwmToken> even when wrapped in <SwmToken path="pydantic/_internal/_typing_extra.py" pos="97:17:17" line-data="_classvar_re = re.compile(r&#39;((\w+\.)?Annotated\[)?(\w+\.)?ClassVar\[&#39;)">`Annotated`</SwmToken>, or when the annotation is a string (forward reference). It uses both type inspection and a regex fallback for string annotations. This is needed because class variables must be detected before evaluating forward references, and Python's typing system is inconsistent here.

```python
_classvar_re = re.compile(r'((\w+\.)?Annotated\[)?(\w+\.)?ClassVar\[')


def is_classvar_annotation(tp: Any, /) -> bool:
    """Return whether the provided argument represents a class variable annotation.

    Although not explicitly stated by the typing specification, `ClassVar` can be used
    inside `Annotated` and as such, this function checks for this specific scenario.

    Because this function is used to detect class variables before evaluating forward references
    (or because evaluation failed), we also implement a naive regex match implementation. This is
    required because class variables are inspected before fields are collected, so we try to be
    as accurate as possible.
    """
    if typing_objects.is_classvar(tp):
        return True

    origin = get_origin(tp)

    if typing_objects.is_classvar(origin):
        return True

    if typing_objects.is_annotated(origin):
        annotated_type = tp.__origin__
        if typing_objects.is_classvar(annotated_type) or typing_objects.is_classvar(get_origin(annotated_type)):
            return True

    str_ann: str | None = None
    if isinstance(tp, typing.ForwardRef):
        str_ann = tp.__forward_arg__
    if isinstance(tp, str):
        str_ann = tp

    if str_ann is not None and _classvar_re.match(str_ann):
        # stdlib dataclasses do something similar, although a bit more advanced
        # (see `dataclass._is_type`).
        return True

    return False
```

---

</SwmSnippet>

# Robust annotation evaluation

Evaluating type annotations is tricky: forward references, missing symbols, and new typing syntax can all cause failures. The code provides a layered approach to evaluation, with graceful fallback and error reporting.

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="384">

---

The function <SwmToken path="pydantic/_internal/_typing_extra.py" pos="384:2:2" line-data="def try_eval_type(">`try_eval_type`</SwmToken> attempts to evaluate an annotation, returning both the result and a success flag. If evaluation fails (<SwmToken path="pydantic/_internal/_typing_extra.py" pos="234:30:32" line-data="    this only allows us to fetch the parent frame namespace and not other parents (e.g. a model defined in a function,">`e.g`</SwmToken>., due to a missing symbol), it returns the original value and `False`. This lets Pydantic distinguish between successfully and unsuccessfully resolved types, which is important for error messages and deferred evaluation.

```python
def try_eval_type(
    value: Any,
    globalns: GlobalsNamespace | None = None,
    localns: MappingNamespace | None = None,
) -> tuple[Any, bool]:
    """Try evaluating the annotation using the provided namespaces.

    Args:
        value: The value to evaluate. If `None`, it will be replaced by `type[None]`. If an instance
            of `str`, it will be converted to a `ForwardRef`.
        localns: The global namespace to use during annotation evaluation.
        globalns: The local namespace to use during annotation evaluation.

    Returns:
        A two-tuple containing the possibly evaluated type and a boolean indicating
            whether the evaluation succeeded or not.
    """
    value = _type_convert(value)

    try:
        return eval_type_backport(value, globalns, localns), True
    except NameError:
        return value, False
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="439">

---

For actual evaluation, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="439:2:2" line-data="def eval_type_backport(">`eval_type_backport`</SwmToken> is used. It tries to use the standard library's <SwmToken path="pydantic/_internal/_typing_extra.py" pos="445:15:15" line-data="    &quot;&quot;&quot;An enhanced version of `typing._eval_type` which will fall back to using the `eval_type_backport`">`_eval_type`</SwmToken>, but if that fails (<SwmToken path="pydantic/_internal/_typing_extra.py" pos="474:23:25" line-data="            &quot;If you made use of an implicit recursive type alias (e.g. `MyType = list[&#39;MyType&#39;]), &quot;">`e.g`</SwmToken>., due to new syntax on old Python), it falls back to the <SwmToken path="pydantic/_internal/_typing_extra.py" pos="439:2:2" line-data="def eval_type_backport(">`eval_type_backport`</SwmToken> package if available. This allows Pydantic to support new typing features even on older Python versions, and gives clear errors if evaluation is impossible.

```python
def eval_type_backport(
    value: Any,
    globalns: GlobalsNamespace | None = None,
    localns: MappingNamespace | None = None,
    type_params: tuple[Any, ...] | None = None,
) -> Any:
    """An enhanced version of `typing._eval_type` which will fall back to using the `eval_type_backport`
    package if it's installed to let older Python versions use newer typing constructs.

    Specifically, this transforms `X | Y` into `typing.Union[X, Y]` and `list[X]` into `typing.List[X]`
    (as well as all the types made generic in PEP 585) if the original syntax is not supported in the
    current Python version.

    This function will also display a helpful error if the value passed fails to evaluate.
    """
    try:
        return _eval_type_backport(value, globalns, localns, type_params)
    except TypeError as e:
        if 'Unable to evaluate type annotation' in str(e):
            raise

        # If it is a `TypeError` and value isn't a `ForwardRef`, it would have failed during annotation definition.
        # Thus we assert here for type checking purposes:
        assert isinstance(value, typing.ForwardRef)

        message = f'Unable to evaluate type annotation {value.__forward_arg__!r}.'
        if sys.version_info >= (3, 11):
            e.add_note(message)
            raise
        else:
            raise TypeError(message) from e
    except RecursionError as e:
        # TODO ideally recursion errors should be checked in `eval_type` above, but `eval_type_backport`
        # is used directly in some places.
        message = (
            "If you made use of an implicit recursive type alias (e.g. `MyType = list['MyType']), "
            'consider using PEP 695 type aliases instead. For more details, refer to the documentation: '
            f'https://docs.pydantic.dev/{version_short()}/concepts/types/#named-recursive-types'
        )
        if sys.version_info >= (3, 11):
            e.add_note(message)
            raise
        else:
            raise RecursionError(f'{e.args[0]}\n{message}')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="485">

---

&nbsp;

```python
def _eval_type_backport(
    value: Any,
    globalns: GlobalsNamespace | None = None,
    localns: MappingNamespace | None = None,
    type_params: tuple[Any, ...] | None = None,
) -> Any:
    try:
        return _eval_type(value, globalns, localns, type_params)
    except TypeError as e:
        if not (isinstance(value, typing.ForwardRef) and is_backport_fixable_error(e)):
            raise

        try:
            from eval_type_backport import eval_type_backport
        except ImportError:
            raise TypeError(
                f'Unable to evaluate type annotation {value.__forward_arg__!r}. If you are making use '
                'of the new typing syntax (unions using `|` since Python 3.10 or builtins subscripting '
                'since Python 3.9), you should either replace the use of new syntax with the existing '
                '`typing` constructs or install the `eval_type_backport` package.'
            ) from e

        return eval_type_backport(
            value,
            globalns,
            localns,  # pyright: ignore[reportArgumentType], waiting on a new `eval_type_backport` release.
            try_default=False,
        )
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="515">

---

&nbsp;

```python
def _eval_type(
    value: Any,
    globalns: GlobalsNamespace | None = None,
    localns: MappingNamespace | None = None,
    type_params: tuple[Any, ...] | None = None,
) -> Any:
    if sys.version_info >= (3, 13):
        return typing._eval_type(  # type: ignore
            value, globalns, localns, type_params=type_params
        )
    else:
        return typing._eval_type(  # type: ignore
            value, globalns, localns
        )
```

---

</SwmSnippet>

# Namespace and forward reference resolution

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="206">

---

Type annotations often refer to names not yet defined, or defined in local scopes. The function <SwmToken path="pydantic/_internal/_typing_extra.py" pos="213:2:2" line-data="def parent_frame_namespace(*, parent_depth: int = 2, force: bool = False) -&gt; dict[str, Any] | None:">`parent_frame_namespace`</SwmToken> fetches the local namespace of the parent frame, which is needed to resolve forward references in models defined inside functions. This is a workaround for limitations in Python's standard library, which can't always resolve such references.

````python
# Similarly, we shouldn't rely on this `_Final` class, which is even more private than `_GenericAlias`:
typing_base: Any = typing._Final  # pyright: ignore[reportAttributeAccessIssue]


### Annotation evaluations functions:


def parent_frame_namespace(*, parent_depth: int = 2, force: bool = False) -> dict[str, Any] | None:
    """Fetch the local namespace of the parent frame where this function is called.

    Using this function is mostly useful to resolve forward annotations pointing to members defined in a local namespace,
    such as assignments inside a function. Using the standard library tools, it is currently not possible to resolve
    such annotations:

    ```python {lint="skip" test="skip"}
    from typing import get_type_hints

    def func() -> None:
        Alias = int

        class C:
            a: 'Alias'

        # Raises a `NameError: 'Alias' is not defined`
        get_type_hints(C)
    ```

    Pydantic uses this function when a Pydantic model is being defined to fetch the parent frame locals. However,
    this only allows us to fetch the parent frame namespace and not other parents (e.g. a model defined in a function,
    itself defined in another function). Inspecting the next outer frames (using `f_back`) is not reliable enough
    (see https://discuss.python.org/t/20659).

    Because this function is mostly used to better resolve forward annotations, nothing is returned if the parent frame's
    code object is defined at the module level. In this case, the locals of the frame will be the same as the module
    globals where the class is defined (see `_namespace_utils.get_module_ns_of`). However, if you still want to fetch
    the module globals (e.g. when rebuilding a model, where the frame where the rebuild call is performed might contain
    members that you want to use for forward annotations evaluation), you can use the `force` parameter.

    Args:
        parent_depth: The depth at which to get the frame. Defaults to 2, meaning the parent frame where this function
            is called will be used.
        force: Whether to always return the frame locals, even if the frame's code object is defined at the module level.

    Returns:
        The locals of the namespace, or `None` if it was skipped as per the described logic.
    """
    frame = sys._getframe(parent_depth)

    if frame.f_code.co_name.startswith('<generic parameters of'):
        # As `parent_frame_namespace` is mostly called in `ModelMetaclass.__new__`,
        # the parent frame can be the annotation scope if the PEP 695 generic syntax is used.
        # (see https://docs.python.org/3/reference/executionmodel.html#annotation-scopes,
        # https://docs.python.org/3/reference/compound_stmts.html#generic-classes).
        # In this case, the code name is set to `<generic parameters of MyClass>`,
        # and we need to skip this frame as it is irrelevant.
        frame = cast(types.FrameType, frame.f_back)  # guaranteed to not be `None`

    # note, we don't copy frame.f_locals here (or during the last return call), because we don't expect the namespace to be
    # modified down the line if this becomes a problem, we could implement some sort of frozen mapping structure to enforce this.
    if force:
        return frame.f_locals

    # If either of the following conditions are true, the class is defined at the top module level.
    # To better understand why we need both of these checks, see
    # https://github.com/pydantic/pydantic/pull/10113#discussion_r1714981531.
    if frame.f_back is None or frame.f_code.co_name == '<module>':
        return None

    return frame.f_locals
````

---

</SwmSnippet>

# Collecting type hints from classes and functions

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="308">

---

To build models, Pydantic needs to collect type hints from classes, including parent classes, and handle forward references and evaluation errors. The function <SwmToken path="pydantic/_internal/_typing_extra.py" pos="308:2:2" line-data="def get_model_type_hints(">`get_model_type_hints`</SwmToken> walks the MRO, collects annotations, and tries to evaluate them, recording which succeed and which fail. This is more robust than the standard library, especially for private attributes and advanced typing features.

```python
def get_model_type_hints(
    obj: type[BaseModel],
    *,
    ns_resolver: NsResolver | None = None,
) -> dict[str, tuple[Any, bool]]:
    """Collect annotations from a Pydantic model class, including those from parent classes.

    Args:
        obj: The Pydantic model to inspect.
        ns_resolver: A namespace resolver instance to use. Defaults to an empty instance.

    Returns:
        A dictionary mapping annotation names to a two-tuple: the first element is the evaluated
        type or the original annotation if a `NameError` occurred, the second element is a boolean
        indicating if whether the evaluation succeeded.
    """
    hints: dict[str, Any] | dict[str, tuple[Any, bool]] = {}
    ns_resolver = ns_resolver or NsResolver()

    for base in reversed(obj.__mro__):
        # For Python 3.14, we could also use `Format.VALUE` and pass the globals/locals
        # from the ns_resolver, but we want to be able to know which specific field failed
        # to evaluate:
        ann = safe_get_annotations(base)

        if not ann:
            continue

        with ns_resolver.push(base):
            globalns, localns = ns_resolver.types_namespace
            for name, value in ann.items():
                if name.startswith('_'):
                    # For private attributes, we only need the annotation to detect the `ClassVar` special form.
                    # For this reason, we still try to evaluate it, but we also catch any possible exception (on
                    # top of the `NameError`s caught in `try_eval_type`) that could happen so that users are free
                    # to use any kind of forward annotation for private fields (e.g. circular imports, new typing
                    # syntax, etc).
                    try:
                        hints[name] = try_eval_type(value, globalns, localns)
                    except Exception:
                        hints[name] = (value, False)
                else:
                    hints[name] = try_eval_type(value, globalns, localns)
    return hints
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="531">

---

For functions, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="537:2:2" line-data="def get_function_type_hints(">`get_function_type_hints`</SwmToken> collects type hints, supporting <SwmToken path="pydantic/_internal/_typing_extra.py" pos="547:6:8" line-data="    - Support `functools.partial` by using the underlying `func` attribute.">`functools.partial`</SwmToken>, and avoids wrapping types with <SwmToken path="pydantic/_internal/_typing_extra.py" pos="548:22:22" line-data="    - Do not wrap type annotation of a parameter with `Optional` if it has a default value of `None`">`Optional`</SwmToken> just because a default value is `None` (a bug in Python <<SwmToken path="pydantic/_internal/_typing_extra.py" pos="549:28:30" line-data="      (related bug: https://github.com/python/cpython/issues/90353, only fixed in 3.11+).">`3.11`</SwmToken>). It also handles forward references and uses the correct namespace for evaluation.

```python
def is_backport_fixable_error(e: TypeError) -> bool:
    msg = str(e)

    return sys.version_info < (3, 10) and msg.startswith('unsupported operand type(s) for |: ')


def get_function_type_hints(
    function: Callable[..., Any],
    *,
    include_keys: set[str] | None = None,
    globalns: GlobalsNamespace | None = None,
    localns: MappingNamespace | None = None,
) -> dict[str, Any]:
    """Return type hints for a function.

    This is similar to the `typing.get_type_hints` function, with a few differences:
    - Support `functools.partial` by using the underlying `func` attribute.
    - Do not wrap type annotation of a parameter with `Optional` if it has a default value of `None`
      (related bug: https://github.com/python/cpython/issues/90353, only fixed in 3.11+).
    """
    try:
        if isinstance(function, partial):
            annotations = function.func.__annotations__
        else:
            annotations = function.__annotations__
    except AttributeError:
        # Some functions (e.g. builtins) don't have annotations:
        return {}

    if globalns is None:
        globalns = get_module_ns_of(function)
    type_params: tuple[Any, ...] | None = None
    if localns is None:
        # If localns was specified, it is assumed to already contain type params. This is because
        # Pydantic has more advanced logic to do so (see `_namespace_utils.ns_for_function`).
        type_params = getattr(function, '__type_params__', ())

    type_hints = {}
    for name, value in annotations.items():
        if include_keys is not None and name not in include_keys:
            continue
        if value is None:
            value = NoneType
        elif isinstance(value, str):
            value = _make_forward_ref(value)

        type_hints[name] = eval_type_backport(value, globalns, localns, type_params)

    return type_hints
```

---

</SwmSnippet>

# Compatibility with Python versions

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="582">

---

The file contains several shims and backports to handle differences in typing support across Python versions. For example, <SwmToken path="pydantic/_internal/_typing_extra.py" pos="584:3:3" line-data="    def _make_forward_ref(">`_make_forward_ref`</SwmToken> is a wrapper for <SwmToken path="pydantic/_internal/_typing_extra.py" pos="589:7:7" line-data="    ) -&gt; typing.ForwardRef:">`ForwardRef`</SwmToken> that works around missing arguments in older Python versions. This ensures that forward references are handled consistently.

```python
if sys.version_info < (3, 9, 8) or (3, 10) <= sys.version_info < (3, 10, 1):

    def _make_forward_ref(
        arg: Any,
        is_argument: bool = True,
        *,
        is_class: bool = False,
    ) -> typing.ForwardRef:
        """Wrapper for ForwardRef that accounts for the `is_class` argument missing in older versions.
        The `module` argument is omitted as it breaks <3.9.8, =3.10.0 and isn't used in the calls below.

        See https://github.com/python/cpython/pull/28560 for some background.
        The backport happened on 3.9.8, see:
        https://github.com/pydantic/pydantic/discussions/6244#discussioncomment-6275458,
        and on 3.10.1 for the 3.10 branch, see:
        https://github.com/pydantic/pydantic/issues/6912

        Implemented as EAFP with memory.
        """
        return typing.ForwardRef(arg, is_argument)

else:
    _make_forward_ref = typing.ForwardRef


if sys.version_info >= (3, 10):
    get_type_hints = typing.get_type_hints

else:
    """
    For older versions of python, we have a custom implementation of `get_type_hints` which is a close as possible to
    the implementation in CPython 3.10.8.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/_internal/_typing_extra.py" line="616">

---

For older Python versions, a custom <SwmToken path="pydantic/_internal/_typing_extra.py" pos="617:3:3" line-data="    def get_type_hints(  # noqa: C901">`get_type_hints`</SwmToken> implementation is provided, closely matching the standard library but patched to use the correct forward reference handling and evaluation logic. This is only used when the standard library version is insufficient.

```python
    @typing.no_type_check
    def get_type_hints(  # noqa: C901
        obj: Any,
        globalns: dict[str, Any] | None = None,
        localns: dict[str, Any] | None = None,
        include_extras: bool = False,
    ) -> dict[str, Any]:  # pragma: no cover
        """Taken verbatim from python 3.10.8 unchanged, except:
        * type annotations of the function definition above.
        * prefixing `typing.` where appropriate
        * Use `_make_forward_ref` instead of `typing.ForwardRef` to handle the `is_class` argument.

        https://github.com/python/cpython/blob/aaaf5174241496afca7ce4d4584570190ff972fe/Lib/typing.py#L1773-L1875

        DO NOT CHANGE THIS METHOD UNLESS ABSOLUTELY NECESSARY.
        ======================================================

        Return type hints for an object.

        This is often the same as obj.__annotations__, but it handles
        forward references encoded as string literals, adds Optional[t] if a
        default value equal to None is set and recursively replaces all
        'Annotated[T, ...]' with 'T' (unless 'include_extras=True').

        The argument may be a module, class, method, or function. The annotations
        are returned as a dictionary. For classes, annotations include also
        inherited members.

        TypeError is raised if the argument is not of a type that can contain
        annotations, and an empty dictionary is returned if no annotations are
        present.

        BEWARE -- the behavior of globalns and localns is counterintuitive
        (unless you are familiar with how eval() and exec() work).  The
        search order is locals first, then globals.

        - If no dict arguments are passed, an attempt is made to use the
          globals from obj (or the respective module's globals for classes),
          and these are also used as the locals.  If the object does not appear
          to have globals, an empty dictionary is used.  For classes, the search
          order is globals first then locals.

        - If one dict argument is passed, it is used for both globals and
          locals.

        - If two dict arguments are passed, they specify globals and
          locals, respectively.
        """
        if getattr(obj, '__no_type_check__', None):
            return {}
        # Classes require a special treatment.
        if isinstance(obj, type):
            hints = {}
            for base in reversed(obj.__mro__):
                if globalns is None:
                    base_globals = getattr(sys.modules.get(base.__module__, None), '__dict__', {})
                else:
                    base_globals = globalns
                ann = base.__dict__.get('__annotations__', {})
                if isinstance(ann, types.GetSetDescriptorType):
                    ann = {}
                base_locals = dict(vars(base)) if localns is None else localns
                if localns is None and globalns is None:
                    # This is surprising, but required.  Before Python 3.10,
                    # get_type_hints only evaluated the globalns of
                    # a class.  To maintain backwards compatibility, we reverse
                    # the globalns and localns order so that eval() looks into
                    # *base_globals* first rather than *base_locals*.
                    # This only affects ForwardRefs.
                    base_globals, base_locals = base_locals, base_globals
                for name, value in ann.items():
                    if value is None:
                        value = type(None)
                    if isinstance(value, str):
                        value = _make_forward_ref(value, is_argument=False, is_class=True)

                    value = eval_type_backport(value, base_globals, base_locals)
                    hints[name] = value
            if not include_extras and hasattr(typing, '_strip_annotations'):
                return {
                    k: typing._strip_annotations(t)  # type: ignore
                    for k, t in hints.items()
                }
            else:
                return hints

        if globalns is None:
            if isinstance(obj, types.ModuleType):
                globalns = obj.__dict__
            else:
                nsobj = obj
                # Find globalns for the unwrapped object.
                while hasattr(nsobj, '__wrapped__'):
                    nsobj = nsobj.__wrapped__
                globalns = getattr(nsobj, '__globals__', {})
            if localns is None:
                localns = globalns
        elif localns is None:
            localns = globalns
        hints = getattr(obj, '__annotations__', None)
        if hints is None:
            # Return empty annotations for something that _could_ have them.
            if isinstance(obj, typing._allowed_types):  # type: ignore
                return {}
            else:
                raise TypeError(f'{obj!r} is not a module, class, method, or function.')
        defaults = typing._get_defaults(obj)  # type: ignore
        hints = dict(hints)
        for name, value in hints.items():
            if value is None:
                value = type(None)
            if isinstance(value, str):
                # class-level forward refs were handled above, this must be either
                # a module-level annotation or a function argument annotation

                value = _make_forward_ref(
                    value,
                    is_argument=not isinstance(obj, types.ModuleType),
                    is_class=False,
                )
            value = eval_type_backport(value, globalns, localns)
            if name in defaults and defaults[name] is None:
                value = typing.Optional[value]
            hints[name] = value
        return hints if include_extras else {k: typing._strip_annotations(t) for k, t in hints.items()}  # type: ignore
```

---

</SwmSnippet>

This approach ensures that Pydantic's typing utilities work reliably across Python versions and with a wide range of user code.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
