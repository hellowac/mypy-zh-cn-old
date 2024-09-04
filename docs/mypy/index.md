# 开始

Getting started

转自: <https://mypy.readthedocs.io/en/stable/getting_started.html>

=== "中文"

    本章介绍了mypy的一些核心概念，包括函数注解、[typing](https://docs.python.org/3/library/typing.html#module-typing) 模块、存根文件等。
    
    如果你想要一个快速的介绍，请查看[mypy速查表](./mypy/cheat_sheet_py3.md)。
    
    如果你不熟悉静态和动态类型检查的概念，请确保仔细阅读本章，因为否则文档的其余部分可能不太容易理解。

=== "英文"

    This chapter introduces some core concepts of mypy, including function annotations, the [typing](https://docs.python.org/3/library/typing.html#module-typing) module, stub files, and more.
    
    If you’re looking for a quick intro, see the [mypy cheatsheet](./mypy/cheat_sheet_py3.md).
    
    If you’re unfamiliar with the concepts of static and dynamic type checking, be sure to read this chapter carefully, as the rest of the documentation may not make much sense otherwise.

## 安装和运行mypy

**Installing and running mypy**

=== "中文"

    Mypy 需要 Python 3.8 或更高版本才能运行。你可以使用 pip 安装 mypy：
    
    ```
    $ python3 -m pip install mypy
    ```
    
    安装 mypy 后，使用 `mypy` 工具运行它：
    
    ```
    $ mypy program.py
    ```
    
    这个命令会让 mypy 对你的 `program.py` 文件进行类型检查，并打印出它发现的任何错误。Mypy 会对你的代码进行*静态*类型检查：这意味着它会在不运行你的代码的情况下检查错误，就像一个 linter。
    
    这也意味着，如果你愿意，你总是可以忽略 mypy 报告的错误。即使 mypy 报告错误，你也可以使用 Python 解释器来运行你的代码。
    
    然而，如果你尝试直接在现有的 Python 代码上运行 mypy，它很可能报告很少或没有错误。这是一个特性！它使得逐步采用 mypy 变得容易。
    
    为了从 mypy 获得有用的诊断信息，你必须在代码中添加类型注解。详见下面的部分。
    

=== "英文"

    Mypy requires Python 3.8 or later to run. You can install mypy using pip:
    
    `$ python3 -m pip install mypy`
    
    Once mypy is installed, run it by using the `mypy` tool:
    
    `$ mypy program.py`
    
    This command makes mypy type check your `program.py` file and print out any errors it finds. Mypy will type check your code *statically*: this means that it will check for errors without ever running your code, just like a linter.
    
    This also means that you are always free to ignore the errors mypy reports, if you so wish. You can always use the Python interpreter to run your code, even if mypy reports errors.
    
    However, if you try directly running mypy on your existing Python code, it will most likely report little to no errors. This is a feature! It makes it easy to adopt mypy incrementally.
    
    In order to get useful diagnostics from mypy, you must add type annotations to your code. See the section below for details.

## 动态 vs 静态 类型

**Dynamic vs static typing**

=== "中文"
    
    没有类型注释的函数被 mypy 认为是*动态类型*的：
    
    ```python
    def greeting(name):
        return 'Hello ' + name
    ```
    
    默认情况下，mypy **不会**对动态类型函数进行类型检查。这意味着，除了一些例外情况，mypy 不会报告常规未注释 Python 代码的任何错误。
    
    即使你误用了函数，情况也是如此！
    
    ```python
    def greeting(name):
        return 'Hello ' + name
    
    # 当程序运行时，这些调用将失败，但 mypy 不报告错误
    # 因为 "greeting" 没有类型注释。
    greeting(123)
    greeting(b"Alice")
    ```
    
    我们可以通过添加类型注释（也称为类型提示）来让 mypy 检测这类错误。例如，你可以告诉 mypy `greeting` 函数既接受又返回一个字符串，如下所示：
    
    ```python
    # "name: str" 注释表明 "name" 参数应该是一个字符串
    # "-> str" 注释表明 "greeting" 将返回一个字符串
    def greeting(name: str) -> str:
        return 'Hello ' + name
    ```
    
    这个函数现在是静态类型的：mypy 将使用提供的类型提示来检测 greeting 函数的错误使用以及 greeting 函数内部变量的错误使用。例如：
    
    ```python
    def greeting(name: str) -> str:
        return 'Hello ' + name
    
    greeting(3)         # "greeting" 的参数 1 有不兼容的类型 "int"；预期 "str"
    greeting(b'Alice')  # "greeting" 的参数 1 有不兼容的类型 "bytes"；预期 "str"
    greeting("World!")  # 无错误
    
    def bad_greeting(name: str) -> str:
        return 'Hello ' * name  # "str" 和 "str" 对于 * 运算符不支持的操作数类型
    ```
    
    能够选择函数是动态类型还是静态类型非常有用。例如，如果你正在将现有的 Python 代码库迁移到使用静态类型，通常逐个添加类型注释比一次性全部添加要容易。同样，当你在原型设计一个新特性时，最初使用动态类型实现代码，代码更稳定后再添加类型注释可能更方便。
    
    一旦你完成了迁移或原型设计，你可以使用 [--disallow-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-disallow-untyped-defs) 标志让 mypy 在你不小心添加了一个动态函数时警告你。你还可以使用 [--check-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-check-untyped-defs) 标志让 mypy 对动态类型函数进行一些有限的检查。有关配置 mypy 的更多信息，请参见 [The mypy command line](https://mypy.readthedocs.io/en/stable/command_line.html#command-line)。
    


=== "英文"
    
    A function without type annotations is considered to be *dynamically typed* by mypy:
    
    ```python
    def greeting(name):
        return 'Hello ' + name
    ```
    
    By default, mypy will **not** type check dynamically typed functions. This means that with a few exceptions, mypy will not report any errors with regular unannotated Python.
    
    This is the case even if you misuse the function!
    
    ```python
    def greeting(name):
        return 'Hello ' + name
    
    # These calls will fail when the program runs, but mypy does not report an error
    # because "greeting" does not have type annotations.
    greeting(123)
    greeting(b"Alice")
    ```
    
    We can get mypy to detect these kinds of bugs by adding type annotations (also known as type hints). For example, you can tell mypy that `greeting` both accepts and returns a string like so:
    
    ```python
    # The "name: str" annotation says that the "name" argument should be a string
    # The "-> str" annotation says that "greeting" will return a string
    def greeting(name: str) -> str:
        return 'Hello ' + name
    ```
    
    This function is now statically typed: mypy will use the provided type hints to detect incorrect use of the greeting function and incorrect use of variables within the greeting function. For example:
    
    ```python
    def greeting(name: str) -> str:
        return 'Hello ' + name
    
    greeting(3)         # Argument 1 to "greeting" has incompatible type "int"; expected "str"
    greeting(b'Alice')  # Argument 1 to "greeting" has incompatible type "bytes"; expected "str"
    greeting("World!")  # No error
    
    def bad_greeting(name: str) -> str:
        return 'Hello ' * name  # Unsupported operand types for * ("str" and "str")
    ```
    
    Being able to pick whether you want a function to be dynamically or statically typed can be very helpful. For example, if you are migrating an existing Python codebase to use static types, it’s usually easier to migrate by incrementally adding type hints to your code rather than adding them all at once. Similarly, when you are prototyping a new feature, it may be convenient to initially implement the code using dynamic typing and only add type hints later once the code is more stable.
    
    Once you are finished migrating or prototyping your code, you can make mypy warn you if you add a dynamic function by mistake by using the [--disallow-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-disallow-untyped-defs) flag. You can also get mypy to provide some limited checking of dynamically typed functions by using the [--check-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-check-untyped-defs) flag. See [The mypy command line](https://mypy.readthedocs.io/en/stable/command_line.html#command-line) for more information on configuring mypy.

## 严格模式以及配置

**Strict mode and configuration**

=== "中文"

    Mypy 提供了一个严格模式，该模式启用了许多额外的检查，比如 [--disallow-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-disallow-untyped-defs)。
    
    如果你使用 [--strict](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-strict) 标志运行 mypy，你基本上不会再在运行时遇到任何类型相关的错误，除非你以某种方式明确规避了 mypy。
    
    然而，如果你正试图为一个大型的、现有的代码库添加静态类型，这个标志可能会过于激进。有关如何处理这种情况的建议，请参见[在现有代码库中使用 mypy](https://mypy.readthedocs.io/en/stable/existing_code.html#existing-code)。
    
    Mypy 是非常可配置的，你可以从使用 `--strict` 开始，然后关闭个别检查。例如，如果你使用许多没有类型的第三方库，`--ignore-missing-imports` 可能会很有用。关于如何逐步采用 `--strict`，请参见[引入更严格的选项](https://mypy.readthedocs.io/en/stable/existing_code.html#getting-to-strict)。
    
    有关配置选项的完整参考，请参见 [mypy 命令行](https://mypy.readthedocs.io/en/stable/command_line.html#command-line) 和 [mypy 配置文件](https://mypy.readthedocs.io/en/stable/config_file.html#config-file)。
    

=== "英文"

    Mypy has a strict mode that enables a number of additional checks, like [--disallow-untyped-defs](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-disallow-untyped-defs).
    
    If you run mypy with the [--strict](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-strict) flag, you will basically never get a type related error at runtime without a corresponding mypy error, unless you explicitly circumvent mypy somehow.
    
    However, this flag will probably be too aggressive if you are trying to add static types to a large, existing codebase. See [Using mypy with an existing codebase](https://mypy.readthedocs.io/en/stable/existing_code.html#existing-code) for suggestions on how to handle that case.
    
    Mypy is very configurable, so you can start with using `--strict` and toggle off individual checks. For instance, if you use many third party libraries that do not have types, [--ignore-missing-imports](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-ignore-missing-imports) may be useful. See [Introduce stricter options](https://mypy.readthedocs.io/en/stable/existing_code.html#getting-to-strict) for how to build up to `--strict`.
    
    See [The mypy command line](https://mypy.readthedocs.io/en/stable/command_line.html#command-line) and [The mypy configuration file](https://mypy.readthedocs.io/en/stable/config_file.html#config-file) for a complete reference on configuration options.

## 更多复杂类型

**More complex types**

=== "中文"

    到目前为止，我们已经添加了使用基本具体类型（如 `str` 和 `float`）的类型提示。如果我们想表达更复杂的类型，例如“一个字符串列表”或“一个整数的可迭代对象”怎么办？
    
    例如，要表示某个函数可以接受一个字符串列表，可以使用 `list[str]` 类型（适用于 Python 3.9 及更高版本）：
    
    ```python
    def greet_all(names: list[str]) -> None:
        for name in names:
            print('Hello ' + name)
    
    names = ["Alice", "Bob", "Charlie"]
    ages = [10, 20, 30]
    
    greet_all(names)   # 可以！
    greet_all(ages)    # 由于类型不兼容，出现错误
    ```
    
    [list](https://docs.python.org/3/library/stdtypes.html#list) 类型是一个称为泛型类型的示例：它可以接受一个或多个类型参数。在这种情况下，我们通过写 `list[str]` 来对 [list](https://docs.python.org/3/library/stdtypes.html#list) 进行参数化。这使得 mypy 知道 `greet_all` 仅接受包含字符串的列表，而不是包含整数或其他类型的列表。
    
    在上面的例子中，类型签名可能有点过于严格。毕竟，这个函数并不一定非得接受 *具体的* 列表——如果你传递一个元组、集合或任何其他自定义的可迭代对象，它也能正常运行。
    
    你可以使用 [collections.abc.Iterable](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable) 来表达这一想法：
    
    ```python
    from collections.abc import Iterable  # 或者 "from typing import Iterable"
    
    def greet_all(names: Iterable[str]) -> None:
        for name in names:
            print('Hello ' + name)
    ```
    
    这种行为实际上是 PEP 484 类型系统的一个基本方面：当我们用类型 `T` 注解某个变量时，我们实际上是在告诉 mypy，这个变量可以被赋值为 `T` 的实例，或者 `T` 的 *子类型* 的实例。也就是说，`list[str]` 是 `Iterable[str]` 的子类型。
    
    这也适用于继承，因此如果你有一个 `Child` 类继承自 `Parent`，那么 `Child` 类型的值可以赋给 `Parent` 类型的变量。例如，一个 `RuntimeError` 实例可以传递给注解为接受 `Exception` 的函数。
    
    另一个例子，假设你想编写一个函数，该函数可以接受 *整数或字符串*，但不接受其他类型。你可以使用 [Union](https://docs.python.org/3/library/typing.html#typing.Union) 类型来表达这种情况。例如，`int` 是 `Union[int, str]` 的子类型：
    
    ```python
    from typing import Union
    
    def normalize_id(user_id: Union[int, str]) -> str:
        if isinstance(user_id, int):
            return f'user-{100_000 + user_id}'
        else:
            return user_id
    ```
    
    [typing](https://docs.python.org/3/library/typing.html#module-typing) 模块包含许多其他有用的类型。
    
    要快速了解，可以查看 [mypy cheatsheet](./mypy/cheat_sheet_py3.md)。
    
    要获得详细的概述（包括如何创建自己的泛型类型或类型别名），可以查看 **类型系统参考**。
    
    !!! note "注意"
    
        添加类型时，约定是使用 `from typing import Union` 形式导入类型（而不是仅仅 `import typing` 或 `import typing as t` 或 `from typing import *`）。
        
        为了简洁，我们在代码示例中经常省略了从 [typing](https://docs.python.org/3/library/typing.html#module-typing) 或 [collections.abc](https://docs.python.org/3/library/collections.abc.html#module-collections.abc) 进行的导入，但如果你在没有首先导入的情况下使用像 [Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable) 这样的类型，mypy 会报错。
    
    !!! note "注意"
        
        在一些例子中，我们使用了大写的类型名称，如 `List`，而有时我们使用普通的 `list`。它们是等价的，但如果你使用的是 Python 3.8 或更早的版本，则需要使用前者。

=== "英文"

    So far, we’ve added type hints that use only basic concrete types like `str` and `float`. What if we want to express more complex types, such as “a list of strings” or “an iterable of ints”?
    
    For example, to indicate that some function can accept a list of strings, use the `list[str]` type (Python 3.9 and later):
    
    ```python
    def greet_all(names: list[str]) -> None:
        for name in names:
            print('Hello ' + name)
    
    names = ["Alice", "Bob", "Charlie"]
    ages = [10, 20, 30]
    
    greet_all(names)   # Ok!
    greet_all(ages)    # Error due to incompatible types
    ```
    
    The [list](https://docs.python.org/3/library/stdtypes.html#list) type is an example of something called a generic type: it can accept one or more type parameters. In this case, we parameterized [list](https://docs.python.org/3/library/stdtypes.html#list) by writing `list[str]`. This lets mypy know that `greet_all` accepts specifically lists containing strings, and not lists containing ints or any other type.
    
    In the above examples, the type signature is perhaps a little too rigid. After all, there’s no reason why this function must accept *specifically* a list – it would run just fine if you were to pass in a tuple, a set, or any other custom iterable.
    
    You can express this idea using [collections.abc.Iterable](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable):
    
    ```python
    from collections.abc import Iterable  # or "from typing import Iterable"
    
    def greet_all(names: Iterable[str]) -> None:
        for name in names:
            print('Hello ' + name)
    ```
    
    This behavior is actually a fundamental aspect of the PEP 484 type system: when we annotate some variable with a type `T`, we are actually telling mypy that variable can be assigned an instance of `T`, or an instance of a *subtype* of `T`. That is, `list[str]` is a subtype of `Iterable[str]`.
    
    This also applies to inheritance, so if you have a class `Child` that inherits from `Parent`, then a value of type `Child` can be assigned to a variable of type `Parent`. For example, a `RuntimeError` instance can be passed to a function that is annotated as taking an `Exception`.
    
    As another example, suppose you want to write a function that can accept *either* ints or strings, but no other types. You can express this using the [Union](https://docs.python.org/3/library/typing.html#typing.Union) type. For example, `int` is a subtype of `Union[int, str]`:
    
    ```python
    from typing import Union
    
    def normalize_id(user_id: Union[int, str]) -> str:
        if isinstance(user_id, int):
            return f'user-{100_000 + user_id}'
        else:
            return user_id
    ```
    
    The [typing](https://docs.python.org/3/library/typing.html#module-typing) module contains many other useful types.
    
    For a quick overview, look through the [mypy cheatsheet](./mypy/cheat_sheet_py3.md).
    
    For a detailed overview (including information on how to make your own generic types or your own type aliases), look through the **type system reference**.
    
    !!! note "Note"
    
        When adding types, the convention is to import types using the form `from typing import Union` (as opposed to doing just `import typing` or `import typing as t` or `from typing import *`).
        
        For brevity, we often omit imports from [typing](https://docs.python.org/3/library/typing.html#module-typing) or [collections.abc](https://docs.python.org/3/library/collections.abc.html#module-collections.abc) in code examples, but mypy will give an error if you use types such as [Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable) without first importing them.
    
    !!! note "Note"
        
        In some examples we use capitalized variants of types, such as `List`, and sometimes we use plain `list`. They are equivalent, but the prior variant is needed if you are using Python 3.8 or earlier.

## 局部类型推断

**Local type inference**

=== "中文"

    一旦你为函数添加了类型提示（即使它变成了静态类型），mypy 将自动对该函数的主体进行类型检查。在此过程中，mypy 将尝试推断尽可能多的细节。
    
    我们在上面的 `normalize_id` 函数中看到了一个例子——mypy 能理解基本的 [isinstance](https://docs.python.org/3/library/functions.html#isinstance) 检查，因此可以推断出 `user_id` 变量在 if 分支中是 `int` 类型，在 else 分支中是 `str` 类型。
    
    再举一个例子，考虑以下函数。Mypy 可以毫无问题地对该函数进行类型检查：它将利用可用的上下文推断出 `output` 必须是 `list[float]` 类型，并且 `num` 必须是 `float` 类型：
    
    ```python
    def nums_below(numbers: Iterable[float], limit: float) -> list[float]:
        output = []
        for num in numbers:
            if num < limit:
                output.append(num)
        return output
    ```
    
    有关更多详细信息，请参阅 [类型推断和类型注解](https://mypy.readthedocs.io/en/stable/type_inference_and_annotations.html#type-inference-and-annotations)。


=== "英文"

    Once you have added type hints to a function (i.e. made it statically typed), mypy will automatically type check that function’s body. While doing so, mypy will try and infer as many details as possible.
    
    We saw an example of this in the `normalize_id` function above – mypy understands basic [isinstance](https://docs.python.org/3/library/functions.html#isinstance) checks and so can infer that the `user_id` variable was of type `int` in the if-branch and of type `str` in the else-branch.
    
    As another example, consider the following function. Mypy can type check this function without a problem: it will use the available context and deduce that `output` must be of type `list[float]` and that `num` must be of type `float`:
    
    ```python
    def nums_below(numbers: Iterable[float], limit: float) -> list[float]:
        output = []
        for num in numbers:
            if num < limit:
                output.append(num)
        return output
    ```
    
    For more details, see [Type inference and type annotations](https://mypy.readthedocs.io/en/stable/type_inference_and_annotations.html#type-inference-and-annotations).

## 来自库的类型

**Types from libraries**

=== "中文"

    Mypy 也可以理解如何处理你所使用的库中的类型。
    
    例如，mypy 内置了对 Python 标准库的深入了解。以下是一个使用 [pathlib 标准库模块](https://docs.python.org/3/library/pathlib.html) 中的 Path 对象的函数示例：
    
    ```python
    from pathlib import Path
    
    def load_template(template_path: Path, name: str) -> str:
        # Mypy 知道 `template_path` 有一个返回字符串的 `read_text` 方法
        template = template_path.read_text()
        # 因此它理解这行代码的类型检查是正确的
        return template.replace('USERNAME', name)
    ```
    
    如果你使用的第三方库 [声明支持类型检查](https://mypy.readthedocs.io/en/stable/installed_packages.html#installed-packages)，mypy 将根据该库中的类型提示对你使用该库的代码进行类型检查。
    
    然而，如果第三方库没有类型提示，mypy 将会提示缺少类型信息。
    
    ```text
    prog.py:1: error: Library stubs not installed for "yaml"
    prog.py:1: note: Hint: "python3 -m pip install types-PyYAML"
    prog.py:2: error: Library stubs not installed for "requests"
    prog.py:2: note: Hint: "python3 -m pip install types-requests"
    ...
    ```
    
    在这种情况下，你可以通过安装存根包来为 mypy 提供另一种类型信息来源。存根包是一个仅包含类型提示但不包含实际代码的包。
    
    ```bash
    $ python3 -m pip install types-PyYAML types-requests
    ```
    
    分发包的存根包通常命名为 `types-<distribution>`。注意，分发包的名称可能与导入的包名不同。例如，`types-PyYAML` 包含 `yaml` 包的存根。
    
    有关处理缺少类型信息的库错误的策略的更多讨论，请参阅 [缺少导入](https://mypy.readthedocs.io/en/stable/running_mypy.html#fix-missing-imports)。
    
    有关存根的更多信息，请参阅 [存根文件](https://mypy.readthedocs.io/en/stable/stubs.html#stub-files)。

=== "英文"

    Mypy can also understand how to work with types from libraries that you use.
    
    For instance, mypy comes out of the box with an intimate knowledge of the Python standard library. For example, here is a function which uses the Path object from the [pathlib standard library module](https://docs.python.org/3/library/pathlib.html):
    
    ```python
    from pathlib import Path
    
    def load_template(template_path: Path, name: str) -> str:
        # Mypy knows that `template_path` has a `read_text` method that returns a str
        template = template_path.read_text()
        # ...so it understands this line type checks
        return template.replace('USERNAME', name)
    ```
    
    If a third party library you use [declares support for type checking](https://mypy.readthedocs.io/en/stable/installed_packages.html#installed-packages), mypy will type check your use of that library based on the type hints it contains.
    
    However, if the third party library does not have type hints, mypy will complain about missing type information.
    
    ```text
    prog.py:1: error: Library stubs not installed for "yaml"
    prog.py:1: note: Hint: "python3 -m pip install types-PyYAML"
    prog.py:2: error: Library stubs not installed for "requests"
    prog.py:2: note: Hint: "python3 -m pip install types-requests"
    ...
    ```
    
    In this case, you can provide mypy a different source of type information, by installing a stub package. A stub package is a package that contains type hints for another library, but no actual code.
    
    `$ python3 -m pip install types-PyYAML types-requests`
    
    Stubs packages for a distribution are often named `types-<distribution>`. Note that a distribution name may be different from the name of the package that you import. For example, `types-PyYAML` contains stubs for the `yaml` package.
    
    For more discussion on strategies for handling errors about libraries without type information, refer to [Missing imports](https://mypy.readthedocs.io/en/stable/running_mypy.html#fix-missing-imports).
    
    For more information about stubs, see [Stub files](https://mypy.readthedocs.io/en/stable/stubs.html#stub-files).


## 下一步

**Next steps**

=== "中文"

    如果你时间紧迫，不想在开始之前阅读大量文档，以下是一些快速学习资源的指引：
    
    - 阅读 [mypy cheatsheet](./mypy/cheat_sheet_py3.md)。
    
    - 如果你有一个现有的大型代码库但没有很多类型注解，可以阅读 [如何在现有代码库中使用 mypy](https://mypy.readthedocs.io/en/stable/existing_code.html#existing-code)。
    
    - 阅读关于 Zulip 项目采纳 mypy 的 [博客文章](https://blog.zulip.org/2016/10/13/static-types-in-python-oh-mypy/)。
    
    - 如果你更喜欢观看讲座而不是阅读，以下是一些建议：
    
        - Carl Meyer: [Type Checked Python in the Real World](https://www.youtube.com/watch?v=pMgmKJyWKn8) (PyCon 2018)
    
        - Greg Price: [Clearer Code at Scale: Static Types at Zulip and Dropbox](https://www.youtube.com/watch?v=0c46YHS3RY8) (PyCon 2018)
    
    - 如果你遇到问题，可以查看 [mypy 的常见问题解决方案](https://mypy.readthedocs.io/en/stable/common_issues.html#common-issues)。
    
    - 你可以在 [mypy 问题跟踪器](https://github.com/python/mypy/issues) 和 typing 的 [Gitter 聊天](https://gitter.im/python/typing) 中提出有关 mypy 的问题。
    
    - 对于有关 Python 类型的一般问题，可以尝试在 [typing discussions](https://github.com/python/typing/discussions) 中发帖。
    
    你也可以继续阅读本文档，并跳过那些与你无关的部分。你不需要按顺序阅读各个章节。


=== "英文"

    If you are in a hurry and don’t want to read lots of documentation before getting started, here are some pointers to quick learning resources:
    
    - Read the [mypy cheatsheet](./mypy/cheat_sheet_py3.md).
    
    - Read [Using mypy with an existing codebase](https://mypy.readthedocs.io/en/stable/existing_code.html#existing-code) if you have a significant existing codebase without many type annotations.
    
    - Read the [blog post](https://blog.zulip.org/2016/10/13/static-types-in-python-oh-mypy/) about the Zulip project’s experiences with adopting mypy.
    
    - If you prefer watching talks instead of reading, here are some ideas:
    
        - Carl Meyer: [Type Checked Python in the Real World](https://www.youtube.com/watch?v=pMgmKJyWKn8) (PyCon 2018)
    
        - Greg Price: [Clearer Code at Scale: Static Types at Zulip and Dropbox](https://www.youtube.com/watch?v=0c46YHS3RY8) (PyCon 2018)
    
    - Look at [solutions to common issues](https://mypy.readthedocs.io/en/stable/common_issues.html#common-issues) with mypy if you encounter problems.
    
    - You can ask questions about mypy in the [mypy issue tracker](https://github.com/python/mypy/issues) and typing [Gitter chat](https://gitter.im/python/typing).
    
    - For general questions about Python typing, try posting at [typing discussions](https://github.com/python/typing/discussions).
    
    You can also continue reading this document and skip sections that aren’t relevant for you. You don’t need to read sections in order.