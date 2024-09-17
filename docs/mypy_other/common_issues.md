# 常见问题及解决方案

**Common issues and solutions**

=== "中文"

    本节提供了在需要更新代码以使用静态类型时的示例，以及当 mypy 无法按预期工作时的解决方案建议。静态类型代码通常与普通 Python 代码几乎相同（除了类型注解），但有时你需要稍微调整方法。

=== "英文"

    This section has examples of cases when you need to update your code to use static typing, and ideas for working around issues if mypy doesn't work as expected. Statically typed code is often identical to normal Python code (except for type annotations), but sometimes you need to do things slightly differently.

## 显然错误的代码未报告错误

**No errors reported for obviously wrong code**

=== "中文"

    有几个常见的原因导致明显错误的代码没有被标记为错误。

    **函数没有注解。**

    没有任何注解（既没有参数注解，也没有返回类型注解）的函数不会进行类型检查，即使是最明显的类型错误（例如 ``2 + 'a'``）也会被默默忽略。解决方法是添加注解。如果无法添加注解，可以使用 [--check-untyped-defs](../mypy_conf/command_line.md#check-untyped-defs) 进行检查。

    示例：

    ```python
    def foo(a):
        return '(' + a.split() + ')'  # 没有错误！
    ```

    即使 ``a.split()`` 显然是一个列表（作者可能想用 ``a.strip()``），也不会报告错误。添加注解后错误将会被报告：

    ```python
    def foo(a: str) -> str:
        return '(' + a.split() + ')'
    # 错误: 不支持的操作数类型用于 + ("str" 和 "list[str]")
    ```

    如果不知道要添加什么类型，可以使用 ``Any``，但要注意：

    **涉及的某些值具有类型 'Any'。**

    扩展上述示例，如果我们省略了 ``a`` 的注解，则不会出现错误：

    ```python
    def foo(a) -> str:
        return '(' + a.split() + ')'  # 没有错误！
    ```

    原因是，如果 ``a`` 的类型未知，则 ``a.split()`` 的类型也未知，因此被推断为 ``Any``，将一个字符串添加到 ``Any`` 上不会引发错误。

    如果在调试这些情况时遇到困难，`reveal_type()` 可能会有所帮助。

    请注意，有时库的存根由于类型信息不准确，也可能是 ``Any`` 值的来源。

    **[\_\_init\_\_](https://docs.python.org/3/reference/datamodel.html#object.__init__) 方法没有注解的参数和返回类型注解。**

    这基本上是以上两种情况的结合，因为没有注解的 ``__init__`` 可能会导致 ``Any`` 类型渗透到实例变量中：

    ```python
    class Bad:
        def __init__(self):
            self.value = "asdf"
            1 + "asdf"  # 没有错误！

    bad = Bad()
    bad.value + 1           # 没有错误！
    reveal_type(bad)        # 显示的类型是 "__main__.Bad"
    reveal_type(bad.value)  # 显示的类型是 "Any"

    class Good:
        def __init__(self) -> None:  # 显式返回 None
            self.value = value
    ```

    **一些导入可能会被默默忽略。**

    一个常见的 ``Any`` 值来源是 [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports) 标志。

    当使用 [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports) 时，任何无法找到的导入模块会被默默替换为 ``Any``。

    为了解决这个问题，可以简单地省略 [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports)。如 [缺失的导入](../mypy_conf/running_mypy.md#缺失的导入) 中所提到的，在每个模块上设置 ``ignore_missing_imports=True`` 会减少意外情况的发生，强烈建议这样做。

    使用 [--follow-imports=skip](../mypy_conf/command_line.md#follow-imports) 标志也可能导致问题。强烈不推荐使用这些标志，除非在相对小众的情况下需要。有关更多信息，请参阅 [跟随导入](../mypy_conf/running_mypy.md#跟随导入)。

    **mypy 认为你的代码无法到达。**

    有关更多信息，请参见 [无法到达的代码](#无法到达的代码)。

    **一个注解为非可选类型的函数返回 'None' 而 mypy 不报错。**

    ```python
    def foo() -> str:
        return None  # 没有错误！
    ```

    你可能已禁用严格的可选检查（有关更多信息，请参见 [--no-strict-optional](../mypy_conf/command_line.md#no-strict-optional)）。

=== "英文"

    There are several common reasons why obviously wrong code is not flagged as an error.

    **The function containing the error is not annotated.**

    Functions that do not have any annotations (neither for any argument nor for the return type) are not type-checked, and even the most blatant type errors (e.g. ``2 + 'a'``) pass silently.  The solution is to add annotations. Where that isn't possible, functions without annotations can be checked using [--check-untyped-defs](../mypy_conf/command_line.md#check-untyped-defs).

    Example:

    ```python
    def foo(a):
        return '(' + a.split() + ')'  # No error!
    ```

    This gives no error even though ``a.split()`` is "obviously" a list (the author probably meant ``a.strip()``).  The error is reported once you add annotations:

    ```python
    def foo(a: str) -> str:
        return '(' + a.split() + ')'
    # error: Unsupported operand types for + ("str" and "list[str]")
    ```

    If you don't know what types to add, you can use ``Any``, but beware:

    **One of the values involved has type 'Any'.**

    Extending the above example, if we were to leave out the annotation for ``a``, we'd get no error:

    ```python
    def foo(a) -> str:
        return '(' + a.split() + ')'  # No error!
    ```

    The reason is that if the type of ``a`` is unknown, the type of ``a.split()`` is also unknown, so it is inferred as having type ``Any``, and it is no error to add a string to an ``Any``.

    If you're having trouble debugging such situations, `reveal_type()` might come in handy.

    Note that sometimes library stubs with imprecise type information can be a source of ``Any`` values.

    [\_\_init\_\_](https://docs.python.org/3/reference/datamodel.html#object.__init__) **method has no annotated arguments and no return type annotation.**

    This is basically a combination of the two cases above, in that ``__init__`` without annotations can cause ``Any`` types leak into instance variables:

    ```python
    class Bad:
        def __init__(self):
            self.value = "asdf"
            1 + "asdf"  # No error!

    bad = Bad()
    bad.value + 1           # No error!
    reveal_type(bad)        # Revealed type is "__main__.Bad"
    reveal_type(bad.value)  # Revealed type is "Any"

    class Good:
        def __init__(self) -> None:  # Explicitly return None
            self.value = value
    ```

    **Some imports may be silently ignored**.

    A common source of unexpected ``Any`` values is the [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports) flag.

    When you use [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports), any imported module that cannot be found is silently replaced with ``Any``.

    To help debug this, simply leave out [--ignore-missing-imports](../mypy_conf/command_line.md#ignore-missing-imports). As mentioned in [Missing imports](../mypy_conf/running_mypy.md#缺失的导入), setting ``ignore_missing_imports=True`` on a per-module basis will make bad surprises less likely and is highly encouraged.

    Use of the [--follow-imports=skip](../mypy_conf/command_line.md#follow-imports) flags can also cause problems. Use of these flags is strongly discouraged and only required in relatively niche situations. See [Following imports](../mypy_conf/running_mypy.md#跟随导入) for more information.

    **mypy considers some of your code unreachable**.

    See [unreachable](#无法到达的代码) for more information.

    **A function annotated as returning a non-optional type returns 'None' and mypy doesn't complain**.

    ```python
    def foo() -> str:
        return None  # No error!
    ```

    You may have disabled strict optional checking (see [--no-strict-optional](../mypy_conf/command_line.md#no-strict-optional) for more).

## 虚假错误和局部静默检查器

**Spurious errors and locally silencing the checker**

=== "中文"

    你可以使用 ``# type: ignore`` 注释来在特定行上抑制类型检查器的错误。例如，假设我们的代码使用了 C 扩展模块 ``frobnicate``，但没有可用的存根。Mypy 会对此发出警告，因为它没有该模块的信息：

    ```python
    import frobnicate  # 错误：没有模块 "frobnicate"
    frobnicate.start()
    ```

    你可以添加 ``# type: ignore`` 注释来告诉 mypy 忽略此错误：

    ```python
    import frobnicate  # type: ignore
    frobnicate.start()  # 没问题！
    ```

    第二行现在是正常的，因为忽略注释使得名称 ``frobnicate`` 得到了隐式的 ``Any`` 类型。

    !!! note

        你可以使用 ``# type: ignore[<code>]`` 的形式只忽略特定的错误。这样你不太可能忽略那些不安全的意外错误，同时这也会记录注释的目的。有关更多信息，请参见 [错误代码](./error_codes.md)。

    !!! note

        ``# type: ignore`` 注释只会在 mypy 无法找到特定模块的信息时才会分配隐式的 ``Any`` 类型。因此，如果我们确实有 ``frobnicate`` 的存根，那么 mypy 会忽略 ``# type: ignore`` 注释，像平常一样检查存根。

    另一种选择是显式地将值注解为 ``Any`` 类型 -- mypy 允许你对 ``Any`` 类型的值执行任意操作。有时，对于某个特定值没有更精确的类型，尤其是当你使用动态 Python 特性如 [\_\_getattr\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattr__) 时：

    ```python
    class Wrapper:
        ...
        def __getattr__(self, a: str) -> Any:
            return getattr(self._wrapped, a)
    ```

    最后，你可以为生成虚假错误的文件创建一个存根文件（``.pyi``）。Mypy 将只查看存根文件并忽略实现，因为存根文件的优先级高于 ``.py`` 文件。

=== "英文"

    You can use a ``# type: ignore`` comment to silence the type checker on a particular line. For example, let's say our code is using the C extension module ``frobnicate``, and there's no stub available. Mypy will complain about this, as it has no information about the module:

    ```python
    import frobnicate  # Error: No module "frobnicate"
    frobnicate.start()
    ```

    You can add a ``# type: ignore`` comment to tell mypy to ignore this error:

    ```python
    import frobnicate  # type: ignore
    frobnicate.start()  # Okay!
    ```

    The second line is now fine, since the ignore comment causes the name ``frobnicate`` to get an implicit ``Any`` type.

    !!! note 

        You can use the form ``# type: ignore[<code>]`` to only ignore specific errors on the line. This way you are less likely to silence unexpected errors that are not safe to ignore, and this will also document what the purpose of the comment is.  See [Error codes](./error_codes.md) for more information.

    !!! note 

        The ``# type: ignore`` comment will only assign the implicit ``Any`` type if mypy cannot find information about that particular module. So, if we did have a stub available for ``frobnicate`` then mypy would ignore the ``# type: ignore`` comment and typecheck the stub as usual.

    Another option is to explicitly annotate values with type ``Any`` -- mypy will let you perform arbitrary operations on ``Any`` values. Sometimes there is no more precise type you can use for a particular value, especially if you use dynamic Python features such as [\_\_getattr\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattr__):

    ```python
    class Wrapper:
        ...
        def __getattr__(self, a: str) -> Any:
            return getattr(self._wrapped, a)
    ```

    Finally, you can create a stub file (``.pyi``) for a file that generates spurious errors. Mypy will only look at the stub file and ignore the implementation, since stub files take precedence over ``.py`` files.

## 忽略整个文件

**Ignoring a whole file**

=== "中文"

    * 若只想忽略错误，可以使用顶层的 ``# mypy: ignore-errors`` 注释。
    * 若只想忽略特定错误代码的错误，可以使用顶层的 ``# mypy: disable-error-code="..."`` 注释。例如：``# mypy: disable-error-code="truthy-bool, ignore-without-code"``。
    * 若要将模块的内容替换为 ``Any`` 类型，可以使用每个模块的 ``follow_imports = skip`` 设置。有关详细信息，请参见 [跟随导入](../mypy_conf/running_mypy.md#跟随导入)。

    请注意，模块顶部的 ``# type: ignore`` 注释（在任何语句之前，包括导入或文档字符串）会导致忽略整个模块的内容。这种行为可能会令人惊讶，并导致出现“模块 ... 没有属性 ... [attr-defined]”的错误。

=== "英文"

    * To only ignore errors, use a top-level ``# mypy: ignore-errors`` comment instead.
    * To only ignore errors with a specific error code, use a top-level ``# mypy: disable-error-code="..."`` comment. Example: ``# mypy: disable-error-code="truthy-bool, ignore-without-code"``
    * To replace the contents of a module with ``Any``, use a per-module ``follow_imports = skip``. See [Following imports](../mypy_conf/running_mypy.md#跟随导入) for details.

    Note that a ``# type: ignore`` comment at the top of a module (before any statements, including imports or docstrings) has the effect of ignoring the entire contents of the module. This behaviour can be surprising and result in "Module ... has no attribute ... [attr-defined]" errors.

## 运行时代码问题

**Issues with code at runtime**

=== "中文"

    使用类型注解的惯用法有时会与某个 Python 版本认为合法的代码产生冲突。这些冲突可能导致在运行代码时出现以下一些错误：

    * **``ImportError``**：由于循环导入。
    * **``NameError: name "X" is not defined``**：由于前向引用。
    * **``TypeError: 'type' object is not subscriptable``**：由于在运行时类型不是通用类型。
    * **``ImportError``** 或 **``ModuleNotFoundError``**：由于使用了在运行时不可用的存根定义。
    * **``TypeError: unsupported operand type(s) for |: 'type' and 'type'``**：由于使用了新的语法。

    有关如何处理这些问题，请参见 [运行时注解问题](../mypy/annotation_issue_at_runtime.md)。

=== "英文"

    Idiomatic use of type annotations can sometimes run up against what a given version of Python considers legal code. These can result in some of the following errors when trying to run your code:

    * ``ImportError`` from circular imports
    * ``NameError: name "X" is not defined`` from forward references
    * ``TypeError: 'type' object is not subscriptable`` from types that are not generic at runtime
    * ``ImportError`` or ``ModuleNotFoundError`` from use of stub definitions not available at runtime
    * ``TypeError: unsupported operand type(s) for |: 'type' and 'type'`` from use of new syntax

    For dealing with these, see [Annotation issues at runtime](../mypy/annotation_issue_at_runtime.md).

## Mypy 运行速度慢

**Mypy runs are slow**

=== "中文"

    如果你觉得运行 mypy 很慢，你可能应该使用 [mypy 守护进程](../mypy_conf/mypy_daemon.md)，它可以将增量 mypy 运行的速度提高 10 倍或更多。 [远程缓存](./additional_features.md#使用远程缓存加速-mypy-运行) 可以使冷启动的 mypy 运行速度提高几倍。

=== "英文"

    If your mypy runs feel slow, you should probably use the [mypy daemon](../mypy_conf/mypy_daemon.md), which can speed up incremental mypy runtimes by a factor of 10 or more. [Remote caching](./additional_features.md#使用远程缓存加速-mypy-运行) can make cold mypy runs several times faster.

## 空集合的类型

**Types of empty collections**

=== "中文"

    当你将一个空列表或字典分配给一个新变量时，通常需要指定类型，如前面提到的：

    ```python
    a: list[int] = []
    ```

    如果没有类型注解，mypy 并不总是能够准确推断 ``a`` 的类型。

    在动态类型的函数中，你可以使用简单的空列表字面量（因为此时 ``a`` 的类型将隐式为 ``Any``，无需推断），如果变量的类型在之前已声明或推断过，或者在相同作用域内进行简单的修改操作（如列表的 ``append``）：

    ```python
    a = []  # 可以，因为紧接着有 append，推断类型为 list[int]
    for i in range(n):
        a.append(i * i)
    ```

    然而，在更复杂的情况下，可能需要显式的类型注解（mypy 会告诉你）。通常，注解不仅有助于 mypy，也使阅读代码的每个人更容易理解代码！

=== "英文"

    You often need to specify the type when you assign an empty list or dict to a new variable, as mentioned earlier:

    ```python
    a: list[int] = []
    ```

    Without the annotation mypy can't always figure out the precise type of ``a``.

    You can use a simple empty list literal in a dynamically typed function (as the type of ``a`` would be implicitly ``Any`` and need not be inferred), if type of the variable has been declared or inferred before, or if you perform a simple modification operation in the same scope (such as ``append`` for a list):

    ```python
    a = []  # Okay because followed by append, inferred type list[int]
    for i in range(n):
        a.append(i * i)
    ```

    However, in more complex cases an explicit type annotation can be required (mypy will tell you this). Often the annotation can make your code easier to understand, so it doesn't only help mypy but everybody who is reading the code!

## 具有不兼容类型的重新定义

**Redefinitions with incompatible types**

=== "中文"

    在函数中，每个名称只有一个“声明”的类型。你可以重用 `for` 循环索引等，但如果你希望在一个函数内使用具有多种类型的变量，你可能需要使用多个变量（或者可能声明变量为 ``Any`` 类型）。

    ```python
    def f() -> None:
        n = 1
        ...
        n = 'x'  # 错误：赋值中的类型不兼容（表达式的类型为 "str"，变量的类型为 "int"）
    ```

    !!! note

    使用 [--allow-redefinition](../mypy_conf/command_line.md#allow-redefinition) 标志可以在某些情况下抑制此错误。

    请注意，你可以将变量重新定义为更*精确*或更具体的类型。例如，你可以将一个序列（它不支持 ``sort()``）重新定义为列表，并在原地对其进行排序：

    ```python
    def f(x: Sequence[int]) -> None:
        # 这里 x 的类型是 Sequence[int]；我们不知道具体类型。
        x = list(x)
        # 这里 x 的类型是 list[int]。
        x.sort()  # 可以！
    ```

    有关更多信息，请参见 [类型缩小](../mypy/type_narrowing.md)。

=== "英文"

    Each name within a function only has a single 'declared' type. You can reuse for loop indices etc., but if you want to use a variable with multiple types within a single function, you may need to instead use multiple variables (or maybe declare the variable with an ``Any`` type).

    ```python
    def f() -> None:
        n = 1
        ...
        n = 'x'  # error: Incompatible types in assignment (expression has type "str", variable has type "int")
    ```

    !!! note 

    Using the [--allow-redefinition](../mypy_conf/command_line.md#allow-redefinition) flag can suppress this error in several cases.

    Note that you can redefine a variable with a more *precise* or a more concrete type. For example, you can redefine a sequence (which does not support ``sort()``) as a list and sort it in-place:

    ```python
    def f(x: Sequence[int]) -> None:
        # Type of x is Sequence[int] here; we don't know the concrete type.
        x = list(x)
        # Type of x is list[int] here.
        x.sort()  # Okay!
    ```

    See [Type narrowing](../mypy/type_narrowing.md) for more information.

## 不变性 vs 协变性

**Invariance vs covariance**

=== "中文"

    大多数可变的泛型集合都是不可变的，而 mypy 默认将所有用户定义的泛型类视为不可变（有关动机，请参见 [泛型类型的变异性](../mypy/generics.md)）。这可能会在与类型推断结合使用时导致一些意外的错误。例如：

    ```python
    class A: ...
    class B(A): ...

    lst = [A(), A()]  # 推断类型是 list[A]
    new_lst = [B(), B()]  # 推断类型是 list[B]
    lst = new_lst  # mypy 会对此提出警告，因为 List 是不可变的
    ```

    在这种情况下的可能策略包括：

    * 使用显式类型注解：

        ```python
        new_lst: list[A] = [B(), B()]
        lst = new_lst  # 可以
        ```

    * 复制右侧的值：

        ```python
        lst = list(new_lst) # 也可以
        ```

    * 尽可能使用不可变集合作为注解：

        ```python
        def f_bad(x: list[A]) -> A:
            return x[0]
        f_bad(new_lst) # 失败
        
        def f_good(x: Sequence[A]) -> A:
            return x[0]
        f_good(new_lst) # 可以
        ```

=== "英文"

    Most mutable generic collections are invariant, and mypy considers all user-defined generic classes invariant by default (see [Variance of generic types](../mypy/generics.md) for motivation). This could lead to some unexpected errors when combined with type inference. For example:

    ```python
    class A: ...
    class B(A): ...

    lst = [A(), A()]  # Inferred type is list[A]
    new_lst = [B(), B()]  # inferred type is list[B]
    lst = new_lst  # mypy will complain about this, because List is invariant
    ```

    Possible strategies in such situations are:

    * Use an explicit type annotation:

        ```python
        new_lst: list[A] = [B(), B()]
        lst = new_lst  # OK
        ```

    * Make a copy of the right hand side:

        ```python
        lst = list(new_lst) # Also OK
        ```

    * Use immutable collections as annotations whenever possible:

        ```python
        
            def f_bad(x: list[A]) -> A:
                return x[0]
            f_bad(new_lst) # Fails
        
            def f_good(x: Sequence[A]) -> A:
                return x[0]
            f_good(new_lst) # OK
        ```

## 将超类型声明为变量类型

**Declaring a supertype as variable type**

=== "中文"

    有时，推断出的类型是所需类型的子类型（子类）。类型推断使用第一次赋值来推断名称的类型：

    ```python
    class Shape: ...
    class Circle(Shape): ...
    class Triangle(Shape): ...

    shape = Circle()    # mypy 推断 shape 的类型为 Circle
    shape = Triangle()  # 错误：赋值中的类型不兼容（表达式类型为 "Triangle"，变量类型为 "Circle"）
    ```

    在这种情况下，你可以为变量提供一个显式的类型注解：

    ```python
    shape: Shape = Circle()  # 变量 shape 可以是任何 Shape，不仅仅是 Circle
    shape = Triangle()       # 可以
    ```

=== "英文"

    Sometimes the inferred type is a subtype (subclass) of the desired type. The type inference uses the first assignment to infer the type of a name:

    ```python
    class Shape: ...
    class Circle(Shape): ...
    class Triangle(Shape): ...

    shape = Circle()    # mypy infers the type of shape to be Circle
    shape = Triangle()  # error: Incompatible types in assignment (expression has type "Triangle", variable has type "Circle")
    ```

    You can just give an explicit type for the variable in cases such the above example:

    ```python
    shape: Shape = Circle()  # The variable s can be any Shape, not just Circle
    shape = Triangle()       # OK
    ```

## 复杂类型测试

**Complex type tests**

=== "中文"

    Mypy 通常可以在使用 [isinstance]、[issubclass] 或 ``type(obj) is some_class`` 这样的类型检查时正确推断类型，甚至在使用 [用户定义的类型保护](../mypy/type_narrowing.md) 时也是如此。但是，对于其他类型的检查，你可能需要添加显式的类型转换：

    ```python
    from typing import Sequence, cast

    def find_first_str(a: Sequence[object]) -> str:
        index = next((i for i, s in enumerate(a) if isinstance(s, str)), -1)
        if index < 0:
            raise ValueError('No str found')

        found = a[index]  # 类型为 "object"，尽管我们知道它是 "str"
        return cast(str, found)  # 需要显式的转换以使 mypy 满意
    ```

    另外，你可以使用 ``assert`` 语句配合一些支持的类型推断技术：

    ```python
    def find_first_str(a: Sequence[object]) -> str:
        index = next((i for i, s in enumerate(a) if isinstance(s, str)), -1)
        if index < 0:
            raise ValueError('No str found')

        found = a[index]  # 类型为 "object"，尽管我们知道它是 "str"
        assert isinstance(found, str)  # 现在，"found" 将被推断为 "str"
        return found  # 不再需要显式的 "cast()"
    ```

    !!! note 

        请注意，上述示例中的 `object` 类型类似于 Java 中的 `Object`：它仅支持定义在 *所有* 对象上的操作，例如相等性和 [isinstance]。相比之下，类型 ``Any`` 支持所有操作，即使这些操作可能在运行时失败。如果 ``o`` 的类型是 ``Any``，上面的类型转换将是不必要的。

    !!! note 

        你可以在 [这里](../mypy/type_narrowing.md) 阅读更多关于类型缩小技术的信息。

    Mypy 中的类型推断旨在在常见情况下表现良好，具有可预测性，并让类型检查器提供有用的错误信息。更强大的类型推断策略通常具有复杂且难以预测的失败模式，可能导致非常令人困惑的错误信息。权衡之下，作为程序员你有时需要为类型检查器提供一些帮助。

=== "英文"

    Mypy can usually infer the types correctly when using [isinstance], [issubclass], or ``type(obj) is some_class`` type tests, and even [user-defined type guards](../mypy/type_narrowing.md),  but for other kinds of checks you may need to add an explicit type cast:

    ```python

    from typing import Sequence, cast

    def find_first_str(a: Sequence[object]) -> str:
        index = next((i for i, s in enumerate(a) if isinstance(s, str)), -1)
        if index < 0:
            raise ValueError('No str found')

        found = a[index]  # Has type "object", despite the fact that we know it is "str"
        return cast(str, found)  # We need an explicit cast to make mypy happy
    ```

    Alternatively, you can use an ``assert`` statement together with some of the supported type inference techniques:

    ```python

    def find_first_str(a: Sequence[object]) -> str:
        index = next((i for i, s in enumerate(a) if isinstance(s, str)), -1)
        if index < 0:
            raise ValueError('No str found')

        found = a[index]  # Has type "object", despite the fact that we know it is "str"
        assert isinstance(found, str)  # Now, "found" will be narrowed to "str"
        return found  # No need for the explicit "cast()" anymore
    ```

    !!! note 

        Note that the :py:class:`object` type used in the above example is similar to ``Object`` in Java: it only supports operations defined for *all* objects, such as equality and [isinstance]. The type ``Any``, in contrast, supports all operations, even if they may fail at runtime. The cast above would have been unnecessary if the type of ``o`` was ``Any``.

    !!! note 

        You can read more about type narrowing techniques [here](../mypy/type_narrowing.md).

    Type inference in Mypy is designed to work well in common cases, to be predictable and to let the type checker give useful error messages. More powerful type inference strategies often have complex and difficult-to-predict failure modes and could result in very confusing error messages. The tradeoff is that you as a programmer sometimes have to give the type checker a little help.

## Python 版本和系统平台检查

**Python version and system platform checks**

=== "中文"

    Mypy 支持执行 Python 版本检查和平台检查（例如 Windows 与 Posix），忽略在目标 Python 版本或平台上不会运行的代码路径。这使得你可以更有效地对支持多个 Python 版本或多个操作系统的代码进行类型检查。

    更具体地说，mypy 能够理解在 ``if/elif/else`` 语句中使用的 [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) 和 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 检查。例如：

    ```python
    import sys

    # 区分不同版本的 Python：
    if sys.version_info >= (3, 8):
        # Python 3.8+ 特定的定义和导入
    else:
        # 其他定义和导入

    # 区分不同的操作系统：
    if sys.platform.startswith("linux"):
        # Linux 特定的代码
    elif sys.platform == "darwin":
        # Mac 特定的代码
    elif sys.platform == "win32":
        # Windows 特定的代码
    else:
        # 其他系统
    ```

    作为特例，你也可以在顶层（未缩进的）``assert`` 中使用这些检查；这会使 mypy 跳过文件的其余部分。例如：

    ```python
    import sys

    assert sys.platform != 'win32'

    # 这个文件的其余部分不适用于 Windows。
    ```

    一些其他表达式也表现出类似的行为；特别是 [TYPE_CHECKING](), 变量名为 ``MYPY`` 的变量，以及任何变量名被传递给 [--always-true](../mypy_conf/command_line.md#always-true) 或 [--always-false](../mypy_conf/command_line.md#always-false) 的情况。（然而，``True`` 和 ``False`` 不会被特别处理！）

    !!! note 

        Mypy 目前不支持更复杂的检查，也不为将 [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) 或 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 检查赋值给变量提供任何特殊含义。这可能会在未来版本的 mypy 中发生变化。

    默认情况下，mypy 将使用你当前的 Python 版本和当前的操作系统作为 [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) 和 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 的默认值。

    要针对不同的 Python 版本，使用 [--python-version X.Y](../mypy_conf/command_line.md#python-version) 标志。例如，要验证你的代码在使用 Python 3.8 时是否进行类型检查，可以从命令行传入 [--python-version 3.8](../mypy_conf/command_line.md#python-version)。请注意，你不需要安装 Python 3.8 来执行此检查。

    要针对不同的操作系统，使用 [--platform PLATFORM](../mypy_conf/command_line.md#platform) 标志。例如，要验证你的代码在 Windows 上是否进行类型检查，可以传入 [--platform win32](../mypy_conf/command_line.md#platform)。有关有效平台参数的示例，请参阅 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 文档。

=== "英文"

    Mypy supports the ability to perform Python version checks and platform checks (e.g. Windows vs Posix), ignoring code paths that won't be run on the targeted Python version or platform. This allows you to more effectively typecheck code that supports multiple versions of Python or multiple operating systems.

    More specifically, mypy will understand the use of [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) and [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) checks within ``if/elif/else`` statements. For example:

    ```python
    import sys

    # Distinguishing between different versions of Python:
    if sys.version_info >= (3, 8):
        # Python 3.8+ specific definitions and imports
    else:
        # Other definitions and imports

    # Distinguishing between different operating systems:
    if sys.platform.startswith("linux"):
        # Linux-specific code
    elif sys.platform == "darwin":
        # Mac-specific code
    elif sys.platform == "win32":
        # Windows-specific code
    else:
        # Other systems
    ```

    As a special case, you can also use one of these checks in a top-level (unindented) ``assert``; this makes mypy skip the rest of the file. Example:

    ```python
    import sys

    assert sys.platform != 'win32'

    # The rest of this file doesn't apply to Windows.
    ```

    Some other expressions exhibit similar behavior; in particular, [TYPE_CHECKING](), variables named ``MYPY``, and any variable whose name is passed to [--always-true](../mypy_conf/command_line.md#always-true) or [--always-false](../mypy_conf/command_line.md#always-false). (However, ``True`` and ``False`` are not treated specially!)

    !!! note 

        Mypy currently does not support more complex checks, and does not assign any special meaning when assigning a [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) or [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) check to a variable. This may change in future versions of mypy.

    By default, mypy will use your current version of Python and your current operating system as default values for [sys.version_info](https://docs.python.org/3/library/sys.html#sys.version_info) and [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform).

    To target a different Python version, use the [--python-version X.Y](../mypy_conf/command_line.md#python-version) flag. For example, to verify your code typechecks if were run using Python 3.8, pass in [--python-version 3.8](../mypy_conf/command_line.md#python-version) from the command line. Note that you do not need to have Python 3.8 installed to perform this check.

    To target a different operating system, use the [--platform PLATFORM](../mypy_conf/command_line.md#platform) flag. For example, to verify your code typechecks if it were run in Windows, pass in [--platform win32](../mypy_conf/command_line.md#platform). See the documentation for [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) for examples of valid platform parameters.

## 显示表达式的类型

**Displaying the type of an expression**

=== "中文"

    你可以使用 ``reveal_type(expr)`` 来让 mypy 显示表达式的推断静态类型。这在你不太理解 mypy 如何处理特定代码片段时非常有用。例如：

    ```python
    reveal_type((1, 'hello'))  # 显示的类型是 "tuple[builtins.int, builtins.str]"
    ```

    你还可以在文件中的任何一行使用 ``reveal_locals()`` 来查看所有局部变量的类型。例如：

    ```python
    a = 1
    b = 'one'
    reveal_locals()
    # 显示的局部变量类型是：
    #     a: builtins.int
    #     b: builtins.str
    ```

    !!! note 

    ``reveal_type`` 和 ``reveal_locals`` 仅被 mypy 理解，在 Python 中不存在。如果你尝试运行你的程序，你需要在运行代码之前移除任何 ``reveal_type`` 和 ``reveal_locals`` 调用。这两个函数总是可用的，你不需要导入它们。

=== "英文"

    You can use ``reveal_type(expr)`` to ask mypy to display the inferred static type of an expression. This can be useful when you don't quite understand how mypy handles a particular piece of code. Example:

    ```python
    reveal_type((1, 'hello'))  # Revealed type is "tuple[builtins.int, builtins.str]"
    ```

    You can also use ``reveal_locals()`` at any line in a file to see the types of all local variables at once. Example:

    ```python
    a = 1
    b = 'one'
    reveal_locals()
    # Revealed local types are:
    #     a: builtins.int
    #     b: builtins.str
    ```

    !!! note 

        ``reveal_type`` and ``reveal_locals`` are only understood by mypy and don't exist in Python. If you try to run your program, you'll have to remove any ``reveal_type`` and ``reveal_locals`` calls before you can run your code. Both are always available and you don't need to import them.

## 静默处理代码审查工具

**Silencing linters**

=== "中文"

    在某些情况下，代码检查工具会对未使用的导入或代码发出警告。在这些情况下，你可以通过在类型注释之后或与导入语句同一行添加注释来抑制这些警告：

    ```python
    # 抑制对未使用导入的警告
    from typing import List  # noqa
    a = None  # type: List[int]
    ```

    要在类型注释同一行上抑制检查工具的警告，将检查工具的注释放在类型注释之后：

    ```python
    a = some_complex_thing()  # type: ignore  # noqa
    ```

=== "英文"

    In some cases, linters will complain about unused imports or code. In these cases, you can silence them with a comment after type comments, or on the same line as the import:

    ```python
    # to silence complaints about unused imports
    from typing import List  # noqa
    a = None  # type: List[int]
    ```


    To silence the linter on the same line as a type comment put the linter comment *after* the type comment:

    ```python
    a = some_complex_thing()  # type: ignore  # noqa
    ```

## 拒绝可变协议成员的协变子类型

**Covariant subtyping of mutable protocol members is rejected**

=== "中文"

    Mypy 会拒绝这种情况，因为它可能不安全。考虑以下示例：

    ```python
    from typing import Protocol

    class P(Protocol):
        x: float

    def fun(arg: P) -> None:
        arg.x = 3.14

    class C:
        x = 42
    c = C()
    fun(c)  # 这不是安全的
    c.x << 5  # 因为这会失败！
    ```

    为了绕过这个问题，考虑“变更”是否实际上是协议的一部分。如果不是，则可以在协议定义中使用 [@property](https://docs.python.org/3/library/functions.html#property)：

    ```python
    from typing import Protocol

    class P(Protocol):
        @property
        def x(self) -> float:
            pass

    def fun(arg: P) -> None:
        ...

    class C:
        x = 42
    fun(C())  # OK
    ```

=== "英文"

    Mypy rejects this because this is potentially unsafe. Consider this example:

    ```python
    from typing import Protocol

    class P(Protocol):
        x: float

    def fun(arg: P) -> None:
        arg.x = 3.14

    class C:
        x = 42
    c = C()
    fun(c)  # This is not safe
    c.x << 5  # Since this will fail!
    ```

    To work around this problem consider whether "mutating" is actually part of a protocol. If not, then one can use a [@property](https://docs.python.org/3/library/functions.html#property) in the protocol definition:

    ```python
    from typing import Protocol

    class P(Protocol):
        @property
        def x(self) -> float:
            pass

    def fun(arg: P) -> None:
        ...

    class C:
        x = 42
    fun(C())  # OK
    ```

## 处理名称冲突

**Dealing with conflicting names**

=== "中文"

    假设你有一个类，其中一个方法的名称与导入的（或内置的）类型相同，并且你想在另一个方法签名中使用这个类型。例如：

    ```python
    class Message:
        def bytes(self):
            ...
        def register(self, path: bytes):  # 错误：无效的类型 "mod.Message.bytes"
            ...
    ```

    第三行会引发错误，因为 mypy 将参数类型 `bytes` 视为与该名称的方法的引用。除了重命名方法外，另一个解决方法是使用别名：

    ```python
    bytes_ = bytes
    class Message:
        def bytes(self):
            ...
        def register(self, path: bytes_):
            ...
    ```

=== "英文"

    Suppose you have a class with a method whose name is the same as an imported (or built-in) type, and you want to use the type in another method signature.  E.g.:

    ```python
    class Message:
        def bytes(self):
            ...
        def register(self, path: bytes):  # error: Invalid type "mod.Message.bytes"
            ...
    ```

    The third line elicits an error because mypy sees the argument type ``bytes`` as a reference to the method by that name.  Other than renaming the method, a workaround is to use an alias:

    ```python
    bytes_ = bytes
    class Message:
        def bytes(self):
            ...
        def register(self, path: bytes_):
            ...
    ```

## 使用开发中的 mypy 版本

**Using a development mypy build**

=== "中文"

    你可以从源代码安装最新的开发版本的 mypy。首先，克隆 [mypy GitHub 仓库](https://github.com/python/mypy)，然后在本地运行 `pip install`：

    ```shell
    git clone https://github.com/python/mypy.git
    cd mypy
    python3 -m pip install --upgrade .
    ```

    要安装一个已经通过 mypyc 编译的开发版本的 mypy，请参见 [mypyc wheels 仓库](https://github.com/mypyc/mypy_mypyc-wheels) 的说明。

=== "英文"

    You can install the latest development version of mypy from source. Clone the [mypy repository on GitHub](https://github.com/python/mypy), and then run ``pip install`` locally:

    ```shell
    git clone https://github.com/python/mypy.git
    cd mypy
    python3 -m pip install --upgrade .
    ```

    To install a development version of mypy that is mypyc-compiled, see the instructions at the [mypyc wheels repo](https://github.com/mypyc/mypy_mypyc-wheels).

## 变量 vs 类型别名

**Variables vs type aliases**

=== "中文"

    Mypy 既支持 *类型别名*，也支持类似 ``type[...]`` 的类型变量。这两者有微妙的区别，理解它们的不同非常重要，以避免陷入误区。

    1. 类型变量 ``type[...]`` 是通过带有显式类型注解的赋值定义的：

        ```python
        class A: ...
        tp: type[A] = A
        ```

    2. 可以使用没有显式类型注解的赋值来定义类型别名，这通常在模块的顶层定义：

        ```python
        class A: ...
        Alias = A
        ```

        也可以使用 ``TypeAlias`` (:pep:`613`) 来定义一个 *显式类型别名*：

        ```python
        from typing import TypeAlias  # 在 Python 3.9 及更早版本中使用 "from typing_extensions"
        
        class A: ...
        Alias: TypeAlias = A
        ```

        在类体内或函数内部定义类型别名时，应该总是使用 ``TypeAlias``。

    主要区别在于，类型别名的目标在静态类型检查时是精确已知的，这意味着它们可以在类型注解和其他 *类型上下文* 中使用。类型别名不能在条件语句中定义（除非使用 [支持的 Python 版本和系统平台检查](#python-版本和系统平台检查)）：

    ```python
    class A: ...
    class B: ...

    if random() > 0.5:
        Alias = A
    else:
        # 错误：没有显式的 "Type[...]" 注解，无法将多个类型赋值给名称 "Alias"
        Alias = B

    tp: type[object]  # "tp" 是一个类型对象值的变量
    if random() > 0.5:
        tp = A
    else:
        tp = B  # 这是 OK

    def fun1(x: Alias) -> None: ...  # OK
    def fun2(x: tp) -> None: ...  # 错误：“tp” 不能作为类型使用
    ```

=== "英文"

    Mypy has both *type aliases* and variables with types like ``type[...]``. These are subtly different, and it's important to understand how they differ to avoid pitfalls.

    1. A variable with type ``type[...]`` is defined using an assignment with an explicit type annotation:

        ```python
        class A: ...
        tp: type[A] = A
        ```

    2. You can define a type alias using an assignment without an explicit type annotation at the top level of a module:

        ```python
        class A: ...
        Alias = A
        ```

        You can also use ``TypeAlias`` (:pep:`613`) to define an *explicit type alias*:

        ```python
        
        from typing import TypeAlias  # "from typing_extensions" in Python 3.9 and earlier
        
        class A: ...
        Alias: TypeAlias = A
        ```

        You should always use ``TypeAlias`` to define a type alias in a class body or inside a function.

    The main difference is that the target of an alias is precisely known statically, and this means that they can be used in type annotations and other *type contexts*. Type aliases can't be defined conditionally (unless using [supported Python version and platform checks](#python-版本和系统平台检查)):

    ```python

    class A: ...
    class B: ...

    if random() > 0.5:
        Alias = A
    else:
        # error: Cannot assign multiple types to name "Alias" without an
        # explicit "Type[...]" annotation
        Alias = B

    tp: type[object]  # "tp" is a variable with a type object value
    if random() > 0.5:
        tp = A
    else:
        tp = B  # This is OK

    def fun1(x: Alias) -> None: ...  # OK
    def fun2(x: tp) -> None: ...  # Error: "tp" is not valid as a type
    ```

## 不兼容的重写

**Incompatible overrides**

=== "中文"

    重写方法时，如果使用了更具体的参数类型是不安全的，因为这违反了 [里氏替换原则](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle)。对于返回类型，如果重写的方法具有更通用的返回类型，也是危险的。

    在方法重写中，其他不兼容的签名更改，如添加额外的必需参数或移除可选参数，也会产生错误。子类中的方法签名应该接受所有对基类方法的有效调用。Mypy 将子类视为基类的子类型。子类的实例在任何基类实例有效的地方都是有效的。

    下面的示例展示了安全和不安全的重写情况：

    ```python
    from typing import Sequence, List, Iterable

    class A:
        def test(self, t: Sequence[int]) -> Sequence[str]:
            ...

    class GeneralizedArgument(A):
        # 更通用的参数类型是可以的
        def test(self, t: Iterable[int]) -> Sequence[str]:  # OK
            ...

    class NarrowerArgument(A):
        # 更具体的参数类型是不被接受的
        def test(self, t: List[int]) -> Sequence[str]:  # 错误
            ...

    class NarrowerReturn(A):
        # 更具体的返回类型是可以的
        def test(self, t: Sequence[int]) -> List[str]:  # OK
            ...

    class GeneralizedReturn(A):
        # 更通用的返回类型是错误的
        def test(self, t: Sequence[int]) -> Iterable[str]:  # 错误
            ...
    ```

    你可以使用 ``# type: ignore[override]`` 来抑制错误。如果你决定类型安全不是必要的，可以将其添加到生成错误的行上：

    ```python
    class NarrowerArgument(A):
        def test(self, t: List[int]) -> Sequence[str]:  # type: ignore[override]
            ...
    ```

=== "英文"

    It's unsafe to override a method with a more specific argument type, as it violates the [Liskov substitution principle](https://stackoverflow.com/questions/56860/what-is-an-example-of-the-liskov-substitution-principle). For return types, it's unsafe to override a method with a more general return type.

    Other incompatible signature changes in method overrides, such as adding an extra required parameter, or removing an optional parameter, will also generate errors. The signature of a method in a subclass should accept all valid calls to the base class method. Mypy treats a subclass as a subtype of the base class. An instance of a subclass is valid everywhere where an instance of the base class is valid.

    This example demonstrates both safe and unsafe overrides:

    ```python

    from typing import Sequence, List, Iterable

    class A:
        def test(self, t: Sequence[int]) -> Sequence[str]:
            ...

    class GeneralizedArgument(A):
        # A more general argument type is okay
        def test(self, t: Iterable[int]) -> Sequence[str]:  # OK
            ...

    class NarrowerArgument(A):
        # A more specific argument type isn't accepted
        def test(self, t: List[int]) -> Sequence[str]:  # Error
            ...

    class NarrowerReturn(A):
        # A more specific return type is fine
        def test(self, t: Sequence[int]) -> List[str]:  # OK
            ...

    class GeneralizedReturn(A):
        # A more general return type is an error
        def test(self, t: Sequence[int]) -> Iterable[str]:  # Error
            ...
    ```

    You can use ``# type: ignore[override]`` to silence the error. Add it to the line that generates the error, if you decide that type safety is not necessary:

    ```python

    class NarrowerArgument(A):
        def test(self, t: List[int]) -> Sequence[str]:  # type: ignore[override]
            ...
    ```

## 无法到达的代码

**Unreachable code**

=== "中文"

    Mypy 可能会将某些代码视为 *不可达*，即使其原因可能并不立即显而易见。重要的是要注意，mypy *不会* 对这些不可达的代码进行类型检查。考虑以下示例：

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        return
        x: int = 'abc'  # 不可达 -- 不会报错
    ```

    可以很容易地看出，任何在 `return` 语句之后的语句都是不可达的，因此 mypy 不会对下面的错误类型代码发出警告。对于一个更微妙的例子，考虑以下代码：

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        assert foo.bar is None
        x: int = 'abc'  # 不可达 -- 不会报错
    ```

    同样，mypy 不会报告任何错误。`foo.bar` 的类型是 `str`，mypy 认为它永远不可能是 `None`。因此，`assert` 语句总是会失败，下面的语句永远不会被执行。（注意，在 Python 中，`None` 不是一个空引用，而是类型为 `None` 的对象。）

    在这个示例中，mypy 会继续检查最后一行并报告错误，因为 mypy 认为条件可能为 `True` 或 `False`：

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        if not foo.bar:
            return
        x: int = 'abc'  # 可达 -- 错误
    ```

    如果你使用 `--warn-unreachable` 标志，mypy 会对每个不可达的代码块生成错误。

=== "英文"

    Mypy may consider some code as *unreachable*, even if it might not be immediately obvious why.  It's important to note that mypy will *not* type check such code. Consider this example:

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        return
        x: int = 'abc'  # Unreachable -- no error
    ```

    It's easy to see that any statement after ``return`` is unreachable, and hence mypy will not complain about the mis-typed code below it. For a more subtle example, consider this code:

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        assert foo.bar is None
        x: int = 'abc'  # Unreachable -- no error
    ```

    Again, mypy will not report any errors. The type of ``foo.bar`` is ``str``, and mypy reasons that it can never be ``None``.  Hence the ``assert`` statement will always fail and the statement below will never be executed.  (Note that in Python, ``None`` is not an empty reference but an object of type ``None``.)

    In this example mypy will go on to check the last line and report an error, since mypy thinks that the condition could be either True or False:

    ```python
    class Foo:
        bar: str = ''

    def bar() -> None:
        foo: Foo = Foo()
        if not foo.bar:
            return
        x: int = 'abc'  # Reachable -- error
    ```

    If you use the [--warn-unreachable](../mypy_conf/command_line.md#warn-unreachable) flag, mypy will generate an error about each unreachable code block.

## 缩小范围和内部函数

**Narrowing and inner functions**

=== "中文"

    由于 Python 中的闭包是晚绑定的（[late-binding](https://docs.python-guide.org/writing/gotchas/#late-binding-closures)），mypy 不会在内部函数中缩小捕获变量的类型。以下是一个示例，可以更好地理解这一点：

    ```python
    def foo(x: Optional[int]) -> Callable[[], int]:
        if x is None:
            x = 5
        print(x + 1)  # mypy 正确推断此处 x 必须是 int
        def inner() -> int:
            return x + 1  # 但（正确地）对这一行提出警告

        x = None  # 因为 x 以后可能会被赋值为 None
        return inner

    inner = foo(5)
    inner()  # 调用时会引发错误
    ```

    为了使这段代码能够通过类型检查，你可以在 `x` 的类型被缩小后赋值给 `y`，然后在内部函数中使用 `y`，或者在内部函数中添加一个断言。

=== "英文"

    Because closures in Python are late-binding (https://docs.python-guide.org/writing/gotchas/#late-binding-closures), mypy will not narrow the type of a captured variable in an inner function. This is best understood via an example:

    ```python
    def foo(x: Optional[int]) -> Callable[[], int]:
        if x is None:
            x = 5
        print(x + 1)  # mypy correctly deduces x must be an int here
        def inner() -> int:
            return x + 1  # but (correctly) complains about this line

        x = None  # because x could later be assigned None
        return inner

    inner = foo(5)
    inner()  # this will raise an error when called
    ```

    To get this code to type check, you could assign ``y = x`` after ``x`` has been narrowed, and use ``y`` in the inner function, or add an assert in the inner function.

[isinstance]: https://docs.python.org/3/library/functions.html#isinstance
[issubclass]: https://docs.python.org/3/library/functions.html#issubclass