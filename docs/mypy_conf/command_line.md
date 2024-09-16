# mypy 命令行

**The mypy command line**

=== "中文"

    本节文档介绍了 mypy 的命令行接口。你可以通过运行 [mypy --help](#h) 查看可用标志的简要总结。

    !!! note

        命令行标志可能会在不同版本之间发生变化。

=== "英文"

    This section documents mypy's command line interface. You can view a quick summary of the available flags by running [mypy --help](#h).

    !!! note 

        Command line flags are liable to change between releases.

## 指定需要类型检查的内容

**Specifying what to type check**

=== "中文"

    默认情况下，你可以通过传递路径来指定要让 mypy 进行类型检查的代码：
 
    ```shell
    $ mypy foo.py bar.py some_directory
    ```
 
    请注意，目录会被递归检查。
 
    Mypy 还允许你通过其他几种方式指定要检查的代码。以下是相关标志的简要总结：有关详细信息，请参阅 [running-mypy](./running_mypy.md)。
 
    <span id="m"></span>`-m MODULE, --module MODULE`
 
    :    请求 mypy 检查提供的模块。这个标志可以重复使用多次。
 
          Mypy *不会* 递归地检查提供模块的任何子模块。
 
    <span id="p"></span>`-p PACKAGE, --package PACKAGE`
 
    :    请求 mypy 检查提供的包。这个标志可以重复使用多次。
 
          Mypy *会* 递归地检查提供包的所有子模块。这个标志与 [--module](#m) 的唯一区别在于这种行为。
 
    <span id="c"></span>`-c PROGRAM_TEXT, --command PROGRAM_TEXT`
 
    :    请求 mypy 将提供的字符串作为程序进行类型检查。
 
    <span id="exclude"></span>`--exclude`
 
    :    一个正则表达式，用于匹配文件名、目录名和路径，my py 在递归发现要检查的文件时应忽略这些匹配的内容。在所有平台上使用正斜杠。
    
        例如，为了避免发现任何名为 `setup.py` 的文件，你可以使用 ``--exclude '/setup\.py$'``。类似地，你可以通过例如 ``--exclude /build/`` 忽略具有特定名称的目录，或通过 ``--exclude /project/vendor/`` 忽略匹配子路径的目录。要忽略多个文件/目录/路径，你可以多次提供 --exclude 标志，例如 ``--exclude '/setup\.py$' --exclude '/build/'``。
    
        请注意，这个标志只影响递归目录树发现，也就是说，当 mypy 在目录树或包的子模块中发现文件时会被忽略。如果你明确地传递了文件或模块，它仍然会被检查。例如，``mypy --exclude '/setup.py$' but_still_check/setup.py``。
    
        特别是，``--exclude`` 不影响 mypy 的 [导入跟随](./running_mypy.md#跟随导入)。你可以使用每模块的 [follow_imports](./config_file.md#follow_imports) 配置选项，额外避免 mypy 跟随导入并检查你不希望检查的代码。
    
        请注意，mypy 永远不会递归发现名为 "site-packages"、"node_modules" 或 "\_\_pycache\_\_" 的文件和目录，或名称以点号开头的文件夹，就像 ``--exclude '/(site-packages|node_modules|__pycache__|\..*)/$'`` 所做的那样。Mypy 也不会递归发现扩展名不是 ``.py`` 或 ``.pyi`` 的文件。
  
=== "英文"

    By default, you can specify what code you want mypy to type check by passing in the paths to what you want to have type checked
 
    ```shell
    $ mypy foo.py bar.py some_directory
    ```
 
    Note that directories are checked recursively.
 
    Mypy also lets you specify what code to type check in several other ways. A short summary of the relevant flags is included below: for full details, see [running-mypy](./running_mypy.md).
 
    <span id="m"></span>`-m MODULE, --module MODULE`
 
    :    Asks mypy to type check the provided module. This flag may be repeated multiple times.
 
        Mypy *will not* recursively type check any submodules of the provided module.
 
    <span id="p"></span>`-p PACKAGE, --package PACKAGE`
 
    :    Asks mypy to type check the provided package. This flag may be repeated multiple times.
 
        Mypy *will* recursively type check any submodules of the provided package. This flag is identical to [--module](#m) apart from this behavior.
 
    <span id="c"></span>`-c PROGRAM_TEXT, --command PROGRAM_TEXT`
 
    :    Asks mypy to type check the provided string as a program.
 
 
    <span id="exclude"></span>`--exclude`
 
    :    A regular expression that matches file names, directory names and paths which mypy should ignore while recursively discovering files to check. Use forward slashes on all platforms.
 
        For instance, to avoid discovering any files named `setup.py` you could pass ``--exclude '/setup\.py$'``. Similarly, you can ignore discovering directories with a given name by e.g. ``--exclude /build/`` or those matching a subpath with ``--exclude /project/vendor/``. To ignore multiple files / directories / paths, you can provide the --exclude flag more than once, e.g ``--exclude '/setup\.py$' --exclude '/build/'``.
 
        Note that this flag only affects recursive directory tree discovery, that is, when mypy is discovering files within a directory tree or submodules of a package to check. If you pass a file or module explicitly it will still be checked. For instance, ``mypy --exclude '/setup.py$' but_still_check/setup.py``.
 
        In particular, ``--exclude`` does not affect mypy's [import following](./running_mypy.md#跟随导入). You can use a per-module [follow_imports](./config_file.md#follow_imports) config option to additionally avoid mypy from following imports and checking code you do not wish to be checked.
 
        Note that mypy will never recursively discover files and directories named "site-packages", "node_modules" or "\_\_pycache\_\_", or those whose name starts with a period, exactly as ``--exclude '/(site-packages|node_modules|__pycache__|\..*)/$'`` would. Mypy will also never recursively discover files with extensions other than ``.py`` or ``.pyi``.

## 可选参数

**Optional arguments**

=== "中文"

    <span id="h"></span>`-h, --help`

    :    显示帮助信息并退出。

    <span id="v"></span>`-v, --verbose`

    :    显示更详细的消息。

    <span id="V"></span>`-V, --version`

    :    显示程序的版本号并退出。

=== "英文"

    <span id="h"></span>`-h, --help`

    :    Show help message and exit.

    <span id="v"></span>`-v, --verbose`

    :    More verbose messages.

    <span id="V"></span>`-V, --version`

    :    Show program's version number and exit.

## 配置文件

**Config file**

=== "中文"

    <span id="config-file"></span>`--config-file CONFIG_FILE`

    :    该选项使 mypy 从指定的配置文件中读取配置设置。

        默认情况下，设置会从当前目录下的 ``mypy.ini``, ``.mypy.ini``, ``pyproject.toml`` 或 ``setup.cfg`` 文件中读取。配置设置会覆盖 mypy 的内置默认值，而命令行标志可以覆盖这些设置。

        指定 [config-file=](#config-file)（不带文件名）将忽略 *所有* 配置文件。

        有关配置文件的语法，请参见 [config-file](./config_file.md)。

    <span id="warn-unused-configs"></span>`--warn-unused-configs`

    :    该选项使 mypy 对未使用的 ``[mypy-<pattern>]`` 配置文件部分发出警告。（这需要使用 [--no-incremental](#no-incremental) 关闭增量模式。）

=== "英文"

    <span id="config-file"></span>`--config-file CONFIG_FILE`

    :   This flag makes mypy read configuration settings from the given file.

        By default settings are read from ``mypy.ini``, ``.mypy.ini``, ``pyproject.toml``, or ``setup.cfg`` in the current directory. Settings override mypy's built-in defaults and command line flags can override settings.

        Specifying [config-file=](#config-file)(with no filename) will ignore *all* config files.

        See [config-file](./config_file.md) for the syntax of configuration files.

    <span id="warn-unused-configs"></span>`--warn-unused-configs`

    :   This flag makes mypy warn about unused ``[mypy-<pattern>]`` config file sections. (This requires turning off incremental mode using [--no-incremental](#no-incremental).)

## 导入发现

**Import discovery**

=== "中文"

    以下选项用于自定义 mypy 发现和跟随导入的具体方式。

    <span id="explicit-package-bases"></span>`--explicit-package-bases`

    :  该选项告诉 mypy 顶级包将基于当前目录，或者 ``MYPYPATH`` 环境变量或 [mypy_path](./config_file.md#mypy_path) 配置选项的成员。此选项仅在缺少 `__init__.py` 文件时有用。有关详细信息，请参见 [Mapping file paths to modules](./running_mypy.md#映射文件路径到模块)。

    <span id="ignore-missing-imports"></span>`--ignore-missing-imports`

    :  该选项使 mypy 忽略所有缺失的导入。这相当于在代码库中的所有未解析导入处添加 ``# type: ignore`` 注释。

        请注意，此选项 *不会* 抑制关于成功解析的模块中缺失名称的错误。例如，如果有以下文件

        ```text
        package/__init__.py
        package/mod.py
        ```
        那么使用 [--ignore-missing-imports](#ignore-missing-imports) 时，mypy 将生成以下错误：

        ```python
        import package.unknown      # 无错误，已忽略
        x = package.unknown.func()  # OK. 'func' 被假定为 'Any' 类型

        from package import unknown          # 无错误，已忽略
        from package.mod import NonExisting  # 错误: 模块没有属性 'NonExisting'
        ```

        更多细节，请参见 [ignore-missing-imports](./running_mypy.md#缺失的导入)。

    <span id="follow-imports"></span>`--follow-imports {normal,silent,skip,error}`

    :    该选项调整 mypy 如何跟随通过命令行未明确传入的导入模块。

        默认选项是 ``normal``：mypy 将跟随并类型检查所有模块。有关其他选项的作用，请参见 [Following imports](./running_mypy.md#跟随导入)。

    <span id="python-executable"></span>`--python-executable EXECUTABLE`

    :    该选项使 mypy 从为 Python 可执行文件 ``EXECUTABLE`` 安装的 [PEP 561](https://peps.python.org/pep-0561/) 兼容包中收集类型信息。如果未提供，mypy 将使用为运行 mypy 的 Python 可执行文件安装的 PEP 561 兼容包。

        有关如何制作 PEP 561 兼容包的更多信息，请参见 [installed-packages](./installed_packages.md)。

    <span id="no-site-packages"></span>`--no-site-packages`

    :    该选项将禁用对 [PEP 561](https://peps.python.org/pep-0561/) 兼容包的搜索。这也将禁用对可用 Python 可执行文件的搜索。

        如果 mypy 无法找到适用于检查的 Python 版本的 Python 可执行文件，并且不需要使用 PEP 561 类型的包，请使用此选项。否则，请使用 [--python-executable](#python-executable)。

    <span id="no-silence-site-packages"></span>`--no-silence-site-packages`

    :    默认情况下，mypy 会抑制在 [PEP 561](https://peps.python.org/pep-0561/) 兼容包中生成的任何错误消息。添加此选项将禁用此行为。

    <span id="fast-module-lookup"></span>`--fast-module-lookup`

    :    默认逻辑用于扫描搜索路径以解析导入，在某些情况下具有二次最坏情况行为，例如，由大量共享顶级命名空间的文件夹触发，如下所示

        ```text
            foo/
                company/
                    foo/
                        a.py
            bar/
                company/
                    bar/
                        b.py
            baz/
                company/
                    baz/
                        c.py
            ...
        ```

        如果你遇到这种情况，可以通过设置 [--fast-module-lookup](#fast-module-lookup) 选项启用实验性的快速路径。

    <span id="no-namespace-packages"></span>`--no-namespace-packages`

    :    该选项禁用对命名空间包的导入发现（见 [PEP 420](https://peps.python.org/pep-0420/)）。特别是，这会防止发现没有 ``__init__.py``（或 ``__init__.pyi``）文件的包。

        该选项影响 mypy 如何找到在命令行上明确传递的模块和包。它也影响 mypy 如何确定传递在命令行上的文件的完全限定模块名称。有关详细信息，请参见 [Mapping file paths to modules](./running_mypy.md#映射文件路径到模块)。

=== "英文"

    The following flags customize how exactly mypy discovers and follows imports.

    <span id="explicit-package-bases"></span>`--explicit-package-bases`

    :  This flag tells mypy that top-level packages will be based in either the current directory, or a member of the ``MYPYPATH`` environment variable or [mypy_path](./config_file.md#mypy_path) config option. This option is only useful in the absence of `__init__.py`. See [Mapping file paths to modules](./running_mypy.md#映射文件路径到模块) for details.

    <span id="ignore-missing-imports"></span>`--ignore-missing-imports`

    :  This flag makes mypy ignore all missing imports. It is equivalent to adding ``# type: ignore`` comments to all unresolved imports within your codebase.

        Note that this flag does *not* suppress errors about missing names in successfully resolved modules. For example, if one has the following files

        ```text
        package/__init__.py
        package/mod.py
        ```
        Then mypy will generate the following errors with [--ignore-missing-imports](#ignore-missing-imports):

        ```python
        import package.unknown      # No error, ignored
        x = package.unknown.func()  # OK. 'func' is assumed to be of type 'Any'

        from package import unknown          # No error, ignored
        from package.mod import NonExisting  # Error: Module has no attribute 'NonExisting'
        ```

        For more details, see [ignore-missing-imports](./running_mypy.md#缺失的导入).

    <span id="follow-imports"></span>`--follow-imports {normal,silent,skip,error}`

    :    This flag adjusts how mypy follows imported modules that were not explicitly passed in via the command line.

        The default option is ``normal``: mypy will follow and type check all modules. For more information on what the other options do, see [Following imports](./running_mypy.md#跟随导入).

    <span id="python-executable"></span>`--python-executable EXECUTABLE`

    :    This flag will have mypy collect type information from [PEP 561](https://peps.python.org/pep-0561/) compliant packages installed for the Python executable ``EXECUTABLE``. If not provided, mypy will use PEP 561 compliant packages installed for the Python executable running mypy.

        See [installed-packages](./installed_packages.md) for more on making PEP 561 compliant packages.

    <span id="no-site-packages"></span>`--no-site-packages`

    :    This flag will disable searching for [PEP 561](https://peps.python.org/pep-0561/) compliant packages. This
        will also disable searching for a usable Python executable.

        Use this  flag if mypy cannot find a Python executable for the version of Python being checked, and you don't need to use PEP 561 typed packages. Otherwise, use [--python-executable](#python-executable).

    <span id="no-silence-site-packages"></span>`--no-silence-site-packages`

    :    By default, mypy will suppress any error messages generated within [PEP 561](https://peps.python.org/pep-0561/) compliant packages. Adding this flag will disable this behavior.

    <span id="fast-module-lookup"></span>`--fast-module-lookup`

    :    The default logic used to scan through search paths to resolve imports has a quadratic worse-case behavior in some cases, which is for instance triggered by a large number of folders sharing a top-level namespace as in

        ```text
            foo/
                company/
                    foo/
                        a.py
            bar/
                company/
                    bar/
                        b.py
            baz/
                company/
                    baz/
                        c.py
            ...
        ```

        If you are in this situation, you can enable an experimental fast path by setting the [--fast-module-lookup](#fast-module-lookup) option.

    <span id="no-namespace-packages"></span>`--no-namespace-packages`

    :    This flag disables import discovery of namespace packages (see [PEP 420](https://peps.python.org/pep-0420/)). In particular, this prevents discovery of packages that don't have an ``__init__.py`` (or ``__init__.pyi``) file.

        This flag affects how mypy finds modules and packages explicitly passed on the command line. It also affects how mypy determines fully qualified module names for files passed on the command line. See [Mapping file paths to modules](./running_mypy.md#映射文件路径到模块) for details.

## 平台配置

**Platform configuration**

=== "中文"

    默认情况下，mypy 会假定你打算使用与运行 mypy 本身相同的操作系统和 Python 版本来运行你的代码。以下选项允许你修改这一行为。

    有关如何使用这些选项的更多信息，请参见 [Python version and system platform checks](../mypy_other/common_issues.md#python-版本和系统平台检查)。

    <span id="python-version"></span>`--python-version X.Y`

    :    该选项将使 mypy 在类型检查你的代码时假定它在 Python 版本 X.Y 下运行。如果没有此选项，mypy 将默认使用运行 mypy 的 Python 版本。

        此选项会尝试查找对应版本的 Python 可执行文件，以搜索符合 [PEP 561](https://peps.python.org/pep-0561/) 的包。如果你想禁用此功能，可以使用 [--no-site-packages](#no-site-packages) 选项（有关更多细节，请参见 [import-discovery](#导入发现)）。

    <span id="platform"></span>`--platform PLATFORM`

    :    该选项将使 mypy 在类型检查你的代码时假定它在指定的操作系统下运行。如果没有此选项，mypy 将默认使用你当前使用的操作系统。

        ``PLATFORM`` 参数可以是 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 支持的任何字符串。

    <span id="always-true"></span>`--always-true NAME`

    :    该选项将把所有名为 ``NAME`` 的变量视为始终为真（compile-time constants）。此选项可以重复使用。

    <span id="always-false"></span>`--always-false NAME`

    :    该选项将把所有名为 ``NAME`` 的变量视为始终为假（compile-time constants）。此选项可以重复使用。

=== "英文"

    By default, mypy will assume that you intend to run your code using the same operating system and Python version you are using to run mypy itself. The following flags let you modify this behavior.

    For more information on how to use these flags, see [Python version and system platform checks](../mypy_other/common_issues.md#python-版本和系统平台检查).

    <span id="python-version"></span>`--python-version X.Y`

    :    This flag will make mypy type check your code as if it were run under Python version X.Y. Without this option, mypy will default to using whatever version of Python is running mypy.

        This flag will attempt to find a Python executable of the corresponding version to search for [PEP 561](https://peps.python.org/pep-0561/) compliant packages. If you'd like to disable this, use the [--no-site-packages](#no-site-packages) flag (see [import-discovery](#导入发现) for more details).

    <span id="platform"></span>`--platform PLATFORM`

    :    This flag will make mypy type check your code as if it were run under the given operating system. Without this option, mypy will default to using whatever operating system you are currently using.

        The ``PLATFORM`` parameter may be any string supported by [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform).

    <span id="always-true"></span>`--always-true NAME`

    :    This flag will treat all variables named ``NAME`` as compile-time constants that are always true.  This flag may be repeated.

    <span id="always-false"></span>`--always-false NAME`

    :    This flag will treat all variables named ``NAME`` as compile-time constants that are always false.  This flag may be repeated.

## 禁止动态类型

**Disallow dynamic typing**

=== "中文"

    `Any` 类型用于表示具有 [动态类型](../mypy/dynamic_typing.md) 的值。`--disallow-any` 系列标志可以禁止在模块中使用 `Any` 类型，这样可以在受控的方式下战略性地禁用动态类型。

    以下是可用的选项：

    <span id="disallow-any-unimported"></span>`--disallow-any-unimported`

    :    该选项禁止使用来自未跟随导入的类型（这些类型会被视作 `Any` 的别名）。未跟随的导入发生在以下两种情况之一：导入的模块不存在，或者设置了 [--follow-imports=skip](#follow-imports)。

    <span id="disallow-any-expr"></span>`--disallow-any-expr`

    :    该选项禁止模块中所有类型为 `Any` 的表达式。如果模块中出现了类型为 `Any` 的表达式，除非该表达式立即用作 [cast()] 的参数或赋值给具有显式类型注解的变量，否则 mypy 将输出错误。

        此外，声明 `Any` 类型的变量或强制转换为 `Any` 类型也是不允许的。请注意，调用接受 `Any` 类型参数的函数仍然是允许的。

    <span id="disallow-any-decorated"></span>`--disallow-any-decorated`

    :    该选项禁止在装饰器转换后具有 `Any` 类型的函数。

    <span id="disallow-any-explicit"></span>`--disallow-any-explicit`

    :    该选项禁止在类型注解和泛型类型参数等类型位置上显式使用 `Any`。

    <span id="disallow-any-generics"></span>`--disallow-any-generics`

    :    该选项禁止使用未指定显式类型参数的泛型类型。例如，你不能使用裸的 `x: list`，而必须始终写成类似 `x: list[int]` 的形式。

    <span id="disallow-subclassing-any"></span>`--disallow-subclassing-any`

    :    该选项报告错误，当一个类继承了类型为 `Any` 的值时。 这可能发生在基类从一个不存在的模块中导入（当使用 [--ignore-missing-imports](#ignore-missing-imports)）或由于 [--follow-imports=skip](#follow-imports) 或 `import` 语句上的 `# type: ignore` 注释而被忽略时。

        由于模块被静默处理，导入的类被赋予 `Any` 类型。默认情况下，mypy 会假设子类正确地继承了基类，尽管实际情况可能并非如此。此选项会使 mypy 报告错误。

=== "英文"

    The ``Any`` type is used to represent a value that has a [dynamic type](../mypy/dynamic_typing.md). The ``--disallow-any`` family of flags will disallow various uses of the ``Any`` type in a module -- this lets us strategically disallow the use of dynamic typing in a controlled way.

    The following options are available:

    <span id="disallow-any-unimported"></span>`--disallow-any-unimported`

    :    This flag disallows usage of types that come from unfollowed imports (such types become aliases for ``Any``). Unfollowed imports occur either when the imported module does not exist or when [--follow-imports=skip](#follow-imports) is set.

    <span id="disallow-any-expr"></span>`--disallow-any-expr`

    :    This flag disallows all expressions in the module that have type ``Any``. If an expression of type ``Any`` appears anywhere in the module mypy will output an error unless the expression is immediately used as an argument to [cast()] or assigned to a variable with an explicit type annotation.

        In addition, declaring a variable of type ``Any`` or casting to type ``Any`` is not allowed. Note that calling functions that take parameters of type ``Any`` is still allowed.

    <span id="disallow-any-decorated"></span>`--disallow-any-decorated`

    :    This flag disallows functions that have ``Any`` in their signature after decorator transformation.

    <span id="disallow-any-explicit"></span>`--disallow-any-explicit`

    :    This flag disallows explicit ``Any`` in type positions such as type annotations and generic type parameters.

    <span id="disallow-any-generics"></span>`--disallow-any-generics`

    :    This flag disallows usage of generic types that do not specify explicit type parameters. For example, you can't use a bare ``x: list``. Instead, you must always write something like ``x: list[int]``.

    <span id="disallow-subclassing-any"></span>`--disallow-subclassing-any`

    :    This flag reports an error whenever a class subclasses a value of type ``Any``.  This may occur when the base class is imported from a module that doesn't exist (when using [--ignore-missing-imports](#ignore-missing-imports)) or is ignored due to [--follow-imports=skip](#follow-imports) or a ``# type: ignore`` comment on the ``import`` statement.

        Since the module is silenced, the imported class is given a type of ``Any``. By default mypy will assume that the subclass correctly inherited the base class even though that may not actually be the case.  This flag makes mypy raise an error instead.

## 未类型化的定义与调用

**Untyped definitions and calls**

=== "中文"

    以下标志用于配置 mypy 处理未类型注解的函数定义或调用的方式。

    <span id="disallow-untyped-calls"></span>`--disallow-untyped-calls`

    :    该标志在函数调用时，如果调用的函数没有类型注解，则会报告错误。此选项适用于具有类型注解的函数调用未注解的函数。

    <span id="untyped-calls-exclude"></span>`--untyped-calls-exclude`

    :    该标志允许选择性地禁用 [--disallow-untyped-calls](#disallow-untyped-calls) 对特定包、模块或类中定义的函数和方法的限制。请注意，每个排除条目作为前缀进行匹配。例如（假设没有可用的 `third_party_lib` 类型注解）：

        ```python
        # mypy --disallow-untyped-calls
        #      --untyped-calls-exclude=third_party_lib.module_a
        #      --untyped-calls-exclude=foo.A
        from third_party_lib.module_a import some_func
        from third_party_lib.module_b import other_func
        import foo

        some_func()  # OK, 函数来自模块 `third_party_lib.module_a`
        other_func()  # E: 在类型化上下文中调用未注解的函数 "other_func"

        foo.A().meth()  # OK, 方法定义在类 `foo.A` 中
        foo.B().meth()  # E: 在类型化上下文中调用未注解的函数 "meth"

        # 文件 foo.py
        class A:
            def meth(self): pass
        class B:
            def meth(self): pass
        ```

    <span id="disallow-untyped-defs"></span>`--disallow-untyped-defs`

    :    该标志在遇到未注解或注解不完整的函数定义时报告错误。它是 [--disallow-incomplete-defs](#disallow-incomplete-defs) 的超集。

        例如，它会对 `def f(a, b)` 和 `def f(a: int, b)` 报告错误。

    <span id="disallow-incomplete-defs"></span>`--disallow-incomplete-defs`

    :    该标志在遇到部分注解的函数定义时报告错误，但允许完全未注解的定义。

        例如，它会对 `def f(a: int, b)` 报告错误，但不会对 `def f(a, b)` 报告错误。

    <span id="check-untyped-defs"></span>`--check-untyped-defs`

    :    该标志比前两个选项的限制宽松 -- 它会对每个函数的主体进行类型检查，无论函数是否具有类型注解。（默认情况下，未注解的函数主体不会进行类型检查。）

        它会假设所有参数的类型为 `Any`，并且始终推断返回类型为 `Any`。

    <span id="disallow-untyped-decorators"></span>`--disallow-untyped-decorators`

    :    该标志在具有类型注解的函数被未注解的装饰器装饰时报告错误。

=== "英文"

    The following flags configure how mypy handles untyped function definitions or calls.

    <span id="disallow-untyped-calls"></span>`--disallow-untyped-calls`

    :    This flag reports an error whenever a function with type annotations calls a function defined without annotations.

    <span id="untyped-calls-exclude"></span>`--untyped-calls-exclude`

    :    This flag allows to selectively disable [--disallow-untyped-calls](#disallow-untyped-calls) for functions and methods defined in specific packages, modules, or classes. Note that each exclude entry acts as a prefix. For example (assuming there are no type annotations for ``third_party_lib`` available):

        ```python

        # mypy --disallow-untyped-calls
        #      --untyped-calls-exclude=third_party_lib.module_a
        #      --untyped-calls-exclude=foo.A
        from third_party_lib.module_a import some_func
        from third_party_lib.module_b import other_func
        import foo

        some_func()  # OK, function comes from module `third_party_lib.module_a`
        other_func()  # E: Call to untyped function "other_func" in typed context

        foo.A().meth()  # OK, method was defined in class `foo.A`
        foo.B().meth()  # E: Call to untyped function "meth" in typed context

        # file foo.py
        class A:
            def meth(self): pass
        class B:
            def meth(self): pass
        ```

    <span id="disallow-untyped-defs"></span>`--disallow-untyped-defs`

    :    This flag reports an error whenever it encounters a function definition
        without type annotations or with incomplete type annotations.
        (a superset of [--disallow-incomplete-defs](#disallow-incomplete-defs)).

        For example, it would report an error for `def f(a, b)` and `def f(a: int, b)`.

    <span id="disallow-incomplete-defs"></span>`--disallow-incomplete-defs`

    :    This flag reports an error whenever it encounters a partly annotated function definition, while still allowing entirely unannotated definitions.

        For example, it would report an error for `def f(a: int, b)` but not `def f(a, b)`.

    <span id="check-untyped-defs"></span>`--check-untyped-defs`

    :    This flag is less severe than the previous two options -- it type checks the body of every function, regardless of whether it has type annotations. (By default the bodies of functions without annotations are not type checked.)

        It will assume all arguments have type ``Any`` and always infer ``Any`` as the return type.

    <span id="disallow-untyped-decorators"></span>`--disallow-untyped-decorators`

    :    This flag reports an error whenever a function with type annotations is decorated with a decorator without annotations.

## None 和 Optional 的处理

**None and Optional handling**

=== "中文"

    以下标志调整 mypy 处理类型为 ``None`` 的值的方式。

    <span id="implicit-optional"></span>`--implicit-optional`

    :    该标志使 mypy 将具有 ``None`` 默认值的参数视为具有隐式的 [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) 类型。

        例如，如果设置了此标志，则在下面的代码片段中，mypy 将假设 ``x`` 参数实际上是 ``Optional[int]`` 类型，因为默认参数是 ``None``：

        ```python
        def foo(x: int = None) -> None:
            print(x)
        ```

        **注意：** 从 mypy 0.980 开始，此功能默认被禁用。

    <span id="no-strict-optional"></span>`--no-strict-optional`

    :    该标志有效地禁用对 [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) 类型和 ``None`` 值的检查。启用此选项后，mypy 通常不会检查 ``None`` 值的使用 -- 它被视为与所有类型兼容。

        !!! warning

            ``--no-strict-optional`` 是不推荐的。避免使用此选项，并且在不了解其作用的情况下绝对不要使用它。

=== "英文"

    The following flags adjust how mypy handles values of type ``None``.

    <span id="implicit-optional"></span>`--implicit-optional`

    :    This flag causes mypy to treat arguments with a ``None`` default value as having an implicit [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) type.

        For example, if this flag is set, mypy would assume that the ``x`` parameter is actually of type ``Optional[int]`` in the code snippet below since the default parameter is ``None``:

        ```python
        def foo(x: int = None) -> None:
            print(x)
        ```

        **Note:** This was disabled by default starting in mypy 0.980.

    <span id="no-strict-optional"></span>`--no-strict-optional`

    :    This flag effectively disables checking of [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) types and ``None`` values. With this option, mypy doesn't generally check the use of ``None`` values -- it is treated as compatible with every type.

        !!! warning

            ``--no-strict-optional`` is evil. Avoid using it and definitely do not use it without understanding what it does.

## 配置警告(warnings)

**Configuring warnings**

=== "中文"

    以下标志用于启用对代码的警告，这些代码在语义上是正确的，但可能存在潜在问题或冗余。

    <span id="warn-redundant-casts"></span>`--warn-redundant-casts`

    :    该标志使 mypy 在代码中使用了不必要的类型转换时报告错误，这些转换可以安全地移除。

    <span id="warn-unused-ignores"></span>`--warn-unused-ignores`

    :    该标志使 mypy 在代码中使用了 ``# type: ignore`` 注释的行没有实际生成错误消息时报告错误。

        该标志和 [--warn-redundant-casts](#warn-redundant-casts) 标志在升级 mypy 时特别有用。之前，可能需要添加类型转换或 ``# type: ignore`` 注释来绕过 mypy 中的错误或缺少第三方库的类型存根。

        这两个标志帮助您发现那些不再需要的修复方法。

    <span id="no-warn-no-return"></span>`--no-warn-no-return`

    :    默认情况下，当函数在某些执行路径中缺少返回语句时，mypy 会生成错误。唯一的例外是：

        - 函数的返回类型为 ``None`` 或 ``Any``
        - 函数的主体为空，并且标记为抽象方法、在协议类中或在存根文件中
        - 执行路径永远不会返回，例如总是抛出异常

        使用 [--no-warn-no-return](#no-warn-no-return) 选项将禁用所有情况下的这些错误消息。

    <span id="warn-return-any"></span>`--warn-return-any`

    :    该标志使 mypy 在从一个声明了非 ``Any`` 返回类型的函数中返回类型为 ``Any`` 的值时生成警告。

    <span id="warn-unreachable"></span>`--warn-unreachable`

    :    该标志使 mypy 在执行类型分析后遇到被确定为不可达或冗余的代码时报告错误。这是检测代码中某些类型错误的有用方法。

        例如，启用此标志将使 mypy 报告 ``x > 7`` 检查是冗余的，并且以下的 ``else`` 块是不可达的。

        ```python
        def process(x: int) -> None:
            # 错误: "or" 的右操作数永远不会被评估
            if isinstance(x, int) or x > 7:
                # 错误: 不支持的操作数类型 for + ("int" 和 "str")
                print(x + "bad")
            else:
                # 错误: '语句不可达' 错误
                print(x + "bad")
        ```

        为了帮助防止 mypy 生成虚假的警告，“语句不可达”警告将在以下两种情况下被静默：

        1. 当不可达的语句是 ``raise`` 语句、``assert False`` 语句，或调用具有 [NoReturn](https://docs.python.org/3/library/typing.html#typing.NoReturn) 返回类型提示的函数时。换句话说，当不可达的语句抛出错误或以某种方式终止程序时。
        2. 当不可达的语句被 *故意* 标记为不可达时，使用 [Python 版本和系统平台检查](../mypy_other/common_issues.md#python-版本和系统平台检查)。

        !!! note

            目前 mypy 无法检测和报告使用 [值限制的类型变量](../mypy/generics.md#具有值限制的类型变量) 的任何函数内部的不可达或冗余代码。

            这种限制将在 future 版本的 mypy 中移除。

=== "英文"

    The following flags enable warnings for code that is sound but is potentially problematic or redundant in some way.

    <span id="warn-redundant-casts"></span>`--warn-redundant-casts`

    :    This flag will make mypy report an error whenever your code uses an unnecessary cast that can safely be removed.

    <span id="warn-unused-ignores"></span>`--warn-unused-ignores`

    :    This flag will make mypy report an error whenever your code uses a ``# type: ignore`` comment on a line that is not actually generating an error message.

        This flag, along with the [--warn-redundant-casts](#warn-redundant-casts) flag, are both particularly useful when you are upgrading mypy. Previously, you may have needed to add casts or ``# type: ignore`` annotations to work around bugs in mypy or missing stubs for 3rd party libraries.

        These two flags let you discover cases where either workarounds are no longer necessary.

    <span id="no-warn-no-return"></span>`--no-warn-no-return`

    :    By default, mypy will generate errors when a function is missing return statements in some execution paths. The only exceptions are when:

        - The function has a ``None`` or ``Any`` return type
        - The function has an empty body and is marked as an abstract method, is in a protocol class, or is in a stub file
        -  The execution path can never return; for example, if an exception is always raised

        Passing in [--no-warn-no-return](#no-warn-no-return) will disable these error messages in all cases.

    <span id="warn-return-any"></span>`--warn-return-any`

    :    This flag causes mypy to generate a warning when returning a value with type ``Any`` from a function declared with a non-``Any`` return type.

    <span id="warn-unreachable"></span>`--warn-unreachable`

    :    This flag will make mypy report an error whenever it encounters code determined to be unreachable or redundant after performing type analysis. This can be a helpful way of detecting certain kinds of bugs in your code.

        For example, enabling this flag will make mypy report that the ``x > 7`` check is redundant and that the ``else`` block below is unreachable.

        ```python
        def process(x: int) -> None:
            # Error: Right operand of "or" is never evaluated
            if isinstance(x, int) or x > 7:
                # Error: Unsupported operand types for + ("int" and "str")
                print(x + "bad")
            else:
                # Error: 'Statement is unreachable' error
                print(x + "bad")
        ```

        To help prevent mypy from generating spurious warnings, the "Statement is unreachable" warning will be silenced in exactly two cases:

        1.  When the unreachable statement is a ``raise`` statement, is an ``assert False`` statement, or calls a function that has the [NoReturn](https://docs.python.org/3/library/typing.html#typing.NoReturn) return type hint. In other words, when the unreachable statement throws an error or terminates the program in some way.
        2.  When the unreachable statement was *intentionally* marked as unreachable using [Python version and system platform checks](../mypy_other/common_issues.md#python-版本和系统平台检查).

        !!! note 

            Mypy currently cannot detect and report unreachable or redundant code inside any functions using [ Type variables with value restriction](../mypy/generics.md#具有值限制的类型变量).

            This limitation will be removed in future releases of mypy.

## 其他严格性标志

**Miscellaneous strictness flags**

=== "中文"

    以下标志用于配置 mypy 处理不符合上述任何部分的情况。

    <span id="allow-untyped-globals"></span>`--allow-untyped-globals`

    :    该标志使 mypy 抑制因无法完全推断全局和类变量类型而产生的错误。

    <span id="allow-redefinition"></span>`--allow-redefinition`

    :    默认情况下，mypy 不允许将变量重新定义为不相关的类型。此标志启用在某些上下文中重新定义变量为任意类型：仅允许在与原始定义相同的块和嵌套深度内的重新定义。例如，以下代码可能会有所帮助：

        ```python
        def process(items: list[str]) -> None:
            # 'items' 的类型是 list[str]
            items = [item.split() for item in items]
            # 现在 'items' 的类型是 list[list[str]]
        ```

        变量必须在重新定义之前被使用：

        ```python
        def process(items: list[str]) -> None:
            items = "mypy"  # 无效的重新定义为 str，因为变量尚未使用
            print(items)
            items = "100"  # 有效，items 现在是 str 类型
            items = int(items)  # 有效，items 现在是 int 类型
        ```

    <span id="local-partial-types"></span>`--local-partial-types`

    :    在 mypy 中，最常见的部分类型情况是使用 ``None`` 初始化的变量，但没有显式的 ``Optional`` 注解。默认情况下，mypy 不会检查跨模块顶层或类顶层的部分类型。此标志更改了行为，只允许在局部级别进行部分类型，因此不允许在不同作用域中的两个赋值推断变量类型。例如：

        ```python
        from typing import Optional

        a = None  # 如果使用 --local-partial-types，这里需要类型注解
        b: Optional[int] = None

        class Foo:
            bar = None  # 如果使用 --local-partial-types，这里需要类型注解
            baz: Optional[int] = None

            def __init__(self) -> None:
                self.bar = 1

        reveal_type(Foo().bar)  # 如果没有 --local-partial-types，会是 Union[int, None]
        ```

        注意：此选项在 mypy 守护进程中始终隐式启用，并且将在未来的 mypy 版本中默认启用。

    <span id="no-implicit-reexport"></span>`--no-implicit-reexport`

    :    默认情况下，导入到模块的值被视为已导出，mypy 允许其他模块导入它们。此标志更改了行为，只有当项目使用 `from-as` 导入或包含在 ``__all__`` 中时才会重新导出。注意，这在存根文件中始终被视为启用。例如：

        ```python
        # 这不会重新导出值
        from foo import bar

        # 这也不会
        from foo import bar as bang

        # 这将重新导出为 bar 并允许其他模块导入
        from foo import bar as bar

        # 这也将重新导出 bar
        from foo import bar
        __all__ = ['bar']
        ```

    <span id="strict-equality"></span>`--strict-equality`

    :    默认情况下，mypy 允许像 ``42 == 'no'`` 这样的永远为假的比较。使用此标志可以禁止这种非重叠类型的比较，以及类似的身份和容器检查：

        ```python
        from typing import Text

        items: list[int]
        if 'some string' in items:  # 错误: 非重叠容器检查！
            ...

        text: Text
        if text != b'other bytes':  # 错误: 非重叠等于检查！
            ...

        assert text is not None  # OK，检查 None 是允许的特例。
        ```

    <span id="extra-checks"></span>`--extra-checks`

    :    该标志启用额外的检查，这些检查在技术上是正确的，但在实际代码中可能不太实用。特别地，它禁止 ``TypedDict`` 更新中的部分重叠，并使通过 ``Concatenate`` 传递的参数仅限于位置参数。例如：

        ```python
        from typing import TypedDict

        class Foo(TypedDict):
            a: int

        class Bar(TypedDict):
            a: int
            b: int

        def test(foo: Foo, bar: Bar) -> None:
            # 这在技术上是不安全的，因为 foo 可以在运行时具有 Foo 的子类型，
            # 其中键 "b" 的类型与 int 不兼容，如下所示
            bar.update(foo)

        class Bad(Foo):
            b: str
        bad: Bad = {"a": 0, "b": "no"}
        test(bad, bar)
        ```

    <span id="strict"></span>`--strict`

    :    此标志模式启用所有可选错误检查标志。您可以在完整的 [mypy --help](#h) 输出中查看严格模式启用的标志列表。

        注意：运行 [--strict](#strict) 启用的标志的确切列表可能会随着时间而变化。

    <span id="disable-error-code"></span>`--disable-error-code`

    :    该标志允许全局禁用一个或多个错误代码。有关更多信息，请参阅 [错误代码](../mypy_other/error_codes.md)。

        ```python
        # 无标志
        x = 'a string'
        x.trim()  # 错误: "str" 没有属性 "trim"  [attr-defined]

        # 使用 --disable-error-code attr-defined
        x = 'a string'
        x.trim()
        ```

    <span id="enable-error-code"></span>`--enable-error-code`

    :    该标志允许全局启用一个或多个错误代码。有关更多信息，请参阅 [错误代码](../mypy_other/error_codes.md)。

        注意：此标志将覆盖 [--disable-error-code](#disable-error-code) 标志中禁用的错误代码。

        ```python
        # 使用 --disable-error-code attr-defined
        x = 'a string'
        x.trim()

        # --disable-error-code attr-defined --enable-error-code attr-defined
        x = 'a string'
        x.trim()  # 错误: "str" 没有属性 "trim"  [attr-defined]
        ```

=== "英文"

    This section documents any other flags that do not neatly fall under any of the above sections.

    <span id="allow-untyped-globals"></span>`--allow-untyped-globals`

    :    This flag causes mypy to suppress errors caused by not being able to fully infer the types of global and class variables.

    <span id="allow-redefinition"></span>`--allow-redefinition`

    :    By default, mypy won't allow a variable to be redefined with an unrelated type. This flag enables redefinition of a variable with an arbitrary type *in some contexts*: only redefinitions within the same block and nesting depth as the original definition are allowed. Example where this can be useful:

        ```python
        def process(items: list[str]) -> None:
            # 'items' has type list[str]
            items = [item.split() for item in items]
            # 'items' now has type list[list[str]]
        ```

        The variable must be used before it can be redefined:

        ```python
        def process(items: list[str]) -> None:
            items = "mypy"  # invalid redefinition to str because the variable hasn't been used yet
            print(items)
            items = "100"  # valid, items now has type str
            items = int(items)  # valid, items now has type int
        ```

    <span id="local-partial-types"></span>`--local-partial-types`

    :    In mypy, the most common cases for partial types are variables initialized using ``None``, but without explicit ``Optional`` annotations. By default, mypy won't check partial types spanning module top level or class top level. This flag changes the behavior to only allow partial types at local level, therefore it disallows inferring variable type for ``None`` from two assignments in different scopes. For example:

        ```python
        from typing import Optional

        a = None  # Need type annotation here if using --local-partial-types
        b: Optional[int] = None

        class Foo:
            bar = None  # Need type annotation here if using --local-partial-types
            baz: Optional[int] = None

            def __init__(self) -> None:
                self.bar = 1

        reveal_type(Foo().bar)  # Union[int, None] without --local-partial-types
        ```

        Note: this option is always implicitly enabled in mypy daemon and will become enabled by default for mypy in a future release.

    <span id="no-implicit-reexport"></span>`--no-implicit-reexport`

    :    By default, imported values to a module are treated as exported and mypy allows other modules to import them. This flag changes the behavior to not re-export unless the item is imported using from-as or is included in ``__all__``. Note this is always treated as enabled for stub files. For example:

        ```python
        # This won't re-export the value
        from foo import bar

        # Neither will this
        from foo import bar as bang

        # This will re-export it as bar and allow other modules to import it
        from foo import bar as bar

        # This will also re-export bar
        from foo import bar
        __all__ = ['bar']
        ```

    <span id="strict-equality"></span>`--strict-equality`

    :    By default, mypy allows always-false comparisons like ``42 == 'no'``. Use this flag to prohibit such comparisons of non-overlapping types, and similar identity and container checks:

        ```python
        from typing import Text

        items: list[int]
        if 'some string' in items:  # Error: non-overlapping container check!
            ...

        text: Text
        if text != b'other bytes':  # Error: non-overlapping equality check!
            ...

        assert text is not None  # OK, check against None is allowed as a special case.
        ```

    <span id="extra-checks"></span>`--extra-checks`

    :    This flag enables additional checks that are technically correct but may be impractical in real code. In particular, it prohibits partial overlap in ``TypedDict`` updates, and makes arguments prepended via ``Concatenate`` positional-only. For example:

        ```python
        from typing import TypedDict

        class Foo(TypedDict):
            a: int

        class Bar(TypedDict):
            a: int
            b: int

        def test(foo: Foo, bar: Bar) -> None:
            # This is technically unsafe since foo can have a subtype of Foo at
            # runtime, where type of key "b" is incompatible with int, see below
            bar.update(foo)

        class Bad(Foo):
            b: str
        bad: Bad = {"a": 0, "b": "no"}
        test(bad, bar)
        ```

    <span id="strict"></span>`--strict`

    :    This flag mode enables all optional error checking flags.  You can see the list of flags enabled by strict mode in the full [mypy --help](#h) output.

        Note: the exact list of flags enabled by running [--strict](#strict) may change over time.

    <span id="disable-error-code"></span>`--disable-error-code`

    :    This flag allows disabling one or multiple error codes globally. See [Error codes](../mypy_other/error_codes.md) for more information.

        ```python
        # no flag
        x = 'a string'
        x.trim()  # error: "str" has no attribute "trim"  [attr-defined]

        # When using --disable-error-code attr-defined
        x = 'a string'
        x.trim()
        ```

    <span id="enable-error-code"></span>`--enable-error-code`

    :    This flag allows enabling one or multiple error codes globally. See [Error codes](../mypy_other/error_codes.md) for more information.

        Note: This flag will override disabled error codes from the [--disable-error-code](#disable-error-code) flag.

        ```python
        # When using --disable-error-code attr-defined
        x = 'a string'
        x.trim()

        # --disable-error-code attr-defined --enable-error-code attr-defined
        x = 'a string'
        x.trim()  # error: "str" has no attribute "trim"  [attr-defined]
        ```

## 配置错误消息

**Configuring error messages**

=== "中文"

    以下标志允许您调整 mypy 在错误消息中显示的详细程度。

    <span id="show-error-context"></span>`--show-error-context`

    :    该标志会在所有错误消息前面添加“note”信息，解释错误的上下文。例如，考虑以下程序：

        ```python
        class Test:
            def foo(self, x: int) -> int:
                return x + "bar"
        ```

        默认情况下，mypy 显示的错误消息如下：

        ```text
        main.py:3: error: Unsupported operand types for + ("int" and "str")
        ```

        如果启用此标志，错误消息将变为：

        ```text
        main.py: note: In member "foo" of class "Test":
        main.py:3: error: Unsupported operand types for + ("int" and "str")
        ```

    <span id="show-column-numbers"></span>`--show-column-numbers`

    :    该标志会在错误消息中添加列偏移。例如，以下表示在第 12 行，第 9 列发生错误（注意列偏移从 0 开始）：

        ```text
        main.py:12:9: error: Unsupported operand types for / ("int" and "str")
        ```

    <span id="show-error-code-links"></span>`--show-error-code-links`

    :    该标志还会显示一个指向错误代码文档的链接，链接到由 mypy 报告的错误代码。相应的错误代码将在文档页面中高亮显示。如果启用此标志，错误消息将如下所示：

        ```text
        main.py:3: error: Unsupported operand types for - ("int" and "str")  [operator]
        main.py:3: note: See 'https://mypy.rtfd.io/en/stable/_refs.html#code-operator' for more info
        ```

    <span id="show-error-end"></span>`--show-error-end`

    :    该标志使 mypy 显示错误的起始位置和相关表达式的结束位置。这样，各种工具可以轻松高亮显示整个错误范围。格式为 ``file:line:column:end_line:end_column``。此选项隐含启用 ``--show-column-numbers``。

    <span id="hide-error-codes"></span>`--hide-error-codes`

    :    该标志会在错误消息中隐藏错误代码 ``[<code>]``。默认情况下，错误代码在每条错误消息之后显示：

        ```text
        prog.py:1: error: "str" has no attribute "trim"  [attr-defined]
        ```

        有关更多信息，请参阅 [错误代码](../mypy_other/error_codes.md)。

    <span id="pretty"></span>`--pretty`

    :    在错误消息中使用视觉上更美观的输出：使用软换行，显示源代码片段，并显示错误位置标记。

    <span id="no-color-output"></span>`--no-color-output`

    :    该标志会禁用错误消息中的彩色输出，默认启用。

    <span id="no-error-summary"></span>`--no-error-summary`

    :    该标志会禁用错误总结。默认情况下，mypy 会显示一行总结，包括错误总数、包含错误的文件数和检查过的文件数。

    <span id="show-absolute-path"></span>`--show-absolute-path`

    :    显示文件的绝对路径。

    <span id="soft-error-limit"></span>`--soft-error-limit N`

    :    该标志会调整一个限制，超出该限制后，mypy 将（有时）禁用报告大多数额外错误。该限制仅在大多数剩余错误似乎不有用或过于嘈杂时适用。如果 ``N`` 为负数，则没有限制。默认限制为 -1。

    <span id="force-uppercase-builtins"></span>`--force-uppercase-builtins`

    :    始终在错误消息中使用 ``List`` 而不是 ``list``，即使在 Python 3.9+ 中也是如此。

    <span id="force-union-syntax"></span>`--force-union-syntax`

    :    始终在错误消息中使用 ``Union[]`` 和 ``Optional[]`` 表示联合类型（而不是 ``|`` 运算符），即使在 Python 3.10+ 中也是如此。

=== "英文"

    The following flags let you adjust how much detail mypy displays in error messages.

    <span id="show-error-context"></span>`--show-error-context`

    :    This flag will precede all errors with "note" messages explaining the context of the error. For example, consider the following program:

        ```python
        class Test:
            def foo(self, x: int) -> int:
                return x + "bar"
        ```

        Mypy normally displays an error message that looks like this
        
        ```text
        main.py:3: error: Unsupported operand types for + ("int" and "str")
        ```

        If we enable this flag, the error message now looks like this

        ```text
        main.py: note: In member "foo" of class "Test":
        main.py:3: error: Unsupported operand types for + ("int" and "str")
        ```

    <span id="show-column-numbers"></span>`--show-column-numbers`

    :    This flag will add column offsets to error messages. For example, the following indicates an error in line 12, column 9 (note that column offsets are 0-based)

        ```text
        main.py:12:9: error: Unsupported operand types for / ("int" and "str")
        ```

    <span id="show-error-code-links"></span>`--show-error-code-links`

    :    This flag will also display a link to error code documentation, anchored to the error code reported by mypy. The corresponding error code will be highlighted within the documentation page. If we enable this flag, the error message now looks like this

        ```text
        main.py:3: error: Unsupported operand types for - ("int" and "str")  [operator]
        main.py:3: note: See 'https://mypy.rtfd.io/en/stable/_refs.html#code-operator' for more info
        ```

    <span id="show-error-end"></span>`--show-error-end`

    :    This flag will make mypy show not just that start position where an error was detected, but also the end position of the relevant expression. This way various tools can easily highlight the whole error span. The format is ``file:line:column:end_line:end_column``. This option implies ``--show-column-numbers``.

    <span id="hide-error-codes"></span>`--hide-error-codes`

    :    This flag will hide the error code ``[<code>]`` from error messages. By default, the error code is shown after each error message

        ```text
        prog.py:1: error: "str" has no attribute "trim"  [attr-defined]
        ```

        See [Error codes](../mypy_other/error_codes.md) for more information.

    <span id="pretty"></span>`--pretty`

    :    Use visually nicer output in error messages: use soft word wrap, show source code snippets, and show error location markers.

    <span id="no-color-output"></span>`--no-color-output`

    :    This flag will disable color output in error messages, enabled by default.

    <span id="no-error-summary"></span>`--no-error-summary`

    :    This flag will disable error summary. By default mypy shows a summary line including total number of errors, number of files with errors, and number of files checked.

    <span id="show-absolute-path"></span>`--show-absolute-path`

    :    Show absolute paths to files.

    <span id="soft-error-limit"></span>`--soft-error-limit N`

    :    This flag will adjust the limit after which mypy will (sometimes) disable reporting most additional errors. The limit only applies if it seems likely that most of the remaining errors will not be useful or they may be overly noisy. If ``N`` is negative, there is no limit. The default limit is -1.

    <span id="force-uppercase-builtins"></span>`--force-uppercase-builtins`

    :    Always use ``List`` instead of ``list`` in error messages, even on Python 3.9+.

    <span id="force-union-syntax"></span>`--force-union-syntax`

    :    Always use ``Union[]`` and ``Optional[]`` for union types in error messages (instead of the ``|`` operator), even on Python 3.10+.

## 增量模式

**Incremental mode**

=== "中文"

    默认情况下，mypy 会将类型信息存储到缓存中。mypy 会利用这些信息来避免在重新检查代码时进行不必要的重新计算。这可以帮助加快类型检查过程，特别是当大多数程序部分自上次 mypy 执行以来没有变化时。

    如果您希望在增量模式所能提供的速度提升之外进一步加快代码重新检查的时间，可以尝试在 [守护进程模式](./mypy_daemon.md) 下运行 mypy。

    <span id="no-incremental"></span>`--no-incremental`

    :    该标志禁用增量模式：mypy 在重新运行时将不再参考缓存。

        请注意，即使增量模式被禁用，mypy 仍会将数据写入缓存：有关更多详细信息，请参见下面的 [--cache-dir](#cache-dir) 标志。

    <span id="cache-dir"></span>`--cache-dir DIR`

    :    默认情况下，mypy 将所有缓存数据存储在当前目录下名为 ``.mypy_cache`` 的文件夹中。此标志允许您更改此文件夹的位置。此标志还可用于在使用 [远程缓存](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行) 时控制缓存的使用。

        此设置会覆盖 ``MYPY_CACHE_DIR`` 环境变量（如果已设置）。

        即使在禁用增量模式时，mypy 也会始终写入缓存，以便“预热”缓存。要禁用写入缓存，请使用 ``--cache-dir=/dev/null``（UNIX）或 ``--cache-dir=nul``（Windows）。

    <span id="sqlite-cache"></span>`--sqlite-cache`

    :    使用 [SQLite] 数据库存储缓存。

    <span id="cache-fine-grained"></span>`--cache-fine-grained`

    :    在缓存中包含细粒度的依赖信息，用于 mypy 守护进程。

    <span id="skip-version-check"></span>`--skip-version-check`

    :    默认情况下，mypy 会忽略由不同版本的 mypy 生成的缓存数据。此标志禁用此行为。

    <span id="skip-cache-mtime-checks"></span>`--skip-cache-mtime-checks`

    :    跳过基于 mtime 的缓存内部一致性检查。

=== "英文"

    By default, mypy will store type information into a cache. Mypy will use this information to avoid unnecessary recomputation when it type checks your code again.  This can help speed up the type checking process, especially when most parts of your program have not changed since the previous mypy run.

    If you want to speed up how long it takes to recheck your code beyond what incremental mode can offer, try running mypy in [daemon mode](./mypy_daemon.md).

    <span id="no-incremental"></span>`--no-incremental`

    :    This flag disables incremental mode: mypy will no longer reference the cache when re-run.

        Note that mypy will still write out to the cache even when incremental mode is disabled: see the [--cache-dir](#cache-dir) flag below for more details.

    <span id="cache-dir"></span>`--cache-dir DIR`

    :    By default, mypy stores all cache data inside of a folder named ``.mypy_cache`` in the current directory. This flag lets you change this folder. This flag can also be useful for controlling cache use when using [remote caching](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行).

        This setting will override the ``MYPY_CACHE_DIR`` environment variable if it is set.

        Mypy will also always write to the cache even when incremental mode is disabled so it can "warm up" the cache. To disable writing to the cache, use ``--cache-dir=/dev/null`` (UNIX) or ``--cache-dir=nul`` (Windows).

    <span id="sqlite-cache"></span>`--sqlite-cache`

    :    Use an [SQLite] database to store the cache.

    <span id="cache-fine-grained"></span>`--cache-fine-grained`

    :    Include fine-grained dependency information in the cache for the mypy daemon.

    <span id="skip-version-check"></span>`--skip-version-check`

    :    By default, mypy will ignore cache data generated by a different version of mypy. This flag disables that behavior.

    <span id="skip-cache-mtime-checks"></span>`--skip-cache-mtime-checks`

    :    Skip cache internal consistency checks based on mtime.

## 高级选项

**Advanced options**

=== "中文"

    以下标志主要适用于对开发或调试 mypy 内部工作感兴趣的用户。

    <span id="pdb"></span>`--pdb`

    :    这个标志会在 mypy 遇到致命错误时调用 Python 调试器（pdb）。

    <span id="show-traceback"></span>`--show-traceback, --tb`

    :    如果设置了此标志，mypy 在遇到致命错误时会显示完整的回溯信息。

    <span id="raise-exceptions"></span>`--raise-exceptions`

    :    在遇到致命错误时引发异常。

    <span id="custom-typing-module"></span>`--custom-typing-module MODULE`

    :    这个标志允许您使用自定义模块来替代 :py:mod:`typing` 模块。

    <span id="custom-typeshed-dir"></span>`--custom-typeshed-dir DIR`

    :    这个标志指定 mypy 查找标准库类型的目录，而不是使用 mypy 自带的 typeshed。这主要用于在将 typeshed 更改提交到上游之前进行测试，但也允许您使用 typeshed 的分叉版本。

        请注意，这不会影响第三方库的存根。要测试第三方存根，例如可以尝试 ``MYPYPATH=stubs/six mypy ...``。

    <span id="warn-incomplete-stub"></span>`--warn-incomplete-stub`

    :    这个标志会修改 [--disallow-untyped-defs](#disallow-untyped-defs) 和 [--disallow-incomplete-defs](#disallow-incomplete-defs) 标志，使它们在 typeshed 中的存根缺少类型注解或存在不完整的注解时也报告错误。如果两个标志都没有设置，[--warn-incomplete-stub](#warn-incomplete-stub) 也不会起作用。

        这个标志主要供那些希望为 typeshed 贡献代码的人使用，以便方便地发现类型注解的缺口和遗漏。

        如果您希望 mypy 报告代码库中使用未注解的函数的错误（无论该函数是否在 typeshed 中定义），请使用 [--disallow-untyped-calls](#disallow-untyped-calls) 标志。有关更多详细信息，请参见 [未类型化的定义与调用](#未类型化的定义与调用)。

    <span id="shadow-file"></span>`--shadow-file SOURCE_FILE SHADOW_FILE`

    :    当 mypy 被要求检查 ``SOURCE_FILE`` 时，这个标志会使 mypy 从 ``SHADOW_FILE`` 中读取并检查内容。然而，诊断信息仍会参考 ``SOURCE_FILE``。

        指定此参数多次（``--shadow-file X1 Y1 --shadow-file X2 Y2``）将允许 mypy 执行多个替换。

        这使得工具可以创建具有有用修改的临时文件，而不需要直接更改源文件。例如，假设我们有一个管道，它为某些变量添加 ``reveal_type``。这个管道在 ``original.py`` 上运行以生成 ``temp.py``。运行 ``mypy --shadow-file original.py temp.py original.py`` 将使 mypy 检查 ``temp.py`` 的内容，而不是 ``original.py``，但错误信息仍会引用 ``original.py``。

=== "英文"

    The following flags are useful mostly for people who are interested in developing or debugging mypy internals.

    <span id="pdb"></span>`--pdb`

    :    This flag will invoke the Python debugger when mypy encounters a fatal error.

    <span id="show-traceback"></span>`--show-traceback, --tb`

    :    If set, this flag will display a full traceback when mypy encounters a fatal error.

    <span id="raise-exceptions"></span>`--raise-exceptions`

    :    Raise exception on fatal error.

    <span id="custom-typing-module"></span>`--custom-typing-module MODULE`

    :    This flag lets you use a custom module as a substitute for the :py:mod:`typing` module.

    <span id="custom-typeshed-dir"></span>`--custom-typeshed-dir DIR`

    :    This flag specifies the directory where mypy looks for standard library typeshed stubs, instead of the typeshed that ships with mypy.  This is primarily intended to make it easier to test typeshed changes before submitting them upstream, but also allows you to use a forked version of typeshed.

        Note that this doesn't affect third-party library stubs. To test third-party stubs, for example try ``MYPYPATH=stubs/six mypy ...``.

    <span id="warn-incomplete-stub"></span>`--warn-incomplete-stub`

    :    This flag modifies both the [--disallow-untyped-defs](#disallow-untyped-defs) and [--disallow-incomplete-defs](#disallow-incomplete-defs) flags so they also report errors if stubs in typeshed are missing type annotations or has incomplete annotations. If both flags are missing, [--warn-incomplete-stub](#warn-incomplete-stub) also does nothing.

        This flag is mainly intended to be used by people who want contribute to typeshed and would like a convenient way to find gaps and omissions.

        If you want mypy to report an error when your codebase *uses* an untyped function, whether that function is defined in typeshed or not, use the [--disallow-untyped-calls](#disallow-untyped-calls) flag. See [Untyped definitions and calls](#未类型化的定义与调用) for more details.

    <span id="shadow-file"></span>`--shadow-file SOURCE_FILE SHADOW_FILE`

    :    When mypy is asked to type check ``SOURCE_FILE``, this flag makes mypy read from and type check the contents of ``SHADOW_FILE`` instead. However, diagnostics will continue to refer to ``SOURCE_FILE``.

        Specifying this argument multiple times (``--shadow-file X1 Y1 --shadow-file X2 Y2``) will allow mypy to perform multiple substitutions.

        This allows tooling to create temporary files with helpful modifications without having to change the source file in place. For example, suppose we have a pipeline that adds ``reveal_type`` for certain variables. This pipeline is run on ``original.py`` to produce ``temp.py``. Running ``mypy --shadow-file original.py temp.py original.py`` will then cause mypy to type check the contents of ``temp.py`` instead of  ``original.py``, but error messages will still reference ``original.py``.

## 报告生成

**Report generation**

=== "中文"

    如果设置了这些标志，mypy 将会以指定的格式生成报告，并将其输出到指定的目录。

    <span id="any-exprs-report"></span>`--any-exprs-report DIR`

    :    使 mypy 生成一个文本文件报告，记录代码库中存在的 ``Any`` 类型表达式的数量。

    <span id="cobertura-xml-report"></span>`--cobertura-xml-report DIR`

    :    使 mypy 生成一个 Cobertura XML 类型检查覆盖率报告。

        要生成此报告，您必须手动安装 [lxml] 库，或指定带有 setuptools 附加选项 ``mypy[reports]`` 的 mypy 安装。

    <span id="html-report"></span>`--html-report / --xslt-html-report DIR`

    :    使 mypy 生成一个 HTML 类型检查覆盖率报告。

        要生成此报告，您必须手动安装 [lxml] 库，或指定带有 setuptools 附加选项 ``mypy[reports]`` 的 mypy 安装。

    <span id="linecount-report"></span>`--linecount-report DIR`

    :    使 mypy 生成一个文本文件报告，记录代码库中已类型化和未类型化的函数和行数。

    <span id="linecoverage"></span>`--linecoverage-report DIR`

    :    使 mypy 生成一个 JSON 文件，将每个源文件的绝对文件名映射到该文件中属于已类型化函数的行号列表。

    <span id="lineprecision"></span>`--lineprecision-report DIR`

    :    使 mypy 生成一个平面文本文件报告，包含每个模块的统计数据，如检查了多少行代码等。

    <span id="txt-report"></span>`--txt-report / --xslt-txt-report DIR`

    :    使 mypy 生成一个文本文件类型检查覆盖率报告。

        要生成此报告，您必须手动安装 [lxml] 库，或指定带有 setuptools 附加选项 ``mypy[reports]`` 的 mypy 安装。

    <span id="xml-report"></span>`--xml-report DIR`

    :    使 mypy 生成一个 XML 类型检查覆盖率报告。

        要生成此报告，您必须手动安装 [lxml] 库，或指定带有 setuptools 附加选项 ``mypy[reports]`` 的 mypy 安装。

=== "英文"

    If these flags are set, mypy will generate a report in the specified format into the specified directory.

    <span id="any-exprs-report"></span>`--any-exprs-report DIR`

    :    Causes mypy to generate a text file report documenting how many expressions of type ``Any`` are present within your codebase.

    <span id="cobertura-xml-report"></span>`--cobertura-xml-report DIR`

    :    Causes mypy to generate a Cobertura XML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="html-report"></span>`--html-report / --xslt-html-report DIR`

    :    Causes mypy to generate an HTML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="linecount-report"></span>`--linecount-report DIR`

    :    Causes mypy to generate a text file report documenting the functions and lines that are typed and untyped within your codebase.

    <span id="linecoverage"></span>`--linecoverage-report DIR`

    :    Causes mypy to generate a JSON file that maps each source file's absolute filename to a list of line numbers that belong to typed functions in that file.

    <span id="lineprecision"></span>`--lineprecision-report DIR`

    :    Causes mypy to generate a flat text file report with per-module statistics of how many lines are typechecked etc.

    <span id="txt-report"></span>`--txt-report / --xslt-txt-report DIR`

    :    Causes mypy to generate a text file type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="xml-report"></span>`--xml-report DIR`

    :    Causes mypy to generate an XML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

## 启用不完整或实验性功能

**Enabling incomplete/experimental features**

=== "中文"

    <span id="enable-incomplete-feature"></span>`--enable-incomplete-feature {PreciseTupleTypes, NewGenericSyntax, InlineTypedDict}`

    :    一些功能可能需要多个 mypy 版本来实现，例如由于其复杂性、潜在的向后不兼容性或可能从社区反馈中获益的模糊语义。您可以使用此标志启用这些功能以进行早期预览。请注意，并不保证所有功能最终都会默认启用。在 *极少数情况下*，我们可能决定不继续推进某些功能。

    当前不完整/实验性功能列表：

    * ``PreciseTupleTypes``: 此功能将在各种场景中推断更精确的元组类型。在 [PEP 646](https://peps.python.org/pep-0646/) 添加变参类型之前，无法表达像“至少包含两个整数的元组”这样的类型。最好的类型是 ``tuple[int, ...]``。因此，mypy 对变长元组进行了非常宽松的检查。现在，这种类型可以表示为 ``tuple[int, int, *tuple[int, ...]]``。对于这样的更精确类型（当用户显式 *定义* 时），例如，mypy 会警告不安全的索引访问，并以类型安全的方式处理它们。然而，为了避免现有代码中的问题，mypy 并不会 *推断* 这些精确类型，即使技术上可以。以下是 ``PreciseTupleTypes`` 推断更精确类型的显著示例：

        ```python
        numbers: tuple[int, ...]

        more_numbers = (1, *numbers, 1)
        reveal_type(more_numbers)
        # 没有 PreciseTupleTypes: tuple[int, ...]
        # 有 PreciseTupleTypes: tuple[int, *tuple[int, ...], int]

        other_numbers = (1, 1) + numbers
        reveal_type(other_numbers)
        # 没有 PreciseTupleTypes: tuple[int, ...]
        # 有 PreciseTupleTypes: tuple[int, int, *tuple[int, ...]]

        if len(numbers) > 2:
            reveal_type(numbers)
            # 没有 PreciseTupleTypes: tuple[int, ...]
            # 有 PreciseTupleTypes: tuple[int, int, int, *tuple[int, ...]]
        else:
            reveal_type(numbers)
            # 没有 PreciseTupleTypes: tuple[int, ...]
            # 有 PreciseTupleTypes: tuple[()] | tuple[int] | tuple[int, int]
        ```

    * ``NewGenericSyntax``: 此功能启用对 [PEP 695](https://peps.python.org/pep-0695/) 定义的语法的支持。例如：

        ```python

        class Container[T]:  # 定义一个泛型类
            content: T

        def first[T](items: list[T]) -> T:  # 定义一个泛型函数
            return items[0]

        type Items[T] = list[tuple[T, T]]  # 定义一个泛型类型别名
        ```

    * ``InlineTypedDict``: 此功能启用用于内联 [TypedDicts](https://docs.python.org/3/library/typing.html#typing.TypedDict) 的非标准语法，例如：

        ```python

        def test_values() -> {"int": int, "str": str}:
            return {"int": 42, "str": "test"}
        ```

=== "英文"

    <span id="enable-incomplete-feature"></span>`--enable-incomplete-feature {PreciseTupleTypes, NewGenericSyntax, InlineTypedDict}`

    :    Some features may require several mypy releases to implement, for example due to their complexity, potential for backwards incompatibility, or ambiguous semantics that would benefit from feedback from the community. You can enable such features for early preview using this flag. Note that it is not guaranteed that all features will be ultimately enabled by default. In *rare cases* we may decide to not go ahead with certain features.

    List of currently incomplete/experimental features:

    * ``PreciseTupleTypes``: this feature will infer more precise tuple types in various scenarios. Before variadic types were added to the Python type system by [PEP 646](https://peps.python.org/pep-0646/), it was impossible to express a type like "a tuple with at least two integers". The best type available was ``tuple[int, ...]``. Therefore, mypy applied very lenient checking for variable-length tuples. Now this type can be expressed as ``tuple[int, int, *tuple[int, ...]]``. For such more precise types (when explicitly *defined* by a user) mypy, for example, warns about unsafe index access, and generally handles them in a type-safe manner. However, to avoid problems in existing code, mypy does not *infer* these precise types when it technically can. Here are notable examples where ``PreciseTupleTypes`` infers more precise types:

        ```python
        numbers: tuple[int, ...]

        more_numbers = (1, *numbers, 1)
        reveal_type(more_numbers)
        # Without PreciseTupleTypes: tuple[int, ...]
        # With PreciseTupleTypes: tuple[int, *tuple[int, ...], int]

        other_numbers = (1, 1) + numbers
        reveal_type(other_numbers)
        # Without PreciseTupleTypes: tuple[int, ...]
        # With PreciseTupleTypes: tuple[int, int, *tuple[int, ...]]

        if len(numbers) > 2:
            reveal_type(numbers)
            # Without PreciseTupleTypes: tuple[int, ...]
            # With PreciseTupleTypes: tuple[int, int, int, *tuple[int, ...]]
        else:
            reveal_type(numbers)
            # Without PreciseTupleTypes: tuple[int, ...]
            # With PreciseTupleTypes: tuple[()] | tuple[int] | tuple[int, int]
        ```

    * ``NewGenericSyntax``: this feature enables support for syntax defined by [PEP 695](https://peps.python.org/pep-0695/). For example:

        ```python

        class Container[T]:  # defines a generic class
            content: T

        def first[T](items: list[T]) -> T:  # defines a generic function
            return items[0]

        type Items[T] = list[tuple[T, T]]  # defines a generic type alias
        ```

    * ``InlineTypedDict``: this feature enables non-standard syntax for inline [TypedDicts](https://docs.python.org/3/library/typing.html#typing.TypedDict), for example:

        ```python

        def test_values() -> {"int": int, "str": str}:
            return {"int": 42, "str": "test"}
        ```

## 其他

**Miscellaneous**

=== "中文"

    <span id="install-types"></span>`--install-types`

    :    此标志使 mypy 使用 pip 安装已知缺失的第三方库的 stub 包。它将显示将要运行的 pip 命令，并在安装任何内容之前要求确认。出于安全原因，这些 stub 包仅限于一个小的手动选择的子集，这些包已由 typeshed 团队验证。这些包仅包含 stub 文件，不包含可执行代码。

        如果使用此选项而未提供任何文件或模块进行类型检查，mypy 将安装在之前的 mypy 运行中建议的 stub 包。如果提供了文件或模块进行类型检查，mypy 会首先检查这些文件，然后在运行结束时建议安装缺失的 stub，但仅当检测到缺失的模块时才会这样做。

        !!! 注意
        
            这是 mypy 0.900 中的新功能。之前的 mypy 版本包括了第三方包 stub 的选择，而不是单独安装它们。

    <span id="non-interactive"></span>`--non-interactive`

    :    与 [--install-types](#install-types) 一起使用时，此标志会使 mypy 使用 pip 自动安装所有建议的 stub 包，而无需确认，然后继续使用已安装的 stubs 进行类型检查（如果提供了文件或模块进行类型检查）。

        这在内部实现为最多两次 mypy 运行。第一次运行用于查找缺失的 stub 包，仅在未找到缺失的 stub 包时才会显示此运行的输出。如果发现缺失的 stub 包，则会安装它们，然后执行另一次运行。

    <span id="junit-xml"></span>`--junit-xml JUNIT_XML`

    :    此标志使 mypy 生成一个包含类型检查结果的 JUnit XML 测试结果文档。这可以更容易地将 mypy 集成到持续集成（CI）工具中。

    <span id="find-occurrences"></span>`--find-occurrences CLASS.MEMBER`

    :    此标志将使 mypy 打印出基于静态类型信息的类成员的所有使用情况。此功能是实验性的。

    <span id="scripts-are-modules"></span>`--scripts-are-modules`

    :    此标志将使命令行参数中看起来像脚本的文件（即名称不以 ``.py`` 结尾的文件）获得一个从脚本名称派生的模块名称，而不是固定的名称 [\_\_main\_\_](https://docs.python.org/3/library/__main__.html#module-__main__)。

        这使您可以在一次 mypy 调用中检查多个脚本。（默认的 [\_\_main\_\_](https://docs.python.org/3/library/__main__.html#module-__main__) 从技术上讲更正确，但如果您有许多脚本导入大型包，则此标志启用的行为通常更方便。）

=== "英文"

    <span id="install-types"></span>`--install-types`

    :    This flag causes mypy to install known missing stub packages for third-party libraries using pip.  It will display the pip command that will be run, and expects a confirmation before installing anything. For security reasons, these stubs are limited to only a small subset of manually selected packages that have been verified by the typeshed team. These packages include only stub files and no executable code.

        If you use this option without providing any files or modules to type check, mypy will install stub packages suggested during the previous mypy run. If there are files or modules to type check, mypy first type checks those, and proposes to install missing stubs at the end of the run, but only if any missing modules were detected.

        !!! note 
        
            This is new in mypy 0.900. Previous mypy versions included a selection of third-party package stubs, instead of having them installed separately.

    <span id="non-interactive"></span>`--non-interactive`

    :    hen used together with [--install-types](#install-types), this causes mypy to install all suggested stub packages using pip without asking for confirmation, and then continues to perform type checking using the installed stubs, if some files or modules are provided to type check.

    This is implemented as up to two mypy runs internally. The first run is used to find missing stub packages, and output is shown from this run only if no missing stub packages were found. If missing stub packages were found, they are installed and then another run is performed.

    <span id="junit-xml"></span>`--junit-xml JUNIT_XML`

    :    Causes mypy to generate a JUnit XML test result document with type checking results. This can make it easier to integrate mypy with continuous integration (CI) tools.

    <span id="find-occurrences"></span>`--find-occurrences CLASS.MEMBER`

    :    This flag will make mypy print out all usages of a class member based on static type information. This feature is experimental.

    <span id="scripts-are-modules"></span>`--scripts-are-modules`

    :    This flag will give command line arguments that appear to be scripts (i.e. files whose name does not end in ``.py``) a module name derived from the script name rather than the fixed name [\_\_main\_\_](https://docs.python.org/3/library/__main__.html#module-__main__).

        This lets you check more than one script in a single mypy invocation. (The default [\_\_main\_\_](https://docs.python.org/3/library/__main__.html#module-__main__) is technically more correct, but if you have many scripts that import a large package, the behavior enabled by this flag is often more convenient.)

[SQLite]: https://www.sqlite.org/

[cast()]: https://docs.python.org/3/library/typing.html#typing.cast
