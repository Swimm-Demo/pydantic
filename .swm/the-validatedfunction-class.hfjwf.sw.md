---
title: The ValidatedFunction class
---
This document covers the <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> class. We'll address:

1. What <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken>, with code citations for each.

# What is <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken>

<SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is a class that encapsulates the logic for validating function arguments using Pydantic models. It is used internally by the <SwmToken path="pydantic/v1/decorator.py" pos="69:23:23" line-data="                f&#39;are not permitted as argument names when using the &quot;{validate_arguments.__name__}&quot; decorator&#39;">`validate_arguments`</SwmToken> decorator to dynamically construct a model that represents the function's signature, validate incoming arguments, and then call the original function with validated data. This enables runtime type and value checking for any decorated function, ensuring that arguments conform to the specified type hints and constraints.

<SwmSnippet path="/pydantic/v1/decorator.py" line="72">

---

The variable <SwmToken path="pydantic/v1/decorator.py" pos="72:3:3" line-data="        self.raw_function = function">`raw_function`</SwmToken> stores the original function that is being wrapped and validated.

```python
        self.raw_function = function
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="73">

---

The variable <SwmToken path="pydantic/v1/decorator.py" pos="73:3:3" line-data="        self.arg_mapping: Dict[int, str] = {}">`arg_mapping`</SwmToken> is a dictionary mapping positional argument indices to their parameter names, used to reconstruct arguments during validation.

```python
        self.arg_mapping: Dict[int, str] = {}
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="74">

---

The variable <SwmToken path="pydantic/v1/decorator.py" pos="74:3:3" line-data="        self.positional_only_args = set()">`positional_only_args`</SwmToken> is a set that tracks the names of parameters that are <SwmToken path="pydantic/v1/decorator.py" pos="250:7:9" line-data="                raise TypeError(f&#39;positional-only argument{plural} passed as keyword argument{plural}: {keys}&#39;)">`positional-only`</SwmToken> in the function signature.

```python
        self.positional_only_args = set()
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="75">

---

The variable <SwmToken path="pydantic/v1/decorator.py" pos="75:3:3" line-data="        self.v_args_name = &#39;args&#39;">`v_args_name`</SwmToken> holds the name of the \*args parameter if present, or defaults to 'args'.

```python
        self.v_args_name = 'args'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="76">

---

The variable <SwmToken path="pydantic/v1/decorator.py" pos="76:3:3" line-data="        self.v_kwargs_name = &#39;kwargs&#39;">`v_kwargs_name`</SwmToken> holds the name of the \*\*kwargs parameter if present, or defaults to 'kwargs'.

```python
        self.v_kwargs_name = 'kwargs'
```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="61">

---

The **init** function analyzes the function signature, extracts type hints, builds a mapping of argument names and types, and prepares the internal model for validation. It also checks for reserved argument names and sets up fields for positional and keyword arguments.

```python
    def __init__(self, function: 'AnyCallableT', config: 'ConfigType'):  # noqa C901
        from inspect import Parameter, signature

        parameters: Mapping[str, Parameter] = signature(function).parameters

        if parameters.keys() & {ALT_V_ARGS, ALT_V_KWARGS, V_POSITIONAL_ONLY_NAME, V_DUPLICATE_KWARGS}:
            raise ConfigError(
                f'"{ALT_V_ARGS}", "{ALT_V_KWARGS}", "{V_POSITIONAL_ONLY_NAME}" and "{V_DUPLICATE_KWARGS}" '
                f'are not permitted as argument names when using the "{validate_arguments.__name__}" decorator'
            )

        self.raw_function = function
        self.arg_mapping: Dict[int, str] = {}
        self.positional_only_args = set()
        self.v_args_name = 'args'
        self.v_kwargs_name = 'kwargs'

        type_hints = get_all_type_hints(function)
        takes_args = False
        takes_kwargs = False
        fields: Dict[str, Tuple[Any, Any]] = {}
        for i, (name, p) in enumerate(parameters.items()):
            if p.annotation is p.empty:
                annotation = Any
            else:
                annotation = type_hints[name]

            default = ... if p.default is p.empty else p.default
            if p.kind == Parameter.POSITIONAL_ONLY:
                self.arg_mapping[i] = name
                fields[name] = annotation, default
                fields[V_POSITIONAL_ONLY_NAME] = List[str], None
                self.positional_only_args.add(name)
            elif p.kind == Parameter.POSITIONAL_OR_KEYWORD:
                self.arg_mapping[i] = name
                fields[name] = annotation, default
                fields[V_DUPLICATE_KWARGS] = List[str], None
            elif p.kind == Parameter.KEYWORD_ONLY:
                fields[name] = annotation, default
            elif p.kind == Parameter.VAR_POSITIONAL:
                self.v_args_name = name
                fields[name] = Tuple[annotation, ...], None
                takes_args = True
            else:
                assert p.kind == Parameter.VAR_KEYWORD, p.kind
                self.v_kwargs_name = name
                fields[name] = Dict[str, annotation], None  # type: ignore
                takes_kwargs = True

        # these checks avoid a clash between "args" and a field with that name
        if not takes_args and self.v_args_name in fields:
            self.v_args_name = ALT_V_ARGS

        # same with "kwargs"
        if not takes_kwargs and self.v_kwargs_name in fields:
            self.v_kwargs_name = ALT_V_KWARGS

        if not takes_args:
            # we add the field so validation below can raise the correct exception
            fields[self.v_args_name] = List[Any], None

        if not takes_kwargs:
            # same with kwargs
            fields[self.v_kwargs_name] = Dict[Any, Any], None

        self.create_model(fields, takes_args, takes_kwargs, config)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="128">

---

The <SwmToken path="pydantic/v1/decorator.py" pos="128:3:3" line-data="    def init_model_instance(self, *args: Any, **kwargs: Any) -&gt; BaseModel:">`init_model_instance`</SwmToken> function builds a dictionary of argument values from provided args and kwargs, then instantiates the internal Pydantic model with these values for validation.

```python
    def init_model_instance(self, *args: Any, **kwargs: Any) -> BaseModel:
        values = self.build_values(args, kwargs)
        return self.model(**values)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="132">

---

The call function is the main entry point for invoking the validated function. It creates a validated model instance from the arguments and then executes the original function with validated data.

```python
    def call(self, *args: Any, **kwargs: Any) -> Any:
        m = self.init_model_instance(*args, **kwargs)
        return self.execute(m)

```

---

</SwmSnippet>

<SwmSnippet path="/pydantic/v1/decorator.py" line="136">

---

The <SwmToken path="pydantic/v1/decorator.py" pos="136:3:3" line-data="    def build_values(self, args: Tuple[Any, ...], kwargs: Dict[str, Any]) -&gt; Dict[str, Any]:">`build_values`</SwmToken> function reconstructs a dictionary of argument values from the provided positional and keyword arguments, handling <SwmToken path="pydantic/v1/decorator.py" pos="250:7:9" line-data="                raise TypeError(f&#39;positional-only argument{plural} passed as keyword argument{plural}: {keys}&#39;)">`positional-only`</SwmToken>, keyword-only, \*args, and \*\*kwargs, as well as error tracking for invalid argument usage.

```python
    def build_values(self, args: Tuple[Any, ...], kwargs: Dict[str, Any]) -> Dict[str, Any]:
        values: Dict[str, Any] = {}
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

        var_kwargs: Dict[str, Any] = {}
        wrong_positional_args = []
        duplicate_kwargs = []
        fields_alias = [
            field.alias
            for name, field in self.model.__fields__.items()
            if name not in (self.v_args_name, self.v_kwargs_name)
        ]
        non_var_fields = set(self.model.__fields__) - {self.v_args_name, self.v_kwargs_name}
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

<SwmSnippet path="/pydantic/v1/decorator.py" line="179">

---

The execute function takes a validated model instance, extracts the argument values, and calls the original function with the correct positional and keyword arguments, handling \*args and \*\*kwargs as needed.

```python
    def execute(self, m: BaseModel) -> Any:
        d = {k: v for k, v in m._iter() if k in m.__fields_set__ or m.__fields__[k].default_factory}
        var_kwargs = d.pop(self.v_kwargs_name, {})

        if self.v_args_name in d:
            args_: List[Any] = []
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

<SwmSnippet path="/pydantic/v1/decorator.py" line="208">

---

The <SwmToken path="pydantic/v1/decorator.py" pos="208:3:3" line-data="    def create_model(self, fields: Dict[str, Any], takes_args: bool, takes_kwargs: bool, config: &#39;ConfigType&#39;) -&gt; None:">`create_model`</SwmToken> function dynamically constructs a Pydantic model class based on the function's signature, argument types, and configuration. It also defines custom validators for argument count, unexpected keywords, <SwmToken path="pydantic/v1/decorator.py" pos="250:7:9" line-data="                raise TypeError(f&#39;positional-only argument{plural} passed as keyword argument{plural}: {keys}&#39;)">`positional-only`</SwmToken> arguments, and duplicate keyword arguments.

```python
    def create_model(self, fields: Dict[str, Any], takes_args: bool, takes_kwargs: bool, config: 'ConfigType') -> None:
        pos_args = len(self.arg_mapping)

        class CustomConfig:
            pass

        if not TYPE_CHECKING:  # pragma: no branch
            if isinstance(config, dict):
                CustomConfig = type('Config', (), config)  # noqa: F811
            elif config is not None:
                CustomConfig = config  # noqa: F811

        if hasattr(CustomConfig, 'fields') or hasattr(CustomConfig, 'alias_generator'):
            raise ConfigError(
                'Setting the "fields" and "alias_generator" property on custom Config for '
                '@validate_arguments is not yet supported, please remove.'
            )

        class DecoratorBaseModel(BaseModel):
            @validator(self.v_args_name, check_fields=False, allow_reuse=True)
            def check_args(cls, v: Optional[List[Any]]) -> Optional[List[Any]]:
                if takes_args or v is None:
                    return v

                raise TypeError(f'{pos_args} positional arguments expected but {pos_args + len(v)} given')

            @validator(self.v_kwargs_name, check_fields=False, allow_reuse=True)
            def check_kwargs(cls, v: Optional[Dict[str, Any]]) -> Optional[Dict[str, Any]]:
                if takes_kwargs or v is None:
                    return v

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v.keys()))
                raise TypeError(f'unexpected keyword argument{plural}: {keys}')

            @validator(V_POSITIONAL_ONLY_NAME, check_fields=False, allow_reuse=True)
            def check_positional_only(cls, v: Optional[List[str]]) -> None:
                if v is None:
                    return

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v))
                raise TypeError(f'positional-only argument{plural} passed as keyword argument{plural}: {keys}')

            @validator(V_DUPLICATE_KWARGS, check_fields=False, allow_reuse=True)
            def check_duplicate_kwargs(cls, v: Optional[List[str]]) -> None:
                if v is None:
                    return

                plural = '' if len(v) == 1 else 's'
                keys = ', '.join(map(repr, v))
                raise TypeError(f'multiple values for argument{plural}: {keys}')

            class Config(CustomConfig):
                extra = getattr(CustomConfig, 'extra', Extra.forbid)

        self.model = create_model(to_camel(self.raw_function.__name__), __base__=DecoratorBaseModel, **fields)
```

---

</SwmSnippet>

# Usage

## <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken>

<SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is used to create a validated wrapper around a function, enabling input data validation based on the function's type hints.

An example usage of <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is found in the validate decorator, where an instance of <SwmToken path="pydantic/v1/decorator.py" pos="36:5:5" line-data="        vd = ValidatedFunction(_func, config)">`ValidatedFunction`</SwmToken> is created by passing the target function and configuration.

The decorator then defines a wrapper function that calls the validated function, ensuring that the inputs are validated before the original function executes.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
