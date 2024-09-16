# Mypy 守护进程 (mypy server)

**Mypy daemon (mypy server)**

=== "中文"

    除了将 mypy 作为命令行工具运行外，你还可以将其作为一个长期运行的守护进程（服务器）来运行，并使用命令行客户端将类型检查请求发送到服务器。这样，mypy 可以更快地执行类型检查，因为从以前运行中缓存的程序状态保存在内存中，无需在每次运行时从文件系统读取。服务器还使用更细粒度的依赖追踪来减少需要完成的工作量。

    如果你有一个大型代码库需要检查，使用 mypy 守护进程运行 mypy 可以比常规的命令行 ``mypy`` 工具快 *10 倍或更多*，特别是当你的工作流程涉及在小编辑后重复运行 mypy 时——这通常是一个好主意，因为这样你会更早发现错误。

    !!! 注意

        mypy 守护进程的命令行接口可能会在未来的 mypy 版本中发生变化。

    !!! 注意

        每个 mypy 守护进程实例支持一个用户和一组源文件，并且一次只能处理一个类型检查请求。你可以运行多个 mypy 守护进程实例来检查多个代码库。

=== "英文"

    Instead of running mypy as a command-line tool, you can also run it as a long-running daemon (server) process and use a command-line client to send type-checking requests to the server.  This way mypy can perform type checking much faster, since program state cached from previous runs is kept in memory and doesn't have to be read from the file system on each run. The server also uses finer-grained dependency tracking to reduce the amount of work that needs to be done.

    If you have a large codebase to check, running mypy using the mypy daemon can be *10 or more times faster* than the regular command-line ``mypy`` tool, especially if your workflow involves running mypy repeatedly after small edits -- which is often a good idea, as this way you'll find errors sooner.

    !!! note 

        The command-line interface of mypy daemon may change in future mypy releases.

    !!! note 

        Each mypy daemon process supports one user and one set of source files, and it can only process one type checking request at a time. You can run multiple mypy daemon processes to type check multiple repositories.


## 基本用法

**Basic usage**

=== "中文"

    客户端工具 `dmypy` 用于控制 mypy 守护进程。使用 `dmypy run -- <flags> <files>` 来检查一组文件（或目录）的类型。如果守护进程未运行，这条命令将启动守护进程。你可以在 `--` 后面使用几乎任意的 mypy 标志。守护进程将始终在当前主机上运行。例如：

    ```shell
    dmypy run -- prog.py pkg/*.py
    ```

    `dmypy run` 会在配置或 mypy 版本更改时自动重启守护进程。

    初次运行会处理所有代码，可能需要一些时间才能完成，但后续运行将会很快，特别是如果你只修改了几个文件的话。（你可以使用 [远程缓存](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行) 来加速初次运行。如果你有一个大型代码库，这种加速可能会非常显著。）

    !!! 注意

        Mypy 0.780 版本增加了对 `dmypy` 中跟踪导入的支持（默认启用）。这个功能仍处于实验阶段。你可以使用 `--follow-imports=skip` 或 `--follow-imports=error` 来回退到稳定功能。有关这些功能的详细信息，请参见 :ref:`follow-imports`。

    !!! 注意

        mypy 守护进程需要 `--local-partial-types` 选项，并会自动启用该选项。
=== "英文"

    The client utility ``dmypy`` is used to control the mypy daemon. Use ``dmypy run -- <flags> <files>`` to type check a set of files (or directories). This will launch the daemon if it is not running. You can use almost arbitrary mypy flags after ``--``.  The daemon will always run on the current host. Example

    ```shell
    dmypy run -- prog.py pkg/*.py
    ```

    ``dmypy run`` will automatically restart the daemon if the configuration or mypy version changes.

    The initial run will process all the code and may take a while to finish, but subsequent runs will be quick, especially if you've only changed a few files. (You can use [remote caching](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行) to speed up the initial run. The speedup can be significant if you have a large codebase.)

    !!! note 

        Mypy 0.780 added support for following imports in dmypy (enabled by default). This functionality is still experimental. You can use ``--follow-imports=skip`` or ``--follow-imports=error`` to fall back to the stable functionality.  See :ref:`follow-imports` for details on how these work.

    !!! note 

        The mypy daemon requires ``--local-partial-types`` and automatically enables it.


## 守护进程客户端命令

**Daemon client commands**

=== "中文"

    虽然 `dmypy run` 对大多数用途来说已经足够，但有些工作流（例如使用 [远程缓存](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行) 的工作流）可能需要对守护进程的生命周期有更精确的控制：

    * `dmypy stop` 终止守护进程。

    * `dmypy start -- <flags>` 启动守护进程但不检查任何文件。你可以在 `--` 后面使用几乎任意的 mypy 标志。

    * `dmypy restart -- <flags>` 重启守护进程。标志与 `dmypy start` 相同。这等同于先执行停止命令，然后再启动命令。

    * 使用 `dmypy run --timeout SECONDS -- <flags>`（或 `start` 或 `restart`）来在不活动后自动关闭守护进程。默认情况下，守护进程会一直运行，直到被明确停止。

    * `dmypy check <files>` 使用已经运行的守护进程检查一组文件。

    * `dmypy recheck` 检查与最近的 `check` 或 `recheck` 命令相同的文件集。（你也可以使用 [--update](#update) 和 [--remove](#remove) 选项来修改文件集，并定义应处理哪些文件。）

    * `dmypy status` 检查守护进程是否正在运行。如果有运行中的守护进程，它会打印诊断信息并以 `0` 退出。

    使用 `dmypy --help` 获取有关额外命令和命令行选项的帮助，使用 `dmypy <command> --help` 获取有关特定命令选项的帮助。

=== "英文"

    While ``dmypy run`` is sufficient for most uses, some workflows (ones using [remote caching](../mypy_other/additional_features.md#使用远程缓存加速-mypy-运行), perhaps), require more precise control over the lifetime of the daemon process:

    * ``dmypy stop`` stops the daemon.

    * ``dmypy start -- <flags>`` starts the daemon but does not check any files. You can use almost arbitrary mypy flags after ``--``.

    * ``dmypy restart -- <flags>`` restarts the daemon. The flags are the same as with ``dmypy start``. This is equivalent to a stop command followed by a start.

    * Use ``dmypy run --timeout SECONDS -- <flags>`` (or ``start`` or ``restart``) to automatically shut down the daemon after inactivity. By default, the daemon runs until it's explicitly stopped.

    * ``dmypy check <files>`` checks a set of files using an already running daemon.

    * ``dmypy recheck`` checks the same set of files as the most recent ``check`` or ``recheck`` command. (You can also use the [--update](#update) and [--remove](#remove) options to alter the set of files, and to define which files should be processed.)

    * ``dmypy status`` checks whether a daemon is running. It prints a diagnostic and exits with ``0`` if there is a running daemon.

    Use ``dmypy --help`` for help on additional commands and command-line options not discussed here, and ``dmypy <command> --help`` for help on command-specific options.

## 额外的守护进程标志

**Additional daemon flags**

=== "中文"

    <span id="status-file"></span>`--status-file FILE`

    :    使用 ``FILE`` 作为状态文件来存储守护进程的运行状态。这通常是一个 JSON 文件，包含关于守护进程和连接的信息。默认路径是当前工作目录下的 ``.dmypy.json``。

    <span id="log-file"></span>`--log-file FILE`

    :    将守护进程的标准输出/标准错误输出定向到 ``FILE``。这对调试守护进程崩溃非常有用，因为服务器的回溯信息不总是由客户端打印。此选项适用于 ``start``、``restart`` 和 ``run`` 命令。

    <span id="timeout"></span>`--timeout TIMEOUT`

    :    在 ``TIMEOUT`` 秒的非活动后自动关闭服务器。此选项适用于 ``start``、``restart`` 和 ``run`` 命令。

    <span id="update"></span>`--update FILE`

    :    重新检查 ``FILE``，或将其添加到正在检查的文件集（并进行检查）。此选项可以重复使用，仅适用于 ``recheck`` 命令。默认情况下，mypy 查找并检查自上次运行以来更改的所有文件及其依赖的文件。然而，如果使用此选项（和/或 [--remove](#remove)），mypy 仅假设显式指定的文件已更改。这对加速检查大量文件很有用，前提是使用外部的快速文件系统监视器，如 [watchman](https://facebook.github.io/watchman/) 或 [watchdog](https://pypi.org/project/watchdog/)，来确定哪些文件被编辑或删除。*注意：*此选项并非必需，仅用于性能调优。

    <span id="remove"></span>`--remove FILE`

    :    从正在检查的文件集中移除 ``FILE``。此选项可以重复使用，仅适用于 ``recheck`` 命令。有关何时有用的信息，请参见上面的 [--update](#update)。*注意：*此选项并非必需，仅用于性能调优。

    <span id="fswatcher-dump-file"></span>`--fswatcher-dump-file FILE`

    :    收集当前内部文件状态的信息。仅适用于 ``status`` 命令。这将以 JSON 格式将信息转储到 ``FILE`` 中，格式为 ``{path: [modification_time, size, content_hash]}``。这对调试内置的文件系统监视器非常有用。*注意：*这是一个内部标志，格式可能会有所变化。

    <span id="perf-stats-file"></span>`--perf-stats-file FILE`

    :    将性能分析信息写入 ``FILE``。仅适用于 ``check``、``recheck`` 和 ``run`` 命令。

    <span id="export-types"></span>`--export-types`

    :    将所有表达式类型存储在内存中以供将来使用。这对加速未来对 ``dmypy inspect`` 的调用非常有用（但会使用更多内存）。仅适用于 ``check``、``recheck`` 和 ``run`` 命令。

=== "英文"

    <span id="status-file"></span>`--status-file FILE`

    :    Use ``FILE`` as the status file for storing daemon runtime state. This is normally a JSON file that contains information about daemon process and connection. The default path is ``.dmypy.json`` in the current working directory.

    <span id="log-file"></span>`--log-file FILE`

    :    Direct daemon stdout/stderr to ``FILE``. This is useful for debugging daemon crashes, since the server traceback is not always printed by the client. This is available for the ``start``, ``restart``, and ``run`` commands.

    <span id="timeout"></span>`--timeout TIMEOUT`

    :    Automatically shut down server after ``TIMEOUT`` seconds of inactivity. This is available for the ``start``, ``restart``, and ``run`` commands.

    <span id="update"></span>`--update FILE`

    :    Re-check ``FILE``, or add it to the set of files being checked (and check it). This option may be repeated, and it's only available for the ``recheck`` command.  By default, mypy finds and checks all files changed since the previous run and files that depend on them.  However, if you use this option (and/or [--remove](#remove)), mypy assumes that only the explicitly specified files have changed. This is only useful to speed up mypy if you type check a very large number of files, and use an external, fast file system watcher, such as [watchman](https://facebook.github.io/watchman/) or [watchdog](https://pypi.org/project/watchdog/), to determine which files got edited or deleted. *Note:* This option is never required and is only available for performance tuning.

    <span id="remove"></span>`--remove FILE`

    :    Remove ``FILE`` from the set of files being checked. This option may be repeated. This is only available for the ``recheck`` command. See [--update](#update) above for when this may be useful. *Note:* This option is never required and is only available for performance tuning.

    <span id="fswatcher-dump-file"></span>`--fswatcher-dump-file FILE`

    :    Collect information about the current internal file state. This is only available for the ``status`` command. This will dump JSON to ``FILE`` in the format ``{path: [modification_time, size, content_hash]}``. This is useful for debugging the built-in file system watcher. *Note:* This is an internal flag and the format may change.

    <span id="perf-stats-file"></span>`--perf-stats-file FILE`

    :    Write performance profiling information to ``FILE``. This is only available for the ``check``, ``recheck``, and ``run`` commands.

    <span id="export-types"></span>`--export-types`

    :    Store all expression types in memory for future use. This is useful to speed up future calls to ``dmypy inspect`` (but uses more memory). Only valid for ``check``, ``recheck``, and ``run`` command.

## 注解的静态推断

**Static inference of annotations**

=== "中文"

    mypy 守护进程支持（作为实验性功能）静态推断函数和方法的草稿类型注解。使用 ``dmypy suggest FUNCTION`` 来生成草稿签名，格式为 ``(param_type_1, param_type_2, ...) -> ret_type``（类型包含所有参数，包括仅关键字参数、``*args`` 和 ``**kwargs``）。

    这是一个低级功能，旨在由编辑器集成、IDE 和其他工具（例如 [PyCharm 的 mypy 插件](https://github.com/dropbox/mypy-PyCharm-plugin)）使用，用于自动向源文件添加注解或提议函数签名。

    在这个示例中，函数 ``format_id()`` 没有注解：

    ```python
    def format_id(user):
        return f"User: {user}"

    root = format_id(0)
    ```

    ``dmypy suggest`` 使用调用点、返回语句和其他启发式方法（例如查找基类中的签名）来推断 ``format_id()`` 接受一个 ``int`` 参数并返回一个 ``str``。使用 ``dmypy suggest module.format_id`` 打印函数的建议签名。

    更一般地，目标函数可以通过两种方式指定：

    * 通过其完全限定名，即 ``[package.]module.[class.]function``。

    * 通过其在源文件中的位置，即 ``/path/to/file.py:line``。路径可以是绝对的或相对的，而 ``line`` 可以是函数体内的任何行号。

    此命令还可用于为现有的、不精确的注解提供更精确的替代方案，这些注解中包含一些 ``Any`` 类型。

    以下标志自定义 ``dmypy suggest`` 命令的各种方面：

    <span id="json"></span>`--json`

    :    将签名输出为 JSON，以便 [PyAnnotate](https://github.com/dropbox/pyannotate) 可以读取并将签名添加到源文件中。JSON 格式如下所示：

    ```python
    [{"func_name": "example.format_id",
        "line": 1,
        "path": "/absolute/path/to/example.py",
        "samples": 0,
        "signature": {"arg_types": ["int"], "return_type": "str"}}]
    ```

    <span id="no-errors"></span>`--no-errors`

    :    仅生成不会导致检查代码出错的建议。默认情况下，mypy 会尝试找到最精确的类型，即使这会导致一些类型错误。

    <span id="no-any"></span>`--no-any`

    :    仅生成不包含 ``Any`` 类型的建议。默认情况下，mypy 会建议找到的最精确签名，即使它包含 ``Any`` 类型。

    <span id="flex-any"></span>`--flex-any FRACTION`

    :    仅允许建议签名中的某些比例类型为 ``Any`` 类型。该比例范围从 ``0``（相当于 ``--no-any``）到 ``1``。

    <span id="callsites"></span>`--callsites`

    :    仅查找给定函数的调用点，而不是建议类型。这将生成一个列表，包含每个调用的行号和实际参数的类型：``/path/to/file.py:line: (arg_type_1, arg_type_2, ...)``。

    <span id="use-fixme"></span>`--use-fixme NAME`

    :    对于无法推断的类型，使用一个虚拟名称而不是普通的 ``Any``。这可能有助于向用户强调某个类型无法推断，并需要手动输入。

    <span id="max-guesses"></span>`--max-guesses NUMBER`

    :    设置尝试的函数类型的最大数量（默认值：``64``）。

=== "英文"

    The mypy daemon supports (as an experimental feature) statically inferring draft function and method type annotations. Use ``dmypy suggest FUNCTION`` to generate a draft signature in the format ``(param_type_1, param_type_2, ...) -> ret_type`` (types are included for all arguments, including keyword-only arguments, ``*args`` and ``**kwargs``).

    This is a low-level feature intended to be used by editor integrations, IDEs, and other tools (for example, the [mypy plugin for PyCharm](https://github.com/dropbox/mypy-PyCharm-plugin)), to automatically add annotations to source files, or to propose function signatures.

    In this example, the function ``format_id()`` has no annotation:

    ```python
    def format_id(user):
        return f"User: {user}"

    root = format_id(0)
    ```

    ``dmypy suggest`` uses call sites, return statements, and other heuristics (such as looking for signatures in base classes) to infer that ``format_id()`` accepts an ``int`` argument and returns a ``str``. Use ``dmypy suggest module.format_id`` to print the suggested signature for the function.

    More generally, the target function may be specified in two ways:

    * By its fully qualified name, i.e. ``[package.]module.[class.]function``.

    * By its location in a source file, i.e. ``/path/to/file.py:line``. The path can be absolute or relative, and ``line`` can refer to any line number within the function body.

    This command can also be used to find a more precise alternative for an existing, imprecise annotation with some ``Any`` types.

    The following flags customize various aspects of the ``dmypy suggest`` command.

    <span id="json"></span>`--json`

    :    Output the signature as JSON, so that [PyAnnotate](https://github.com/dropbox/pyannotate) can read it and add the signature to the source file. Here is what the JSON looks like:

    ```python
    [{"func_name": "example.format_id",
        "line": 1,
        "path": "/absolute/path/to/example.py",
        "samples": 0,
        "signature": {"arg_types": ["int"], "return_type": "str"}}]
    ```

    <span id="no-errors"></span>`--no-errors`

    :    Only produce suggestions that cause no errors in the checked code. By default, mypy will try to find the most precise type, even if it causes some type errors.

    <span id="no-any"></span>`--no-any`

    :    Only produce suggestions that don't contain ``Any`` types. By default mypy proposes the most precise signature found, even if it contains ``Any`` types.

    <span id="flex-any"></span>`--flex-any FRACTION`

    :    Only allow some fraction of types in the suggested signature to be ``Any`` types. The fraction ranges from ``0`` (same as ``--no-any``) to ``1``.

    <span id="callsites"></span>`--callsites`

    :    Only find call sites for a given function instead of suggesting a type. This will produce a list with line numbers and types of actual arguments for each call: ``/path/to/file.py:line: (arg_type_1, arg_type_2, ...)``.

    <span id="use-fixme"></span>`--use-fixme NAME`

    :    Use a dummy name instead of plain ``Any`` for types that cannot be inferred. This may be useful to emphasize to a user that a given type couldn't be inferred and needs to be entered manually.

    <span id="max-guesses"></span>`--max-guesses NUMBER`

    :    Set the maximum number of types to try for a function (default: ``64``).

## 静态检查表达式

**Statically inspect expressions**

=== "中文"

    守护进程支持通过 ``dmypy inspect LOCATION`` 命令获取表达式的声明或推断类型（或其他有关表达式的信息，例如已知属性或定义位置）。表达式的位置应以 ``path/to/file.py:line:column[:end_line:end_column]`` 格式指定。行和列都是从 1 开始的。开始和结束位置都是包含在内的。这些规则与 mypy 在错误消息中打印错误位置的方式相匹配。

    如果提供了一个跨度（即四个数字），则仅检查完全匹配的表达式。如果只提供了位置（即两个数字，行和列），mypy 将检查包括该位置的所有 *表达式*，从最内层的开始。

    考虑以下 Python 代码片段：

    ```python
    def foo(x: int, longer_name: str) -> None:
        x
        longer_name
    ```

    要查找 ``x`` 的类型，需要调用 ``dmypy inspect src.py:2:5:2:5`` 或 ``dmypy inspect src.py:2:5``。而要查找 ``longer_name`` 的类型，则需要调用 ``dmypy inspect src.py:3:5:3:15`` 或 ``dmypy inspect src.py:3:10``。请注意，此命令仅在守护进程成功进行类型检查（没有解析错误）后有效，以便类型被填充，例如使用 ``dmypy check``。如果多个表达式匹配提供的位置，它们的类型会以换行符分隔返回。

    重要说明：建议使用 [--export-types](#export-types) 检查文件，否则大多数检查将无法工作，除非使用 [--force-reload](#force-reload)。

    <span id="show"></span>`--show INSPECTION`

    :    指定要对找到的表达式运行的检查类型。目前支持的检查包括：

    * ``type``（默认）：显示给定表达式的最佳已知类型。
    * ``attrs``：显示表达式的有效属性（例如，用于自动补全）。格式为 ``{"Base1": ["name_1", "name_2", ...]; "Base2": ...}``。名称按方法解析顺序排序。如果表达式引用的是模块，则模块属性将位于类似 ``"<full.module.name>"`` 的键下。
    * ``definition``（实验性）：显示名称表达式或成员表达式的定义位置。格式为 ``path/to/file.py:line:column:Symbol``。 如果找到多个定义（例如，对于 Union 属性），它们用逗号分隔。

    <span id="verbose"></span>`--verbose`

    :    增加类型字符串表示的详细程度（可以重复）。例如，这将打印实例类型的完全限定名称（如 ``"builtins.str"``），而不是简短名称（如 ``"str"``）。

    <span id="limit"></span>`--limit NUM`

    :    如果位置以 ``line:column`` 给出，则此选项将导致守护进程仅返回最多 ``NUM`` 个最内层表达式的检查结果。值为 0 表示无限制（这是默认值）。例如，如果调用 ``dmypy inspect src.py:4:10 --limit=1``，代码如下：

        ```python
        def foo(x: int) -> str: ..
        def bar(x: str) -> None: ...
        baz: int
        bar(foo(baz))
        ```

        这将输出一个类型 ``"int"``（针对 ``baz`` 名称表达式）。而没有限制选项的情况下，它将输出所有三个类型：``"int"``, ``"str"``, 和 ``"None"``。

    <span id="include-span"></span>`--include-span`

    :    启用此选项后，守护进程将为每个检查结果附加对应表达式的完整跨度，格式为 ``1:2:1:4 -> "int"``。这在多个表达式匹配一个位置的情况下可能很有用。

    <span id="include-kind"></span>`--include-kind`

    :    启用此选项后，守护进程将为每个检查结果附加对应表达式的类型，格式为 ``NameExpr -> "int"``。如果同时启用此选项和 [--include-span](#include-span)，类型会首先出现，例如 ``NameExpr:1:2:1:4 -> "int"``。

    <span id="include-object-attrs"></span>`--include-object-attrs`

    :    在 ``attrs`` 检查中，包括 ``object`` 的属性（默认情况下排除）。

    <span id="union-attrs"></span>`--union-attrs`

    :    包括对某些可能的表达式类型有效的属性（默认情况下返回交集）。这对具有值的类型变量的联合类型很有用。例如，对于以下代码：

        ```python
        from typing import Union

        class A:
            x: int
            z: int
        class B:
            y: int
            z: int
        var: Union[A, B]
        var
        ```

        使用命令 ``dmypy inspect --show attrs src.py:10:1`` 将返回 ``{"A": ["z"], "B": ["z"]}``，而使用 ``--union-attrs`` 将返回 ``{"A": ["x", "z"], "B": ["y", "z"]}``。

    <span id="force-reload"></span>`--force-reload`

    :    在检查之前强制重新解析和重新进行类型检查。默认情况下，仅在需要时执行此操作（例如，文件未从缓存中加载或守护进程最初运行时没有 ``--export-types`` mypy 选项），因为重新加载可能会很慢（对于非常大的文件，可能需要几秒钟）。

=== "英文"

    The daemon allows to get declared or inferred type of an expression (or other information about an expression, such as known attributes or definition location) using ``dmypy inspect LOCATION`` command. The location of the expression should be specified in the format ``path/to/file.py:line:column[:end_line:end_column]``. Both line and column are 1-based. Both start and end position are inclusive. These rules match how mypy prints the error location in error messages.

    If a span is given (i.e. all 4 numbers), then only an exactly matching expression is inspected. If only a position is given (i.e. 2 numbers, line and column), mypy will inspect all *expressions*, that include this position, starting from the innermost one.

    Consider this Python code snippet:

    ```python
    def foo(x: int, longer_name: str) -> None:
        x
        longer_name
    ```


    Here to find the type of ``x`` one needs to call ``dmypy inspect src.py:2:5:2:5`` or ``dmypy inspect src.py:2:5``. While for ``longer_name`` one needs to call ``dmypy inspect src.py:3:5:3:15`` or, for example, ``dmypy inspect src.py:3:10``. Please note that this command is only valid after daemon had a successful type check (without parse errors), so that types are populated, e.g. using ``dmypy check``. In case where multiple expressions match the provided location, their types are returned separated by a newline.

    Important note: it is recommended to check files with [--export-types](#export-types) since otherwise most inspections will not work without [--force-reload](#force-reload).

    <span id="show"></span>`--show INSPECTION`

    :    What kind of inspection to run for expression(s) found. Currently the supported inspections are:

    * ``type`` (default): Show the best known type of a given expression.
    * ``attrs``: Show which attributes are valid for an expression (e.g. for auto-completion). Format is ``{"Base1": ["name_1", "name_2", ...]; "Base2": ...}``. Names are sorted by method resolution order. If expression refers to a module, then module attributes will be under key like ``"<full.module.name>"``.
    * ``definition`` (experimental): Show the definition location for a name expression or member expression. Format is ``path/to/file.py:line:column:Symbol``. If multiple definitions are found (e.g. for a Union attribute), they are separated by comma.

    <span id="verbose"></span>`--verbose`

    :    Increase verbosity of types string representation (can be repeated). For example, this will print fully qualified names of instance types (like ``"builtins.str"``), instead of just a short name (like ``"str"``).

    <span id="limit"></span>`--limit NUM`

    :    If the location is given as ``line:column``, this will cause daemon to return only at most ``NUM`` inspections of innermost expressions. Value of 0 means no limit (this is the default). For example, if one calls ``dmypy inspect src.py:4:10 --limit=1`` with this code

        ```python
        def foo(x: int) -> str: ..
        def bar(x: str) -> None: ...
        baz: int
        bar(foo(baz))
        ```

        This will output just one type ``"int"`` (for ``baz`` name expression). While without the limit option, it would output all three types: ``"int"``, ``"str"``, and ``"None"``.

    <span id="include-span"></span>`--include-span`

    :    With this option on, the daemon will prepend each inspection result with the full span of corresponding expression, formatted as ``1:2:1:4 -> "int"``. This may be useful in case multiple expressions match a location.

    <span id="include-kind"></span>`--include-kind`

    :    With this option on, the daemon will prepend each inspection result with the kind of corresponding expression, formatted as ``NameExpr -> "int"``. If both this option and [--include-span](#include-span) are on, the kind will appear first, for example ``NameExpr:1:2:1:4 -> "int"``.

    <span id="include-object-attrs"></span>`--include-object-attrs`

    :    This will make the daemon include attributes of ``object`` (excluded by default) in case of an ``atts`` inspection.

    <span id="union-attrs"></span>`--union-attrs`

    :    Include attributes valid for some of possible expression types (by default an intersection is returned). This is useful for union types of type variables with values. For example, with this code:

        ```python
        from typing import Union
    
        class A:
            x: int
            z: int
        class B:
            y: int
            z: int
        var: Union[A, B]
        var
        ```

        The command ``dmypy inspect --show attrs src.py:10:1`` will return ``{"A": ["z"], "B": ["z"]}``, while with ``--union-attrs`` it will return ``{"A": ["x", "z"], "B": ["y", "z"]}``.

    <span id="force-reload"></span>`--force-reload`

    :    Force re-parsing and re-type-checking file before inspection. By default this is done only when needed (for example file was not loaded from cache or daemon was initially run without ``--export-types`` mypy option), since reloading may be slow (up to few seconds for very large files).

.. TODO: Add similar section about find usages when added, and then move this to a separate file.


[watchman]: https://facebook.github.io/watchman/
[watchdog]: https://pypi.org/project/watchdog/
[PyAnnotate]: https://github.com/dropbox/pyannotate
[mypy plugin for PyCharm]: https://github.com/dropbox/mypy-PyCharm-plugin

