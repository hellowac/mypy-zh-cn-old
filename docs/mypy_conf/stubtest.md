# 自动存根测试 (stubtest)

**Automatic stub testing (stubtest)**

=== "中文"

    存根文件是包含类型注释的文件。有关更多动机和细节，请参见 [PEP 484](https://www.python.org/dev/peps/pep-0484/#stub-files)。

    存根文件的一个常见问题是，它们往往与实际实现有所偏离。Mypy 包含了 ``stubtest`` 工具，可以在运行时自动检查存根与实现之间的差异。

=== "英文"

    Stub files are files containing type annotations. See [PEP 484](https://www.python.org/dev/peps/pep-0484/#stub-files) for more motivation and details.

    A common problem with stub files is that they tend to diverge from the actual implementation. Mypy includes the ``stubtest`` tool that can automatically check for discrepancies between the stubs and the implementation at runtime.

## stubtest 能做什么和不能做什么

**What stubtest does and does not do**

=== "中文"

    `stubtest` 将导入你的代码并在运行时通过使用 [inspect](https://docs.python.org/3/library/inspect.html#module-inspect) 模块的功能来进行对象检查。然后，`stubtest` 将分析存根文件，并将两者进行比较，指出存根与实际实现之间的差异。

    需要注意的是，这种比较有其局限性。`stubtest` 不会尝试静态分析你的实际代码，而仅依赖动态运行时检查（特别是这种方法意味着 `stubtest` 对扩展模块的工作效果良好）。然而，这也意味着 `stubtest` 的可见性有限；例如，它无法判断函数的返回类型在存根中是否被准确地标注。

    为了明确，以下是一些 `stubtest` 无法做到的事情：

    * 对你的代码进行类型检查 — 使用 `mypy` 替代
    * 生成存根 — 使用 `stubgen` 或 `pyright --createstub` 替代
    * 基于运行你的应用程序或测试套件生成存根 — 使用 `monkeytype` 替代
    * 将存根应用到代码中生成内联类型 — 使用 `retype` 或 `libcst` 替代

    总的来说，`stubtest` 非常适合确保存根与实现之间的基本一致性或检查存根的完整性。它被用来测试 Python 官方的库存根集合 [typeshed](https://github.com/python/typeshed)。

    !!! 警告

        `stubtest` 会导入并执行它检查的包中的 Python 代码。

=== "英文"

    Stubtest will import your code and introspect your code objects at runtime, for example, by using the capabilities of the [inspect](https://docs.python.org/3/library/inspect.html#module-inspect) module. Stubtest will then analyse the stub files, and compare the two, pointing out things that differ between stubs and the implementation at runtime.

    It's important to be aware of the limitations of this comparison. Stubtest will not make any attempt to statically analyse your actual code and relies only on dynamic runtime introspection (in particular, this approach means stubtest works well with extension modules). However, this means that stubtest has limited visibility; for instance, it cannot tell if a return type of a function is accurately typed in the stubs.

    For clarity, here are some additional things stubtest can't do:

    * Type check your code -- use ``mypy`` instead
    * Generate stubs -- use ``stubgen`` or ``pyright --createstub`` instead
    * Generate stubs based on running your application or test suite -- use ``monkeytype`` instead
    * Apply stubs to code to produce inline types -- use ``retype`` or ``libcst`` instead

    In summary, stubtest works very well for ensuring basic consistency between stubs and implementation or to check for stub completeness. It's used to test Python's official collection of library stubs, [typeshed](https://github.com/python/typeshed).

    !!! warning

        stubtest will import and execute Python code from the packages it checks.

## 示例

**Example**

=== "中文"

    以下是 `stubtest` 可以做的一个快速示例：

    ```shell
    $ python3 -m pip install mypy

    $ cat library.py
    x = "hello, stubtest"

    def foo(x=None):
        print(x)

    $ cat library.pyi
    x: int

    def foo(x: int) -> None: ...

    $ python3 -m mypy.stubtest library
    error: library.foo 与运行时不一致，运行时参数 "x" 有默认值，但存根中的参数没有
    存根: 第 3 行
    def (x: builtins.int)
    运行时: 在文件 ~/library.py:3
    def (x=None)

    error: library.x 变量与运行时类型 Literal['hello, stubtest'] 不一致
    存根: 第 1 行
    builtins.int
    运行时:
    'hello, stubtest'
    ```

=== "英文"

    Here's a quick example of what stubtest can do:

    ```shell
    $ python3 -m pip install mypy

    $ cat library.py
    x = "hello, stubtest"

    def foo(x=None):
        print(x)

    $ cat library.pyi
    x: int

    def foo(x: int) -> None: ...

    $ python3 -m mypy.stubtest library
    error: library.foo is inconsistent, runtime argument "x" has a default value but stub argument does not
    Stub: at line 3
    def (x: builtins.int)
    Runtime: in file ~/library.py:3
    def (x=None)

    error: library.x variable differs from runtime type Literal['hello, stubtest']
    Stub: at line 1
    builtins.int
    Runtime:
    'hello, stubtest'
    ```

## 使用方法

**Usage**

=== "中文"

    运行 `stubtest` 可以简单地通过 `stubtest module_to_check` 来完成。运行 `stubtest --help` 可以获取选项的简要说明。

    `stubtest` 需要能够导入要检查的代码，因此请确保 `mypy` 已安装在与要测试的库相同的环境中。在某些情况下，设置 `PYTHONPATH` 可以帮助 `stubtest` 找到要导入的代码。

    类似地，`stubtest` 也需要能够找到要检查的存根文件。`stubtest` 会尊重 `MYPYPATH` 环境变量——如果遇到“未能找到存根”之类的错误，可以考虑使用这个变量。

    请注意，`stubtest` 需要 `mypy` 能够分析存根。如果 `mypy` 无法分析存根，可能会出现“由于 mypy 构建错误未检查存根”之类的错误。在这种情况下，需要解决这些错误才能运行 `stubtest`。尽管这里可能存在错误的重叠，但 `stubtest` 并不打算替代直接运行 `mypy`。

    如果希望忽略一些 `stubtest` 的警告，`stubtest` 支持一个相当有用的允许列表系统。

    本节其余部分描述了 `stubtest` 的命令行接口。

    <span id="concise"></span>`--concise`

    :    使 `stubtest` 的输出更简洁，每个错误一行

    <span id="ignore-missing-stub"></span>`--ignore-missing-stub`

    :    忽略存根缺少运行时存在的内容的错误

    <span id="ignore-positional-only"></span>`--ignore-positional-only`

    :    忽略参数是否应为位置参数的错误

    <span id="allowlist"></span>`--allowlist FILE`

    :    使用文件作为允许列表。可以多次传递以组合多个允许列表。允许列表可以使用 `--generate-allowlist` 创建。允许列表支持正则表达式。

        允许列表中的条目表示 `stubtest` 不会对对应的定义生成任何错误。

    <span id="generate-allowlist"></span>`--generate-allowlist`

    :    打印允许列表（到标准输出），以便与 `--allowlist` 一起使用

        当将 `stubtest` 引入现有项目时，这是消除所有现有错误的简单方法。

    <span id="ignore-unused-allowlist"></span>`--ignore-unused-allowlist`

    :    忽略未使用的允许列表条目

        启用此选项之前的默认行为是如果允许列表条目对 `stubtest` 成功通过没有必要，`stubtest` 会发出警告。

        请注意，如果允许列表条目是匹配空字符串的正则表达式，`stubtest` 将永远不会将其视为未使用。例如，要为单个允许列表条目如 `foo.bar` 获取 `--ignore-unused-allowlist` 行为，可以添加允许列表条目 `(foo\.bar)?`。这在错误仅在特定平台上出现时可能会很有用。

    <span id="mypy-config-file"></span>`--mypy-config-file FILE`

    :    使用指定的 mypy 配置文件来确定 mypy 插件和 mypy 路径

    <span id="custom-typeshed-dir"></span>`--custom-typeshed-dir DIR`

    :    使用 DIR 中的自定义 typeshed

    <span id="check-typeshed"></span>`--check-typeshed`

    :    检查 typeshed 中的所有标准库模块

    <span id="help"></span>`--help`

    :    显示帮助信息 :-)

=== "英文"

    Running stubtest can be as simple as ``stubtest module_to_check``. Run [stubtest --help](#help) for a quick summary of options.

    Stubtest must be able to import the code to be checked, so make sure that mypy is installed in the same environment as the library to be tested. In some cases, setting ``PYTHONPATH`` can help stubtest find the code to import.

    Similarly, stubtest must be able to find the stubs to be checked. Stubtest respects the ``MYPYPATH`` environment variable -- consider using this if you receive a complaint along the lines of "failed to find stubs".

    Note that stubtest requires mypy to be able to analyse stubs. If mypy is unable to analyse stubs, you may get an error on the lines of "not checking stubs due to mypy build errors". In this case, you will need to mitigate those errors before stubtest will run. Despite potential overlap in errors here, stubtest is not intended as a substitute for running mypy directly.

    If you wish to ignore some of stubtest's complaints, stubtest supports a pretty handy allowlist system.

    The rest of this section documents the command line interface of stubtest.

    <span id="concise"></span>`--concise`

    :    Makes stubtest's output more concise, one line per error

    <span id="ignore-missing-stub"></span>`--ignore-missing-stub`

    :    Ignore errors for stub missing things that are present at runtime

    <span id="ignore-positional-only"></span>`--ignore-positional-only`

    :    Ignore errors for whether an argument should or shouldn't be positional-only

    <span id="allowlist"></span>`--allowlist FILE`

    :    Use file as an allowlist. Can be passed multiple times to combine multiple allowlists. Allowlists can be created with --generate-allowlist. Allowlists support regular expressions.

        The presence of an entry in the allowlist means stubtest will not generate any errors for the corresponding definition.

    <span id="generate-allowlist"></span>`--generate-allowlist`

    :    Print an allowlist (to stdout) to be used with --allowlist

        When introducing stubtest to an existing project, this is an easy way to silence all existing errors.

    <span id="ignore-unused-allowlist"></span>`--ignore-unused-allowlist`

    :    Ignore unused allowlist entries

        Without this option enabled, the default is for stubtest to complain if an allowlist entry is not necessary for stubtest to pass successfully.

        Note if an allowlist entry is a regex that matches the empty string, stubtest will never consider it unused. For example, to get `--ignore-unused-allowlist` behaviour for a single allowlist entry like ``foo.bar`` you could add an allowlist entry ``(foo\.bar)?``. This can be useful when an error only occurs on a specific platform.

    <span id="mypy-config-file"></span>`--mypy-config-file FILE`

    :    Use specified mypy config file to determine mypy plugins and mypy path

    <span id="custom-typeshed-dir"></span>`--custom-typeshed-dir DIR`

    :    Use the custom typeshed in DIR

    <span id="check-typeshed"></span>`--check-typeshed`

    :    Check all stdlib modules in typeshed

    <span id="help"></span>`--help`

    :    Show a help message :-)
