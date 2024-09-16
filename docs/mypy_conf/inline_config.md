# 内联配置

**Inline configuration**

=== "中文"

    Mypy 支持在文件内部使用 ``# mypy:`` 注释来设置每个文件的配置选项。例如：

    ```python
    # mypy: disallow-any-generics
    ```

    内联配置注释优先于所有其他配置机制。

=== "英文"

    Mypy supports setting per-file configuration options inside files themselves using ``# mypy:`` comments. For example:

    ```python
    # mypy: disallow-any-generics
    ```

    Inline configuration comments take precedence over all other configuration mechanisms.

## 配置注释格式

**Configuration comment format**

=== "中文"

    标志对应于 [配置文件标志](./config_file.md)，但允许使用连字符代替下划线。

    值通过 ``=`` 指定，但 ``= True`` 可以省略：

    ```python
    # mypy: disallow-any-generics
    # mypy: always-true=FOO
    ```

    多个标志可以用逗号分隔，也可以放在不同的行上。要将逗号包含在选项值的一部分中，可以将值放在引号内：

    ```python
    # mypy: disallow-untyped-defs, always-false="FOO,BAR"
    ```

    如同在配置文件中一样，接受布尔值的选项可以通过在名称前添加 ``no-`` 来取反，或（在适用时）将其前缀从 ``disallow`` 更改为 ``allow``（反之亦然）：

    ```python
    # mypy: allow-untyped-defs, no-strict-optional
    ```

=== "英文"

    Flags correspond to [config file flags](./config_file.md) but allow hyphens to be substituted for underscores.

    Values are specified using ``=``, but ``= True`` may be omitted:

    ```python
    # mypy: disallow-any-generics
    # mypy: always-true=FOO
    ```

    Multiple flags can be separated by commas or placed on separate lines. To include a comma as part of an option's value, place the value inside quotes:

    ```python
    # mypy: disallow-untyped-defs, always-false="FOO,BAR"
    ```

    Like in the configuration file, options that take a boolean value may be inverted by adding ``no-`` to their name or by (when applicable) swapping their prefix from ``disallow`` to ``allow`` (and vice versa):

    ```python
    # mypy: allow-untyped-defs, no-strict-optional
    ```
