# 错误代码

**Error codes**

=== "中文"

    Mypy 可以选择在每个错误信息后显示一个错误代码，例如 ``[attr-defined]``。错误代码有两个用途：

    1. 可以使用 ``# type: ignore[code]`` 来仅静默特定的错误代码。这样，您不会不小心忽略其他可能更严重的错误。

    2. 错误代码可用于查找有关错误的文档。接下来的两个主题（[默认启用的错误代码](./error_code_list.md) 和 [可选检查的错误代码](./error_code_list2.md)）记录了 mypy 可以报告的各种错误代码。

    大多数错误代码在多个相关错误消息之间是共享的。错误代码可能会在未来的 mypy 版本中发生变化。

=== "英文"

    Mypy can optionally display an error code such as ``[attr-defined]`` after each error message. Error codes serve two purposes:

    1. It's possible to silence specific error codes on a line using ``# type: ignore[code]``. This way you won't accidentally ignore other, potentially more serious errors.

    2. The error code can be used to find documentation about the error. The next two topics ([Error codes enabled by default](./error_code_list.md) and [Error codes for optional checks](./error_code_list2.md)) document the various error codes mypy can report.

    Most error codes are shared between multiple related error messages. Error codes may change in future mypy releases.

## 根据错误代码静默错误

**Silencing errors based on error codes**

=== "中文"

    您可以使用特殊注释 ``# type: ignore[code, ...]`` 仅忽略特定错误代码（或多个错误代码）的错误。这即使在您没有配置 mypy 显示错误代码的情况下也可以使用。

    以下示例展示了如何忽略一个关于导入名称的错误，该名称 mypy 认为是未定义的：

    ```python
    # 'foo' 在 'foolib' 中定义，尽管 mypy 看不到定义。
    from foolib import foo  # type: ignore[attr-defined]
    ```

=== "英文"

    You can use a special comment ``# type: ignore[code, ...]`` to only ignore errors with a specific error code (or codes) on a particular line.  This can be used even if you have not configured mypy to show error codes.

    This example shows how to ignore an error about an imported name mypy thinks is undefined:

    ```python
    # 'foo' is defined in 'foolib', even though mypy can't see the definition.
    from foolib import foo  # type: ignore[attr-defined]
    ```

## 全局启用/禁用特定错误代码

**Enabling/disabling specific error codes globally**

=== "中文"

    有一些命令行标志和配置文件设置可以启用某些可选的错误代码，例如 [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs)，它启用了 ``no-untyped-def`` 错误代码。

    您可以使用 [--enable-error-code](../mypy_conf/command_line.md#enable-error-code) 和 [--disable-error-code](../mypy_conf/command_line.md#disable-error-code) 来启用或禁用特定的错误代码，这些错误代码没有专门的命令行标志或配置文件设置。

=== "英文"

    There are command-line flags and config file settings for enabling certain optional error codes, such as [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs), which enables the ``no-untyped-def`` error code.

    You can use [--enable-error-code](../mypy_conf/command_line.md#enable-error-code) and [--disable-error-code](../mypy_conf/command_line.md#disable-error-code) to enable or disable specific error codes that don't have a dedicated command-line flag or config file setting.

## 每模块启用/禁用错误代码

**Per-module enabling/disabling error codes**

=== "中文"

    您可以使用 [配置文件](../mypy_conf/config_file.md) 部分来仅在某些模块中启用或禁用特定的错误代码。例如，以下 ``mypy.ini`` 配置将允许在测试中使用未注解的空容器，同时保持代码的其他部分在严格模式下进行检查：

    ```ini
    [mypy]
    strict = True

    [mypy-tests.*]
    allow_untyped_defs = True
    allow_untyped_calls = True
    disable_error_code = var-annotated, has-type
    ```

    请注意，每个模块的启用/禁用设置会覆盖全局选项。因此，如果您在全局配置部分中定义了错误代码列表，则无需在每个模块中重复定义。例如：

    ```ini
    [mypy]
    enable_error_code = truthy-bool, ignore-without-code, unused-awaitable

    [mypy-extensions.*]
    disable_error_code = unused-awaitable
    ```

    上述配置将允许扩展模块中存在未使用的 awaitable，但仍将保留其他两个错误代码的启用状态。整体逻辑如下：

    * 命令行和/或配置文件主部分设置全局错误代码。
    * 单独的配置部分 *调整* 这些代码以适应特定的 glob/模块。
    * 内联 ``# mypy: disable-error-code="..."`` 和 ``# mypy: enable-error-code="..."`` 注释可以进一步 *调整* 这些设置以适应特定文件。例如：

    ```python
    # mypy: enable-error-code="truthy-bool, ignore-without-code"
    ```

    因此，可以例如在全局范围内启用某些代码，在相应的配置部分中为所有测试禁用它，然后通过内联注释在某些特定测试中重新启用它。

=== "英文"

    You can use [configuration file](../mypy_conf/config_file.md) sections to enable or disable specific error codes only in some modules. For example, this ``mypy.ini`` config will enable non-annotated empty containers in tests, while keeping other parts of code checked in strict mode:

    ```ini
    [mypy]
    strict = True

    [mypy-tests.*]
    allow_untyped_defs = True
    allow_untyped_calls = True
    disable_error_code = var-annotated, has-type
    ```

    Note that per-module enabling/disabling acts as override over the global options. So that you don't need to repeat the error code lists for each module if you have them in global config section. For example:

    ```ini
    [mypy]
    enable_error_code = truthy-bool, ignore-without-code, unused-awaitable

    [mypy-extensions.*]
    disable_error_code = unused-awaitable
    ```

    The above config will allow unused awaitables in extension modules, but will still keep the other two error codes enabled. The overall logic is following:

    * Command line and/or config main section set global error codes
    * Individual config sections *adjust* them per glob/module
    * Inline ``# mypy: disable-error-code="..."`` and ``# mypy: enable-error-code="..."``
      comments can further *adjust* them for a specific file.
      For example:

    ```python
    # mypy: enable-error-code="truthy-bool, ignore-without-code"
    ```

    So one can e.g. enable some code globally, disable it for all tests in the corresponding config section, and then re-enable it with an inline comment in some specific test.

## 错误代码的子代码

**Subcodes of error codes**

=== "中文"

    在某些情况下，主要出于向后兼容的原因，一个错误代码可能也被另一个更广泛的错误代码覆盖。例如，代码为 ``[method-assign]`` 的错误可以通过 ``# type: ignore[assignment]`` 被忽略。类似的逻辑适用于全局禁用错误代码。如果某个错误代码是另一个错误代码的子代码，它会在更具体的错误代码的文档中提到。这种层次结构不是嵌套的：子代码不能有其他子代码的子代码。

=== "英文"

    In some cases, mostly for backwards compatibility reasons, an error code may be covered also by another, wider error code. For example, an error with code ``[method-assign]`` can be ignored by ``# type: ignore[assignment]``. Similar logic works for disabling error codes globally. If a given error code is a subcode of another one, it will be mentioned in the documentation for the narrower code. This hierarchy is not nested: there cannot be subcodes of other subcodes.


## 要求错误代码

**Requiring error codes**

=== "中文"

    可以要求在 ``type: ignore`` 注释中指定错误代码。有关更多信息，请参见 [ignore-without-code](./error_code_list2.md#检查--type-ignore-是否包含错误代码-ignore-without-code)。

=== "英文"

    It's possible to require error codes be specified in ``type: ignore`` comments. See [ignore-without-code](./error_code_list2.md#检查--type-ignore-是否包含错误代码-ignore-without-code) for more information.

