# 扩展和集成 mypy

**Extending and integrating mypy**

## 将 mypy 集成到其他 Python 应用中

**Integrating mypy into another Python application**

=== "中文"

    可以通过导入 ``mypy.api`` 并调用 ``run`` 函数来将 mypy 集成到其他 Python 3 应用程序中。调用 ``run`` 函数时，参数类型为 ``list[str]``，包含通常作为 mypy 命令行参数的内容。

    ``run`` 函数返回一个 ``tuple[str, str, int]``，即 ``(<normal_report>, <error_report>, <exit_status>)``，其中 ``<normal_report>`` 是 mypy 通常写入 [sys.stdout](https://docs.python.org/3/library/sys.html#sys.stdout) 的内容，``<error_report>`` 是 mypy 通常写入 [sys.stderr](https://docs.python.org/3/library/sys.html#sys.stderr) 的内容，``exit_status`` 是 mypy 通常返回给操作系统的退出状态。

    使用该 API 的一个简单示例如下：

    ```python
    import sys
    from mypy import api

    result = api.run(sys.argv[1:])

    if result[0]:
        print('\n类型检查报告:\n')
        print(result[0])  # stdout

    if result[1]:
        print('\n错误报告:\n')
        print(result[1])  # stderr

    print('\n退出状态:', result[2])
    ```

=== "英文"

    It is possible to integrate mypy into another Python 3 application by importing ``mypy.api`` and calling the ``run`` function with a parameter of type ``list[str]``, containing what normally would have been the command line arguments to mypy.

    Function ``run`` returns a ``tuple[str, str, int]``, namely ``(<normal_report>, <error_report>, <exit_status>)``, in which ``<normal_report>`` is what mypy normally writes to [sys.stdout](https://docs.python.org/3/library/sys.html#sys.stdout), ``<error_report>`` is what mypy normally writes to [sys.stderr](https://docs.python.org/3/library/sys.html#sys.stderr) and ``exit_status`` is the exit status mypy normally returns to the operating system.

    A trivial example of using the api is the following

    ```python
    import sys
    from mypy import api

    result = api.run(sys.argv[1:])

    if result[0]:
        print('\nType checking report:\n')
        print(result[0])  # stdout

    if result[1]:
        print('\nError report:\n')
        print(result[1])  # stderr

    print('\nExit status:', result[2])
    ```

## 使用插件扩展 mypy

**Extending mypy using plugins**

=== "中文"

    Python 是一种高度动态的语言，具有广泛的元编程能力。许多流行的库利用这些能力创建的 API 可能对人类更灵活和自然，但使用静态类型表达这些 API 却很困难。扩展 [PEP 484](https://peps.python.org/pep-0484/) 类型系统以适应所有现有的动态模式是不切实际的，且往往是不可能的。

    Mypy 支持一个插件系统，允许你自定义 mypy 检查代码的方式。如果你想扩展 mypy 以便它可以检查使用了难以仅用 [PEP 484](https://peps.python.org/pep-0484/) 类型表达的库的代码，这可能会很有用。

    插件系统的重点是改善 mypy 对第三方框架 *语义* 的理解。目前尚无定义新的第一类类型的方法。

    !!! note 

        插件系统仍处于实验阶段，可能会发生变化。如果你想编写 mypy 插件，我们建议你先通过 [gitter](https://gitter.im/python/typing) 联系 mypy 核心开发人员。特别是，对于向后兼容性没有保证。

        可能会在没有弃用期的情况下进行不兼容的更改，但我们会在 [插件 API 更改公告问题](https://github.com/python/mypy/issues/6617) 中宣布这些更改。

=== "英文"

    Python is a highly dynamic language and has extensive metaprogramming capabilities. Many popular libraries use these to create APIs that may be more flexible and/or natural for humans, but are hard to express using static types. Extending the [PEP 484](https://peps.python.org/pep-0484/) type system to accommodate all existing dynamic patterns is impractical and often just impossible.

    Mypy supports a plugin system that lets you customize the way mypy type checks code. This can be useful if you want to extend mypy so it can type check code that uses a library that is difficult to express using just [PEP 484](https://peps.python.org/pep-0484/) types.

    The plugin system is focused on improving mypy's understanding of *semantics* of third party frameworks. There is currently no way to define new first class kinds of types.

    !!! note 

        The plugin system is experimental and prone to change. If you want to write a mypy plugin, we recommend you start by contacting the mypy core developers on [gitter](https://gitter.im/python/typing). In particular, there are no guarantees about backwards compatibility.

        Backwards incompatible changes may be made without a deprecation period, but we will announce them in [the plugin API changes announcement issue](https://github.com/python/mypy/issues/6617).

## 配置 mypy 使用插件

**Configuring mypy to use plugins**

=== "中文"

    插件是 Python 文件，可以通过 mypy 的 [配置文件](./config_file.md) 使用 [plugins](./config_file.md#plugins) 选项来指定。插件的路径可以是相对路径或绝对路径，或者是模块名称（如果插件是在与 mypy 运行相同的虚拟环境中通过 `pip install` 安装的）。这两种格式可以混合使用，例如：

    ```ini
    [mypy]
    plugins = /one/plugin.py, other.plugin
    ```

    Mypy 会尝试导入插件，并查找一个名为 `plugin` 的入口点函数。如果插件的入口点函数名称不同，可以在冒号后指定：

    ```ini
    [mypy]
    plugins = custom_plugin:custom_entry_point
    ```

    在以下部分，我们将介绍插件系统的基础知识和一些示例。有关更多技术细节，请阅读 mypy 源代码中的 [mypy/plugin.py](https://github.com/python/mypy/blob/master/mypy/plugin.py) 文件中的文档字符串。你还可以在 [mypy/plugins](https://github.com/python/mypy/tree/master/mypy/plugins) 中找到捆绑插件的良好示例。

=== "英文"

    Plugins are Python files that can be specified in a mypy [config file](./config_file.md) using the [plugins](./config_file.md#plugins) option and one of the two formats: relative or absolute path to the plugin file, or a module name (if the plugin is installed using ``pip install`` in the same virtual environment where mypy is running). The two formats can be mixed, for example:

    ```ini

        [mypy]
        plugins = /one/plugin.py, other.plugin
    ```

    Mypy will try to import the plugins and will look for an entry point function named ``plugin``. If the plugin entry point function has a different name, it can be specified after colon:

    ```ini

        [mypy]
        plugins = custom_plugin:custom_entry_point
    ```

    In the following sections we describe the basics of the plugin system with some examples. For more technical details, please read the docstrings in [mypy/plugin.py](https://github.com/python/mypy/blob/master/mypy/plugin.py) in mypy source code. Also you can find good examples in the bundled plugins located in [mypy/plugins](https://github.com/python/mypy/tree/master/mypy/plugins).

## 高级概览

**High-level overview**

=== "中文"

    每个入口点函数应接受一个字符串参数，该参数是完整的 mypy 版本，并返回 `mypy.plugin.Plugin` 的子类：

    ```python
    from mypy.plugin import Plugin

    class CustomPlugin(Plugin):
        def get_type_analyze_hook(self, fullname: str):
            # 参见下面的解释
            ...

    def plugin(version: str):
        # 如果插件适用于所有 mypy 版本，则忽略版本参数
        return CustomPlugin
    ```

    在分析代码的不同阶段（首先是语义分析，然后是类型检查）中，mypy 会调用插件的方法，例如 `get_type_analyze_hook()`。例如，这个特定的方法可以返回一个回调，mypy 将使用这个回调来分析具有给定全名的未绑定类型。有关插件钩子方法的完整列表，请参见 [下方](#当前插件钩子列表)。

    Mypy 从配置文件中获取插件列表，并加上始终启用的默认（内置）插件。Mypy 会对列表中的每个插件调用一次方法，直到其中一个方法返回非 `None` 的值。然后，这个回调将用于定制当前抽象语法树节点的分析/检查方面。

    `get_xxx` 方法返回的回调将获得详细的当前上下文和一个 API，用于创建新节点、新类型、发出错误消息等，并将结果用于进一步处理。

    插件开发者应确保他们的插件在增量模式和守护进程模式下都能正常工作。特别是，插件不应因缓存插件钩子结果而持有全局状态。

=== "英文"

    Every entry point function should accept a single string argument that is a full mypy version and return a subclass of ``mypy.plugin.Plugin``:

    ```python
    from mypy.plugin import Plugin

    class CustomPlugin(Plugin):
        def get_type_analyze_hook(self, fullname: str):
            # see explanation below
            ...

    def plugin(version: str):
        # ignore version argument if the plugin works with all mypy versions.
        return CustomPlugin
    ```

    During different phases of analyzing the code (first in semantic analysis, and then in type checking) mypy calls plugin methods such as ``get_type_analyze_hook()`` on user plugins. This particular method, for example, can return a callback that mypy will use to analyze unbound types with the given full name. See the full plugin hook method list [below](#当前插件钩子列表).

    Mypy maintains a list of plugins it gets from the config file plus the default (built-in) plugin that is always enabled. Mypy calls a method once for each plugin in the list until one of the methods returns a non-``None`` value. This callback will be then used to customize the corresponding aspect of analyzing/checking the current abstract syntax tree node.

    The callback returned by the ``get_xxx`` method will be given a detailed current context and an API to create new nodes, new types, emit error messages, etc., and the result will be used for further processing.

    Plugin developers should ensure that their plugins work well in incremental and daemon modes. In particular, plugins should not hold global state due to caching of plugin hook results.

## 当前插件钩子列表

**Current list of plugin hooks**

=== "中文"

    **get_type_analyze_hook()** 自定义类型分析器的行为。例如，[PEP 484](https://peps.python.org/pep-0484/) 不支持定义可变参数泛型类型：

    ```python
    from lib import Vector

    a: Vector[int, int]
    b: Vector[int, int, int]
    ```

    在分析这段代码时，mypy 会调用 `get_type_analyze_hook("lib.Vector")`，插件可以为每个变量返回一个有效的类型。

    **get_function_hook()** 用于调整函数调用的返回类型。这个钩子也会在类实例化时被调用。如果返回类型过于复杂，无法通过常规的 Python 类型表达，这个钩子是一个不错的选择。

    **get_function_signature_hook()** 用于调整函数的签名。

    **get_method_hook()** 与 `get_function_hook()` 相同，但用于方法，而不是模块级函数。

    **get_method_signature_hook()** 用于调整方法的签名。这包括特殊的 Python 方法，除了 [__init__()](https://docs.python.org/3/reference/datamodel.html#object.__init__) 和 [__new__()](https://docs.python.org/3/reference/datamodel.html#object.__new__)。例如，在下面的代码中：

    ```python
    from ctypes import Array, c_int

    x: Array[c_int]
    x[0] = 42
    ```

    mypy 会调用 `get_method_signature_hook("ctypes.Array.__setitem__")`，以便插件可以模仿 [ctypes](https://docs.python.org/3/library/ctypes.html#module-ctypes) 的自动转换行为。

    **get_attribute_hook()** 用于覆盖实例成员字段查找和属性访问（不是赋值，也不是方法调用）。这个钩子仅在类中已经存在字段时被调用。*例外：* 如果 [__getattr__](https://docs.python.org/3/reference/datamodel.html#object.__getattr__) 或 [__getattribute__](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__) 是类中的方法，钩子将会被调用，以处理所有不引用方法的字段。

    **get_class_attribute_hook()** 类似于上面的钩子，但用于类上的属性，而不是实例上的属性。与上面不同，这个钩子没有对 [__getattr__](https://docs.python.org/3/reference/datamodel.html#object.__getattr__) 或 [__getattribute__](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__) 的特殊处理。

    **get_class_decorator_hook()** 用于更新给定类装饰器的类定义。例如，你可以向类中添加一些属性，以匹配运行时行为：

    ```python
    from dataclasses import dataclass

    @dataclass  # 内置插件在这里添加了 `__init__` 方法
    class User:
        name: str

    user = User(name='example')  # mypy 可以通过插件理解这一点
    ```

    **get_metaclass_hook()** 类似于上述钩子，但用于 metaclass。

    **get_base_class_hook()** 类似于上述钩子，但用于基类。

    **get_dynamic_class_hook()** 用于允许 mypy 中的动态类定义。这个插件钩子会在每个简单名称的赋值，其中右侧是一个函数调用时被调用：

    ```python
    from lib import dynamic_class

    X = dynamic_class('X', [])
    ```

    对于这样的定义，mypy 将调用 `get_dynamic_class_hook("lib.dynamic_class")`。插件应创建相应的 `mypy.nodes.TypeInfo` 对象，并将其放入相关的符号表中。（这个类的实例表示 mypy 中的类，并持有如限定名称、方法解析顺序等基本信息。）

    **get_customize_class_mro_hook()** 可以用来在类体分析之前修改类的 MRO（例如，插入一些条目）。

    **get_additional_deps()** 用于为模块添加新依赖项。它在语义分析之前被调用。例如，如果库有基于配置动态加载的依赖项，可以使用这个钩子。

    **report_config_data()** 用于插件具有某种每模块配置的情况，这些配置可能影响类型检查。在这种情况下，当模块的配置更改时，我们希望使 mypy 的缓存失效，以便重新检查模块。这个钩子应该用于报告任何相关的配置数据，以便 mypy 知道在配置更改时重新检查模块。钩子应返回可以编码为 JSON 的数据。

=== "英文"

    **get_type_analyze_hook()** customizes behaviour of the type analyzer. For example, [PEP 484](https://peps.python.org/pep-0484/) doesn't support defining variadic generic types:

    ```python

    from lib import Vector

    a: Vector[int, int]
    b: Vector[int, int, int]
    ```

    When analyzing this code, mypy will call ``get_type_analyze_hook("lib.Vector")``, so the plugin can return some valid type for each variable.

    **get_function_hook()** is used to adjust the return type of a function call. This hook will be also called for instantiation of classes. This is a good choice if the return type is too complex to be expressed by regular python typing.

    **get_function_signature_hook()** is used to adjust the signature of a function.

    **get_method_hook()** is the same as ``get_function_hook()`` but for methods instead of module level functions.

    **get_method_signature_hook()** is used to adjust the signature of a method. This includes special Python methods except [\_\_init\_\_()](https://docs.python.org/3/reference/datamodel.html#object.__init__) and [\_\_new\_\_()](https://docs.python.org/3/reference/datamodel.html#object.__new__). For example in this code:

    ```python
    from ctypes import Array, c_int

    x: Array[c_int]
    x[0] = 42
    ```

    mypy will call ``get_method_signature_hook("ctypes.Array.__setitem__")`` so that the plugin can mimic the [ctypes](https://docs.python.org/3/library/ctypes.html#module-ctypes) auto-convert behavior.

    **get_attribute_hook()** overrides instance member field lookups and property access (not assignments, and not method calls). This hook is only called for fields which already exist on the class. *Exception:* if [\_\_getattr\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattr__) or [\_\_getattribute\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__) is a method on the class, the hook is called for all fields which do not refer to methods.

    **get_class_attribute_hook()** is similar to above, but for attributes on classes rather than instances. Unlike above, this does not have special casing for [\_\_getattr\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattr__) or [\_\_getattribute\_\_](https://docs.python.org/3/reference/datamodel.html#object.__getattribute__).

    **get_class_decorator_hook()** can be used to update class definition for given class decorators. For example, you can add some attributes to the class to match runtime behaviour:

    ```python
    from dataclasses import dataclass

    @dataclass  # built-in plugin adds `__init__` method here
    class User:
        name: str

    user = User(name='example')  # mypy can understand this using a plugin
    ```

    **get_metaclass_hook()** is similar to above, but for metaclasses.

    **get_base_class_hook()** is similar to above, but for base classes.

    **get_dynamic_class_hook()** can be used to allow dynamic class definitions in mypy. This plugin hook is called for every assignment to a simple name where right hand side is a function call:

    ```python
    from lib import dynamic_class

    X = dynamic_class('X', [])
    ```

    For such definition, mypy will call ``get_dynamic_class_hook("lib.dynamic_class")``. The plugin should create the corresponding ``mypy.nodes.TypeInfo`` object, and place it into a relevant symbol table. (Instances of this class represent classes in mypy and hold essential information such as qualified name, method resolution order, etc.)

    **get_customize_class_mro_hook()** can be used to modify class MRO (for example insert some entries there) before the class body is analyzed.

    **get_additional_deps()** can be used to add new dependencies for a module. It is called before semantic analysis. For example, this can be used if a library has dependencies that are dynamically loaded based on configuration information.

    **report_config_data()** can be used if the plugin has some sort of per-module configuration that can affect typechecking. In that case, when the configuration for a module changes, we want to invalidate mypy's cache for that module so that it can be rechecked. This hook should be used to report to mypy any relevant configuration data, so that mypy knows to recheck the module if the configuration changes. The hooks should return data encodable as JSON.

## 实用工具

**Useful tools**

=== "中文"

    Mypy 提供了 `mypy.plugins.proper_plugin` 插件，这对于插件作者来说非常有用，因为它可以帮助发现遗漏的 `get_proper_type()` 调用，这是一个非常常见的错误。

    建议将其作为插件 CI 流程的一部分启用。

=== "英文"

    Mypy ships ``mypy.plugins.proper_plugin`` plugin which can be useful for plugin authors, since it finds missing ``get_proper_type()`` calls, which is a pretty common mistake.

    It is recommended to enable it is a part of your plugin's CI.
