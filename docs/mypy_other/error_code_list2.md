# 可选检查的错误代码

**Error codes for optional checks**

=== "中文"

    本节文档记录了 mypy 在启用特定选项时生成的各种错误代码。有关错误代码及其配置的一般文档，请参见 [错误代码](./error_codes.md)。[默认启用的错误代码](./error_code_list.md) 记录了默认启用的错误代码。

    !!! note

        本节中的示例使用 [内联配置](../mypy_conf/inline_config.md) 来指定 mypy 选项。您也可以通过使用 [配置文件](../mypy_conf/config_file.md) 或 [命令行选项](../mypy_conf/command_line.md) 来设置相同的选项。

=== "英文"

    This section documents various errors codes that mypy generates only if you enable certain options. See [Error codes](./error_codes.md) for general documentation about error codes and their configuration. [Error codes enabled by default](./error_code_list.md) documents error codes that are enabled by default.

    !!! note 

        The examples in this section use [inline configuration](../mypy_conf/inline_config.md) to specify mypy options. You can also set the same options by using a [configuration file](../mypy_conf/config_file.md) or [command-line options](../mypy_conf/command_line.md).

## 检查类型参数是否存在 [type-arg]

**Check that type arguments exist [type-arg]**

=== "中文"

    如果您使用 [--disallow-any-generics](../mypy_conf/command_line.md#disallow-any-generics)，mypy 要求每个泛型类型都必须为每个类型参数提供具体的值。例如，类型 ``list`` 或 ``dict`` 会被拒绝。您应该改用类似 ``list[int]`` 或 ``dict[str, int]`` 的类型。任何省略的泛型类型参数会被隐式地视为 ``Any``。类型 ``list`` 等同于 ``list[Any]``，依此类推。

    示例：

    ```python
    # mypy: disallow-any-generics

    # 错误：缺少泛型类型 "list" 的类型参数 [type-arg]
    def remove_dups(items: list) -> list:
        ...
    ```

=== "英文"

    If you use [--disallow-any-generics](../mypy_conf/command_line.md#disallow-any-generics), mypy requires that each generic type has values for each type argument. For example, the types ``list`` or ``dict`` would be rejected. You should instead use types like ``list[int]`` or ``dict[str, int]``. Any omitted generic type arguments get implicit ``Any`` values. The type ``list`` is equivalent to ``list[Any]``, and so on.

    Example:

    ```python
    # mypy: disallow-any-generics

    # Error: Missing type parameters for generic type "list"  [type-arg]
    def remove_dups(items: list) -> list:
        ...
    ```

## 检查每个函数是否有注解 [no-untyped-def]

**Check that every function has an annotation [no-untyped-def]**

=== "中文"

    如果您使用 [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs)，mypy 要求所有函数都有类型注解（无论是 Python 3 注解还是类型注释）。

    示例：

    ```python
    # mypy: disallow-untyped-defs

    def inc(x):  # 错误：函数缺少类型注解 [no-untyped-def]
        return x + 1

    def inc_ok(x: int) -> int:  # 正确
        return x + 1

    class Counter:
            # 错误：函数缺少类型注解 [no-untyped-def]
            def __init__(self):
                self.value = 0

    class CounterOk:
            # 正确：如果 "__init__" 不接受参数，则需要显式的 "-> None"
            def __init__(self) -> None:
                self.value = 0
    ```

=== "英文"

    If you use [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs), mypy requires that all functions have annotations (either a Python 3 annotation or a type comment).

    Example:

    ```python
    # mypy: disallow-untyped-defs

    def inc(x):  # Error: Function is missing a type annotation  [no-untyped-def]
        return x + 1

    def inc_ok(x: int) -> int:  # OK
        return x + 1

    class Counter:
            # Error: Function is missing a type annotation  [no-untyped-def]
            def __init__(self):
                self.value = 0

    class CounterOk:
            # OK: An explicit "-> None" is needed if "__init__" takes no arguments
            def __init__(self) -> None:
                self.value = 0
    ```


## 检查类型转换是否冗余 [redundant-cast]

**Check that cast is not redundant [redundant-cast]**

=== "中文"

    如果您使用 [--warn-redundant-casts](../mypy_conf/command_line.md#warn-redundant-casts)，mypy 会生成错误，如果强制转换的源类型与目标类型相同。

    示例：

    ```python
    # mypy: warn-redundant-casts

    from typing import cast

    Count = int

    def example(x: Count) -> int:
        # 错误：对 "int" 的冗余强制转换 [redundant-cast]
        return cast(int, x)
    ```

=== "英文"

    If you use [--warn-redundant-casts](../mypy_conf/command_line.md#warn-redundant-casts), mypy will generate an error if the source type of a cast is the same as the target type.

    Example:

    ```python
    # mypy: warn-redundant-casts

    from typing import cast

    Count = int

    def example(x: Count) -> int:
        # Error: Redundant cast to "int"  [redundant-cast]
        return cast(int, x)
    ```


## 检查方法是否有冗余的 Self 注解 [redundant-self]

**Check that methods do not have redundant Self annotations [redundant-self]**

=== "中文"

    如果一个方法在返回类型或非 `self` 参数的类型中使用了 `Self` 类型，那么就不需要显式地注解 `self` 参数。这种注解虽然在 [PEP 673](https://peps.python.org/pep-0673/) 中被允许，但它们是多余的。如果启用这个错误代码，mypy 会在发现冗余的 `Self` 类型时生成错误。

    示例：

    ```python
    # mypy: enable-error-code="redundant-self"

    from typing import Self

    class C:
        # 错误：第一个方法参数的 "Self" 注解是多余的
        def copy(self: Self) -> Self:
            return type(self)()
    ```

=== "英文"

    If a method uses the ``Self`` type in the return type or the type of a non-self argument, there is no need to annotate the ``self`` argument explicitly. Such annotations are allowed by :pep:`673` but are redundant. If you enable this error code, mypy will generate an error if there is a redundant ``Self`` type.

    Example:

    ```python
    # mypy: enable-error-code="redundant-self"

    from typing import Self

    class C:
        # Error: Redundant "Self" annotation for the first method argument
        def copy(self: Self) -> Self:
            return type(self)()
    ```


## 检查比较是否重叠 [comparison-overlap]

**Check that comparisons are overlapping [comparison-overlap]**

=== "中文"

    如果您使用 [--strict-equality](../mypy_conf/command_line.md#strict-equality)，mypy 会在认为比较操作总是为真或为假时生成错误。这些通常是错误。有时 mypy 可能过于挑剔，而实际上比较操作可能是有用的。您可以使用 ``# type: ignore[comparison-overlap]`` 仅在特定行忽略此问题，而不是在所有地方禁用严格的相等检查。

    示例：

    ```python
    # mypy: strict-equality

    def is_magic(x: bytes) -> bool:
        # 错误：非重叠的相等检查（左操作数类型："bytes"，
        #        右操作数类型："str"） [comparison-overlap]
        return x == 'magic'
    ```

    我们可以通过将字符串字面量更改为字节字面量来修复此错误：

    ```python
    # mypy: strict-equality

    def is_magic(x: bytes) -> bool:
        return x == b'magic'  # 正确
    ```

=== "英文"

    If you use [--strict-equality](../mypy_conf/command_line.md#strict-equality), mypy will generate an error if it thinks that a comparison operation is always true or false. These are often bugs. Sometimes mypy is too picky and the comparison can actually be useful. Instead of disabling strict equality checking everywhere, you can use ``# type: ignore[comparison-overlap]`` to ignore the issue on a particular line only.

    Example:

    ```python
    # mypy: strict-equality

    def is_magic(x: bytes) -> bool:
        # Error: Non-overlapping equality check (left operand type: "bytes",
        #        right operand type: "str")  [comparison-overlap]
        return x == 'magic'
    ```

    We can fix the error by changing the string literal to a bytes literal:

    ```python
    # mypy: strict-equality

    def is_magic(x: bytes) -> bool:
        return x == b'magic'  # OK
    ```


## 检查是否没有调用未注解的函数 [no-untyped-call]

**Check that no untyped functions are called [no-untyped-call]**

=== "中文"

    如果您使用 [--disallow-untyped-calls](../mypy_conf/command_line.md#disallow-untyped-calls)，当您在注解了类型的函数中调用一个未注解的函数时，mypy 会生成错误。

    示例：

    ```python
    # mypy: disallow-untyped-calls

    def do_it() -> None:
        # 错误：在类型上下文中调用未注解的函数 "bad"  [no-untyped-call]
        bad()

    def bad():
        ...
    ```

=== "英文"

    If you use [--disallow-untyped-calls](../mypy_conf/command_line.md#disallow-untyped-calls), mypy generates an error when you call an unannotated function in an annotated function.

    Example:

    ```python
    # mypy: disallow-untyped-calls

    def do_it() -> None:
        # Error: Call to untyped function "bad" in typed context  [no-untyped-call]
        bad()

    def bad():
        ...
    ```


## 检查函数是否不返回 Any 值 [no-any-return]

**Check that function does not return Any value [no-any-return]**

=== "中文"

    如果您使用 [--warn-return-any](../mypy_conf/command_line.md#warn-return-any)，当您在一个被注解为返回非 ``Any`` 值的函数中返回一个 ``Any`` 类型的值时，mypy 会生成错误。

    示例：

    ```python
    # mypy: warn-return-any

    def fields(s):
        return s.split(',')

    def first_field(x: str) -> str:
        # 错误：从声明返回 "str" 的函数中返回了 Any  [no-any-return]
        return fields(x)[0]
    ```

=== "英文"

    If you use [--warn-return-any](../mypy_conf/command_line.md#warn-return-any), mypy generates an error if you return a value with an ``Any`` type in a function that is annotated to return a non-``Any`` value.

    Example:

    ```python
    # mypy: warn-return-any

    def fields(s):
            return s.split(',')

    def first_field(x: str) -> str:
        # Error: Returning Any from function declared to return "str"  [no-any-return]
        return fields(x)[0]
    ```


## 检查类型是否由于缺少导入而包含 Any 组件 [no-any-unimported]

**Check that types have no Any components due to missing imports [no-any-unimported]**

=== "中文"

    如果您使用 [--disallow-any-unimported](../mypy_conf/command_line.md#disallow-any-unimported)，当由于 mypy 无法解析导入而导致类型的某个组件变为 ``Any`` 时，mypy 会生成错误。这些“隐形”的 ``Any`` 类型可能会令人惊讶，并且意外地导致不准确的类型检查。

    在这个示例中，我们假设 mypy 无法找到模块 ``animals``，这意味着 ``Cat`` 在类型注解中回退为 ``Any``：

    ```python
    # mypy: disallow-any-unimported

    from animals import Cat  # type: ignore

    # 错误：由于未解析的导入，"feed" 的第一个参数变为 "Any"  [no-any-unimported]
    def feed(cat: Cat) -> None:
        ...
    ```

=== "英文"

    If you use [--disallow-any-unimported](../mypy_conf/command_line.md#disallow-any-unimported), mypy generates an error if a component of a type becomes ``Any`` because mypy couldn't resolve an import. These "stealth" ``Any`` types can be surprising and accidentally cause imprecise type checking.

    In this example, we assume that mypy can't find the module ``animals``, which means that ``Cat`` falls back to ``Any`` in a type annotation:

    ```python
    # mypy: disallow-any-unimported

    from animals import Cat  # type: ignore

    # Error: Argument 1 to "feed" becomes "Any" due to an unfollowed import  [no-any-unimported]
    def feed(cat: Cat) -> None:
        ...
    ```

## 检查语句或表达式是否不可达 [unreachable]

**Check that statement or expression is unreachable [unreachable]**

=== "中文"

    如果您使用 [--warn-unreachable](../mypy_conf/command_line.md#warn-unreachable)，当 mypy 认为某个语句或表达式永远不会被执行时，会生成错误。在大多数情况下，这是由于控制流或条件检查不正确，导致检查意外地总是为真或为假。

    示例：

    ```python
    # mypy: warn-unreachable

    def example(x: int) -> None:
        # 错误： "or" 的右操作数永远不会被计算  [unreachable]
        assert isinstance(x, int) or x == 'unused'

        return
        # 错误：语句不可达  [unreachable]
        print('unreachable')
    ```

=== "英文"

    If you use [--warn-unreachable](../mypy_conf/command_line.md#warn-unreachable), mypy generates an error if it thinks that a statement or expression will never be executed. In most cases, this is due to incorrect control flow or conditional checks that are accidentally always true or false.

    ```python
    # mypy: warn-unreachable

    def example(x: int) -> None:
        # Error: Right operand of "or" is never evaluated  [unreachable]
        assert isinstance(x, int) or x == 'unused'

        return
        # Error: Statement is unreachable  [unreachable]
        print('unreachable')
    ```


## 检查表达式是否冗余 [redundant-expr]

**Check that expression is redundant [redundant-expr]**

=== "中文"

    如果您使用 [--enable-error-code redundant-expr](../mypy_conf/command_line.md#enable-error-code)，当 mypy 认为某个表达式是冗余的时，会生成错误。

    示例：

    ```python
    # mypy: enable-error-code="redundant-expr"

    def example(x: int) -> None:
        # 错误： "and" 的左操作数总是为真  [redundant-expr]
        if isinstance(x, int) and x > 0:
            pass

        # 错误：条件总是为真  [redundant-expr]
        1 if isinstance(x, int) else 0

        # 错误：列表推导式中的条件总是为真  [redundant-expr]
        [i for i in range(x) if isinstance(i, int)]
    ```

=== "英文"

    If you use [--enable-error-code redundant-expr](../mypy_conf/command_line.md#enable-error-code), mypy generates an error if it thinks that an expression is redundant.

    ```python
    # mypy: enable-error-code="redundant-expr"

    def example(x: int) -> None:
        # Error: Left operand of "and" is always true  [redundant-expr]
        if isinstance(x, int) and x > 0:
            pass

        # Error: If condition is always true  [redundant-expr]
        1 if isinstance(x, int) else 0

        # Error: If condition in comprehension is always true  [redundant-expr]
        [i for i in range(x) if isinstance(i, int)]
    ```

## 警告只在某些执行路径中定义的变量 [possibly-undefined]

**Warn about variables that are defined only in some execution paths [possibly-undefined]**

=== "中文"

    如果您使用 [--enable-error-code possibly-undefined](../mypy_conf/command_line.md#enable-error-code)，当 mypy 无法验证一个变量在所有执行路径中都会被定义时，会生成错误。这包括变量定义出现在循环中、条件分支中、异常处理器中等情况。例如：

    ```python
    # mypy: enable-error-code="possibly-undefined"

    from typing import Iterable

    def test(values: Iterable[int], flag: bool) -> None:
        if flag:
            a = 1
        z = a + 1  # 错误：名称 "a" 可能未定义  [possibly-undefined]

        for v in values:
            b = v
        z = b + 1  # 错误：名称 "b" 可能未定义  [possibly-undefined]
    ```

=== "英文"

    If you use [--enable-error-code possibly-undefined](../mypy_conf/command_line.md#enable-error-code), mypy generates an error if it cannot verify that a variable will be defined in all execution paths. This includes situations when a variable definition appears in a loop, in a conditional branch, in an except handler, etc. For example:

    ```python
    # mypy: enable-error-code="possibly-undefined"

    from typing import Iterable

    def test(values: Iterable[int], flag: bool) -> None:
        if flag:
            a = 1
        z = a + 1  # Error: Name "a" may be undefined [possibly-undefined]

        for v in values:
            b = v
        z = b + 1  # Error: Name "b" may be undefined [possibly-undefined]
    ```


## 检查表达式在布尔上下文中是否隐式为真 [truthy-bool]

**Check that expression is not implicitly true in boolean context [truthy-bool]**

=== "中文"

    当布尔上下文中的表达式类型没有实现 `__bool__` 或 `__len__` 时发出警告。除非子类型实现了这些方法，否则表达式将始终被认为是 `True`，这可能会导致条件判断中的错误。

    作为例外，`object` 类型在布尔上下文中是被允许的。使用可迭代对象值作为布尔上下文中的条件有一个单独的错误代码（见下文）。

    示例：

    ```python
    # mypy: enable-error-code="truthy-bool"

    class Foo:
        pass
    foo = Foo()
    # 错误："foo" 的类型是 "Foo"，没有实现 __bool__ 或 __len__，因此在布尔上下文中总是为真
    if foo:
            ...
    ```

=== "英文"

    Warn when the type of an expression in a boolean context does not implement ``__bool__`` or ``__len__``. Unless one of these is implemented by a subtype, the expression will always be considered true, and there may be a bug in the condition.

    As an exception, the ``object`` type is allowed in a boolean context. Using an iterable value in a boolean context has a separate error code (see below).

    ```python
    # mypy: enable-error-code="truthy-bool"

    class Foo:
        pass
    foo = Foo()
    # Error: "foo" has type "Foo" which does not implement __bool__ or __len__ so it could always be true in boolean context
    if foo:
            ...
    ```


## 检查可迭代对象在布尔上下文中是否隐式为真 [truthy-iterable]

**Check that iterable is not implicitly true in boolean context [truthy-iterable]**

=== "中文"

    如果一个类型为 `Iterable` 的值用作布尔条件，会生成错误，因为 `Iterable` 不实现 `__len__` 或 `__bool__`。

    示例：

    ```python
    from typing import Iterable

    def transform(items: Iterable[int]) -> list[int]:
        # 错误："items" 的类型是 "Iterable[int]"，在布尔上下文中总是为真。考虑使用 "Collection[int]" 代替。  [truthy-iterable]
        if not items:
            return [42]
        return [x + 1 for x in items]
    ```

    如果 `transform` 函数被传入 `Generator` 参数，如 `int(x) for x in []`，这个函数将不会返回 `[42]`，这可能不是预期的结果。当然，也有可能 `transform` 只会被 `list` 或其他容器对象调用，并且 `if not items` 检查实际上是有效的。如果是这种情况，建议将 `items` 注解为 `Collection[int]` 而不是 `Iterable[int]`。

=== "英文"

    Generate an error if a value of type ``Iterable`` is used as a boolean condition, since ``Iterable`` does not implement ``__len__`` or ``__bool__``.

    Example:

    ```python
    from typing import Iterable

    def transform(items: Iterable[int]) -> list[int]:
        # Error: "items" has type "Iterable[int]" which can always be true in boolean context. Consider using "Collection[int]" instead.  [truthy-iterable]
        if not items:
            return [42]
        return [x + 1 for x in items]
    ```


    If ``transform`` is called with a ``Generator`` argument, such as ``int(x) for x in []``, this function would not return ``[42]`` unlike what might be intended. Of course, it's possible that ``transform`` is only called with ``list`` or other container objects, and the ``if not items`` check is actually valid. If that is the case, it is recommended to annotate ``items`` as ``Collection[int]`` instead of ``Iterable[int]``.

## 检查 ``# type: ignore`` 是否包含错误代码 [ignore-without-code]

**Check that ``# type: ignore`` include an error code [ignore-without-code]**

=== "中文"

    当 `# type: ignore` 注释没有指定任何错误代码时发出警告。这可以明确忽略的意图，并确保仅忽略预期的错误。

    示例：

    ```python
    # mypy: enable-error-code="ignore-without-code"

    class Foo:
        def __init__(self, name: str) -> None:
            self.name = name

    f = Foo('foo')

    # 这一行的注释没有指定错误代码，因此：
    # - 预期错误 'assignment' 和
    # - 意外错误 'attr-defined'
    # 都被忽略了。
    # 错误： "type: ignore" 注释没有错误代码（考虑使用 "type: ignore[attr-defined]"）
    f.nme = 42  # type: ignore

    # 这一行正确地警告了属性名称中的错字
    # 错误："Foo" 没有属性 "nme"；也许是 "name"？
    f.nme = 42  # type: ignore[assignment]
    ```

=== "英文"

    Warn when a ``# type: ignore`` comment does not specify any error codes. This clarifies the intent of the ignore and ensures that only the expected errors are silenced.

    Example:

    ```python
    # mypy: enable-error-code="ignore-without-code"

    class Foo:
        def __init__(self, name: str) -> None:
            self.name = name

    f = Foo('foo')

    # This line has a typo that mypy can't help with as both:
    # - the expected error 'assignment', and
    # - the unexpected error 'attr-defined'
    # are silenced.
    # Error: "type: ignore" comment without error code (consider "type: ignore[attr-defined]" instead)
    f.nme = 42  # type: ignore

    # This line warns correctly about the typo in the attribute name
    # Error: "Foo" has no attribute "nme"; maybe "name"?
    f.nme = 42  # type: ignore[assignment]
    ```


## 检查 awaitable 返回值是否被使用 [unused-awaitable]

**Check that awaitable return value is used [unused-awaitable]**

=== "中文"

    如果你使用 [--enable-error-code unused-awaitable](../mypy_conf/command_line.md#enable-error-code)，当你不使用定义了 `__await__` 的返回值时，mypy 会生成错误。

    示例：

    ```python
    # mypy: enable-error-code="unused-awaitable"

    import asyncio

    async def f() -> int: ...

    async def g() -> None:
        # 错误：类型为 "Task[int]" 的值必须被使用
        #       你是否遗漏了一个 await？
        asyncio.create_task(f())
    ```

    你可以将值赋给一个临时的、未使用的变量来消除这个错误：

    ```python
    async def g() -> None:
        _ = asyncio.create_task(f())  # 无错误
    ```

=== "英文"

    If you use [--enable-error-code unused-awaitable](../mypy_conf/command_line.md#enable-error-code), mypy generates an error if you don't use a returned value that defines ``__await__``.

    Example:

    ```python
    # mypy: enable-error-code="unused-awaitable"

    import asyncio

    async def f() -> int: ...

    async def g() -> None:
        # Error: Value of type "Task[int]" must be used
        #        Are you missing an await?
        asyncio.create_task(f())
    ```

    You can assign the value to a temporary, otherwise unused variable to silence the error:

    ```python
    async def g() -> None:
        _ = asyncio.create_task(f())  # No error
    ```


## 检查 ``# type: ignore`` 注释是否被使用 [unused-ignore]

**Check that ``# type: ignore`` comment is used [unused-ignore]**

=== "中文"

    如果你使用 [--enable-error-code unused-ignore](../mypy_conf/command_line.md#enable-error-code) 或 [--warn-unused-ignores](../mypy_conf/command_line.md#warn-unused-ignores)，当你使用了 `# type: ignore` 注释但实际上该行不会生成错误时，mypy 会生成错误。

    示例：

    ```python
    # 使用 "mypy --warn-unused-ignores ..."

    def add(a: int, b: int) -> int:
        # 错误：未使用的 "type: ignore" 注释
        return a + b  # type: ignore
    ```

    请注意，由于这种注释的特定性质，唯一可以选择性地忽略它的方式是明确包含错误代码。同时，如果 `# type: ignore` 注释未被使用，因为代码由于平台或版本检查等原因静态上不可达，则不会显示此错误。

    示例：

    ```python
    # 使用 "mypy --warn-unused-ignores ..."

    import sys

    try:
        # "[unused-ignore]" 是在 Python 3.8 和 3.9 中干净运行 mypy 的必要条件
        # 在这些版本中，该模块被添加
        import graphlib  # type: ignore[import,unused-ignore]
    except ImportError:
        pass

    if sys.version_info >= (3, 9):
        # 以下代码在 Python 3.8 和 Python 3.9 上都不会生成错误
        42 + "testing..."  # type: ignore
    ```

=== "英文"

    If you use [--enable-error-code unused-ignore](../mypy_conf/command_line.md#enable-error-code), or [--warn-unused-ignores](../mypy_conf/command_line.md#warn-unused-ignores) mypy generates an error if you don't use a ``# type: ignore`` comment, i.e. if there is a comment, but there would be no error generated by mypy on this line anyway.

    Example:

    ```python
    # Use "mypy --warn-unused-ignores ..."

    def add(a: int, b: int) -> int:
        # Error: unused "type: ignore" comment
        return a + b  # type: ignore
    ```


    Note that due to a specific nature of this comment, the only way to selectively silence it, is to include the error code explicitly. Also note that this error is not shown if the ``# type: ignore`` is not used due to code being statically unreachable (e.g. due to platform or version checks).

    Example:

    ```python
    # Use "mypy --warn-unused-ignores ..."

    import sys

    try:
        # The "[unused-ignore]" is needed to get a clean mypy run
        # on both Python 3.8, and 3.9 where this module was added
        import graphlib  # type: ignore[import,unused-ignore]
    except ImportError:
        pass

    if sys.version_info >= (3, 9):
        # The following will not generate an error on either
        # Python 3.8, or Python 3.9
        42 + "testing..."  # type: ignore
    ```


## 检查在重写基类方法时是否使用 ``@override`` [explicit-override]

**Check that ``@override`` is used when overriding a base class method [explicit-override]**

=== "中文"

    如果你使用 [--enable-error-code explicit-override](../mypy_conf/command_line.md#enable-error-code)，当你覆盖基类方法而没有使用 `@override` 装饰器时，mypy 会生成错误。对于 `__init__` 或 `__new__` 的覆盖不会触发错误。有关更多信息，请参见 [PEP 698](https://peps.python.org/pep-0698/#strict-enforcement-per-project)。

    !!! 注

        从 Python 3.12 开始，`@override` 装饰器可以从 `typing` 导入。要在旧版本的 Python 中使用它，请从 `typing_extensions` 导入。

    示例：

    ```python
    # mypy: enable-error-code="explicit-override"

    from typing import override

    class Parent:
        def f(self, x: int) -> None:
            pass

        def g(self, y: int) -> None:
            pass


    class Child(Parent):
        def f(self, x: int) -> None:  # 错误：缺少 @override 装饰器
            pass

        @override
        def g(self, y: int) -> None:
            pass
    ```

=== "英文"

    If you use [--enable-error-code explicit-override](../mypy_conf/command_line.md#enable-error-code) mypy generates an error if you override a base class method without using the ``@override`` decorator. An error will not be emitted for overrides of ``__init__`` or ``__new__``. See [PEP 698](https://peps.python.org/pep-0698/#strict-enforcement-per-project).

    !!! note 

        Starting with Python 3.12, the ``@override`` decorator can be imported from ``typing``. To use it with older Python versions, import it from ``typing_extensions`` instead.

    Example:

    ```python
    # mypy: enable-error-code="explicit-override"

    from typing import override

    class Parent:
        def f(self, x: int) -> None:
            pass

        def g(self, y: int) -> None:
            pass


    class Child(Parent):
        def f(self, x: int) -> None:  # Error: Missing @override decorator
            pass

        @override
        def g(self, y: int) -> None:
            pass
    ```


## 检查可变属性的重写是否安全 [mutable-override]

**Check that overrides of mutable attributes are safe [mutable-override]**

=== "中文"

    `mutable-override` 将启用对可变属性的安全覆盖检查。由于历史原因，以及这是 Python 中相对常见的模式，此检查默认情况下未启用。以下示例是不安全的，当启用此错误代码时将会被标记：

    ```python
    from typing import Any

    class C:
        x: float
        y: float
        z: float

    class D(C):
        x: int  # 错误：对可变属性的协变覆盖
                # (基类 "C" 定义的类型为 "float",
                # 表达式的类型为 "int")  [mutable-override]
        y: float  # 正确
        z: Any  # 正确

    def f(c: C) -> None:
        c.x = 1.1
    d = D()
    f(d)
    d.x >> 1  # 这将在运行时崩溃，因为 d.x 现在是 float，而不是 int
    ```

=== "英文"

    `mutable-override` will enable the check for unsafe overrides of mutable attributes. For historical reasons, and because this is a relatively common pattern in Python, this check is not enabled by default. The example below is unsafe, and will be flagged when this error code is enabled:

    ```python
    from typing import Any

    class C:
        x: float
        y: float
        z: float

    class D(C):
        x: int  # Error: Covariant override of a mutable attribute
                # (base class "C" defined the type as "float",
                # expression has type "int")  [mutable-override]
        y: float  # OK
        z: Any  # OK

    def f(c: C) -> None:
        c.x = 1.1
    d = D()
    f(d)
    d.x >> 1  # This will crash at runtime, because d.x is now float, not an int
    ```


## 检查 ``reveal_type`` 是否从 typing 或 typing_extensions 导入 [unimported-reveal]

**Check that ``reveal_type`` is imported from typing or typing_extensions [unimported-reveal]**

=== "中文"

    Mypy 曾经有一个 `reveal_type` 特殊内置函数，它仅在类型检查期间存在。在运行时，它会导致预期的 `NameError`，这可能会在生产环境中造成实际问题，并且对 mypy 隐藏。

    但在 Python 3.11 中，添加了 `typing.reveal_type()`。`typing_extensions` 将这个帮助函数移植到了所有受支持的 Python 版本中。

    现在用户实际上可以导入 `reveal_type` 以使运行时代码安全。

    !!! note

        从 Python 3.11 开始，`reveal_type` 函数可以从 `typing` 导入。要在旧版本的 Python 中使用它，请从 `typing_extensions` 导入。

    示例：

    ```python
    # mypy: enable-error-code="unimported-reveal"

    x = 1
    reveal_type(x)  # 注：揭示的类型是 "builtins.int" \
                    # 错误：名称 "reveal_type" 未定义
    ```

    正确用法：

    ```python
    # mypy: enable-error-code="unimported-reveal"
    from typing import reveal_type   # 或 `typing_extensions`

    x = 1
    # 这不会引发错误：
    reveal_type(x)  # 注：揭示的类型是 "builtins.int"
    ```

    当启用此代码时，使用 `reveal_locals` 始终会引发错误，因为没有办法导入它。

=== "英文"

    Mypy used to have ``reveal_type`` as a special builtin that only existed during type-checking. In runtime it fails with expected ``NameError``, which can cause real problem in production, hidden from mypy.

    But, in Python3.11 [typing.reveal_type()] was added. ``typing_extensions`` ported this helper to all supported Python versions.

    Now users can actually import ``reveal_type`` to make the runtime code safe.

    !!! note 

        Starting with Python 3.11, the ``reveal_type`` function can be imported from ``typing``. To use it with older Python versions, import it from ``typing_extensions`` instead.

    ```python
    # mypy: enable-error-code="unimported-reveal"

    x = 1
    reveal_type(x)  # Note: Revealed type is "builtins.int" \
                    # Error: Name "reveal_type" is not defined
    ```


    Correct usage:

    ```python
    # mypy: enable-error-code="unimported-reveal"
    from typing import reveal_type   # or `typing_extensions`

    x = 1
    # This won't raise an error:
    reveal_type(x)  # Note: Revealed type is "builtins.int"
    ```


    When this code is enabled, using ``reveal_locals`` is always an error, because there's no way one can import it.

## 检查 ``TypeIs`` 是否缩小了类型 [narrowed-type-not-subtype]

**Check that ``TypeIs`` narrows types [narrowed-type-not-subtype]**

=== "中文"

    [PEP 742](https://peps.python.org/pep-0742/) 要求当使用 `TypeIs` 时，缩小后的类型必须是原始类型的子类型：

    ```python
    from typing_extensions import TypeIs

    def f(x: int) -> TypeIs[str]:  # 错误，str 不是 int 的子类型
        ...

    def g(x: object) -> TypeIs[str]:  # 正确
        ...
    ```

=== "英文"

    [PEP 742](https://peps.python.org/pep-0742/) requires that when ``TypeIs`` is used, the narrowed type must be a subtype of the original type

    ```python
    from typing_extensions import TypeIs

    def f(x: int) -> TypeIs[str]:  # Error, str is not a subtype of int
        ...

    def g(x: object) -> TypeIs[str]:  # OK
        ...
    ```

[typing.reveal_type()]: https://docs.python.org/3/library/typing.html#typing.reveal_type