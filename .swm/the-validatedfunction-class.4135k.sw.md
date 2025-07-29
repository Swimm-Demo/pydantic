---
title: The ValidatedFunction class
---
This document covers the <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> class. We'll address:

1. What <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken>, with code citations for each.

# What is <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken>

<SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is a class in <SwmPath>[pydantic/deprecated/decorator.py](pydantic/deprecated/decorator.py)</SwmPath> that wraps a Python function to enable argument validation based on type hints and configuration. It is used internally by the deprecated <SwmToken path="pydantic/deprecated/decorator.py" pos="85:23:23" line-data="                f&#39;are not permitted as argument names when using the &quot;{validate_arguments.__name__}&quot; decorator&#39;,">`validate_arguments`</SwmToken> decorator to ensure that function arguments are validated before the function is executed. The class dynamically constructs a Pydantic model to represent the function's signature, validates incoming arguments, and then calls the original function with the validated data.

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="89">

---

The variable <SwmToken path="pydantic/deprecated/decorator.py" pos="89:3:3" line-data="        self.raw_function = function">`raw_function`</SwmToken> stores the original function that is being wrapped and validated.

```python
        self.raw_function = function
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="90">

---

The variable <SwmToken path="pydantic/deprecated/decorator.py" pos="90:3:3" line-data="        self.arg_mapping: dict[int, str] = {}">`arg_mapping`</SwmToken> is a dictionary mapping positional argument indices to their parameter names, used to reconstruct arguments during validation.

```python
        self.arg_mapping: dict[int, str] = {}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="91">

---

The variable <SwmToken path="pydantic/deprecated/decorator.py" pos="91:3:3" line-data="        self.positional_only_args: set[str] = set()">`positional_only_args`</SwmToken> is a set containing the names of parameters that are <SwmToken path="pydantic/deprecated/decorator.py" pos="270:7:9" line-data="                raise TypeError(f&#39;positional-only argument{plural} passed as keyword argument{plural}: {keys}&#39;)">`positional-only`</SwmToken> in the wrapped function.

```python
        self.positional_only_args: set[str] = set()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="92">

---

The variable <SwmToken path="pydantic/deprecated/decorator.py" pos="92:3:3" line-data="        self.v_args_name = &#39;args&#39;">`v_args_name`</SwmToken> holds the name of the \*args parameter if present, or defaults to 'args'.

```python
        self.v_args_name = 'args'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="93">

---

The variable <SwmToken path="pydantic/deprecated/decorator.py" pos="93:3:3" line-data="        self.v_kwargs_name = &#39;kwargs&#39;">`v_kwargs_name`</SwmToken> holds the name of the \*\*kwargs parameter if present, or defaults to 'kwargs'.

```python
        self.v_kwargs_name = 'kwargs'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="77">

---

The **init** function initializes the <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> instance by analyzing the wrapped function's signature, extracting type hints, and building the internal argument mapping and validation model.

```python
    def __init__(self, function: 'AnyCallable', config: 'ConfigType'):
        from inspect import Parameter, signature

        parameters: Mapping[str, Parameter] = signature(function).parameters

        if parameters.keys() & {ALT_V_ARGS, ALT_V_KWARGS, V_POSITIONAL_ONLY_NAME, V_DUPLICATE_KWARGS}:
            raise PydanticUserError(
                f'"{ALT_V_ARGS}", "{ALT_V_KWARGS}", "{V_POSITIONAL_ONLY_NAME}" and "{V_DUPLICATE_KWARGS}" '
                f'are not permitted as argument names when using the "{validate_arguments.__name__}" decorator',
                code=None,
            )

        self.raw_function = function
        self.arg_mapping: dict[int, str] = {}
        self.positional_only_args: set[str] = set()
        self.v_args_name = 'args'
        self.v_kwargs_name = 'kwargs'

        type_hints = _typing_extra.get_type_hints(function, include_extras=True)
        takes_args = False
        takes_kwargs = False
        fields: dict[str, tuple[Any, Any]] = {}
        for i, (name, p) in enumerate(parameters.items()):
            if p.annotation is p.empty:
                annotation = Any
            else:
                annotation = type_hints[name]

            default = ... if p.default is p.empty else p.default
            if p.kind == Parameter.POSITIONAL_ONLY:
                self.arg_mapping[i] = name
                fields[name] = annotation, default
                fields[V_POSITIONAL_ONLY_NAME] = list[str], None
                self.positional_only_args.add(name)
            elif p.kind == Parameter.POSITIONAL_OR_KEYWORD:
                self.arg_mapping[i] = name
                fields[name] = annotation, default
                fields[V_DUPLICATE_KWARGS] = list[str], None
            elif p.kind == Parameter.KEYWORD_ONLY:
                fields[name] = annotation, default
            elif p.kind == Parameter.VAR_POSITIONAL:
                self.v_args_name = name
                fields[name] = tuple[annotation, ...], None
                takes_args = True
            else:
                assert p.kind == Parameter.VAR_KEYWORD, p.kind
                self.v_kwargs_name = name
                fields[name] = dict[str, annotation], None
                takes_kwargs = True

        # these checks avoid a clash between "args" and a field with that name
        if not takes_args and self.v_args_name in fields:
            self.v_args_name = ALT_V_ARGS

        # same with "kwargs"
        if not takes_kwargs and self.v_kwargs_name in fields:
            self.v_kwargs_name = ALT_V_KWARGS

        if not takes_args:
            # we add the field so validation below can raise the correct exception
            fields[self.v_args_name] = list[Any], None

        if not takes_kwargs:
            # same with kwargs
            fields[self.v_kwargs_name] = dict[Any, Any], None

        self.create_model(fields, takes_args, takes_kwargs, config)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="145">

---

The <SwmToken path="pydantic/deprecated/decorator.py" pos="145:3:3" line-data="    def init_model_instance(self, *args: Any, **kwargs: Any) -&gt; BaseModel:">`init_model_instance`</SwmToken> function builds a dictionary of validated argument values and instantiates the internal Pydantic model with them.

```python
    def init_model_instance(self, *args: Any, **kwargs: Any) -> BaseModel:
        values = self.build_values(args, kwargs)
        return self.model(**values)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="149">

---

The call function is the main entry point for invoking the validated function. It validates the arguments, creates the model instance, and then executes the original function with the validated data.

```python
    def call(self, *args: Any, **kwargs: Any) -> Any:
        m = self.init_model_instance(*args, **kwargs)
        return self.execute(m)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="153">

---

The <SwmToken path="pydantic/deprecated/decorator.py" pos="153:3:3" line-data="    def build_values(self, args: tuple[Any, ...], kwargs: dict[str, Any]) -&gt; dict[str, Any]:">`build_values`</SwmToken> function constructs a dictionary of argument values from the provided args and kwargs, handling positional, keyword, \*args, and \*\*kwargs, as well as error tracking for invalid argument usage.

```python
    def build_values(self, args: tuple[Any, ...], kwargs: dict[str, Any]) -> dict[str, Any]:
        values: dict[str, Any] = {}
        if args:
            arg_iter = enumerate(args)
            while True:
                try:
                    i, a = next(arg_iter)
                except StopIteration:
                    break
                arg_name = self.arg_mapping.get(i)
                if arg_name is not None:
                    values[arg_name] = a
                else:
                    values[self.v_args_name] = [a] + [a for _, a in arg_iter]
                    break

        var_kwargs: dict[str, Any] = {}
        wrong_positional_args = []
        duplicate_kwargs = []
        fields_alias = [
            field.alias
            for name, field in self.model.__pydantic_fields__.items()
            if name not in (self.v_args_name, self.v_kwargs_name)
        ]
        non_var_fields = set(self.model.__pydantic_fields__) - {self.v_args_name, self.v_kwargs_name}
        for k, v in kwargs.items():
            if k in non_var_fields or k in fields_alias:
                if k in self.positional_only_args:
                    wrong_positional_args.append(k)
                if k in values:
                    duplicate_kwargs.append(k)
                values[k] = v
            else:
                var_kwargs[k] = v

        if var_kwargs:
            values[self.v_kwargs_name] = var_kwargs
        if wrong_positional_args:
            values[V_POSITIONAL_ONLY_NAME] = wrong_positional_args
        if duplicate_kwargs:
            values[V_DUPLICATE_KWARGS] = duplicate_kwargs
        return values
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="196">

---

The execute function takes a validated model instance, extracts the arguments, and calls the original function with the correct positional and keyword arguments.

```python
    def execute(self, m: BaseModel) -> Any:
        d = {
            k: v
            for k, v in m.__dict__.items()
            if k in m.__pydantic_fields_set__ or m.__pydantic_fields__[k].default_factory
        }
        var_kwargs = d.pop(self.v_kwargs_name, {})

        if self.v_args_name in d:
            args_: list[Any] = []
            in_kwargs = False
            kwargs = {}
            for name, value in d.items():
                if in_kwargs:
                    kwargs[name] = value
                elif name == self.v_args_name:
                    args_ += value
                    in_kwargs = True
                else:
                    args_.append(value)
            return self.raw_function(*args_, **kwargs, **var_kwargs)
        elif self.positional_only_args:
            args_ = []
            kwargs = {}
            for name, value in d.items():
                if name in self.positional_only_args:
                    args_.append(value)
                else:
                    kwargs[name] = value
            return self.raw_function(*args_, **kwargs, **var_kwargs)
        else:
            return self.raw_function(**d, **var_kwargs)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/deprecated/decorator.py" line="229">

---

The <SwmToken path="pydantic/deprecated/decorator.py" pos="229:3:3" line-data="    def create_model(self, fields: dict[str, Any], takes_args: bool, takes_kwargs: bool, config: &#39;ConfigType&#39;) -&gt; None:">`create_model`</SwmToken> function dynamically creates a Pydantic model class to represent the function's arguments, including custom field validators for argument count, unexpected keywords, <SwmToken path="pydantic/deprecated/decorator.py" pos="270:7:9" line-data="                raise TypeError(f&#39;positional-only argument{plural} passed as keyword argument{plural}: {keys}&#39;)">`positional-only`</SwmToken> enforcement, and duplicate keyword arguments.

```python
    def create_model(self, fields: dict[str, Any], takes_args: bool, takes_kwargs: bool, config: 'ConfigType') -> None:
        pos_args = len(self.arg_mapping)

        config_wrapper = _config.ConfigWrapper(config)

        if config_wrapper.alias_generator:
            raise PydanticUserError(
                'Setting the "alias_generator" property on custom Config for '
                '@validate_arguments is not yet supported, please remove.',
                code=None,
            )
        if config_wrapper.extra is None:
            config_wrapper.config_dict['extra'] = 'forbid'

        class DecoratorBaseModel(BaseModel):
            @field_validator(self.v_args_name, check_fields=False)
            @classmethod
            def check_args(cls, v: Optional[list[Any]]) -> Optional[list[Any]]:
                if takes_args or v is None:
                    return v

                raise TypeError(f'{pos_args} positional arguments expected but {pos_args + len(v)} given')

            @field_validator(self.v_kwargs_name, check_fields=False)
            @classmethod
            def check_kwargs(cls, v: Optional[dict[str, Any]]) -> Optional[dict[str, Any]]:
                if takes_kwargs or v is None:
                    return v

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v.keys()))
                raise TypeError(f'unexpected keyword argument{plural}: {keys}')

            @field_validator(V_POSITIONAL_ONLY_NAME, check_fields=False)
            @classmethod
            def check_positional_only(cls, v: Optional[list[str]]) -> None:
                if v is None:
                    return

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v))
                raise TypeError(f'positional-only argument{plural} passed as keyword argument{plural}: {keys}')

            @field_validator(V_DUPLICATE_KWARGS, check_fields=False)
            @classmethod
            def check_duplicate_kwargs(cls, v: Optional[list[str]]) -> None:
                if v is None:
                    return

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v))
                raise TypeError(f'multiple values for argument{plural}: {keys}')

            model_config = config_wrapper.config_dict

        self.model = create_model(to_pascal(self.raw_function.__name__), __base__=DecoratorBaseModel, **fields)
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> Usage in Decorator

The <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> class is instantiated within a decorator function named validate. This decorator takes a function as input and creates a <SwmToken path="pydantic/deprecated/decorator.py" pos="52:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> instance with it, along with a configuration object. The decorator then defines a wrapper function that preserves the original function's metadata using functools.wraps and is intended to handle the validation logic when the wrapped function is called.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
