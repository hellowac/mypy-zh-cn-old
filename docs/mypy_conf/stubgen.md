# 自动生成存根 (stubgen)

**Automatic stub generation (stubgen)**

=== "中文"

    存根文件（参见 [PEP 484](https://peps.python.org/pep-0484/)）仅包含模块公共接口的类型提示，并且函数体为空。Mypy 可以使用存根文件代替真实实现来提供模块的类型信息。这些文件对于那些尚未添加类型提示的第三方模块（且在 typeshed 中没有可用的存根）以及 C 扩展模块（mypy 不能直接处理）非常有用。

    Mypy 包含了 `stubgen` 工具，可以自动生成 Python 模块和 C 扩展模块的存根文件（`.pyi` 文件）。例如，考虑以下源文件：

    ```python
    from other_module import dynamic

    BORDER_WIDTH = 15

    class Window:
        parent = dynamic()
        def __init__(self, width, height):
            self.width = width
            self.height = height

    def create_empty() -> Window:
        return Window(0, 0)
    ```

    `stubgen` 可以基于上述文件生成以下存根文件：

    ```python
    from typing import Any

    BORDER_WIDTH: int = ...

    class Window:
        parent: Any = ...
        width: Any = ...
        height: Any = ...
        def __init__(self, width, height) -> None: ...

    def create_empty() -> Window: ...
    ```

    `stubgen` 生成的是 *草稿* 存根。自动生成的存根文件通常需要一些手动更新，大多数类型会默认设置为 `Any`。如果你为最常用的功能添加更精确的类型注释，存根会更有用。

    本节的其余部分介绍了 `stubgen` 的命令行接口。运行 [stubgen --help](#h) 可以快速查看选项概述。

    !!! note

        命令行标志可能会在版本之间发生变化。

=== "英文"

    A stub file (see [PEP 484](https://peps.python.org/pep-0484/)) contains only type hints for the public interface of a module, with empty function bodies. Mypy can use a stub file instead of the real implementation to provide type information for the module. They are useful for third-party modules whose authors have not yet added type hints (and when no stubs are available in typeshed) and C extension modules (which mypy can't directly process).

    Mypy includes the ``stubgen`` tool that can automatically generate stub files (``.pyi`` files) for Python modules and C extension modules. For example, consider this source file:

    ```python
    from other_module import dynamic

    BORDER_WIDTH = 15

    class Window:
        parent = dynamic()
        def __init__(self, width, height):
            self.width = width
            self.height = height

    def create_empty() -> Window:
        return Window(0, 0)
    ```

    Stubgen can generate this stub file based on the above file:

    ```python
    from typing import Any

    BORDER_WIDTH: int = ...

    class Window:
        parent: Any = ...
        width: Any = ...
        height: Any = ...
        def __init__(self, width, height) -> None: ...

    def create_empty() -> Window: ...
    ```

    Stubgen generates *draft* stubs. The auto-generated stub files often require some manual updates, and most types will default to ``Any``. The stubs will be much more useful if you add more precise type annotations, at least for the most commonly used functionality.

    The rest of this section documents the command line interface of stubgen. Run [stubgen --help](#h) for a quick summary of options.

    !!! note 

        The command-line flags may change between releases.

## 指定要生成存根的内容

**Specifying what to stub**

=== "中文"

    你可以将 `stubgen` 指定要生成存根的源文件路径：

    ```shell
    $ stubgen foo.py bar.py
    ```

    这会生成存根文件 `out/foo.pyi` 和 `out/bar.pyi`。默认的输出目录是 `out`，可以通过 [-o DIR](#o) 选项覆盖。

    你也可以传递目录，`stubgen` 会递归地搜索目录中的所有 `.py` 文件，并为它们生成存根：

    ```shell
    $ stubgen my_pkg_dir
    ```

    另外，你可以使用 [-m](#m) 或 [-p](#p) 选项指定模块或包名：

    ```shell
    $ stubgen -m foo -m bar -p my_pkg_dir
    ```

    选项的详细说明：

    <span id="m"></span>`-m MODULE, --module MODULE`

    :    为指定的模块生成存根文件。此标志可以重复多次使用。

        `stubgen` *不会* 递归地为所提供模块的任何子模块生成存根。

    <span id="p"></span>`-p PACKAGE, --package PACKAGE`

    :    为指定的包生成存根。此标志可以重复多次使用。

        `stubgen` *会* 递归地为所提供包的所有子模块生成存根。除了这个行为，`-p` 选项与 [--module](#m) 选项是相同的。

    !!! note

        你不能在同一 `stubgen` 调用中混合使用路径和 [-m](#m)/[-p](#p) 选项。

    `stubgen` 使用启发式方法来避免为包含测试或供应商提供的第三方包的子模块生成存根。

=== "英文"

    You can give stubgen paths of the source files for which you want to generate stubs

    ```shell
    $ stubgen foo.py bar.py
    ```

    This generates stubs ``out/foo.pyi`` and ``out/bar.pyi``. The default output directory ``out`` can be overridden with [-o DIR](#o).

    You can also pass directories, and stubgen will recursively search them for any ``.py`` files and generate stubs for all of them

    ```shell
    $ stubgen my_pkg_dir
    ```

    Alternatively, you can give module or package names using the [-m](#m) or [-p](#p) options

    ```shell
    $ stubgen -m foo -m bar -p my_pkg_dir
    ```

    Details of the options:

    <span id="m"></span>`-m MODULE, --module MODULE`

    :    Generate a stub file for the given module. This flag may be repeated multiple times.

        Stubgen *will not* recursively generate stubs for any submodules of the provided module.

    <span id="p"></span>`-p PACKAGE, --package PACKAGE`

    :    Generate stubs for the given package. This flag maybe repeated multiple times.

        Stubgen *will* recursively generate stubs for all submodules of the provided package. This flag is identical to [--module](#m) apart from this behavior.

    !!! note 

        You can't mix paths and [-m](#m)/[-p](#p) options in the same stubgen invocation.

    Stubgen applies heuristics to avoid generating stubs for submodules that include tests or vendored third-party packages.

## 指定如何生成存根

**Specifying how to generate stubs**

=== "中文"

    默认情况下，`stubgen` 会尝试导入目标模块和包。这使得 `stubgen` 能够使用运行时反射来为 C 扩展模块生成存根，并提高生成的存根的质量。默认情况下，`stubgen` 还会使用 mypy 对任何 Python 模块进行轻量级语义分析。你可以使用以下标志来更改默认行为：

    <span id="no-import"></span>`--no-import`

    :    不尝试导入模块。只使用 mypy 的正常搜索机制来查找源文件。这不支持 C 扩展模块。此标志还禁用运行时反射功能，mypy 使用它来查找 `__all__` 的值。因此，存根中导出的名称集可能会不完整。这个标志通常只有在导入模块会引起不希望的副作用（例如，运行测试）时才有用。即使没有这个选项，`stubgen` 也会尽量跳过测试模块，但这并不总是有效。

    <span id="no-analysis"></span>`--no-analysis`

    :    不对源文件执行语义分析。这可能会生成更差的存根——特别是，一些模块、类和函数的别名可能会被表示为 `Any` 类型的变量。这个选项通常只有在语义分析导致了严重的 mypy 错误时才有用。不适用于 C 扩展模块。与 [--inspect-mode](#inspect-mode) 不兼容。

    <span id="inspect-mode"></span>`--inspect-mode`

    :    导入并检查模块，而不是解析源代码。这是 C 模块和仅 pyc 包的默认行为。这个标志在纯 Python 模块中很有用，特别是当这些模块使用动态生成的成员时，这些成员在默认的代码解析行为中可能会被忽略。隐式地使用 [--no-analysis](#no-analysis) 因为分析需要源代码。

    <span id="doc-dir"></span>`--doc-dir PATH`

    :    通过解析 `PATH` 中的 `.rst` 文档来推断更好的签名。这可能会生成更好的存根，但目前仅适用于 C 扩展模块。

=== "英文"

    By default stubgen will try to import the target modules and packages. This allows stubgen to use runtime introspection to generate stubs for C extension modules and to improve the quality of the generated stubs. By default, stubgen will also use mypy to perform light-weight semantic analysis of any Python modules. Use the following flags to alter the default behavior:

    <span id="no-import"></span>`--no-import`

    :    Don't try to import modules. Instead only use mypy's normal search mechanism to find sources. This does not support C extension modules. This flag also disables runtime introspection functionality, which mypy uses to find the value of ``__all__``. As result the set of exported imported names in stubs may be incomplete. This flag is generally only useful when importing a module causes unwanted side effects, such as the running of tests. Stubgen tries to skip test modules even without this option, but this does not always work.

    <span id="no-analysis"></span>`--no-analysis`

    :    Don't perform semantic analysis of source files. This may generate worse stubs -- in particular, some module, class, and function aliases may be represented as variables with the ``Any`` type. This is generally only useful if semantic analysis causes a critical mypy error.  Does not apply to C extension modules.  Incompatible with [--inspect-mode](#inspect-mode).

    <span id="inspect-mode"></span>`--inspect-mode`

    :    Import and inspect modules instead of parsing source code. This is the default behavior for C modules and pyc-only packages.  The flag is useful to force inspection for pure Python modules that make use of dynamically generated members that would otherwise be omitted when using the default behavior of code parsing.  Implies [--no-analysis](#no-analysis) as analysis requires source code.

    <span id="doc-dir"></span>`--doc-dir PATH`

    :    Try to infer better signatures by parsing .rst documentation in ``PATH``. This may result in better stubs, but currently it only works for C extension modules.

## 额外标志

**Additional flags**

=== "中文"

    <span id="h"></span>`-h, --help`

    :    显示帮助信息并退出。

    <span id="ignore-errors"></span>`--ignore-errors`

    :    如果在生成存根时发生异常，继续处理剩余的模块，而不是立即因错误而失败。

    <span id="include-private"></span>`--include-private`

    :    在存根中包含被认为是私有的定义（例如带有单个前导下划线且没有后缀下划线的名称，如 ``_foo``）。

    <span id="export-less"></span>`--export-less`

    :    不导出从同一包的其他模块中导入的所有名称。仅导出未在包含导入的模块中引用的导入名称。

    <span id="include-docstrings"></span>`--include-docstrings`

    :    在存根中包含文档字符串。这将为 Python 函数和类的存根以及 C 扩展函数的存根添加文档字符串。

    <span id="search-path"></span>`--search-path PATH`

    :    指定模块搜索目录，以冒号分隔（仅在使用 [--no-import](#no-import) 时有效）。

    <span id="o"></span>`-o PATH, --output PATH`

    :    更改输出目录。默认情况下，存根写入 ``./out`` 目录。如果该目录不存在，将创建该目录。输出目录中的现有存根将被覆盖，且不会发出警告。

    <span id="v"></span>`-v, --verbose`

    :    生成更详细的输出。

    <span id="q"></span>`-q, --quiet`

    :    生成更简洁的输出。

=== "英文"

    <span id="h"></span>`-h, --help`

    :    Show help message and exit.

    <span id="ignore-errors"></span>`--ignore-errors`

    :    If an exception was raised during stub generation, continue to process any remaining modules instead of immediately failing with an error.

    <span id="include-private"></span>`--include-private`

    :    Include definitions that are considered private in stubs (with names such as ``_foo`` with single leading underscore and no trailing underscores).

    <span id="export-less"></span>`--export-less`

    :    Don't export all names imported from other modules within the same package. Instead, only export imported names that are not referenced in the module that contains the import.

    <span id="include-docstrings"></span>`--include-docstrings`

    :    Include docstrings in stubs. This will add docstrings to Python function and classes stubs and to C extension function stubs.

    <span id="search-path"></span>`--search-path PATH`

    :    Specify module search directories, separated by colons (only used if [--no-import](#no-import) is given).

    <span id="o"></span>`-o PATH, --output PATH`

    :    Change the output directory. By default the stubs are written in the ``./out`` directory. The output directory will be created if it doesn't exist. Existing stubs in the output directory will be overwritten without warning.

    <span id="v"></span>`-v, --verbose`

    :    Produce more verbose output.

    <span id="q"></span>`-q, --quiet`

    :    Produce less verbose output.
