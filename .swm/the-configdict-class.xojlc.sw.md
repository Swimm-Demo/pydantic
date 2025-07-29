---
title: The ConfigDict class
---
This document covers the <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> class in Pydantic, focusing on:

1. What <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken>, with explanations and code citations.

# What is <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken>

<SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is a <SwmToken path="pydantic/config.py" pos="9:9:9" line-data="from typing_extensions import TypeAlias, TypedDict, Unpack, deprecated">`TypedDict`</SwmToken> defined in Pydantic for configuring the behavior of Pydantic models and related types. It allows users to specify a wide range of configuration options that control validation, serialization, schema generation, and other aspects of how Pydantic models operate. <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is typically used by assigning it to the <SwmToken path="pydantic/config.py" pos="74:1:1" line-data="          model_config = ConfigDict(extra=&#39;ignore&#39;)  # (1)!">`model_config`</SwmToken> attribute of a Pydantic model or by passing it to decorators or dataclasses, enabling fine-grained control over model behavior.

<SwmSnippet path="/pydantic/config.py" line="39">

---

The variable <SwmToken path="pydantic/config.py" pos="39:1:1" line-data="    title: str | None">`title`</SwmToken> specifies the title for the generated JSON schema. If not set, it defaults to the model's name.

```python
    title: str | None
    """The title for the generated JSON schema, defaults to the model's name"""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="42">

---

The variable <SwmToken path="pydantic/config.py" pos="42:1:1" line-data="    model_title_generator: Callable[[type], str] | None">`model_title_generator`</SwmToken> is a callable that takes a model class and returns a title for it. It allows dynamic generation of schema titles.

```python
    model_title_generator: Callable[[type], str] | None
    """A callable that takes a model class and returns the title for it. Defaults to `None`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="45">

---

The variable <SwmToken path="pydantic/config.py" pos="45:1:1" line-data="    field_title_generator: Callable[[str, FieldInfo | ComputedFieldInfo], str] | None">`field_title_generator`</SwmToken> is a callable that takes a field's name and info and returns a title for it. This enables custom field title generation in schemas.

```python
    field_title_generator: Callable[[str, FieldInfo | ComputedFieldInfo], str] | None
    """A callable that takes a field's name and info and returns title for it. Defaults to `None`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="48">

---

The variable <SwmToken path="pydantic/config.py" pos="48:1:1" line-data="    str_to_lower: bool">`str_to_lower`</SwmToken> determines whether all characters in string fields should be converted to lowercase during validation. Defaults to False.

```python
    str_to_lower: bool
    """Whether to convert all characters to lowercase for str types. Defaults to `False`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="51">

---

The variable <SwmToken path="pydantic/config.py" pos="51:1:1" line-data="    str_to_upper: bool">`str_to_upper`</SwmToken> determines whether all characters in string fields should be converted to uppercase during validation. Defaults to False.

```python
    str_to_upper: bool
    """Whether to convert all characters to uppercase for str types. Defaults to `False`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="54">

---

The variable <SwmToken path="pydantic/config.py" pos="54:1:1" line-data="    str_strip_whitespace: bool">`str_strip_whitespace`</SwmToken> controls whether leading and trailing whitespace should be stripped from string fields.

```python
    str_strip_whitespace: bool
    """Whether to strip leading and trailing whitespace for str types."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="57">

---

The variable <SwmToken path="pydantic/config.py" pos="57:1:1" line-data="    str_min_length: int">`str_min_length`</SwmToken> sets the minimum allowed length for string fields.

```python
    str_min_length: int
    """The minimum length for str types. Defaults to `None`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="60">

---

The variable <SwmToken path="pydantic/config.py" pos="60:1:1" line-data="    str_max_length: int | None">`str_max_length`</SwmToken> sets the maximum allowed length for string fields.

```python
    str_max_length: int | None
    """The maximum length for str types. Defaults to `None`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="63">

---

The variable <SwmToken path="pydantic/config.py" pos="63:1:1" line-data="    extra: ExtraValues | None">`extra`</SwmToken> determines how extra fields (not defined in the model) are handled during model initialization. It can be set to 'ignore', 'forbid', or 'allow', with detailed behavior for each option.

````python
    extra: ExtraValues | None
    '''
    Whether to ignore, allow, or forbid extra data during model initialization. Defaults to `'ignore'`.

    Three configuration values are available:

    - `'ignore'`: Providing extra data is ignored (the default):
      ```python
      from pydantic import BaseModel, ConfigDict

      class User(BaseModel):
          model_config = ConfigDict(extra='ignore')  # (1)!

          name: str

      user = User(name='John Doe', age=20)  # (2)!
      print(user)
      #> name='John Doe'
      ```

        1. This is the default behaviour.
        2. The `age` argument is ignored.

    - `'forbid'`: Providing extra data is not permitted, and a [`ValidationError`][pydantic_core.ValidationError]
      will be raised if this is the case:
      ```python
      from pydantic import BaseModel, ConfigDict, ValidationError


      class Model(BaseModel):
          x: int

          model_config = ConfigDict(extra='forbid')


      try:
          Model(x=1, y='a')
      except ValidationError as exc:
          print(exc)
          """
          1 validation error for Model
          y
            Extra inputs are not permitted [type=extra_forbidden, input_value='a', input_type=str]
          """
      ```

    - `'allow'`: Providing extra data is allowed and stored in the `__pydantic_extra__` dictionary attribute:
      ```python
      from pydantic import BaseModel, ConfigDict


      class Model(BaseModel):
          x: int

          model_config = ConfigDict(extra='allow')


      m = Model(x=1, y='a')
      assert m.__pydantic_extra__ == {'y': 'a'}
      ```
      By default, no validation will be applied to these extra items, but you can set a type for the values by overriding
      the type annotation for `__pydantic_extra__`:
      ```python
      from pydantic import BaseModel, ConfigDict, Field, ValidationError


      class Model(BaseModel):
          __pydantic_extra__: dict[str, int] = Field(init=False)  # (1)!

          x: int

          model_config = ConfigDict(extra='allow')


      try:
          Model(x=1, y='a')
      except ValidationError as exc:
          print(exc)
          """
          1 validation error for Model
          y
            Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='a', input_type=str]
          """

      m = Model(x=1, y='2')
      assert m.x == 1
      assert m.y == 2
      assert m.model_dump() == {'x': 1, 'y': 2}
      assert m.__pydantic_extra__ == {'y': 2}
      ```

        1. The `= Field(init=False)` does not have any effect at runtime, but prevents the `__pydantic_extra__` field from
           being included as a parameter to the model's `__init__` method by type checkers.
    '''
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="158">

---

The variable <SwmToken path="pydantic/config.py" pos="158:1:1" line-data="    frozen: bool">`frozen`</SwmToken> controls whether models are immutable (<SwmToken path="pydantic/config.py" pos="160:7:9" line-data="    Whether models are faux-immutable, i.e. whether `__setattr__` is allowed, and also generates">`faux-immutable`</SwmToken>), disallowing attribute assignment and generating a hash method. Defaults to False.

```python
    frozen: bool
    """
    Whether models are faux-immutable, i.e. whether `__setattr__` is allowed, and also generates
    a `__hash__()` method for the model. This makes instances of the model potentially hashable if all the
    attributes are hashable. Defaults to `False`.

    Note:
        On V1, the inverse of this setting was called `allow_mutation`, and was `True` by default.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="168">

---

The variable <SwmToken path="pydantic/config.py" pos="168:1:1" line-data="    populate_by_name: bool">`populate_by_name`</SwmToken> allows aliased fields to be populated by their attribute name as well as the alias. This setting is deprecated in favor of <SwmToken path="pydantic/config.py" pos="175:14:14" line-data="        Instead, you should use the [`validate_by_name`][pydantic.config.ConfigDict.validate_by_name] configuration setting.">`validate_by_name`</SwmToken> and <SwmToken path="pydantic/config.py" pos="177:12:12" line-data="        When `validate_by_name=True` and `validate_by_alias=True`, this is strictly equivalent to the">`validate_by_alias`</SwmToken>.

````python
    populate_by_name: bool
    """
    Whether an aliased field may be populated by its name as given by the model
    attribute, as well as the alias. Defaults to `False`.

    !!! warning
        `populate_by_name` usage is not recommended in v2.11+ and will be deprecated in v3.
        Instead, you should use the [`validate_by_name`][pydantic.config.ConfigDict.validate_by_name] configuration setting.

        When `validate_by_name=True` and `validate_by_alias=True`, this is strictly equivalent to the
        previous behavior of `populate_by_name=True`.

        In v2.11, we also introduced a [`validate_by_alias`][pydantic.config.ConfigDict.validate_by_alias] setting that introduces more fine grained
        control for validation behavior.

        Here's how you might go about using the new settings to achieve the same behavior:

        ```python
        from pydantic import BaseModel, ConfigDict, Field

        class Model(BaseModel):
            model_config = ConfigDict(validate_by_name=True, validate_by_alias=True)

            my_field: str = Field(alias='my_alias')  # (1)!

        m = Model(my_alias='foo')  # (2)!
        print(m)
        #> my_field='foo'

        m = Model(my_field='foo')  # (3)!
        print(m)
        #> my_field='foo'
        ```

        1. The field `'my_field'` has an alias `'my_alias'`.
        2. The model is populated by the alias `'my_alias'`.
        3. The model is populated by the attribute name `'my_field'`.
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="207">

---

The variable <SwmToken path="pydantic/config.py" pos="207:1:1" line-data="    use_enum_values: bool">`use_enum_values`</SwmToken> determines whether enum fields are populated with their value property instead of the enum instance. This is useful for serialization.

````python
    use_enum_values: bool
    """
    Whether to populate models with the `value` property of enums, rather than the raw enum.
    This may be useful if you want to serialize `model.model_dump()` later. Defaults to `False`.

    !!! note
        If you have an `Optional[Enum]` value that you set a default for, you need to use `validate_default=True`
        for said Field to ensure that the `use_enum_values` flag takes effect on the default, as extracting an
        enum's value occurs during validation, not serialization.

    ```python
    from enum import Enum
    from typing import Optional

    from pydantic import BaseModel, ConfigDict, Field

    class SomeEnum(Enum):
        FOO = 'foo'
        BAR = 'bar'
        BAZ = 'baz'

    class SomeModel(BaseModel):
        model_config = ConfigDict(use_enum_values=True)

        some_enum: SomeEnum
        another_enum: Optional[SomeEnum] = Field(
            default=SomeEnum.FOO, validate_default=True
        )

    model1 = SomeModel(some_enum=SomeEnum.BAR)
    print(model1.model_dump())
    #> {'some_enum': 'bar', 'another_enum': 'foo'}

    model2 = SomeModel(some_enum=SomeEnum.BAR, another_enum=SomeEnum.BAZ)
    print(model2.model_dump())
    #> {'some_enum': 'bar', 'another_enum': 'baz'}
    ```
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="246">

---

The variable <SwmToken path="pydantic/config.py" pos="246:1:1" line-data="    validate_assignment: bool">`validate_assignment`</SwmToken> enables validation when model fields are assigned after initialization. By default, only initial creation is validated.

````python
    validate_assignment: bool
    """
    Whether to validate the data when the model is changed. Defaults to `False`.

    The default behavior of Pydantic is to validate the data when the model is created.

    In case the user changes the data after the model is created, the model is _not_ revalidated.

    ```python
    from pydantic import BaseModel

    class User(BaseModel):
        name: str

    user = User(name='John Doe')  # (1)!
    print(user)
    #> name='John Doe'
    user.name = 123  # (1)!
    print(user)
    #> name=123
    ```

    1. The validation happens only when the model is created.
    2. The validation does not happen when the data is changed.

    In case you want to revalidate the model when the data is changed, you can use `validate_assignment=True`:

    ```python
    from pydantic import BaseModel, ValidationError

    class User(BaseModel, validate_assignment=True):  # (1)!
        name: str

    user = User(name='John Doe')  # (2)!
    print(user)
    #> name='John Doe'
    try:
        user.name = 123  # (3)!
    except ValidationError as e:
        print(e)
        '''
        1 validation error for User
        name
          Input should be a valid string [type=string_type, input_value=123, input_type=int]
        '''
    ```

    1. You can either use class keyword arguments, or `model_config` to set `validate_assignment=True`.
    2. The validation happens when the model is created.
    3. The validation _also_ happens when the data is changed.
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="298">

---

The variable <SwmToken path="pydantic/config.py" pos="298:1:1" line-data="    arbitrary_types_allowed: bool">`arbitrary_types_allowed`</SwmToken> allows fields to have arbitrary types that are not Pydantic models. This is useful for integrating with external classes.

````python
    arbitrary_types_allowed: bool
    """
    Whether arbitrary types are allowed for field types. Defaults to `False`.

    ```python
    from pydantic import BaseModel, ConfigDict, ValidationError

    # This is not a pydantic model, it's an arbitrary class
    class Pet:
        def __init__(self, name: str):
            self.name = name

    class Model(BaseModel):
        model_config = ConfigDict(arbitrary_types_allowed=True)

        pet: Pet
        owner: str

    pet = Pet(name='Hedwig')
    # A simple check of instance type is used to validate the data
    model = Model(owner='Harry', pet=pet)
    print(model)
    #> pet=<__main__.Pet object at 0x0123456789ab> owner='Harry'
    print(model.pet)
    #> <__main__.Pet object at 0x0123456789ab>
    print(model.pet.name)
    #> Hedwig
    print(type(model.pet))
    #> <class '__main__.Pet'>
    try:
        # If the value is not an instance of the type, it's invalid
        Model(owner='Harry', pet='Hedwig')
    except ValidationError as e:
        print(e)
        '''
        1 validation error for Model
        pet
          Input should be an instance of Pet [type=is_instance_of, input_value='Hedwig', input_type=str]
        '''

    # Nothing in the instance of the arbitrary type is checked
    # Here name probably should have been a str, but it's not validated
    pet2 = Pet(name=42)
    model2 = Model(owner='Harry', pet=pet2)
    print(model2)
    #> pet=<__main__.Pet object at 0x0123456789ab> owner='Harry'
    print(model2.pet)
    #> <__main__.Pet object at 0x0123456789ab>
    print(model2.pet.name)
    #> 42
    print(type(model2.pet))
    #> <class '__main__.Pet'>
    ```
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="353">

---

The variable <SwmToken path="pydantic/config.py" pos="353:1:1" line-data="    from_attributes: bool">`from_attributes`</SwmToken> enables building models and looking up discriminators using Python object attributes.

```python
    from_attributes: bool
    """
    Whether to build models and look up discriminators of tagged unions using python object attributes.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="358">

---

The variable <SwmToken path="pydantic/config.py" pos="358:1:1" line-data="    loc_by_alias: bool">`loc_by_alias`</SwmToken> determines whether error locations use the alias or the field name. Defaults to True.

```python
    loc_by_alias: bool
    """Whether to use the actual key provided in the data (e.g. alias) for error `loc`s rather than the field's name. Defaults to `True`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="361">

---

The variable <SwmToken path="pydantic/config.py" pos="361:1:1" line-data="    alias_generator: Callable[[str], str] | AliasGenerator | None">`alias_generator`</SwmToken> allows specifying a callable or <SwmToken path="pydantic/config.py" pos="361:16:16" line-data="    alias_generator: Callable[[str], str] | AliasGenerator | None">`AliasGenerator`</SwmToken> instance to generate field aliases for validation and serialization.

````python
    alias_generator: Callable[[str], str] | AliasGenerator | None
    """
    A callable that takes a field name and returns an alias for it
    or an instance of [`AliasGenerator`][pydantic.aliases.AliasGenerator]. Defaults to `None`.

    When using a callable, the alias generator is used for both validation and serialization.
    If you want to use different alias generators for validation and serialization, you can use
    [`AliasGenerator`][pydantic.aliases.AliasGenerator] instead.

    If data source field names do not match your code style (e. g. CamelCase fields),
    you can automatically generate aliases using `alias_generator`. Here's an example with
    a basic callable:

    ```python
    from pydantic import BaseModel, ConfigDict
    from pydantic.alias_generators import to_pascal

    class Voice(BaseModel):
        model_config = ConfigDict(alias_generator=to_pascal)

        name: str
        language_code: str

    voice = Voice(Name='Filiz', LanguageCode='tr-TR')
    print(voice.language_code)
    #> tr-TR
    print(voice.model_dump(by_alias=True))
    #> {'Name': 'Filiz', 'LanguageCode': 'tr-TR'}
    ```

    If you want to use different alias generators for validation and serialization, you can use
    [`AliasGenerator`][pydantic.aliases.AliasGenerator].

    ```python
    from pydantic import AliasGenerator, BaseModel, ConfigDict
    from pydantic.alias_generators import to_camel, to_pascal

    class Athlete(BaseModel):
        first_name: str
        last_name: str
        sport: str

        model_config = ConfigDict(
            alias_generator=AliasGenerator(
                validation_alias=to_camel,
                serialization_alias=to_pascal,
            )
        )

    athlete = Athlete(firstName='John', lastName='Doe', sport='track')
    print(athlete.model_dump(by_alias=True))
    #> {'FirstName': 'John', 'LastName': 'Doe', 'Sport': 'track'}
    ```

    Note:
        Pydantic offers three built-in alias generators: [`to_pascal`][pydantic.alias_generators.to_pascal],
        [`to_camel`][pydantic.alias_generators.to_camel], and [`to_snake`][pydantic.alias_generators.to_snake].
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="420">

---

The variable <SwmToken path="pydantic/config.py" pos="420:1:1" line-data="    ignored_types: tuple[type, ...]">`ignored_types`</SwmToken> is a tuple of types that can be used as values of class attributes without annotations, typically for custom descriptors.

```python
    ignored_types: tuple[type, ...]
    """A tuple of types that may occur as values of class attributes without annotations. This is
    typically used for custom descriptors (classes that behave like `property`). If an attribute is set on a
    class without an annotation and has a type that is not in this tuple (or otherwise recognized by
    _pydantic_), an error will be raised. Defaults to `()`.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="427">

---

The variable <SwmToken path="pydantic/config.py" pos="427:1:1" line-data="    allow_inf_nan: bool">`allow_inf_nan`</SwmToken> controls whether infinity and <SwmToken path="pydantic/config.py" pos="428:26:26" line-data="    &quot;&quot;&quot;Whether to allow infinity (`+inf` an `-inf`) and NaN values to float and decimal fields. Defaults to `True`.&quot;&quot;&quot;">`NaN`</SwmToken> values are allowed for float and decimal fields. Defaults to True.

```python
    allow_inf_nan: bool
    """Whether to allow infinity (`+inf` an `-inf`) and NaN values to float and decimal fields. Defaults to `True`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="430">

---

The variable <SwmToken path="pydantic/config.py" pos="430:1:1" line-data="    json_schema_extra: JsonDict | JsonSchemaExtraCallable | None">`json_schema_extra`</SwmToken> allows providing extra JSON schema properties as a dict or callable.

```python
    json_schema_extra: JsonDict | JsonSchemaExtraCallable | None
    """A dict or callable to provide extra JSON schema properties. Defaults to `None`."""
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="433">

---

The variable <SwmToken path="pydantic/config.py" pos="433:1:1" line-data="    json_encoders: dict[type[object], JsonEncoder] | None">`json_encoders`</SwmToken> is a dictionary of custom JSON encoders for specific types. This option is deprecated and may be removed in the future.

```python
    json_encoders: dict[type[object], JsonEncoder] | None
    """
    A `dict` of custom JSON encoders for specific types. Defaults to `None`.

    !!! warning "Deprecated"
        This config option is a carryover from v1.
        We originally planned to remove it in v2 but didn't have a 1:1 replacement so we are keeping it for now.
        It is still deprecated and will likely be removed in the future.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="444">

---

The variable <SwmToken path="pydantic/config.py" pos="444:1:1" line-data="    strict: bool">`strict`</SwmToken> enables strict validation for all fields, raising errors if types do not match exactly. This is new in Pydantic <SwmToken path="pydantic/config.py" pos="446:7:7" line-data="    _(new in V2)_ If `True`, strict validation is applied to all fields on the model.">`V2`</SwmToken>.

````python
    strict: bool
    """
    _(new in V2)_ If `True`, strict validation is applied to all fields on the model.

    By default, Pydantic attempts to coerce values to the correct type, when possible.

    There are situations in which you may want to disable this behavior, and instead raise an error if a value's type
    does not match the field's type annotation.

    To configure strict mode for all fields on a model, you can set `strict=True` on the model.

    ```python
    from pydantic import BaseModel, ConfigDict

    class Model(BaseModel):
        model_config = ConfigDict(strict=True)

        name: str
        age: int
    ```

    See [Strict Mode](../concepts/strict_mode.md) for more details.

    See the [Conversion Table](../concepts/conversion_table.md) for more details on how Pydantic converts data in both
    strict and lax modes.
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="471">

---

The variable <SwmToken path="pydantic/config.py" pos="471:1:1" line-data="    revalidate_instances: Literal[&#39;always&#39;, &#39;never&#39;, &#39;subclass-instances&#39;]">`revalidate_instances`</SwmToken> controls when and how to revalidate models and dataclasses during validation. Accepts 'never', 'always', or <SwmToken path="pydantic/config.py" pos="471:17:19" line-data="    revalidate_instances: Literal[&#39;always&#39;, &#39;never&#39;, &#39;subclass-instances&#39;]">`subclass-instances`</SwmToken>.

````python
    revalidate_instances: Literal['always', 'never', 'subclass-instances']
    """
    When and how to revalidate models and dataclasses during validation. Accepts the string
    values of `'never'`, `'always'` and `'subclass-instances'`. Defaults to `'never'`.

    - `'never'` will not revalidate models and dataclasses during validation
    - `'always'` will revalidate models and dataclasses during validation
    - `'subclass-instances'` will revalidate models and dataclasses during validation if the instance is a
        subclass of the model or dataclass

    By default, model and dataclass instances are not revalidated during validation.

    ```python
    from pydantic import BaseModel

    class User(BaseModel, revalidate_instances='never'):  # (1)!
        hobbies: list[str]

    class SubUser(User):
        sins: list[str]

    class Transaction(BaseModel):
        user: User

    my_user = User(hobbies=['reading'])
    t = Transaction(user=my_user)
    print(t)
    #> user=User(hobbies=['reading'])

    my_user.hobbies = [1]  # (2)!
    t = Transaction(user=my_user)  # (3)!
    print(t)
    #> user=User(hobbies=[1])

    my_sub_user = SubUser(hobbies=['scuba diving'], sins=['lying'])
    t = Transaction(user=my_sub_user)
    print(t)
    #> user=SubUser(hobbies=['scuba diving'], sins=['lying'])
    ```

    1. `revalidate_instances` is set to `'never'` by **default.
    2. The assignment is not validated, unless you set `validate_assignment` to `True` in the model's config.
    3. Since `revalidate_instances` is set to `never`, this is not revalidated.

    If you want to revalidate instances during validation, you can set `revalidate_instances` to `'always'`
    in the model's config.

    ```python
    from pydantic import BaseModel, ValidationError

    class User(BaseModel, revalidate_instances='always'):  # (1)!
        hobbies: list[str]

    class SubUser(User):
        sins: list[str]

    class Transaction(BaseModel):
        user: User

    my_user = User(hobbies=['reading'])
    t = Transaction(user=my_user)
    print(t)
    #> user=User(hobbies=['reading'])

    my_user.hobbies = [1]
    try:
        t = Transaction(user=my_user)  # (2)!
    except ValidationError as e:
        print(e)
        '''
        1 validation error for Transaction
        user.hobbies.0
          Input should be a valid string [type=string_type, input_value=1, input_type=int]
        '''

    my_sub_user = SubUser(hobbies=['scuba diving'], sins=['lying'])
    t = Transaction(user=my_sub_user)
    print(t)  # (3)!
    #> user=User(hobbies=['scuba diving'])
    ```

    1. `revalidate_instances` is set to `'always'`.
    2. The model is revalidated, since `revalidate_instances` is set to `'always'`.
    3. Using `'never'` we would have gotten `user=SubUser(hobbies=['scuba diving'], sins=['lying'])`.

    It's also possible to set `revalidate_instances` to `'subclass-instances'` to only revalidate instances
    of subclasses of the model.

    ```python
    from pydantic import BaseModel

    class User(BaseModel, revalidate_instances='subclass-instances'):  # (1)!
        hobbies: list[str]

    class SubUser(User):
        sins: list[str]

    class Transaction(BaseModel):
        user: User

    my_user = User(hobbies=['reading'])
    t = Transaction(user=my_user)
    print(t)
    #> user=User(hobbies=['reading'])

    my_user.hobbies = [1]
    t = Transaction(user=my_user)  # (2)!
    print(t)
    #> user=User(hobbies=[1])

    my_sub_user = SubUser(hobbies=['scuba diving'], sins=['lying'])
    t = Transaction(user=my_sub_user)
    print(t)  # (3)!
    #> user=User(hobbies=['scuba diving'])
    ```

    1. `revalidate_instances` is set to `'subclass-instances'`.
    2. This is not revalidated, since `my_user` is not a subclass of `User`.
    3. Using `'never'` we would have gotten `user=SubUser(hobbies=['scuba diving'], sins=['lying'])`.
    """
````

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="592">

---

The variable <SwmToken path="pydantic/config.py" pos="592:1:1" line-data="    ser_json_timedelta: Literal[&#39;iso8601&#39;, &#39;float&#39;]">`ser_json_timedelta`</SwmToken> sets the format for JSON serialized timedeltas, either <SwmToken path="pydantic/config.py" pos="592:7:7" line-data="    ser_json_timedelta: Literal[&#39;iso8601&#39;, &#39;float&#39;]">`iso8601`</SwmToken> or 'float'.

```python
    ser_json_timedelta: Literal['iso8601', 'float']
    """
    The format of JSON serialized timedeltas. Accepts the string values of `'iso8601'` and
    `'float'`. Defaults to `'iso8601'`.

    - `'iso8601'` will serialize timedeltas to ISO 8601 durations.
    - `'float'` will serialize timedeltas to the total number of seconds.
    """
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/config.py" line="601">

---

The variable <SwmToken path="pydantic/config.py" pos="601:1:1" line-data="    ser_json_bytes: Literal[&#39;utf8&#39;, &#39;base64&#39;, &#39;hex&#39;]">`ser_json_bytes`</SwmToken> sets the encoding for JSON serialized bytes, with options <SwmToken path="pydantic/config.py" pos="601:7:7" line-data="    ser_json_bytes: Literal[&#39;utf8&#39;, &#39;base64&#39;, &#39;hex&#39;]">`utf8`</SwmToken>, <SwmToken path="pydantic/config.py" pos="601:12:12" line-data="    ser_json_bytes: Literal[&#39;utf8&#39;, &#39;base64&#39;, &#39;hex&#39;]">`base64`</SwmToken>, or 'hex'.

```python
    ser_json_bytes: Literal['utf8', 'base64', 'hex']
    """
    The encoding of JSON serialized bytes. Defaults to `'utf8'`.
    Set equal to `val_json_bytes` to get back an equal value after serialization round trip.

    - `'utf8'` will serialize bytes to UTF-8 strings.
    - `'base64'` will serialize bytes to URL safe base64 strings.
    - `'hex'` will serialize bytes to hexadecimal strings.
    """
```

---

</SwmSnippet>

# Usage

## Usage in <SwmPath>[pydantic/mypy.py](pydantic/mypy.py)</SwmPath>

<SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is checked in type analysis code to identify assignments to <SwmToken path="pydantic/config.py" pos="74:1:1" line-data="          model_config = ConfigDict(extra=&#39;ignore&#39;)  # (1)!">`model_config`</SwmToken>. This suggests it is used to represent or validate configuration dictionaries during static type checking or code analysis.

## Usage in <SwmPath>[pydantic/config.py](pydantic/config.py)</SwmPath>

<SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is explicitly exported from the config module, indicating it is a key part of the public API for configuration management. It is grouped with a utility function <SwmToken path="pydantic/config.py" pos="20:11:11" line-data="__all__ = (&#39;ConfigDict&#39;, &#39;with_config&#39;)">`with_config`</SwmToken>, which likely helps in applying or managing configurations.

## Usage in <SwmPath>[pydantic/experimental/arguments_schema.py](pydantic/experimental/arguments_schema.py)</SwmPath>

<SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> is used as an optional parameter in a function that generates schemas for function arguments. This shows that <SwmToken path="pydantic/config.py" pos="71:10:10" line-data="      from pydantic import BaseModel, ConfigDict">`ConfigDict`</SwmToken> can be passed to customize or influence schema generation behavior, highlighting its role in flexible configuration.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
