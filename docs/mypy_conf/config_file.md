# mypy 配置文件

**The mypy configuration file**

=== "中文"

    Mypy 是非常可配置的。这一点在将类型引入现有代码库时尤为重要。有关这种情况的具体建议，请参阅[在现有代码库中使用 mypy](../mypy/existing_code.md)。

    Mypy 支持从文件中读取配置设置，其优先级顺序如下：

    ```text
    1. ./mypy.ini
    2. ./.mypy.ini
    3. ./pyproject.toml
    4. ./setup.cfg
    5. $XDG_CONFIG_HOME/mypy/config
    6. ~/.config/mypy/config
    7. ~/.mypy.ini
    ```

    重要的是要理解，配置文件不会进行合并，因为那样会导致歧义。[--config-file](./command_line.md#config-file) 命令行标志具有最高优先级，必须正确；否则 mypy 会报告错误并退出。如果没有命令行选项，mypy 将按照上述优先级顺序查找配置文件。

    大多数标志与[命令行标志](./command_line.md)非常接近，但也有一些标志名称的不同，且某些标志的值可能会根据正在处理的模块有所不同。

    一些标志支持用户主目录和环境变量扩展。要引用用户主目录，请在路径开头使用 ``~``。要扩展环境变量，请使用 ``$VARNAME`` 或 ``${VARNAME}``。

=== "英文"

    Mypy is very configurable. This is most useful when introducing typing to an existing codebase. See [Using mypy with an existing codebase](../mypy/existing_code.md) for concrete advice for that situation.

    Mypy supports reading configuration settings from a file with the following precedence order:

    ```text
    1. ./mypy.ini
    2. ./.mypy.ini
    3. ./pyproject.toml
    4. ./setup.cfg
    5. $XDG_CONFIG_HOME/mypy/config
    6. ~/.config/mypy/config
    7. ~/.mypy.ini
    ```

    It is important to understand that there is no merging of configuration files, as it would lead to ambiguity. The [--config-file](./command_line.md#config-file) command-line flag has the highest precedence and must be correct; otherwise mypy will report an error and exit. Without the command line option, mypy will look for configuration files in the precedence order above.

    Most flags correspond closely to [command-line flags](./command_line.md) but there are some differences in flag names and some flags may take a different value based on the module being processed.

    Some flags support user home directory and environment variable expansion. To refer to the user home directory, use ``~`` at the beginning of the path. To expand environment variables use ``$VARNAME`` or ``${VARNAME}``.


## 配置文件格式

**Config file format**

=== "中文"

    配置文件格式是通常的 [ini 文件](https://docs.python.org/3/library/configparser.html) 格式。它应包含用方括号括起来的部分名称，以及形式为 `NAME = VALUE` 的标志设置。注释以 ``#`` 字符开头。

    - 必须存在一个名为 ``[mypy]`` 的部分。这指定了全局标志。

    - 可能存在名为 ``[mypy-PATTERN1,PATTERN2,...]`` 的附加部分，其中 ``PATTERN1``、``PATTERN2`` 等是逗号分隔的完全限定模块名称模式，某些组件可以用 `*` 字符替代（例如 ``foo.bar``、``foo.bar.*``、``foo.*.baz``）。这些部分指定了仅适用于名称匹配至少一个模式的 *模块* 的附加标志。

        形式为 ``qualified_module_name`` 的模式仅匹配指定的模块，而 ``dotted_module_name.*`` 匹配 ``dotted_module_name`` 及其所有子模块（例如 ``foo.bar.*`` 会匹配 ``foo.bar``、``foo.bar.baz`` 和 ``foo.bar.baz.quux``）。

        模式也可以是“非结构化”的通配符，其中星号可以出现在名称的中间（例如 ``site.*.migrations.*``）。星号匹配零个或多个模块组件（因此 ``site.*.migrations.*`` 可以匹配 ``site.migrations``）。

        <span id="config-precedence"></span>当选项冲突时，配置的优先级顺序是：

        1.&nbsp;源文件中的[内联配置](./inline_config.md)

        2.&nbsp;具有具体模块名称的部分（``foo.bar``）

        3.&nbsp;具有“非结构化”通配符模式的部分（``foo.*.baz``），在配置文件中后面的部分会覆盖前面的部分。

        4.&nbsp;具有“结构化”通配符模式的部分（``foo.bar.*``），更具体的模式会覆盖更一般的模式。

        5.&nbsp;命令行选项。

        6.&nbsp;顶级配置文件选项。

    “结构化”模式（按特异性）与“非结构化”模式（按文件顺序）的优先级顺序差异是不幸的，并且可能在未来的版本中会发生变化。

    !!! note 

        [warn_unused_configs](./config_file.md#warn_unused_configs) 标志可能有助于调试拼写错误的部分名称。

    !!! note 

        配置标志在版本更新之间可能会发生变化。

=== "英文"

    The configuration file format is the usual [ini file](https://docs.python.org/3/library/configparser.html) format. It should contain section names in square brackets and flag settings of the form `NAME = VALUE`. Comments start with ``#`` characters.

    - A section named ``[mypy]`` must be present.  This specifies the global flags.

    - Additional sections named ``[mypy-PATTERN1,PATTERN2,...]`` may be present, where ``PATTERN1``, ``PATTERN2``, etc., are comma-separated patterns of fully-qualified module names, with some components optionally replaced by the '*' character (e.g. ``foo.bar``, ``foo.bar.*``, ``foo.*.baz``). These sections specify additional flags that only apply to *modules* whose name matches at least one of the patterns.

        A pattern of the form ``qualified_module_name`` matches only the named module, while ``dotted_module_name.*`` matches ``dotted_module_name`` and any submodules (so ``foo.bar.*`` would match all of ``foo.bar``, ``foo.bar.baz``, and ``foo.bar.baz.quux``).

        Patterns may also be "unstructured" wildcards, in which stars may appear in the middle of a name (e.g ``site.*.migrations.*``). Stars match zero or more module components (so ``site.*.migrations.*`` can match ``site.migrations``).

        <span id="config-precedence"></span>When options conflict, the precedence order for configuration is:

        1.&nbsp;[Inline configuration](./inline_config.md) in the source file

        2.&nbsp;Sections with concrete module names (``foo.bar``)

        3.&nbsp;Sections with "unstructured" wildcard patterns (``foo.*.baz``), with sections later in the configuration file overriding sections earlier.

        4.&nbsp;Sections with "well-structured" wildcard patterns (``foo.bar.*``), with more specific overriding more general.
        
        5.&nbsp;Command line options.

        6.&nbsp;Top-level configuration file options.

    The difference in precedence order between "structured" patterns (by specificity) and "unstructured" patterns (by order in the file) is unfortunate, and is subject to change in future versions.

    !!! note 

        The [warn_unused_configs](./config_file.md#warn_unused_configs) flag may be useful to debug misspelled section names.

    !!! note 

        Configuration flags are liable to change between releases.


## 每个模块和全局选项

**Per-module and global options**

=== "中文"

    某些配置选项可以在全局（``[mypy]`` 部分）或按模块（例如 ``[mypy-foo.bar]`` 部分）设置。

    如果一个选项既在全局中设置，也在特定模块中设置，则模块配置选项具有优先权。这允许你设置全局默认值，并在逐个模块的基础上进行覆盖。如果多个模式部分匹配一个模块，[则使用最具体的部分中的选项，当它们不一致时](#config-precedence)。

    根据其描述，某些其他选项可能只能在全局部分（``[mypy]``）中设置。

=== "英文"

    Some of the config options may be set either globally (in the ``[mypy]`` section) or on a per-module basis (in sections like ``[mypy-foo.bar]``).

    If you set an option both globally and for a specific module, the module configuration options take precedence. This lets you set global defaults and override them on a module-by-module basis. If multiple pattern sections match a module, [the options from the most specific section are used where they disagree](#config-precedence).

    Some other options, as specified in their description, may only be set in the global section (``[mypy]``).


## 反转选项值

**Inverting option values**

=== "中文"

    取布尔值的选项可以通过在其名称前添加 ``no_`` 来进行反转，或者（在适用时）通过将其前缀从 ``disallow`` 改为 ``allow``（反之亦然）来进行反转。

=== "英文"

    Options that take a boolean value may be inverted by adding ``no_`` to their name or by (when applicable) swapping their prefix from ``disallow`` to ``allow`` (and vice versa).


## 示例 ``mypy.ini``

**Example ``mypy.ini``**

=== "中文"

    下面是一个 ``mypy.ini`` 文件的示例。要使用此配置文件，将其放置在你的代码库根目录下，然后运行 mypy。

    ```ini
    # 全局选项：

    [mypy]
    warn_return_any = True
    warn_unused_configs = True

    # 按模块设置的选项：

    [mypy-mycode.foo.*]
    disallow_untyped_defs = True

    [mypy-mycode.bar]
    warn_return_any = False

    [mypy-somelibrary]
    ignore_missing_imports = True
    ```

    这个配置文件在 ``[mypy]`` 部分指定了两个全局选项。这两个选项将：

    1. 当函数返回一个被推断为类型 ``Any`` 的值时报告错误。

    2. 报告 mypy 未使用的配置选项。（这有助于我们在修改配置文件时捕获拼写错误）。

    接下来，这个配置文件指定了三个按模块设置的选项。前两个选项更改了 mypy 如何在 ``mycode.foo.*`` 和 ``mycode.bar`` 模块中进行类型检查，我们在这里假设这是你编写的两个模块。最后一个配置选项更改了 mypy 如何对 ``somelibrary`` 进行类型检查，我们在这里假设这是你安装并导入的某个第三方库。这些选项将：

    1. 仅在 ``mycode.foo`` 包内（即仅在 ``mycode/foo`` 目录下定义的函数）选择性地禁止未指定类型的函数定义。

    2. 仅在 ``mycode.bar`` 内选择性地 *禁用* “函数返回任何类型” 的警告。这覆盖了我们之前设置的全局默认值。

    3. 抑制当你的代码库尝试导入模块 ``somelibrary`` 时生成的任何错误消息。如果 ``somelibrary`` 是某个缺少类型提示的第三方库，这一点很有用。

=== "英文"

    Here is an example of a ``mypy.ini`` file. To use this config file, place it at the root of your repo and run mypy.

    ```ini
    # Global options:

    [mypy]
    warn_return_any = True
    warn_unused_configs = True

    # Per-module options:

    [mypy-mycode.foo.*]
    disallow_untyped_defs = True

    [mypy-mycode.bar]
    warn_return_any = False

    [mypy-somelibrary]
    ignore_missing_imports = True
    ```

    This config file specifies two global options in the ``[mypy]`` section. These two options will:

    1.  Report an error whenever a function returns a value that is inferred to have type ``Any``.

    2.  Report any config options that are unused by mypy. (This will help us catch typos when making changes to our config file).

    Next, this module specifies three per-module options. The first two options change how mypy type checks code in ``mycode.foo.*`` and ``mycode.bar``, which we assume here are two modules that you wrote. The final config option changes how mypy type checks ``somelibrary``, which we assume here is some 3rd party library you've installed and are importing. These options will:

    1.  Selectively disallow untyped function definitions only within the ``mycode.foo`` package -- that is, only for function definitions defined in the ``mycode/foo`` directory.

    2.  Selectively *disable* the "function is returning any" warnings within ``mycode.bar`` only. This overrides the global default we set earlier.

    3.  Suppress any error messages generated when your codebase tries importing the module ``somelibrary``. This is useful if ``somelibrary`` is some 3rd party library missing type hints.


## 导入发现

**Import discovery**

=== "中文"

    有关更多信息，请参阅命令行文档中的[导入发现](./command_line.md#导入发现)部分。

    <span id="mypy_path"></span>`mypy_path`

    :    **类型：** 字符串

        指定在尝试 ``MYPYPATH`` 环境变量中的路径后使用的路径。如果你希望将存根保存在代码库中，并与配置文件一起使用，这一点很有用。多个路径始终用 ``:`` 或 ``,`` 分隔，不论平台如何。用户主目录和环境变量将被扩展。

        相对路径是相对于 mypy 命令的工作目录，而不是配置文件的路径。使用 ``MYPY_CONFIG_FILE_DIR`` 环境变量来引用相对于配置文件的路径（例如 ``mypy_path = $MYPY_CONFIG_FILE_DIR/src``）。

        此选项只能在全局部分（``[mypy]``）中设置。

        **注意：** 在 Windows 上，使用 UNC 路径以避免使用 ``:``（例如 ``\\127.0.0.1\X$\MyDir``，其中 ``X`` 是驱动器字母）。

    <span id="files"></span>`files`

    :    **类型：** 以逗号分隔的字符串列表

        一个以逗号分隔的路径列表，如果命令行上未提供路径，则 mypy 将检查这些路径。支持使用 [glob](https://docs.python.org/3/library/glob.html#module-glob) 进行递归文件匹配，其中 ``*``（例如 ``*.py``）匹配当前目录中的文件，``**/``（例如 ``**/*.py``）匹配当前目录下任何目录中的文件。用户主目录和环境变量将被扩展。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="modules"></span>`modules`

    :    **类型：** 以逗号分隔的字符串列表

        一个以逗号分隔的包列表，如果命令行上未提供，则 mypy 将检查这些包。Mypy *不会* 递归地对提供的模块的任何子模块进行类型检查。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="packages"></span>`packages`

    :    **类型：** 以逗号分隔的字符串列表

        一个以逗号分隔的包列表，如果命令行上未提供，则 mypy 将检查这些包。Mypy *会* 递归地对提供的包的任何子模块进行类型检查。此标志与 [modules](#modules) 相同，唯一的区别在于此行为。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="exclude"></span>`exclude`

    :    **类型：** 正则表达式

        一个匹配文件名、目录名和路径的正则表达式，my.py 在递归发现要检查的文件时应忽略这些匹配项。请在所有平台上使用正斜杠（``/``）作为目录分隔符。

        ```ini

        [mypy]
        exclude = (?x)(
            ^one\.py$    # 文件名为 "one.py"
            | two\.pyi$  # 或以 "two.pyi" 结尾的文件
            | ^three\.   # 或以 "three." 开头的文件
            )
        ```

        制作一个既排除多个文件又保持可读性的正则表达式可能是一个挑战。上面的示例展示了一种方法。``(?x)`` 启用后续正则表达式的 ``VERBOSE`` 标志，这样可以 [忽略大部分空白字符并支持注释](https://docs.python.org/3/library/re.html#re.VERBOSE)。上述正则表达式等同于：``(^one\.py$|two\.pyi$|^three\.)``。

        有关更多详细信息，请参见 [--exclude](./command_line.md#exclude)。

        此选项只能在全局部分（``[mypy]``）中设置。

        !!! note 

            请注意，TOML 的等效项略有不同。它可以是一个单一的字符串（包括多行字符串）——作为单个正则表达式处理——或者是这样的字符串数组。以下 TOML 示例与上述 INI 示例等效。

            字符串数组：

            ```toml
            [tool.mypy]
            exclude = [
                "^one\\.py$",  # TOML 的双引号字符串需要转义反斜杠
                'two\.pyi$',  # TOML 的单引号字符串不需要
                '^three\.',
            ]
            ```

            单个多行字符串：

            ```toml
            [tool.mypy]
            exclude = '''(?x)(
                ^one\.py$    # 文件名为 "one.py"
                | two\.pyi$  # 或以 "two.pyi" 结尾的文件
                | ^three\.   # 或以 "three." 开头的文件
            )'''  # TOML 的单引号字符串不需要转义反斜杠
            ```

            参见 [使用 pyproject.toml 文件](#使用-pyprojecttoml-文件)。
    
    <span id="namespace_packages"></span>`namespace_packages`

    :    **类型：** 布尔值

        **默认值：** True

        启用 [PEP 420](https://peps.python.org/pep-0420/) 风格的命名空间包。有关更多信息，请参见对应的标志 [--no-namespace-packages](./command_line.md#no-namespace-packages)。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="explicit_package_bases"></span>`explicit_package_bases`

    :    **类型：** 布尔值

        **默认值：** False

        此标志告知 mypy 顶级包将基于当前目录、``MYPYPATH`` 环境变量或 [mypy_path](#mypy_path) 配置选项中的成员。此选项仅在没有 `__init__.py` 文件的情况下有用。有关详细信息，请参见 [映射文件路径到模块](./running_mypy.md#映射文件路径到模块)。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="ignore_missing_imports"></span>`ignore_missing_imports`

    :    **类型：** 布尔值

        **默认值：** False

        抑制关于无法解析的导入的错误消息。

        如果此选项在按模块部分中使用，模块名称应与 *导入的* 模块名称匹配，而不是包含导入语句的模块名称。

    <span id="follow_imports"></span>`follow_imports`

    :    **类型：** 字符串

        **默认值：** ``normal``

        指定当导入的模块作为 ``.py`` 文件找到且不属于命令行提供的文件、模块和包时应该怎么处理。

        四个可能的值是 ``normal``、``silent``、``skip`` 和 ``error``。有关解释，请参见 [--follow-imports](./command_line.md#follow-imports) 命令行标志的讨论。

        在按模块部分中使用此选项（可能使用通配符，如本页顶部所述）是一种防止 mypy 检查代码部分的好方法。

        如果此选项在按模块部分中使用，模块名称应与 *导入的* 模块名称匹配，而不是包含导入语句的模块名称。

    <span id="follow_imports_for_stubs"></span>`follow_imports_for_stubs`

    :    **类型：** 布尔值

        **默认值：** False

        确定是否即使对于存根（``.pyi``）文件也尊重 [follow_imports](#follow_imports) 设置。

        与 [follow_imports=skip](#follow_imports) 一起使用，可以用于抑制从 ``typeshed`` 导入模块，将其替换为 ``Any``。

        与 [follow_imports=error](#follow_imports) 一起使用，可以用于将特定 ``typeshed`` 模块的任何使用视为错误。

        !!! note 

            这在 mypy 守护进程中不受支持。

    <span id="python_executable"></span>`python_executable`

    :    **类型：** 字符串

        指定要检查的 Python 可执行文件的路径，以收集可用的 [PEP 561 包](https://mypy.readthedocs.io/en/stable/installed_packages.html#installed-packages) 的列表。用户主目录和环境变量将被扩展。默认值为用于运行 mypy 的可执行文件。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="no_site_packages"></span>`no_site_packages`

    :    **类型：** 布尔值

        **默认值：** False

        禁用使用已安装包中的类型信息（参见 [PEP 561](https://peps.python.org/pep-0561/)）。这还将禁用搜索可用的 Python 可执行文件。此选项的行为与 [--no-site-packages](./command_line.md#no-site-packages) 命令行标志相同。

    <span id="no_silence_site_packages"></span>`no_silence_site_packages`

    :    **类型：** 布尔值

        **默认值：** False

        启用报告在已安装包中生成的错误消息（有关分发类型信息的更多详细信息，请参见 [PEP 561](https://peps.python.org/pep-0561/)）。这些错误消息默认被抑制，因为你通常无法控制第三方代码中的错误。

        此选项只能在全局部分（``[mypy]``）中设置。

=== "英文"

    For more information, see the [Import discovery](./command_line.md#导入发现) section of the command line docs.

    <span id="mypy_path"></span>`mypy_path`

    :    **type:** string

        Specifies the paths to use, after trying the paths from ``MYPYPATH`` environment variable.  Useful if you'd like to keep stubs in your repo, along with the config file. Multiple paths are always separated with a ``:`` or ``,`` regardless of the platform. User home directory and environment variables will be expanded.

        Relative paths are treated relative to the working directory of the mypy command, not the config file. Use the ``MYPY_CONFIG_FILE_DIR`` environment variable to refer to paths relative to the config file (e.g. ``mypy_path = $MYPY_CONFIG_FILE_DIR/src``).

        This option may only be set in the global section (``[mypy]``).

        **Note:** On Windows, use UNC paths to avoid using ``:`` (e.g. ``\\127.0.0.1\X$\MyDir`` where ``X`` is the driv letter).

    <span id="files"></span>`files`

    :    **type:** comma-separated list of strings

        A comma-separated list of paths which should be checked by mypy if none are given on the command line. Supports recursive file globbing using [glob](https://docs.python.org/3/library/glob.html#module-glob), where ``*`` (e.g. ``*.py``) matches files in the current directory and ``**/`` (e.g. ``**/*.py``) matches files in any directories below the current one. User home directory and environment variables will be expanded.

        This option may only be set in the global section (``[mypy]``).

    <span id="modules"></span>`modules`

    :    **type:** comma-separated list of strings

        A comma-separated list of packages which should be checked by mypy if none are given on the command line. Mypy *will not* recursively type check any submodules of the provided module.

        This option may only be set in the global section (``[mypy]``).


    <span id="packages"></span>`packages`

    :    **type:** comma-separated list of strings

        A comma-separated list of packages which should be checked by mypy if none are given on the command line.  Mypy *will* recursively type check any submodules of the provided package. This flag is identical to [modules](#modules) apart from this behavior.

        This option may only be set in the global section (``[mypy]``).

    <span id="exclude"></span>`exclude`

    :    **type:** regular expression

        A regular expression that matches file names, directory names and paths which mypy should ignore while recursively discovering files to check. Use forward slashes (``/``) as directory separators on all platforms.

        ```ini

        [mypy]
        exclude = (?x)(
            ^one\.py$    # files named "one.py"
            | two\.pyi$  # or files ending with "two.pyi"
            | ^three\.   # or files starting with "three."
            )
        ```

        Crafting a single regular expression that excludes multiple files while remaining human-readable can be a challenge. The above example demonstrates one approach. ``(?x)`` enables the ``VERBOSE`` flag for the subsequent regular expression, which [ignores most whitespace and supports comments](https://docs.python.org/3/library/re.html#re.VERBOSE). The above is equivalent to: ``(^one\.py$|two\.pyi$|^three\.)``.

        For more details, see [--exclude](./command_line.md#exclude).

        This option may only be set in the global section (``[mypy]``).

        !!! note 

            Note that the TOML equivalent differs slightly. It can be either a single string (including a multi-line string) -- which is treated as a single regular expression -- or an array of such strings. The following TOML examples are equivalent to the above INI example.

            Array of strings:
        
            ```toml
            [tool.mypy]
            exclude = [
                "^one\\.py$",  # TOML's double-quoted strings require escaping backslashes
                'two\.pyi$',  # but TOML's single-quoted strings do not
                '^three\.',
            ]
            ```
        
            A single, multi-line string:
        
            ```toml
            [tool.mypy]
            exclude = '''(?x)(
                ^one\.py$    # files named "one.py"
                | two\.pyi$  # or files ending with "two.pyi"
                | ^three\.   # or files starting with "three."
            )'''  # TOML's single-quoted strings do not require escaping backslashes
            ```
        
            See [Using a pyproject.toml file](#使用-pyprojecttoml-文件).

    <span id="namespace_packages"></span>`namespace_packages`

        :    **type:** boolean

        **default:** True

        Enables [PEP 420](https://peps.python.org/pep-0420/) style namespace packages.  See the corresponding flag [--no-namespace-packages](./command_line.md#no-namespace-packages) for more information.

        This option may only be set in the global section (``[mypy]``).

    <span id="explicit_package_bases"></span>`explicit_package_bases`

        :    **type:** boolean

        **default:** False

        This flag tells mypy that top-level packages will be based in either the current directory, or a member of the ``MYPYPATH`` environment variable or [mypy_path](#mypy_path) config option. This option is only useful in the absence of `__init__.py`. See [Mapping file paths to modules](./running_mypy.md#映射文件路径到模块) for details.

        This option may only be set in the global section (``[mypy]``).

    <span id="ignore_missing_imports"></span>`ignore_missing_imports`

        :    **type:** boolean

        **default:** False

        Suppresses error messages about imports that cannot be resolved.

        If this option is used in a per-module section, the module name should match the name of the *imported* module, not the module containing the import statement.

    <span id="follow_imports"></span>`follow_imports`

        :    **type:** string

        **default:** ``normal``

        Directs what to do with imports when the imported module is found as a ``.py`` file and not part of the files, modules and packages provided on the command line.

        The four possible values are ``normal``, ``silent``, ``skip`` and ``error``.  For explanations see the discussion for the [--follow-imports](./command_line.md#follow-imports) command line flag.

        Using this option in a per-module section (potentially with a wildcard, as described at the top of this page) is a good way to prevent mypy from checking portions of your code.

        If this option is used in a per-module section, the module name should match the name of the *imported* module, not the module containing the import statement.

    <span id="follow_imports_for_stubs"></span>`follow_imports_for_stubs`

        :    **type:** boolean

        **default:** False

        Determines whether to respect the [follow_imports](#follow_imports) setting even for stub (``.pyi``) files.

        Used in conjunction with [follow_imports=skip](#follow_imports), this can be used to suppress the import of a module from ``typeshed``, replacing it with ``Any``.

        Used in conjunction with [follow_imports=error](#follow_imports), this can be used to make any use of a particular ``typeshed`` module an error.

        !!! note 

            This is not supported by the mypy daemon.

    <span id="python_executable"></span>`python_executable`

        :    **type:** string

        Specifies the path to the Python executable to inspect to collect a list of available [PEP 561 packages](https://mypy.readthedocs.io/en/stable/installed_packages.html#installed-packages). User home directory and environment variables will be expanded. Defaults to the executable used to run mypy.

        This option may only be set in the global section (``[mypy]``).

    <span id="no_site_packages"></span>`no_site_packages`

        :    **type:** boolean

        **default:** False

        Disables using type information in installed packages (see [PEP 561](https://peps.python.org/pep-0561/)). This will also disable searching for a usable Python executable. This acts the same as [--no-site-packages](./command_line.md#no-site-packages) command line flag.

    <span id="no_silence_site_packages"></span>`no_silence_site_packages`

        :    **type:** boolean

        **default:** False

        Enables reporting error messages generated within installed packages (see [PEP 561](https://peps.python.org/pep-0561/) for more details on distributing type information). Those error messages are suppressed by default, since you are usually not able to control errors in 3rd party code.

        This option may only be set in the global section (``[mypy]``).


## 平台配置

**Platform configuration**

=== "中文"

    <span id="python_version"></span>`python_version`

    :    **类型：** 字符串

        指定用于解析和检查目标程序的 Python 版本。字符串格式应为 ``MAJOR.MINOR``——例如 ``2.7``。默认值是用于运行 mypy 的 Python 解释器的版本。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="platform"></span>`platform`

    :    **类型：** 字符串

        指定目标程序的操作系统平台，例如 ``darwin`` 或 ``win32``（分别表示 OS X 或 Windows）。默认值是 Python 的 [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) 变量所揭示的当前平台。

        此选项只能在全局部分（``[mypy]``）中设置。

    <span id="always_true"></span>`always_true`

    :    **类型：** 以逗号分隔的字符串列表

        指定一个变量列表，这些变量将被 mypy 视为始终为真（compile-time constants that are always true）的常量。

    <span id="always_false"></span>`always_false`

    :    **类型：** 以逗号分隔的字符串列表

        指定一个变量列表，这些变量将被 mypy 视为始终为假（compile-time constants that are always false）的常量。

=== "英文"

    <span id="python_version"></span>`python_version`

    :    **type:** string

        Specifies the Python version used to parse and check the target program.  The string should be in the format ``MAJOR.MINOR`` -- for example ``2.7``.  The default is the version of the Python interpreter used to run mypy.

        This option may only be set in the global section (``[mypy]``).

    <span id="platform"></span>`platform`

    :    **type:** string

        Specifies the OS platform for the target program, for example ``darwin`` or ``win32`` (meaning OS X or Windows, respectively). The default is the current platform as revealed by Python's [sys.platform](https://docs.python.org/3/library/sys.html#sys.platform) variable.

        This option may only be set in the global section (``[mypy]``).

    <span id="always_true"></span>`always_true`

    :    **type:** comma-separated list of strings

        Specifies a list of variables that mypy will treat as compile-time constants that are always true.

    <span id="always_false"></span>`always_false`

    :    **type:** comma-separated list of strings

        Specifies a list of variables that mypy will treat as compile-time constants that are always false.


## 禁止动态类型

**Disallow dynamic typing**

=== "中文"

    有关更多信息，请参见命令行文档的 [Disallow dynamic typing](./command_line.md#禁止动态类型) 部分。

    <span id="disallow_any_unimported"></span>`disallow_any_unimported`

    :    **类型：** 布尔值

        **默认值：** False

        禁止使用来自未跟踪导入的类型（来自未跟踪导入的任何内容会自动被赋予 ``Any`` 类型）。

    <span id="disallow_any_expr"></span>`disallow_any_expr`

    :    **类型：** 布尔值

        **默认值：** False

        禁止模块中所有具有 ``Any`` 类型的表达式。

    <span id="disallow_any_decorated"></span>`disallow_any_decorated`

    :    **类型：** 布尔值

        **默认值：** False

        禁止装饰器转换后函数签名中的 ``Any`` 类型。

    <span id="disallow_any_explicit"></span>`disallow_any_explicit`

    :    **类型：** 布尔值

        **默认值：** False

        禁止在类型位置（如类型注解和泛型类型参数）中显式使用 ``Any``。

    <span id="disallow_any_generics"></span>`disallow_any_generics`

    :    **类型：** 布尔值

        **默认值：** False

        禁止使用未指定显式类型参数的泛型类型。

    <span id="disallow_subclassing_any"></span>`disallow_subclassing_any`

    :    **类型：** 布尔值

        **默认值：** False

        禁止对 ``Any`` 类型的值进行子类化。

=== "英文"

    For more information, see the [Disallow dynamic typing](./command_line.md#禁止动态类型) section of the command line docs.

    <span id="disallow_any_unimported"></span>`disallow_any_unimported`

    :    **type:** boolean

        **default:** False

        Disallows usage of types that come from unfollowed imports (anything imported from an unfollowed import is automatically given a type of ``Any``).

    <span id="disallow_any_expr"></span>`disallow_any_expr`

    :    **type:** boolean

        **default:** False

        Disallows all expressions in the module that have type ``Any``.

    <span id="disallow_any_decorated"></span>`disallow_any_decorated`

    :    **type:** boolean

        **default:** False

        Disallows functions that have ``Any`` in their signature after decorator transformation.

    <span id="disallow_any_explicit"></span>`disallow_any_explicit`

    :    **type:** boolean

        **default:** False

        Disallows explicit ``Any`` in type positions such as type annotations and generic type parameters.

    <span id="disallow_any_generics"></span>`disallow_any_generics`

    :    **type:** boolean

        **default:** False

        Disallows usage of generic types that do not specify explicit type parameters.

    <span id="disallow_subclassing_any"></span>`disallow_subclassing_any`

    :    **type:** boolean

        **default:** False

        Disallows subclassing a value of type ``Any``.


## 未类型化的定义与调用

**Untyped definitions and calls**

=== "中文"

    有关更多信息，请参见命令行文档的 [Untyped definitions and calls](./command_line.md#未类型化的定义与调用) 部分。

    <span id="disallow_untyped_calls"></span>`disallow_untyped_calls`

    :    **类型：** 布尔值

        **默认值：** False

        禁止从具有类型注解的函数中调用没有类型注解的函数。请注意，当在每个模块选项中使用时，它会在指定的模块**内部**启用/禁用此检查，而不是对来自这些模块的函数进行检查。例如，像这样的配置：

        ```ini
        [mypy]
        disallow_untyped_calls = True

        [mypy-some.library.*]
        disallow_untyped_calls = False
        ```

        将在 ``some.library`` 内部禁用此检查，而不是对导入 ``some.library`` 的代码进行检查。如果你想选择性地对所有导入 ``some.library`` 的代码禁用此检查，你应该使用 [untyped_calls_exclude](#untyped_calls_exclude)，例如：

        ```ini
        [mypy]
        disallow_untyped_calls = True
        untyped_calls_exclude = some.library
        ```

    <span id="untyped_calls_exclude"></span>`untyped_calls_exclude`

    :    **类型：** 以逗号分隔的字符串列表

        选择性地排除特定包、模块和类中定义的函数和方法，以免受 [disallow_untyped_calls](#disallow_untyped_calls) 的影响。这也适用于包的所有子模块（即给定前缀中的所有内容）。请注意，此选项不支持逐文件配置，排除列表是针对你所有代码全局定义的。

    <span id="disallow_untyped_defs"></span>`disallow_untyped_defs`

    :    **类型：** 布尔值

        **默认值：** False

        禁止定义没有类型注解或类型注解不完整的函数（这是 [disallow_incomplete_defs](#disallow_incomplete_defs) 的超集）。

        例如，它会报告 `def f(a, b)` 和 `def f(a: int, b)` 的错误。

    <span id="disallow_incomplete_defs"></span>`disallow_incomplete_defs`

    :    **类型：** 布尔值

        **默认值：** False

        禁止定义具有不完整类型注解的函数，同时允许完全没有注解的定义。

        例如，它会报告 `def f(a: int, b)` 的错误，但不会对 `def f(a, b)` 报错。

    <span id="check_untyped_defs"></span>`check_untyped_defs`

    :    **类型：** 布尔值

        **默认值：** False

        对没有类型注解的函数内部进行类型检查。

    <span id="disallow_untyped_decorators"></span>`disallow_untyped_decorators`

    :    **类型：** 布尔值

        **默认值：** False

        每当一个带有类型注解的函数被一个没有注解的装饰器装饰时，报告一个错误。

=== "英文"

    For more information, see the [Untyped definitions and calls](./command_line.md#未类型化的定义与调用) section of the command line docs.

    <span id="disallow_untyped_calls"></span>`disallow_untyped_calls`

    :    **type:** boolean

        **default:** False

        Disallows calling functions without type annotations from functions with type annotations. Note that when used in per-module options, it enables/disables this check **inside** the module(s) specified, not for functions that come from that module(s), for example config like this:

        ```ini

        [mypy]
        disallow_untyped_calls = True

        [mypy-some.library.*]
        disallow_untyped_calls = False
        ```

        will disable this check inside ``some.library``, not for your code that imports ``some.library``. If you want to selectively disable this check for all your code that imports ``some.library`` you should instead use [untyped_calls_exclude](#untyped_calls_exclude), for example:

        ```ini

            [mypy]
            disallow_untyped_calls = True
            untyped_calls_exclude = some.library
        ```

    <span id="untyped_calls_exclude"></span>`untyped_calls_exclude`

    :    **type:** comma-separated list of strings

        Selectively excludes functions and methods defined in specific packages, modules, and classes from action of [disallow_untyped_calls](#disallow_untyped_calls). This also applies to all submodules of packages (i.e. everything inside a given prefix). Note, this option does not support per-file configuration, the exclusions list is defined globally for all your code.

    <span id="disallow_untyped_defs"></span>`disallow_untyped_defs`

    :    **type:** boolean

        **default:** False

        Disallows defining functions without type annotations or with incomplete type annotations (a superset of [disallow_incomplete_defs](#disallow_incomplete_defs)).

        For example, it would report an error for `def f(a, b)` and `def f(a: int, b)`.

    <span id="disallow_incomplete_defs"></span>`disallow_incomplete_defs`

    :    **type:** boolean

        **default:** False

        Disallows defining functions with incomplete type annotations, while still allowing entirely unannotated definitions.

        For example, it would report an error for `def f(a: int, b)` but not `def f(a, b)`.

    <span id="check_untyped_defs"></span>`check_untyped_defs`

    :    **type:** boolean

        **default:** False

        Type-checks the interior of functions without type annotations.

    <span id="disallow_untyped_decorators"></span>`disallow_untyped_decorators`

    :    **type:** boolean

        **default:** False

        Reports an error whenever a function with type annotations is decorated with a decorator without annotations.


## None 和 Optional 的处理

**None and Optional handling**

=== "中文"

    有关更多信息，请参见命令行文档的 [None 和 Optional 的处理](./command_line.md#none-和-optional-的处理) 部分。

    <span id="implicit_optional"></span>`implicit_optional`

    :    **类型：** 布尔值

        **默认值：** False

        使 mypy 将默认值为 ``None`` 的参数视为具有隐式的 [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) 类型。

        **注意：** 在 mypy 版本 0.980 及更早版本中，这个选项默认为 True。

    <span id="strict_optional"></span>`strict_optional`

    :    **类型：** 布尔值

        **默认值：** True

        实质上禁用对 :py:data:`~typing.Optional` 类型和 ``None`` 值的检查。启用此选项后，mypy 通常不会检查 ``None`` 值的使用——它被视为与所有类型兼容。

        !!! 警告

            ``strict_optional = false`` 是危险的。避免使用它，并且在不了解其作用的情况下绝对不要使用。

=== "英文"

    For more information, see the [None and Optional handling](./command_line.md#none-和-optional-的处理) section of the command line docs.

    <span id="implicit_optional"></span>`implicit_optional`

    :    **type:** boolean

        **default:** False

        Causes mypy to treat arguments with a ``None`` default value as having an implicit [Optional](https://docs.python.org/3/library/typing.html#typing.Optional) type.

        **Note:** This was True by default in mypy versions 0.980 and earlier.

    <span id="strict_optional"></span>`strict_optional`

    :    **type:** boolean

        **default:** True

        Effectively disables checking of :py:data:`~typing.Optional` types and ``None`` values. With this option, mypy doesn't generally check the use of ``None`` values -- it is treated as compatible with every type.

        !!! warning

            ``strict_optional = false`` is evil. Avoid using it and definitely do not use it without understanding what it does.


## 配置警告(warnings)

**Configuring warnings**

=== "中文"

    有关更多信息，请参见命令行文档的 [配置警告](#配置警告warnings) 部分。

    <span id="warn_redundant_casts"></span>`warn_redundant_casts`

    :    **类型：** 布尔值

        **默认值：** False

        当将一个表达式强制转换为其推断出的类型时发出警告。

        该选项只能在全局部分（``[mypy]``）设置。

    <span id="warn_unused_ignores"></span>`warn_unused_ignores`

    :    **类型：** 布尔值

        **默认值：** False

        当存在不需要的 ``# type: ignore`` 注释时发出警告。

    <span id="warn_no_return"></span>`warn_no_return`

    :    **类型：** 布尔值

        **默认值：** True

        显示某些执行路径上缺失返回语句的错误。

    <span id="warn_return_any"></span>`warn_return_any`

    :    **类型：** 布尔值

        **默认值：** False

        当从声明了非 ``Any`` 返回类型的函数中返回 ``Any`` 类型的值时显示警告。

    <span id="warn_unreachable"></span>`warn_unreachable`

    :    **类型：** 布尔值

        **默认值：** False

        当遇到经类型分析推断为不可达或冗余的代码时显示警告。

=== "英文"

    For more information, see the [Configuring warnings](#配置警告warnings) section of the command line docs.

    <span id="warn_redundant_casts"></span>`warn_redundant_casts`

    :    **type:** boolean

        **default:** False

        Warns about casting an expression to its inferred type.

        This option may only be set in the global section (``[mypy]``).

    <span id="warn_unused_ignores"></span>`warn_unused_ignores`

    :    **type:** boolean

        **default:** False

        Warns about unneeded ``# type: ignore`` comments.`

    <span id="warn_no_return"></span>`warn_no_return`

        **type:** boolean

        **default:** True

        Shows errors for missing return statements on some execution paths.

    <span id="warn_return_any"></span>`warn_return_any`

    :    **type:** boolean

        **default:** False

        Shows a warning when returning a value with type ``Any`` from a function declared with a non- ``Any`` return type.

    <span id="warn_unreachable"></span>`warn_unreachable`

    :    **type:** boolean

        **default:** False

        Shows a warning when encountering any code inferred to be unreachable or redundant after performing type analysis.


## 抑制错误

**Suppressing errors**

=== "中文"

    注意：这些配置选项仅在配置文件中可用。命令行选项中没有类似的选项。

    <span id="ignore_errors"></span>`ignore_errors`

    :    **类型：** 布尔值

        **默认值：** False

        忽略所有非致命错误。

=== "英文"

    Note: these configuration options are available in the config file only. There is no analog available via the command line options.

    <span id="ignore_errors"></span>`ignore_errors`

    :    **type:** boolean

        **default:** False

        Ignores all non-fatal errors.


## 其他严格性标志

**Miscellaneous strictness flags**

=== "中文"

    有关更多信息，请参见命令行文档中的 [其他严格性标志](./command_line.md#其他严格性标志) 部分。

    <span id="allow_untyped_globals"></span>`allow_untyped_globals`

    :    **类型：** 布尔值

        **默认值：** False

        使 mypy 忽略由于无法完全推断全局和类变量类型而引发的错误。

    <span id="allow_redefinition"></span>`allow_redefinition`

    :    **类型：** 布尔值

        **默认值：** False

        允许在相同的代码块和嵌套级别中使用任意类型重新定义变量。以下是一个有用的示例：

        ```python
        def process(items: list[str]) -> None:
            # 'items' 的类型是 list[str]
            items = [item.split() for item in items]
            # 现在 'items' 的类型是 list[list[str]]
        ```

        变量必须在重新定义之前被使用：

        ```python
        def process(items: list[str]) -> None:
            items = "mypy"  # 因为变量尚未被使用，这里重新定义为 str 是无效的
            print(items)
            items = "100"  # 有效，现在 items 的类型是 str
            items = int(items)  # 有效，现在 items 的类型是 int
        ```

    <span id="local_partial_types"></span>`local_partial_types`

    :    **类型：** 布尔值

        **默认值：** False

        不允许从不同作用域中的两个赋值推断变量的类型为 `None`。当使用 [mypy daemon](./mypy_daemon.md) 时，这总是会被隐式启用。

    <span id="disable_error_code"></span>`disable_error_code`

    :    **类型：** 以逗号分隔的字符串列表

        允许全局禁用一个或多个错误代码。

    <span id="enable_error_code"></span>`enable_error_code`

    :    **类型：** 以逗号分隔的字符串列表

        允许全局启用一个或多个错误代码。

        注意：此选项会覆盖 `disable_error_code` 选项中禁用的错误代码。

    <span id="implicit_reexport"></span>`implicit_reexport`

    :    **类型：** 布尔值

        **默认值：** True

        默认情况下，模块中的导入值会被视为导出，并允许其他模块导入它们。当此选项为 False 时，mypy 只会在使用 `from-as` 导入或在 `__all__` 中包含项时才重新导出。请注意，mypy 将存根文件视为总是禁用此功能。例如：

        ```python
        # 这不会重新导出该值
        from foo import bar
        # 这将重新导出为 bar，并允许其他模块导入它
        from foo import bar as bar
        # 这也会重新导出 bar
        from foo import bar
        __all__ = ['bar']
        ```

    <span id="strict_concatenate"></span>`strict_concatenate`

    :    **类型：** 布尔值

        **默认值：** False

        使通过 `Concatenate` 预处理的参数真正成为位置参数。

    <span id="strict_equality"></span>`strict_equality`

    :    **类型：** 布尔值

        **默认值：** False

        禁止对不重叠类型进行等式检查、身份检查和容器检查。

    <span id="strict"></span>`strict`

    :    **类型：** 布尔值

        **默认值：** False

        启用所有可选的错误检查标志。您可以在完整的 [mypy --help](./command_line.md#h) 输出中查看严格模式启用的标志列表。

        注意：`strict` 启用的标志列表可能会随着时间的推移而变化。

=== "英文"

    For more information, see the [Miscellaneous strictness flags](./command_line.md#其他严格性标志) section of the command line docs.

    <span id="allow_untyped_globals"></span>`allow_untyped_globals`

    :    **type:** boolean

        **default:** False

        Causes mypy to suppress errors caused by not being able to fully infer the types of global and class variables.

    <span id="allow_redefinition"></span>`allow_redefinition`

    :    **type:** boolean

        **default:** False

        Allows variables to be redefined with an arbitrary type, as long as the redefinition is in the same block and nesting level as the original definition. Example where this can be useful:

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

    <span id="local_partial_types"></span>`local_partial_types`

    :    **type:** boolean

        **default:** False

        Disallows inferring variable type for ``None`` from two assignments in different scopes. This is always implicitly enabled when using the [mypy daemon](./mypy_daemon.md).

    <span id="disable_error_code"></span>`disable_error_code`

    :    **type:** comma-separated list of strings

        Allows disabling one or multiple error codes globally.

    <span id="aaenable_error_codea"></span>`enable_error_code`

    :    **type:** comma-separated list of strings

        Allows enabling one or multiple error codes globally.

        Note: This option will override disabled error codes from the disable_error_code option.

    <span id="implicit_reexport"></span>`implicit_reexport`

    :    **type:** boolean

        **default:** True

        By default, imported values to a module are treated as exported and mypy allows other modules to import them. When false, mypy will not re-export unless the item is imported using from-as or is included in ``__all__``. Note that mypy treats stub files as if this is always disabled. For example:

        ```python
        # This won't re-export the value
        from foo import bar
        # This will re-export it as bar and allow other modules to import it
        from foo import bar as bar
        # This will also re-export bar
        from foo import bar
        __all__ = ['bar']
        ```

    <span id="strict_concatenate"></span>`strict_concatenate`

    :    **type:** boolean

        **default:** False

        Make arguments prepended via ``Concatenate`` be truly positional-only.

    <span id="strict_equality"></span>`strict_equality`

    :    **type:** boolean

        **default:** False

        Prohibit equality checks, identity checks, and container checks between non-overlapping types.

    <span id="strict"></span>`strict`

    :    **type:** boolean

        **default:** False

        Enable all optional error checking flags.  You can see the list of flags enabled by strict mode in the full [mypy --help](./command_line.md#h) output.

        Note: the exact list of flags enabled by [strict](#strict) may change over time.


## 配置错误消息

**Configuring error messages**

=== "中文"

    有关更多信息，请参见命令行文档中的 [配置错误消息](./command_line.md#配置错误消息) 部分。

    这些选项只能在全局部分（``[mypy]``）中设置。

    <span id="show_error_context"></span>`show_error_context`

    :    **类型：** 布尔值

        **默认值：** False

        在每个错误前添加相关的上下文信息。

    <span id="show_column_numbers"></span>`show_column_numbers`

    :    **类型：** 布尔值

        **默认值：** False

        在错误消息中显示列号。

    <span id="show_error_code_links"></span>`show_error_code_links`

    :    **类型：** 布尔值

        **默认值：** False

        显示指向相应错误代码的文档链接。

    <span id="hide_error_codes"></span>`hide_error_codes`

    :    **类型：** 布尔值

        **默认值：** False

        在错误消息中隐藏错误代码。有关更多信息，请参见 [错误代码](../mypy_other/error_codes.md)。

    <span id="pretty"></span>`pretty`

    :    **类型：** 布尔值

        **默认值：** False

        在错误消息中使用更美观的输出：使用软换行，显示源代码片段，并显示错误位置标记。

    <span id="color_output"></span>`color_output`

    :    **类型：** 布尔值

        **默认值：** True

        显示启用颜色的错误消息。

    <span id="error_summary"></span>`error_summary`

    :    **类型：** 布尔值

        **默认值：** True

        在错误消息后显示简短的总结行。

    <span id="show_absolute_path"></span>`show_absolute_path`

    :    **类型：** 布尔值

        **默认值：** False

        显示文件的绝对路径。

    <span id="force_uppercase_builtins"></span>`force_uppercase_builtins`

    :    **类型：** 布尔值

        **默认值：** False

        在错误消息中始终使用 ``List`` 而不是 ``list``，即使在 Python 3.9 及以上版本中也是如此。

    <span id="force_union_syntax"></span>`force_union_syntax`

    :    **类型：** 布尔值

        **默认值：** False

        在错误消息中始终使用 ``Union[]`` 和 ``Optional[]`` 来表示联合类型（而不是 ``|`` 运算符），即使在 Python 3.10 及以上版本中也是如此。

=== "英文"

    For more information, see the [Configuring error messages](./command_line.md#配置错误消息) section of the command line docs.

    These options may only be set in the global section (``[mypy]``).

    <span id="show_error_context"></span>`show_error_context`

    :    **type:** boolean

        **default:** False

        Prefixes each error with the relevant context.

    <span id="show_column_numbers"></span>`show_column_numbers`

    :    **type:** boolean

        **default:** False

        Shows column numbers in error messages.

    <span id="show_error_code_links"></span>`show_error_code_links`

    :    **type:** boolean

        **default:** False

        Shows documentation link to corresponding error code.

    <span id="hide_error_codes"></span>`hide_error_codes`

    :    **type:** boolean

        **default:** False

        Hides error codes in error messages. See [Error codes](../mypy_other/error_codes.md) for more information.

    <span id="pretty"></span>`pretty`

    :    **type:** boolean

        **default:** False

        Use visually nicer output in error messages: use soft word wrap, show source code snippets, and show error location markers.

    <span id="color_output"></span>`color_output`

    :    **type:** boolean

        **default:** True

        Shows error messages with color enabled.

    <span id="error_summary"></span>`error_summary`

    :    **type:** boolean

        **default:** True

        Shows a short summary line after error messages.

    <span id="show_absolute_path"></span>`show_absolute_path`

    :    **type:** boolean

        **default:** False

        Show absolute paths to files.

    <span id="force_uppercase_builtins"></span>`force_uppercase_builtins`

    :    **type:** boolean

        **default:** False

        Always use ``List`` instead of ``list`` in error messages, even on Python 3.9+.

    <span id="force_union_syntax"></span>`force_union_syntax`

    :    **type:** boolean

        **default:** False

        Always use ``Union[]`` and ``Optional[]`` for union types in error messages (instead of the ``|`` operator), even on Python 3.10+.

## 增量模式

**Incremental mode**

=== "中文"

    这些选项只能在全局部分（``[mypy]``）中设置。

    <span id="incremental"></span>`incremental`

    :    **类型：** 布尔值

        **默认值：** True

        启用 [增量模式](./command_line.md#增量模式)。

    <span id="cache_dir"></span>`cache_dir`

    :    **类型：** 字符串

        **默认值：** ``.mypy_cache``

        指定 mypy 存储增量缓存信息的位置。用户主目录和环境变量会被展开。此设置会被 ``MYPY_CACHE_DIR`` 环境变量覆盖。

        请注意，缓存仅在启用增量模式时读取，但总是会写入，除非值设置为 ``/dev/null``（UNIX）或 ``nul``（Windows）。

    <span id="sqlite_cache"></span>`sqlite_cache`

    :    **类型：** 布尔值

        **默认值：** False

        使用 [SQLite](https://www.sqlite.org/) 数据库来存储缓存。

    <span id="cache_fine_grained"></span>`cache_fine_grained`

    :    **类型：** 布尔值

        **默认值：** False

        在缓存中包含精细化的依赖信息，以供 mypy 守护进程使用。

    <span id="skip_version_check"></span>`skip_version_check`

    :    **类型：** 布尔值

        **默认值：** False

        使 mypy 使用增量缓存数据，即使这些数据是由不同版本的 mypy 生成的。（默认情况下，mypy 会执行版本检查，并在缓存由旧版本的 mypy 写入时重新生成缓存。）

    <span id="skip_cache_mtime_checks"></span>`skip_cache_mtime_checks`

    :    **类型：** 布尔值

        **默认值：** False

        跳过基于 mtime 的缓存内部一致性检查。

=== "英文"

    These options may only be set in the global section (``[mypy]``).

    <span id="incremental"></span>`incremental`

    :    **type:** boolean

        **default:** True

        Enables [incremental mode](./command_line.md#增量模式).

    <span id="cache_dir"></span>`cache_dir`

    :    **type:** string

        **default:** ``.mypy_cache``

        Specifies the location where mypy stores incremental cache info. User home directory and environment variables will be expanded. This setting will be overridden by the ``MYPY_CACHE_DIR`` environment variable.

        Note that the cache is only read when incremental mode is enabled but is always written to, unless the value is set to ``/dev/null`` (UNIX) or ``nul`` (Windows).

    <span id="sqlite_cache"></span>`sqlite_cache`

    :    **type:** boolean

        **default:** False

        Use an [SQLite](https://www.sqlite.org/) database to store the cache.

    <span id="cache_fine_grained"></span>`cache_fine_grained`

    :    **type:** boolean

        **default:** False

        Include fine-grained dependency information in the cache for the mypy daemon.

    <span id="skip_version_check"></span>`skip_version_check`

    :    **type:** boolean

        **default:** False

        Makes mypy use incremental cache data even if it was generated by a different version of mypy. (By default, mypy will perform a version check and regenerate the cache if it was written by older versions of mypy.)

    <span id="skip_cache_mtime_checks"></span>`skip_cache_mtime_checks`

    :    **type:** boolean

        **default:** False

        Skip cache internal consistency checks based on mtime.


## 高级选项

**Advanced options**

=== "中文"

    这些选项只能在全局部分（``[mypy]``）中设置。

    <span id="plugins"></span>`plugins`

    :    **类型：** 以逗号分隔的字符串列表

        一个以逗号分隔的 mypy 插件列表。请参阅 [使用插件扩展 mypy](./extending_mypy.md#使用插件扩展-mypy)。

    <span id="pdb"></span>`pdb`

    :    **类型：** 布尔值

        **默认值：** False

        在发生致命错误时调用 [pdb](https://docs.python.org/3/library/pdb.html#module-pdb)。

    <span id="show_traceback"></span>`show_traceback`

    :    **类型：** 布尔值

        **默认值：** False

        在致命错误发生时显示回溯信息。

    <span id="raise_exceptions"></span>`raise_exceptions`

    :    **类型：** 布尔值

        **默认值：** False

        在发生致命错误时引发异常。

    <span id="custom_typing_module"></span>`custom_typing_module`

    :    **类型：** 字符串

        指定一个自定义模块，用作 :py:mod:`typing` 模块的替代品。

    <span id="custom_typeshed_dir"></span>`custom_typeshed_dir`

    :    **类型：** 字符串

        指定 mypy 查找标准库 typeshed 存根的目录，而不是 mypy 附带的 typeshed。这主要用于在提交更改到上游之前测试 typeshed 更改，但也允许使用 forked 版本的 typeshed。

        用户主目录和环境变量会被展开。

        请注意，这不会影响第三方库的存根。要测试第三方存根，例如可以尝试 ``MYPYPATH=stubs/six mypy ...``。

    <span id="warn_incomplete_stub"></span>`warn_incomplete_stub`

    :    **类型：** 布尔值

        **默认值：** False

        对 typeshed 中缺少类型注解发出警告。这仅在与 [disallow_untyped_defs](#disallow_untyped_defs) 或 [disallow_incomplete_defs](#disallow_incomplete_defs) 结合使用时才相关。

=== "英文"

    These options may only be set in the global section (``[mypy]``).

    <span id="plugins"></span>`plugins`

    :    **type:** comma-separated list of strings

        A comma-separated list of mypy plugins. See [Extending mypy using plugins](./extending_mypy.md#使用插件扩展-mypy).

    <span id="pdb"></span>`pdb`

    :    **type:** boolean

        **default:** False

        Invokes [pdb](https://docs.python.org/3/library/pdb.html#module-pdb) on fatal error.

    <span id="show_traceback"></span>`show_traceback`

    :    **type:** boolean

        **default:** False

        Shows traceback on fatal error.

    <span id="raise_exceptions"></span>`raise_exceptions`

    :    **type:** boolean

        **default:** False

        Raise exception on fatal error.

    <span id="custom_typing_module"></span>`custom_typing_module`

    :    **type:** string

        Specifies a custom module to use as a substitute for the :py:mod:`typing` module.

    <span id="custom_typeshed_dir"></span>`custom_typeshed_dir`

    :    **type:** string

        This specifies the directory where mypy looks for standard library typeshed stubs, instead of the typeshed that ships with mypy.  This is primarily intended to make it easier to test typeshed changes before submitting them upstream, but also allows you to use a forked version of typeshed.

        User home directory and environment variables will be expanded.

        Note that this doesn't affect third-party library stubs. To test third-party stubs, for example try ``MYPYPATH=stubs/six mypy ...``.

    <span id="warn_incomplete_stub"></span>`warn_incomplete_stub`

    :    **type:** boolean

        **default:** False

        Warns about missing type annotations in typeshed.  This is only relevant in combination with [disallow_untyped_defs](#disallow_untyped_defs) or [disallow_incomplete_defs](#disallow_incomplete_defs).


## 报告生成

**Report generation**

=== "中文"

    如果设置了这些选项，mypy 将生成指定格式的报告到指定目录。

    !!! warning

        生成报告会禁用增量模式，并可能显著减慢工作流程。建议仅在特定运行（例如 CI 中）启用报告功能。

    <span id="any_exprs_report"></span>`any_exprs_report`

    :    **类型：** 字符串

        使 mypy 生成一个文本文件报告，记录代码库中存在多少个类型为 ``Any`` 的表达式。

    <span id="cobertura_xml_report"></span>`cobertura_xml_report`

    :    **类型：** 字符串

        使 mypy 生成一个 Cobertura XML 类型检查覆盖率报告。

        要生成此报告，必须手动安装 [lxml] 库或通过 setuptools 额外选项 ``mypy[reports]`` 指定 mypy 安装。

    <span id="html_report-xslt_html_report"></span>`html_report / xslt_html_report`

    :    **类型：** 字符串

        使 mypy 生成一个 HTML 类型检查覆盖率报告。

        要生成此报告，必须手动安装 [lxml] 库或通过 setuptools 额外选项 ``mypy[reports]`` 指定 mypy 安装。

    <span id="linecount_report"></span>`linecount_report`

    :    **类型：** 字符串

        使 mypy 生成一个文本文件报告，记录代码库中已类型化和未类型化的函数及其行数。

    <span id="linecoverage_report"></span>`linecoverage_report`

    :    **类型：** 字符串

        使 mypy 生成一个 JSON 文件，将每个源文件的绝对文件名映射到属于该文件中类型化函数的行号列表。

    <span id="lineprecision_report"></span>`lineprecision_report`

    :    **类型：** 字符串

        使 mypy 生成一个平面文本文件报告，包含每个模块的统计信息，如有多少行被类型检查等。

    <span id="txt_report-xslt_txt_report"></span>`txt_report / xslt_txt_report`

    :    **类型：** 字符串

        使 mypy 生成一个文本文件类型检查覆盖率报告。

        要生成此报告，必须手动安装 [lxml] 库或通过 setuptools 额外选项 ``mypy[reports]`` 指定 mypy 安装。

    <span id="xml_report"></span>`xml_report`

    :    **类型：** 字符串

        使 mypy 生成一个 XML 类型检查覆盖率报告。

        要生成此报告，必须手动安装 [lxml] 库或通过 setuptools 额外选项 ``mypy[reports]`` 指定 mypy 安装。

=== "英文"

    If these options are set, mypy will generate a report in the specified format into the specified directory.

    !!! warning

        Generating reports disables incremental mode and can significantly slow down your workflow. It is recommended to enable reporting only for specific runs (e.g. in CI).

    <span id="any_exprs_report"></span>`any_exprs_report`

    :    **type:** string

        Causes mypy to generate a text file report documenting how many expressions of type ``Any`` are present within your codebase.

    <span id="cobertura_xml_report"></span>`cobertura_xml_report`

    :    **type:** string

        Causes mypy to generate a Cobertura XML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="html_report-xslt_html_report"></span>`html_report / xslt_html_report`

    :    **type:** string

        Causes mypy to generate an HTML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="linecount_report"></span>`linecount_report`

    :    **type:** string

        Causes mypy to generate a text file report documenting the functions and lines that are typed and untyped within your codebase.

    <span id="linecoverage_report"></span>`linecoverage_report`

    :    **type:** string

        Causes mypy to generate a JSON file that maps each source file's absolute filename to a list of line numbers that belong to typed functions in that file.

    <span id="lineprecision_report"></span>`lineprecision_report`

    :    **type:** string

        Causes mypy to generate a flat text file report with per-module statistics of how many lines are typechecked etc.

    <span id="txt_report-xslt_txt_report"></span>`txt_report / xslt_txt_report`

    :    **type:** string

        Causes mypy to generate a text file type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.

    <span id="xml_report"></span>`xml_report`

    :    **type:** string

        Causes mypy to generate an XML type checking coverage report.

        To generate this report, you must either manually install the [lxml] library or specify mypy installation with the setuptools extra ``mypy[reports]``.


## 其他

**Miscellaneous**

=== "中文"

    这些选项只能在全局部分（``[mypy]``）中设置。

    <span id="junit_xml"></span>`junit_xml`

    :    **类型：** 字符串

        使 mypy 生成一个 JUnit XML 测试结果文档，包含类型检查的结果。这有助于将 mypy 与持续集成（CI）工具集成。

    <span id="scripts_are_modules"></span>`scripts_are_modules`

    :    **类型：** 布尔值

        **默认值：** False

        使脚本 ``x`` 变成模块 ``x`` 而不是 ``__main__``。在一次运行中检查多个脚本时，这个选项很有用。

    <span id="warn_unused_configs"></span>`warn_unused_configs`

    :    **类型：** 布尔值

        **默认值：** False

        发出警告，指出配置文件中未匹配到任何被 mypy 处理的文件的每个模块部分。（这要求通过设置 [incremental = False](#incremental) 关闭增量模式。）

    <span id="verbosity"></span>`verbosity`

    :    **类型：** 整数

        **默认值：** 0

        控制生成的调试输出量。数值越高，输出越详细。

=== "英文"

    These options may only be set in the global section (``[mypy]``).

    <span id="junit_xml"></span>`junit_xml`

    :    **type:** string

        Causes mypy to generate a JUnit XML test result document with type checking results. This can make it easier to integrate mypy with continuous integration (CI) tools.

    <span id="scripts_are_modules"></span>`scripts_are_modules`

    :    **type:** boolean

        **default:** False

        Makes script ``x`` become module ``x`` instead of ``__main__``.  This is useful when checking multiple scripts in a single run.

    <span id="warn_unused_configs"></span>`warn_unused_configs`

    :    **type:** boolean

        **default:** False

        Warns about per-module sections in the config file that do not match any files processed when invoking mypy. (This requires turning off incremental mode using [incremental = False](#incremental).)

    <span id="verbosity"></span>`verbosity`

    :    **type:** integer

        **default:** 0

        Controls how much debug output will be generated.  Higher numbers are more verbose.


## 使用 pyproject.toml 文件

**Using a pyproject.toml file**

=== "中文"

    除了使用 ``mypy.ini`` 文件之外，还可以使用 ``pyproject.toml`` 文件（如 [PEP 518](https://www.python.org/dev/peps/pep-0518/) 中指定的）。关于使用 ``pyproject.toml`` 文件的一些注意事项：

    * ``[mypy]`` 部分的名称应加上 ``tool.`` 前缀：

        * 即，``[mypy]`` 应变成 ``[tool.mypy]``

    * 模块特定的部分应移到 ``[[tool.mypy.overrides]]`` 部分：

        * 例如，``[mypy-packagename]`` 应变成：

    ```toml
    [[tool.mypy.overrides]]
    module = 'packagename'
    ...
    ```

    * 多模块特定的部分可以移到一个 ``[[tool.mypy.overrides]]`` 部分，并将模块属性设置为模块数组：

        * 例如，``[mypy-packagename,packagename2]`` 应变成：

    ```toml
    [[tool.mypy.overrides]]
    module = [
        'packagename',
        'packagename2'
    ]
    ...
    ```

    * 在 ``pyproject.toml`` 文件中的值应特别注意，与 ``ini`` 文件相比：

        * 字符串必须用双引号括起来，如果字符串包含特殊字符也可以用单引号

        * 布尔值应全部小写

    有关 ``toml`` 文件允许的内容的更多详细信息，请参见 [TOML 文档](https://toml.io/)。有关 ``pyproject.toml`` 文件的布局和结构的更多信息，请参见 [PEP 518](https://www.python.org/dev/peps/pep-0518/)。

=== "英文"

    Instead of using a ``mypy.ini`` file, a ``pyproject.toml`` file (as specified by [PEP 518](https://www.python.org/dev/peps/pep-0518/)) may be used instead. A few notes on doing so:

    * The ``[mypy]`` section should have ``tool.`` prepended to its name:

        * I.e., ``[mypy]`` would become ``[tool.mypy]``

    * The module specific sections should be moved into ``[[tool.mypy.overrides]]`` sections:

        * For example, ``[mypy-packagename]`` would become:

    ```toml
    [[tool.mypy.overrides]]
    module = 'packagename'
    ...
    ```

    * Multi-module specific sections can be moved into a single ``[[tool.mypy.overrides]]`` section with a module property set to an array of modules:

        * For example, ``[mypy-packagename,packagename2]`` would become:

    ```toml
    [[tool.mypy.overrides]]
    module = [
        'packagename',
        'packagename2'
    ]
    ...
    ```

    * The following care should be given to values in the ``pyproject.toml`` files as compared to ``ini`` files:

        * Strings must be wrapped in double quotes, or single quotes if the string contains special characters

        * Boolean values should be all lower case

    Please see the [TOML Documentation](https://toml.io/)` for more details and information on what is allowed in a ``toml`` file. See [PEP 518](https://www.python.org/dev/peps/pep-0518/) for more information on the layout and structure of the ``pyproject.toml`` file.

## 示例 ``pyproject.toml``

**Example ``pyproject.toml``**

=== "中文"

    以下是一个 ``pyproject.toml`` 文件的示例。要使用此配置文件，请将其放置在您的仓库根目录下（或将其追加到现有 ``pyproject.toml`` 文件的末尾），然后运行 mypy。

    ```toml
    # mypy 全局选项：

    [tool.mypy]
    python_version = "2.7"
    warn_return_any = true
    warn_unused_configs = true
    exclude = [
        '^file1\.py$',  # TOML 字符串文字（单引号，不需要转义）
        "^file2\\.py$",  # TOML 基本字符串（双引号，反斜杠和其他字符需要转义）
    ]

    # mypy 模块特定选项：

    [[tool.mypy.overrides]]
    module = "mycode.foo.*"
    disallow_untyped_defs = true

    [[tool.mypy.overrides]]
    module = "mycode.bar"
    warn_return_any = false

    [[tool.mypy.overrides]]
    module = [
        "somelibrary",
        "some_other_library"
    ]
    ignore_missing_imports = true
    ```

=== "英文"

    Here is an example of a ``pyproject.toml`` file. To use this config file, place it at the root of your repo (or append it to the end of an existing ``pyproject.toml`` file) and run mypy.

    ```toml

        # mypy global options:

        [tool.mypy]
        python_version = "2.7"
        warn_return_any = true
        warn_unused_configs = true
        exclude = [
            '^file1\.py$',  # TOML literal string (single-quotes, no escaping necessary)
            "^file2\\.py$",  # TOML basic string (double-quotes, backslash and other characters need escaping)
        ]

        # mypy per-module options:

        [[tool.mypy.overrides]]
        module = "mycode.foo.*"
        disallow_untyped_defs = true

        [[tool.mypy.overrides]]
        module = "mycode.bar"
        warn_return_any = false

        [[tool.mypy.overrides]]
        module = [
            "somelibrary",
            "some_other_library"
        ]
        ignore_missing_imports = true
    ```

[lxml]: https://pypi.org/project/lxml/
[SQLite]: https://www.sqlite.org/
[PEP 518]: https://www.python.org/dev/peps/pep-0518/
[TOML Documentation]: https://toml.io/

