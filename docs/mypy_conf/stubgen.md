# 自动生成存根 (stubgen)

**Automatic stub generation (stubgen)**

=== "中文"

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

   : Generate a stub file for the given module. This flag may be repeated multiple times.

    Stubgen *will not* recursively generate stubs for any submodules of the provided module.

<span id="p"></span>`-p PACKAGE, --package PACKAGE`

   : Generate stubs for the given package. This flag maybe repeated multiple times.

    Stubgen *will* recursively generate stubs for all submodules of the provided package. This flag is identical to [--module](#m) apart from this behavior.

!!! note 

    You can't mix paths and [-m](#m)/[-p](#p) options in the same stubgen invocation.

Stubgen applies heuristics to avoid generating stubs for submodules that include tests or vendored third-party packages.

## 指定如何生成存根

**Specifying how to generate stubs**

=== "中文"

=== "英文"

By default stubgen will try to import the target modules and packages. This allows stubgen to use runtime introspection to generate stubs for C extension modules and to improve the quality of the generated stubs. By default, stubgen will also use mypy to perform light-weight semantic analysis of any Python modules. Use the following flags to alter the default behavior:

<span id="no-import"></span>`--no-import`

   : Don't try to import modules. Instead only use mypy's normal search mechanism to find sources. This does not support C extension modules. This flag also disables runtime introspection functionality, which mypy uses to find the value of ``__all__``. As result the set of exported imported names in stubs may be incomplete. This flag is generally only useful when importing a module causes unwanted side effects, such as the running of tests. Stubgen tries to skip test modules even without this option, but this does not always work.

<span id="no-analysis"></span>`--no-analysis`

   : Don't perform semantic analysis of source files. This may generate worse stubs -- in particular, some module, class, and function aliases may be represented as variables with the ``Any`` type. This is generally only useful if semantic analysis causes a critical mypy error.  Does not apply to C extension modules.  Incompatible with [--inspect-mode](#inspect-mode).

<span id="inspect-mode"></span>`--inspect-mode`

   : Import and inspect modules instead of parsing source code. This is the default behavior for C modules and pyc-only packages.  The flag is useful to force inspection for pure Python modules that make use of dynamically generated members that would otherwise be omitted when using the default behavior of code parsing.  Implies [--no-analysis](#no-analysis) as analysis requires source code.

<span id="doc-dir"></span>`--doc-dir PATH`

   : Try to infer better signatures by parsing .rst documentation in ``PATH``. This may result in better stubs, but currently it only works for C extension modules.

## 额外标志

**Additional flags**

=== "中文"

=== "英文"

<span id="h"></span>`-h, --help`

   : Show help message and exit.

<span id="ignore-errors"></span>`--ignore-errors`

   : If an exception was raised during stub generation, continue to process any remaining modules instead of immediately failing with an error.

<span id="include-private"></span>`--include-private`

   : Include definitions that are considered private in stubs (with names such as ``_foo`` with single leading underscore and no trailing underscores).

<span id="export-less"></span>`--export-less`

   : Don't export all names imported from other modules within the same package. Instead, only export imported names that are not referenced in the module that contains the import.

<span id="include-docstrings"></span>`--include-docstrings`

   : Include docstrings in stubs. This will add docstrings to Python function and classes stubs and to C extension function stubs.

<span id="search-path"></span>`--search-path PATH`

   : Specify module search directories, separated by colons (only used if [--no-import](#no-import) is given).

<span id="o"></span>`-o PATH, --output PATH`

   : Change the output directory. By default the stubs are written in the ``./out`` directory. The output directory will be created if it doesn't exist. Existing stubs in the output directory will be overwritten without warning.

<span id="v"></span>`-v, --verbose`

   : Produce more verbose output.

<span id="q"></span>`-q, --quiet`

   : Produce less verbose output.
