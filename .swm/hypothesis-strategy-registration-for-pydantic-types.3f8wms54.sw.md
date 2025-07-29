---
title: Hypothesis Strategy Registration for Pydantic Types
---
# Introduction

This document explains the main ideas behind the Hypothesis strategy registration for Pydantic types, focusing on why the code is structured as it is and how it achieves its goals.

We will cover:

1. Why Hypothesis strategies are registered for Pydantic types and how this is isolated from runtime.
2. How unsupported or tricky types are handled.
3. How resolver functions are used to generate strategies for constrained types.
4. How the registration mechanism is patched into Pydantic.
5. Why certain design trade-offs were made, especially around maintainability and soundness.

# Why and how Hypothesis strategies are registered

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="1">

---

The file exists to register Hypothesis strategies for Pydantic custom types, enabling automatic test data generation for Pydantic models. This is done without runtime impact on Pydantic itself; the module is only imported by Hypothesis if Pydantic is installed, via a setuptools entry point. The docstring makes this clear and points to relevant documentation for further context.

```python
"""
Register Hypothesis strategies for Pydantic custom types.

This enables fully-automatic generation of test data for most Pydantic classes.

Note that this module has *no* runtime impact on Pydantic itself; instead it
is registered as a setuptools entry point and Hypothesis will import it if
Pydantic is installed.  See also:
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="10">

---

The motivation is to improve user experience by ensuring that generated data is always valid, even if that means not covering every possible valid value (completeness is sacrificed for maintainability and predictability).

```python
https://hypothesis.readthedocs.io/en/latest/strategies.html#registering-strategies-via-setuptools-entry-points
https://hypothesis.readthedocs.io/en/latest/data.html#hypothesis.strategies.register_type_strategy
https://hypothesis.readthedocs.io/en/latest/strategies.html#interaction-with-pytest-cov
https://docs.pydantic.dev/usage/types/#pydantic-types

Note that because our motivation is to *improve user experience*, the strategies
are always sound (never generate invalid data) but sacrifice completeness for
maintainability (ie may be unable to generate some tricky but valid data).
```

---

</SwmSnippet>

# Handling unsupported and tricky types

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="33">

---

Some Pydantic types are explicitly unsupported for automatic strategy generation. This is either because generating valid data would require unsafe or impractical operations (like creating files on disk), or because the type is too open-ended or complex for a general-purpose strategy. The comments in the code document these decisions, making it clear what is and isn't supported and why.

```python
import hypothesis.strategies as st

import pydantic
import pydantic.color
import pydantic.types
from pydantic.v1.utils import lenient_issubclass

# FilePath and DirectoryPath are explicitly unsupported, as we'd have to create
# them on-disk, and that's unsafe in general without being told *where* to do so.
#
# URLs are unsupported because it's easy for users to define their own strategy for
# "normal" URLs, and hard for us to define a general strategy which includes "weird"
# URLs but doesn't also have unpredictable performance problems.
#
# conlist() and conset() are unsupported for now, because the workarounds for
# Cython and Hypothesis to handle parametrized generic types are incompatible.
# We are rethinking Hypothesis compatibility in Pydantic v2.
```

---

</SwmSnippet>

# Example: Filtering for valid emails

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="51">

---

For types like <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="69:7:7" line-data="    st.register_type_strategy(pydantic.EmailStr, st.emails().filter(is_valid_email))  # type: ignore[arg-type]">`EmailStr`</SwmToken>, Hypothesis' built-in strategies are not always sufficient, as they may generate syntactically valid but semantically invalid emails. The code uses the <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="60:13:15" line-data="        # that are invalid according to email-validator, so we filter those out.">`email-validator`</SwmToken> library to filter out invalid emails, ensuring that only truly valid emails are generated.

```python
# Emails
try:
    import email_validator
except ImportError:  # pragma: no cover
    pass
else:

    def is_valid_email(s: str) -> bool:
        # Hypothesis' st.emails() occasionally generates emails like 0@A0--0.ac
        # that are invalid according to email-validator, so we filter those out.
        try:
            email_validator.validate_email(s, check_deliverability=False)
            return True
        except email_validator.EmailNotValidError:  # pragma: no cover
            return False
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="67">

---

The registration then uses this filter to ensure only valid emails are produced for Pydantic's <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="69:7:7" line-data="    st.register_type_strategy(pydantic.EmailStr, st.emails().filter(is_valid_email))  # type: ignore[arg-type]">`EmailStr`</SwmToken> and <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="71:3:3" line-data="        pydantic.NameEmail,">`NameEmail`</SwmToken> types.

```python
    # Note that these strategies deliberately stay away from any tricky Unicode
    # or other encoding issues; we're just trying to generate *something* valid.
    st.register_type_strategy(pydantic.EmailStr, st.emails().filter(is_valid_email))  # type: ignore[arg-type]
    st.register_type_strategy(
        pydantic.NameEmail,
        st.builds(
            '{} <{}>'.format,  # type: ignore[arg-type]
            st.from_regex('[A-Za-z0-9_]+( [A-Za-z0-9_]+){0,5}', fullmatch=True),
            st.emails().filter(is_valid_email),
        ),
    )
```

---

</SwmSnippet>

# Example: Custom strategies for specific types

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="79">

---

For types like <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="79:2:2" line-data="# PyObject - dotted names, in this case taken from the math module.">`PyObject`</SwmToken>, the strategy is to sample from known valid dotted names (from the math module), ensuring that generated values are always valid and importable.

```python
# PyObject - dotted names, in this case taken from the math module.
st.register_type_strategy(
    pydantic.PyObject,  # type: ignore[arg-type]
    st.sampled_from(
        [cast(pydantic.PyObject, f'math.{name}') for name in sorted(vars(math)) if not name.startswith('_')]
    ),
)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="87">

---

<SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="87:2:2" line-data="# CSS3 Colors; as name, hex, rgb(a) tuples or strings, or hsl strings">`CSS3`</SwmToken> color values are another example. The code builds a regex that matches valid color representations and registers a strategy that can generate colors in multiple formats (name, hex, rgb(a), hsl(a)), again prioritizing validity.

```python
# CSS3 Colors; as name, hex, rgb(a) tuples or strings, or hsl strings
_color_regexes = (
    '|'.join(
        (
            pydantic.color.r_hex_short,
            pydantic.color.r_hex_long,
            pydantic.color.r_rgb,
            pydantic.color.r_rgba,
            pydantic.color.r_hsl,
            pydantic.color.r_hsla,
        )
    )
    # Use more precise regex patterns to avoid value-out-of-range errors
    .replace(pydantic.color._r_sl, r'(?:(\d\d?(?:\.\d+)?|100(?:\.0+)?)%)')
    .replace(pydantic.color._r_alpha, r'(?:(0(?:\.\d+)?|1(?:\.0+)?|\.\d+|\d{1,2}%))')
    .replace(pydantic.color._r_255, r'(?:((?:\d|\d\d|[01]\d\d|2[0-4]\d|25[0-4])(?:\.\d+)?|255(?:\.0+)?))')
)
st.register_type_strategy(
    pydantic.color.Color,
    st.one_of(
        st.sampled_from(sorted(pydantic.color.COLORS_BY_NAME)),
        st.tuples(
            st.integers(0, 255),
            st.integers(0, 255),
            st.integers(0, 255),
            st.none() | st.floats(0, 1) | st.floats(0, 100).map('{}%'.format),
        ),
        st.from_regex(_color_regexes, fullmatch=True),
    ),
)
```

---

</SwmSnippet>

# Example: Generating valid payment card numbers

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="119">

---

For <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="126:3:3" line-data="            pydantic.PaymentCardNumber.validate_luhn_check_digit(card_number + digit)">`PaymentCardNumber`</SwmToken>, the code generates numbers matching known patterns and then appends a valid Luhn check digit, ensuring that all generated numbers pass validation.

```python
# Card numbers, valid according to the Luhn algorithm


def add_luhn_digit(card_number: str) -> str:
    # See https://en.wikipedia.org/wiki/Luhn_algorithm
    for digit in '0123456789':
        with contextlib.suppress(Exception):
            pydantic.PaymentCardNumber.validate_luhn_check_digit(card_number + digit)
            return card_number + digit
    raise AssertionError('Unreachable')  # pragma: no cover
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="131">

---

The registration maps the regex-generated numbers through the Luhn digit function.

```python
card_patterns = (
    # Note that these patterns omit the Luhn check digit; that's added by the function above
    '4[0-9]{14}',  # Visa
    '5[12345][0-9]{13}',  # Mastercard
    '3[47][0-9]{12}',  # American Express
    '[0-26-9][0-9]{10,17}',  # other (incomplete to avoid overlap)
)
st.register_type_strategy(
    pydantic.PaymentCardNumber,
    st.from_regex('|'.join(card_patterns), fullmatch=True).map(add_luhn_digit),  # type: ignore[arg-type]
)
```

---

</SwmSnippet>

# Resolver functions for constrained types

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="176">

---

For types that are parameterized (like <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="21:30:30" line-data="`(T, SearchStrategy[T])`, but in most cases we register e.g. `ConstrainedInt`">`ConstrainedInt`</SwmToken>, <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="355:5:5" line-data="@resolves(pydantic.ConstrainedStr)">`ConstrainedStr`</SwmToken>, etc.), the code uses resolver functions. These functions inspect the type's constraints and build a Hypothesis strategy that respects those constraints. The resolver functions are tracked in a RESOLVERS dictionary.

```python
# Constrained-type resolver functions
#
# For these ones, we actually want to inspect the type in order to work out a
# satisfying strategy.  First up, the machinery for tracking resolver functions:

RESOLVERS: Dict[type, Callable[[type], st.SearchStrategy]] = {}  # type: ignore[type-arg]
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="208">

---

The decorator @resolves is used to register resolver functions for each constrained type. This makes it easy to add new resolvers and keeps the mapping explicit.

```python
def resolves(
    typ: Union[type, pydantic.types.ConstrainedNumberMeta]
) -> Callable[[Callable[..., st.SearchStrategy]], Callable[..., st.SearchStrategy]]:  # type: ignore[type-arg]
    def inner(f):  # type: ignore
        assert f not in RESOLVERS
        RESOLVERS[typ] = f
        return f
```

---

</SwmSnippet>

# Example: Resolver for constrained decimals

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="263">

---

The resolver for <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="263:5:5" line-data="@resolves(pydantic.ConstrainedDecimal)">`ConstrainedDecimal`</SwmToken> inspects the type's constraints (ge, le, gt, lt, <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="273:24:24" line-data="    s = st.decimals(min_value, max_value, allow_nan=False, places=cls.decimal_places)">`decimal_places`</SwmToken>) and builds a strategy that generates only valid decimals. It asserts that mutually exclusive constraints are not set together and filters the generated values as needed.

```python
@resolves(pydantic.ConstrainedDecimal)
def resolve_condecimal(cls):  # type: ignore[no-untyped-def]
    min_value = cls.ge
    max_value = cls.le
    if cls.gt is not None:
        assert min_value is None, 'Set `gt` or `ge`, but not both'
        min_value = cls.gt
    if cls.lt is not None:
        assert max_value is None, 'Set `lt` or `le`, but not both'
        max_value = cls.lt
    s = st.decimals(min_value, max_value, allow_nan=False, places=cls.decimal_places)
    if cls.lt is not None:
        s = s.filter(lambda d: d < cls.lt)
    if cls.gt is not None:
        s = s.filter(lambda d: cls.gt < d)
    return s
```

---

</SwmSnippet>

# Example: Resolver for constrained strings

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="355">

---

The resolver for <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="355:5:5" line-data="@resolves(pydantic.ConstrainedStr)">`ConstrainedStr`</SwmToken> handles <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="357:7:7" line-data="    min_size = cls.min_length or 0">`min_length`</SwmToken>, <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="358:7:7" line-data="    max_size = cls.max_length">`max_length`</SwmToken>, regex, and <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="360:17:17" line-data="    if cls.regex is None and not cls.strip_whitespace:">`strip_whitespace`</SwmToken> constraints. It builds a strategy that generates only valid strings, using regex or filtering as appropriate.

```python
@resolves(pydantic.ConstrainedStr)
def resolve_constr(cls):  # type: ignore[no-untyped-def]  # pragma: no cover
    min_size = cls.min_length or 0
    max_size = cls.max_length

    if cls.regex is None and not cls.strip_whitespace:
        return st.text(min_size=min_size, max_size=max_size)
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="363">

---

The logic for regex and whitespace handling is handled here.

```python
    if cls.regex is not None:
        strategy = st.from_regex(cls.regex)
        if cls.strip_whitespace:
            strategy = strategy.filter(lambda s: s == s.strip())
    elif cls.strip_whitespace:
        repeats = '{{{},{}}}'.format(
            min_size - 2 if min_size > 2 else 0,
            max_size - 2 if (max_size or 0) > 2 else '',
        )
        if min_size >= 2:
            strategy = st.from_regex(rf'\W.{repeats}\W')
        elif min_size == 1:
            strategy = st.from_regex(rf'\W(.{repeats}\W)?')
        else:
            assert min_size == 0
            strategy = st.from_regex(rf'(\W(.{repeats}\W)?)?')
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="380">

---

Finally, the strategy is filtered to ensure the length constraints are respected.

```python
    if min_size == 0 and max_size is None:
        return strategy
    elif max_size is None:
        return strategy.filter(lambda s: min_size <= len(s))
    return strategy.filter(lambda s: min_size <= len(s) <= max_size)
```

---

</SwmSnippet>

# Registration and patching

<SwmSnippet path="/pydantic/v1/_hypothesis_plugin.py" line="387">

---

At the end of the file, all <SwmToken path="pydantic/v1/_hypothesis_plugin.py" pos="387:9:11" line-data="# Finally, register all previously-defined types, and patch in our new function">`previously-defined`</SwmToken> types are registered with their strategies, and the \_registered function is patched into Pydantic. This ensures that any new constrained types defined by users will also be registered with Hypothesis.

```python
# Finally, register all previously-defined types, and patch in our new function
for typ in list(pydantic.types._DEFINED_TYPES):
    _registered(typ)
pydantic.types._registered = _registered
st.register_type_strategy(pydantic.Json, resolve_json)
```

---

</SwmSnippet>

# Summary of design trade-offs

- The code prioritizes generating only valid data, even if that means not covering every possible valid value.
- Some types are unsupported due to safety or complexity concerns.
- Resolver functions make it easy to add support for new constrained types.
- The registration mechanism is isolated from Pydantic's runtime, only affecting test-time behavior.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
