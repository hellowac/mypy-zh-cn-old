# 可选检查的错误代码

**Error codes for optional checks**

=== "中文"

=== "英文"

This section documents various errors codes that mypy generates only if you enable certain options. See [Error codes](./error_codes.md) for general documentation about error codes and their configuration. [Error codes enabled by default](./error_code_list.md) documents error codes that are enabled by default.

!!! note 

    The examples in this section use [inline configuration](../mypy_conf/inline_config.md) to specify mypy options. You can also set the same options by using a [configuration file](../mypy_conf/config_file.md) or [command-line options](../mypy_conf/command_line.md).

## 检查类型参数是否存在 [type-arg]

**Check that type arguments exist [type-arg]**

=== "中文"

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

=== "英文"

If you use [--warn-redundant-casts(../mypy_conf/command_line.md#warn-redundant-casts), mypy will generate an error if the source type of a cast is the same as the target type.

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

=== "英文"

If you use [--disallow-untyped-calls](../mypy_conf/command_line.md#disallow-untyped-calls), mypy generates an error when you
call an unannotated function in an annotated function.

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

=== "英文"

If you use [--enable-error-code unused-awaitable](../mypy_conf/command_line.md#enable-error-code),
mypy generates an error if you don't use a returned value that defines ``__await__``.

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