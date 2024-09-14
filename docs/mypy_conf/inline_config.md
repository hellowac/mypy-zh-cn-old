# 内联配置

**Inline configuration**

=== "中文"

=== "英文"

Mypy supports setting per-file configuration options inside files themselves
using ``# mypy:`` comments. For example:

```python
  # mypy: disallow-any-generics
```

Inline configuration comments take precedence over all other
configuration mechanisms.

## 配置注释格式

**Configuration comment format**

=== "中文"

=== "英文"

Flags correspond to :ref:`config file flags <config-file>` but allow
hyphens to be substituted for underscores.

Values are specified using ``=``, but ``= True`` may be omitted:

```python
  # mypy: disallow-any-generics
  # mypy: always-true=FOO
```

Multiple flags can be separated by commas or placed on separate
lines. To include a comma as part of an option's value, place the
value inside quotes:

```python
  # mypy: disallow-untyped-defs, always-false="FOO,BAR"
```

Like in the configuration file, options that take a boolean value may be
inverted by adding ``no-`` to their name or by (when applicable)
swapping their prefix from ``disallow`` to ``allow`` (and vice versa):

```python
  # mypy: allow-untyped-defs, no-strict-optional
```
