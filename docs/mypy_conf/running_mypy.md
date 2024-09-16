# 运行mypy和管理导入

**Running mypy and managing imports**

=== "中文"

    [入门]页面应该已经向你介绍了如何运行 mypy 的基本方法——通过命令行传递你想进行类型检查的文件和目录。

    ```shell
    $ mypy foo.py bar.py some_directory
    ```

    本页面将更详细地讨论如何具体指定你希望 mypy 进行类型检查的文件，mypy 如何发现导入的模块，以及处理你可能遇到的任何问题的建议。

    如果你有兴趣了解如何配置 mypy 实际上是如何对你的代码进行类型检查的，请参阅我们的 [命令行] 指南。

=== "英文"

    The [getting-started] page should have already introduced you to the basics of how to run mypy -- pass in the files and directories you want to type check via the command line

    ```shell
    $ mypy foo.py bar.py some_directory
    ```

    This page discusses in more detail how exactly to specify what files you want mypy to type check, how mypy discovers imported modules, and recommendations on how to handle any issues you may encounter along the way.

    If you are interested in learning about how to configure the actual way mypy type checks your code, see our [command-line] guide.

## 指定要检查的代码

**Specifying code to be checked**

=== "中文"

    Mypy 允许你通过几种不同的方式指定要进行类型检查的文件。

    1. 首先，你可以传递 Python 文件和目录的路径，以便进行类型检查。例如：

        ```shell
        $ mypy file_1.py foo/file_2.py file_3.pyi some/directory
        ```

        上述命令告诉 mypy 对所有提供的文件进行类型检查。此外，mypy 会递归地对任何提供的目录中的全部内容进行类型检查。

        有关具体执行方式的更多细节，请参见 [将文件路径映射到模块 <mapping-paths-to-modules>]。

    2. 其次，你可以使用 [-m] 标志（长格式为 [--module]）来指定要进行类型检查的模块名称。模块的名称与在 Python 程序中导入该模块时使用的名称相同。例如，运行：

        ```shell
        $ mypy -m html.parser
        ```

        ...将对模块 ``html.parser`` 进行类型检查（这恰好是一个库存根）。

        Mypy 将使用类似于 Python 用来查找模块和导入的位置的算法。有关更多细节，请参见 [查找导入的模块]。

    3. 第三，你可以使用 [-p] 标志（长格式为 [--package]）来指定要（递归）进行类型检查的包。这个标志与 [-m] 标志几乎相同，只是如果你提供一个包名，mypy 将递归地对该包的所有子模块和子包进行类型检查。例如，运行：

        ```shell
        $ mypy -p html
        ```

        ...将对整个 ``html`` 包（库存根）进行类型检查。相比之下，如果使用 [-m] 标志，mypy 只会对 ``html`` 的 ``__init__.py`` 文件及其导入的内容进行类型检查。

        请注意，我们可以在命令行上指定多个包和模块。例如：

        ```shell
        $ mypy --package p.a --package p.b --module c
        ```

    4. 第四，你还可以通过使用 [-c] 标志（长格式为 [--command]）直接对小的字符串进行类型检查。例如：

        ```shell
        $ mypy -c 'x = [1, 2]; print(x())'
        ```

        ...将把上述字符串作为迷你程序进行类型检查（在这种情况下，会报告 ``list[int]`` 不能被调用）。

    你还可以在 `mypy.ini` 文件中的 [files](./config_file.md#files) 选项里指定要检查的文件，这样你可以简单地运行 ``mypy`` 而无需任何参数。

=== "英文"

    Mypy lets you specify what files it should type check in several different ways.

    1.  First, you can pass in paths to Python files and directories you want to type check. For example

        ```shell
        $ mypy file_1.py foo/file_2.py file_3.pyi some/directory
        ```

        The above command tells mypy it should type check all of the provided files together. In addition, mypy will recursively type check the entire contents of any provided directories.

        For more details about how exactly this is done, see [Mapping file paths to modules <mapping-paths-to-modules>].

    2.  Second, you can use the [-m] flag (long form: [--module]) to specify a module name to be type checked. The name of a module is identical to the name you would use to import that module within a Python program. For example, running

        ```shell
        $ mypy -m html.parser
        ```

        ...will type check the module ``html.parser`` (this happens to be a library stub).

        Mypy will use an algorithm very similar to the one Python uses to find where modules and imports are located on the file system. For more details, see [finding-imports].

    3.  Third, you can use the [-p] (long form: [--package]) flag to specify a package to be (recursively) type checked. This flag is almost identical to the [-m] flag except that if you give it a package name, mypy will recursively type check all submodules and subpackages of that package. For example, running

        ```shell
        $ mypy -p html
        ```

        ...will type check the entire ``html`` package (of library stubs). In contrast, if we had used the [-m] flag, mypy would have type checked just ``html``'s ``__init__.py`` file and anything imported from there.

        Note that we can specify multiple packages and modules on the command line. For example

        ```shell
        $ mypy --package p.a --package p.b --module c
        ```

    4.  Fourth, you can also instruct mypy to directly type check small strings as programs by using the [-c] (long form: [--command]) flag. For example

        ```shell
        $ mypy -c 'x = [1, 2]; print(x())'
        ```

        ...will type check the above string as a mini-program (and in this case, will report that ``list[int]`` is not callable).

    You can also use the [files](./config_file.md#files) option in your `mypy.ini` file to specify which files to check, in which case you can simply run ``mypy`` with no arguments.

## 从文件中读取文件列表

**Reading a list of files from a file**

=== "中文"

    最后，任何以 ``@`` 开头的命令行参数会从紧随其后的文件中读取额外的命令行参数。这在你有一个包含要进行类型检查的文件列表的文件时特别有用：你可以避免使用类似于以下的 shell 语法：

    ```shell
    $ mypy $(cat file_of_files.txt)
    ```

    而改为使用以下命令：

    ```shell
    $ mypy @file_of_files.txt
    ```

    该文件技术上也可以包含任何命令行标志，而不仅仅是文件路径。然而，如果你想配置很多不同的标志，推荐的做法是使用 [配置文件](./config_file.md)。

=== "英文"

    Finally, any command-line argument starting with ``@`` reads additional command-line arguments from the file following the ``@`` character. This is primarily useful if you have a file containing a list of files that you want to be type-checked: instead of using shell syntax like

    ```shell
    $ mypy $(cat file_of_files.txt)
    ```

    you can use this instead

    ```shell
    $ mypy @file_of_files.txt
    ```

    This file can technically also contain any command line flag, not just file paths. However, if you want to configure many different flags, the recommended approach is to use a [configuration file](./config_file.md) instead.

## 映射文件路径到模块

**Mapping file paths to modules**

=== "中文"

    你可以通过提供路径列表来告诉 mypy 进行类型检查。这是主要的方式之一。例如：

    ```shell
    $ mypy file_1.py foo/file_2.py file_3.pyi some/directory
    ```

    本节描述了 mypy 如何将提供的路径映射到要进行类型检查的模块。

    - mypy 会检查所有对应于文件的提供路径。

    - mypy 会递归地发现并检查所有以 ``.py`` 或 ``.pyi`` 结尾的文件，位于提供的目录路径中，同时考虑到 [--exclude](./command_line.md#exclude) 选项。

    - 对于每个需要检查的文件，mypy 会尝试将该文件（例如 ``project/foo/bar/baz.py``）与一个完全限定的模块名称（例如 ``foo.bar.baz``）相关联。包所在的目录（``project``）会被添加到 mypy 的模块搜索路径中。

    mypy 如何确定完全限定的模块名称取决于选项 [--no-namespace-packages](./command_line.md#no-namespace-packages) 和 [--explicit-package-bases](./command_line.md#explicit-package-bases) 是否被设置。

    1. 如果设置了 [--no-namespace-packages](./command_line.md#no-namespace-packages)，mypy 将仅依赖于 ``__init__.py[i]`` 文件的存在来确定完全限定的模块名称。即，mypy 会向上遍历目录树，只要继续找到 ``__init__.py``（或 ``__init__.pyi``）文件，就继续向上查找。

        例如，如果你的目录树包括 ``pkg/subpkg/mod.py``，mypy 将要求 ``pkg/__init__.py`` 和 ``pkg/subpkg/__init__.py`` 存在，以便正确地将 ``mod.py`` 关联为 ``pkg.subpkg.mod``。

    2. 默认情况。如果 [--namespace-packages](./command_line.md#namespace-packages) 选项开启，但 [--explicit-package-bases](./command_line.md#explicit-package-bases) 选项关闭，mypy 会允许没有 ``__init__.py[i]`` 的目录被视为包。具体来说，mypy 会查看文件的所有父目录，并使用目录树中最高的 ``__init__.py[i]`` 的位置来确定顶级包。

        例如，假设你的目录树仅包含 ``pkg/__init__.py`` 和 ``pkg/a/b/c/d/mod.py``。在确定 ``mod.py`` 的完全限定模块名称时，mypy 会查看 ``pkg/__init__.py`` 并得出关联的模块名称是 ``pkg.a.b.c.d.mod``。

    3. 你会注意到，上述情况仍然依赖于 ``__init__.py``。如果你不能在顶级包中放置 ``__init__.py``，但仍希望传递路径（而不是使用 ``-p`` 或 ``-m`` 标志的包或模块），[--explicit-package-bases](./command_line.md#explicit-package-bases) 提供了一个解决方案。

        使用 [--explicit-package-bases](./command_line.md#explicit-package-bases) 时，mypy 会找到 ``MYPYPATH`` 环境变量、[mypy_path](./config_file.md#mypy_path) 配置或当前工作目录中的最近的父目录。然后，mypy 将使用相对路径来确定完全限定的模块名称。

        例如，假设你的目录树仅包含 ``src/namespace_pkg/mod.py``。如果你运行以下命令，mypy 将正确地将 ``mod.py`` 关联为 ``namespace_pkg.mod``：

        ```shell
        $ MYPYPATH=src mypy --namespace-packages --explicit-package-bases .
        ```

    如果你传递的文件不以 ``.py[i]`` 结尾，假定的模块名称是 ``__main__``（与 Python 解释器的行为一致），除非传递了 [--scripts-are-modules](./command_line.md#scripts-are-modules) 选项。

    传递 [-v](./command_line.md#v) 选项将显示 mypy 将要检查的文件及其关联的模块名称。

=== "英文"

    One of the main ways you can tell mypy what to type check is by providing mypy a list of paths. For example

    ```shell
    $ mypy file_1.py foo/file_2.py file_3.pyi some/directory
    ```

    This section describes how exactly mypy maps the provided paths to modules to type check.

    - Mypy will check all paths provided that correspond to files.

    - Mypy will recursively discover and check all files ending in ``.py`` or ``.pyi`` in directory paths provided, after accounting for [--exclude](./command_line.md#exclude).

    - For each file to be checked, mypy will attempt to associate the file (e.g. ``project/foo/bar/baz.py``) with a fully qualified module name (e.g. ``foo.bar.baz``). The directory the package is in (``project``) is then added to mypy's module search paths.

    How mypy determines fully qualified module names depends on if the options
    [--no-namespace-packages](./command_line.md#no-namespace-packages) and
    [--explicit-package-bases](./command_line.md#explicit-package-bases) are set.

    1. If [--no-namespace-packages](./command_line.md#no-namespace-packages) is set, mypy will rely solely upon the presence of ``__init__.py[i]`` files to determine the fully qualified module name. That is, mypy will crawl up the directory tree for as long as it continues to find ``__init__.py`` (or ``__init__.pyi``) files.

        For example, if your directory tree consists of ``pkg/subpkg/mod.py``, mypy would require ``pkg/__init__.py`` and ``pkg/subpkg/__init__.py`` to exist in order correctly associate ``mod.py`` with ``pkg.subpkg.mod``

    2. The default case. If [--namespace-packages](./command_line.md#namespace-packages) is on, but [--explicit-package-bases](./command_line.md#explicit-package-bases) is off, mypy will allow for the possibility that
        directories without ``__init__.py[i]`` are packages. Specifically, mypy will
        look at all parent directories of the file and use the location of the
        highest ``__init__.py[i]`` in the directory tree to determine the top-level
        package.

        For example, say your directory tree consists solely of ``pkg/__init__.py``
        and ``pkg/a/b/c/d/mod.py``. When determining ``mod.py``'s fully qualified
        module name, mypy will look at ``pkg/__init__.py`` and conclude that the
        associated module name is ``pkg.a.b.c.d.mod``.

    3. You'll notice that the above case still relies on ``__init__.py``. If you can't put an ``__init__.py`` in your top-level package, but still wish to pass paths (as opposed to packages or modules using the ``-p`` or ``-m`` flags), [--explicit-package-bases](./command_line.md#explicit-package-bases) provides a solution.

        With [--explicit-package-bases](./command_line.md#explicit-package-bases), mypy will locate the nearest parent directory that is a member of the ``MYPYPATH`` environment variable, the [mypy_path](./config_file.md#mypy_path) config or is the current working directory. Mypy will then use the relative path to determine the fully qualified module name.

        For example, say your directory tree consists solely of ``src/namespace_pkg/mod.py``. If you run the following command, mypy will correctly associate ``mod.py`` with ``namespace_pkg.mod``

        ```shell
        $ MYPYPATH=src mypy --namespace-packages --explicit-package-bases .
        ```

    If you pass a file not ending in ``.py[i]``, the module name assumed is ``__main__`` (matching the behavior of the Python interpreter), unless [--scripts-are-modules](./command_line.md#scripts-are-modules) is passed.

    Passing [-v](./command_line.md#v) will show you the files and associated module names that mypy will check.

## mypy 如何处理导入

**How mypy handles imports**

=== "中文"

    当 mypy 遇到 ``import`` 语句时，它会首先 [尝试定位](#如何找到导入) 文件系统中的模块或该模块的类型存根。然后，mypy 将对导入的模块进行类型检查。这个过程有三种不同的结果：

    1. **mypy 无法跟随导入**：模块要么不存在，要么是一个不使用类型提示的第三方库。

    2. **mypy 能够跟随并进行类型检查，但你不希望 mypy 对该模块进行类型检查**。

    3. **mypy 能够成功地跟随并进行类型检查，并且你希望 mypy 对该模块进行类型检查**。

    第三种结果是理想情况下 mypy 会执行的操作。接下来的章节将讨论如何处理其他两种情况。

=== "英文"

    When mypy encounters an ``import`` statement, it will first [attempt to locate](#如何找到导入) that module or type stubs for that module in the file system. Mypy will then type check the imported module. There are three different outcomes of this process:

    1.  Mypy is unable to follow the import: the module either does not exist, or is a third party library that does not use type hints.

    2.  Mypy is able to follow and type check the import, but you did not want mypy to type check that module at all.

    3.  Mypy is able to successfully both follow and type check the module, and you want mypy to type check that module.

    The third outcome is what mypy will do in the ideal case. The following sections will discuss what to do in the other two cases.

## 缺失的导入

**Missing imports**

=== "中文"

    当你导入一个模块时，mypy 可能会报告它无法跟随该导入。这可能会导致如下所示的错误：

    ```text
    main.py:1: error: Skipping analyzing 'django': module is installed, but missing library stubs or py.typed marker
    main.py:2: error: Library stubs not installed for "requests"
    main.py:3: error: Cannot find implementation or library stub for module named "this_module_does_not_exist"
    ```

    如果你在导入时遇到这些错误，mypy 将假定该模块的类型为 ``Any``，即动态类型。这意味着访问该模块的任何属性都会自动成功：

    ```python
    # 错误: 找不到名为 'does_not_exist' 的实现或库存根
    import does_not_exist

    # 但这段代码会通过类型检查，x 的类型为 'Any'
    x = does_not_exist.foobar()
    ```

    这可能导致 mypy 无法警告你代码中的错误。由于对 ``Any`` 的操作结果仍然是 ``Any``，这些动态类型可以在代码中传播，使类型检查效果降低。有关更多信息，请参见 [动态类型](../mypy/dynamic_typing.md)。

    接下来的章节描述了这些错误的含义以及推荐的解决步骤；请滚动到与您的错误相匹配的部分。

=== "英文"

    When you import a module, mypy may report that it is unable to follow the import. This can cause errors that look like the following:

    ```text
    main.py:1: error: Skipping analyzing 'django': module is installed, but missing library stubs or py.typed marker
    main.py:2: error: Library stubs not installed for "requests"
    main.py:3: error: Cannot find implementation or library stub for module named "this_module_does_not_exist"
    ```

    If you get any of these errors on an import, mypy will assume the type of that module is ``Any``, the dynamic type. This means attempting to access any attribute of the module will automatically succeed:

    ```python
    # Error: Cannot find implementation or library stub for module named 'does_not_exist'
    import does_not_exist

    # But this type checks, and x will have type 'Any'
    x = does_not_exist.foobar()
    ```

    This can result in mypy failing to warn you about errors in your code. Since operations on ``Any`` result in ``Any``, these dynamic types can propagate through your code, making type checking less effective. See [dynamic-typing](../mypy/dynamic_typing.md) for more information.

    The next sections describe what each of these errors means and recommended next steps; scroll to the section that matches your error.

### 缺失的库存根或 py.typed 标记

**Missing library stubs or py.typed marker**

=== "中文"

    如果你遇到错误信息 ``Skipping analyzing X: module is installed, but missing library stubs or py.typed marker``，这意味着 mypy 能够找到你导入的模块，但没有找到相应的类型提示。

    除非第三方库已声明为[PEP 561 合规的存根包](./installed_packages.md)（例如，包含 ``py.typed`` 文件），或者已在 [typeshed](https://github.com/python/typeshed) 注册，mypy 不会尝试推断你安装的任何第三方库的类型提示。typeshed 是一个标准库及一些第三方库的类型存储库。

    如果你遇到这个错误，请尝试获取你正在使用的库的类型提示：

    1. **升级库的版本**，以查看较新版本是否开始包含类型提示。

    2. **搜索是否有与第三方库对应的 [PEP 561 合规存根包](./installed_packages.md)**。存根包允许你独立于库本身安装类型提示。

        例如，如果你需要 ``django`` 库的类型提示，可以安装 [django-stubs](https://pypi.org/project/django-stubs/) 包。

    3. **[编写自己的存根文件](../mypy/stub_files.md)**，包含库的类型提示。你可以通过命令行传递、使用 [files](./config_file.md#files) 或 [mypy_path](./config_file.md#mypy_path) 配置文件选项，或将位置添加到 ``MYPYPATH`` 环境变量来指向你的类型提示。

        这些存根文件不需要完整！一个好的策略是使用 [stubgen <stubgen>](../mypy/stubgen.md) 生成存根的初步草稿。然后你可以仅对需要的库部分进行迭代。

        如果你想分享你的工作，可以尝试将存根贡献回库——请参阅我们关于创建 [PEP 561 合规包](./installed_packages.md) 的文档。

    如果你无法找到现有的类型提示或没有时间编写自己的提示，你可以选择 *抑制* 错误。

    这将使 mypy 停止报告包含导入的行上的错误：导入的模块将继续为 ``Any`` 类型，mypy 可能无法捕捉其使用中的错误。

    1. **要抑制单个** 缺失导入错误，请在包含导入的行末尾添加 ``# type: ignore``。

    2. **要抑制单个库的所有** 缺失导入错误，请在 [mypy 配置文件](./config_file.md) 中添加每个模块的部分，将 [ignore_missing_imports](./config_file.md#ignore_missing_imports) 设置为 True。例如，假设你的代码库大量使用了一个（未类型化的）库名为 ``foobar``。你可以通过在配置文件中添加以下部分来消除与该库相关的所有导入错误：

        ```ini
        [mypy-foobar.*]
        ignore_missing_imports = True
        ```

        注意：这个选项等同于在你的代码库中的每个 ``foobar`` 导入上添加 ``# type: ignore``。有关更多信息，请参见关于配置 [导入发现](./config_file.md#导入发现) 的文档。``.*`` 后缀将忽略 ``foobar`` 模块和子包的导入，除了 ``foobar`` 顶级包命名空间。

    3. **要抑制代码库中所有未类型化库的所有** 缺失导入错误，请使用 [disable-error-code=import-untyped](./command_line.md#ignore-missing-imports)。有关此错误代码的更多详细信息，请参见 [code-import-untyped](../mypy_other/error_code_list.md#检查导入目标是否可以找到-import-untyped)。

        你也可以设置 [disable_error_code](./config_file.md#disable_error_code)，如下所示：

        ```ini
        [mypy]
        disable_error_code = import-untyped
        ```

        你还可以使用 [--ignore-missing-imports](./command_line.md#ignore-missing-imports) 命令行标志，或将 [ignore_missing_imports](./config_file.md#ignore_missing_imports) 配置文件选项设置为 True 在 mypy 配置文件的 *global* 部分。我们建议尽可能避免使用 ``--ignore-missing-imports``：它等同于在代码库中所有未解析的导入上添加 ``# type: ignore``。

=== "英文"

    If you are getting a ``Skipping analyzing X: module is installed, but missing library stubs or py.typed marker``, error, this means mypy was able to find the module you were importing, but no corresponding type hints.

    Mypy will not try inferring the types of any 3rd party libraries you have installed unless they either have declared themselves to be [PEP 561 compliant stub package](./installed_packages.md) (e.g. with a ``py.typed`` file) or have registered themselves on [typeshed](https://github.com/python/typeshed), the repository of types for the standard library and some 3rd party libraries.

    If you are getting this error, try to obtain type hints for the library you're using:

    1.  Upgrading the version of the library you're using, in case a newer version has started to include type hints.

    2.  Searching to see if there is a [PEP 561 compliant stub package][PEP 561 compliant stub package](./installed_packages.md) corresponding to your third party library. Stub packages let you install type hints independently from the library itself.

        For example, if you want type hints for the ``django`` library, you can install the [django-stubs](https://pypi.org/project/django-stubs/) package.

    3.  [Writing your own stub files](../mypy/stub_files.md) containing type hints for the library. You can point mypy at your type hints either by passing them in via the command line, by using the  [files](./config_file.md#files) or [mypy_path](./config_file.md#mypy_path) config file options, or by adding the location to the ``MYPYPATH`` environment variable.

        These stub files do not need to be complete! A good strategy is to use [stubgen <stubgen>], a program that comes bundled with mypy, to generate a first rough draft of the stubs. You can then iterate on just the parts of the library you need.

        If you want to share your work, you can try contributing your stubs back to the library -- see our documentation on creating [PEP 561 compliant packages](./installed_packages.md).

    If you are unable to find any existing type hints nor have time to write your own, you can instead *suppress* the errors.

    All this will do is make mypy stop reporting an error on the line containing the import: the imported module will continue to be of type ``Any``, and mypy may not catch errors in its use.

    1.  To suppress a *single* missing import error, add a ``# type: ignore`` at the end of the line containing the import.

    2.  To suppress *all* missing import errors from a single library, add a per-module section to your [mypy config file](./config_file.md) setting [ignore_missing_imports](./config_file.md#ignore_missing_imports) to True for that library. For example, suppose your codebase makes heavy use of an (untyped) library named ``foobar``. You can silence all import errors associated with that library and that library alone by adding the following section to your config file
        
        ```ini  
            [mypy-foobar.*]
            ignore_missing_imports = True
        ```

        Note: this option is equivalent to adding a ``# type: ignore`` to every import of ``foobar`` in your codebase. For more information, see the documentation about configuring [import discovery](./config_file.md#导入发现) in config files. The ``.*`` after ``foobar`` will ignore imports of ``foobar`` modules and subpackages in addition to the ``foobar`` top-level package namespace.

    3.  To suppress *all* missing import errors for *all* untyped libraries in your codebase, use [disable-error-code=import-untyped](./command_line.md#ignore-missing-imports). See [code-import-untyped](../mypy_other/error_code_list.md#检查导入目标是否可以找到-import-untyped) for more details on this error code.

        You can also set [disable_error_code](./config_file.md#disable_error_code), like so
        
        ```ini
        [mypy]
        disable_error_code = import-untyped
        ```

        You can also set the [--ignore-missing-imports](./command_line.md#ignore-missing-imports) command line flag or set the [ignore_missing_imports](./config_file.md#ignore_missing_imports) config file option to True in the *global* section of your mypy config file. We recommend avoiding ``--ignore-missing-imports`` if possible: it's equivalent to adding a ``# type: ignore`` to all unresolved imports in your codebase.

### 库存根未安装

**Library stubs not installed**

=== "中文"

    如果 mypy 无法找到某个第三方库的存根文件，并且它知道该库有存根文件，你会看到类似以下的消息：

    ```text
    main.py:1: error: Library stubs not installed for "yaml"
    main.py:1: note: Hint: "python3 -m pip install types-PyYAML"
    main.py:1: note: (or run "mypy --install-types" to install all missing stub packages)
    ```

    你可以通过运行建议的 pip 命令来解决这个问题。如果你在 CI 环境中运行 mypy，你可以像处理其他测试依赖一样确保所需的存根包的存在，例如，通过将它们添加到适当的 ``requirements.txt`` 文件中。

    另一种方法是向你的 mypy 命令添加 [--install-types](./command_line.md#install-types) 选项，以安装所有已知的缺失存根：

    ```shell
    mypy --install-types
    ```

    这比显式安装存根要慢，因为它实际上会运行两次 mypy——第一次是查找缺失的存根，第二次是在 mypy 安装存根后正确地类型检查你的代码。这也可能使控制存根版本变得更加困难，从而导致类型检查的可重现性降低。

    默认情况下，[--install-types](./command_line.md#install-types) 会显示一个确认提示。使用 [--non-interactive](./command_line.md#non-interactive) 可以在不需要确认的情况下安装所有建议的存根包 *并且* 类型检查你的代码：

    如果你已经在 mypy 运行所在环境之外的环境中安装了相关的第三方库，你可以使用 [--python-executable](./command_line.md#python-executable) 标志来指定该环境的 Python 可执行文件，mypy 将找到为该 Python 可执行文件安装的包。

    如果你已经安装了相关的存根包，但仍然遇到此错误，请参阅 [下面的部分](#无法找到实现或库存根)。

=== "英文"

    If mypy can't find stubs for a third-party library, and it knows that stubs exist for the library, you will get a message like this:

    ```text
    main.py:1: error: Library stubs not installed for "yaml"
    main.py:1: note: Hint: "python3 -m pip install types-PyYAML"
    main.py:1: note: (or run "mypy --install-types" to install all missing stub packages)
    ```

    You can resolve the issue by running the suggested pip commands. If you're running mypy in CI, you can ensure the presence of any stub packages you need the same as you would any other test dependency, e.g. by adding them to the appropriate ``requirements.txt`` file.

    Alternatively, add the [--install-types](./command_line.md#install-types) to your mypy command to install all known missing stubs:

    ```shell
    mypy --install-types
    ```

    This is slower than explicitly installing stubs, since it effectively runs mypy twice -- the first time to find the missing stubs, and the second time to type check your code properly after mypy has installed the stubs. It also can make controlling stub versions harder, resulting in less reproducible type checking.

    By default, [--install-types](./command_line.md#install-types) shows a confirmation prompt. Use [--non-interactive](./command_line.md#non-interactive) to install all suggested stub packages without asking for confirmation *and* type check your code:

    If you've already installed the relevant third-party libraries in an environment other than the one mypy is running in, you can use [--python-executable](./command_line.md#python-executable) flag to point to the Python executable for that environment, and mypy will find packages installed for that Python executable.

    If you've installed the relevant stub packages and are still getting this error, see the [section below](#无法找到实现或库存根).

### 无法找到实现或库存根

**Cannot find implementation or library stub**

=== "中文"

    如果你遇到 ``Cannot find implementation or library stub for module`` 错误，这意味着 mypy 无法找到你试图导入的模块，无论该模块是否包含类型提示。如果你遇到这个错误，请尝试以下方法：

    1. **确保你的导入语句没有拼写错误。**

    2. **如果模块是第三方库，确保 mypy 能够找到包含已安装库的解释器。**

        例如，如果你在 virtualenv 中运行代码，请确保在 virtualenv 中安装和使用 mypy。或者，如果你希望使用全局安装的 mypy，请设置 [--python-executable](./command_line.md#python-executable) 命令行标志，指向包含已安装第三方包的 Python 解释器。

        你可以通过像 ``python -m mypy ...`` 这样运行 mypy 来确认你是否在预期的环境中运行它。你可以通过像 ``python -m pip ...`` 这样运行 pip 来确认你是否在预期的环境中安装包。

    3. **阅读下面的 [finding-imports](#如何找到导入) 部分，以确保你理解 mypy 是如何搜索和找到模块的，并根据需要调整你调用 mypy 的方式。**

    4. **通过命令行直接指定包含你想要类型检查的模块的目录，使用 [mypy_path](./config_file.md#mypy_path) 或 [files](./config_file.md#files) 配置文件选项，或通过设置 ``MYPYPATH`` 环境变量。**

        注意：如果你试图导入的模块实际上是某个包的 *子模块*，你应该指定包含 *整个* 包的目录。例如，假设你要添加模块 ``foo.bar.baz``，它位于 ``~/foo-project/src/foo/bar/baz.py``。在这种情况下，你必须运行 ``mypy ~/foo-project/src``（或将 ``MYPYPATH`` 设置为 ``~/foo-project/src``）。

=== "英文"

    If you are getting a ``Cannot find implementation or library stub for module`` error, this means mypy was not able to find the module you are trying to import, whether it comes bundled with type hints or not. If you are getting this error, try:

    1.  Making sure your import does not contain a typo.

    2.  If the module is a third party library, making sure that mypy is able to find the interpreter containing the installed library.

        For example, if you are running your code in a virtualenv, make sure to install and use mypy within the virtualenv. Alternatively, if you want to use a globally installed mypy, set the [--python-executable](./command_line.md#python-executable) command line flag to point the Python interpreter containing your installed third party packages.

        You can confirm that you are running mypy from the environment you expect by running it like ``python -m mypy ...``. You can confirm that you are installing into the environment you expect by running pip like ``python -m pip ...``.

    3.  Reading the [finding-imports](#如何找到导入) section below to make sure you
        understand how exactly mypy searches for and finds modules and modify
        how you're invoking mypy accordingly.

    4.  Directly specifying the directory containing the module you want to type check from the command line, by using the [mypy_path](./config_file.md#mypy_path) or [files](./config_file.md#files) config file options, or by using the ``MYPYPATH`` environment variable.

        Note: if the module you are trying to import is actually a *submodule* of some package, you should specify the directory containing the *entire* package. For example, suppose you are trying to add the module ``foo.bar.baz`` which is located at ``~/foo-project/src/foo/bar/baz.py``. In this case, you must run ``mypy ~/foo-project/src`` (or set the ``MYPYPATH`` to ``~/foo-project/src``).

## 如何找到导入

**How imports are found**

=== "中文"

    当 mypy 遇到 ``import`` 语句或通过 [--module](./command_line.md#m) 或 [--package](./command_line.md#p) 标志从命令行接收模块名称时，mypy 尝试在文件系统上找到该模块，这与 Python 查找模块的方式类似。然而，存在一些差异。

    首先，mypy 有自己的搜索路径。这些路径由以下项目计算得出：

    - ``MYPYPATH`` 环境变量（在 UNIX 系统上用冒号分隔的目录列表，在 Windows 上用分号分隔）。
    - [mypy_path](./config_file.md#mypy_path) 配置文件选项。
    - 命令行上提供的源文件所在的目录（参见 [将文件路径映射到模块](#映射文件路径到模块)）。
    - 被标记为类型检查安全的已安装包（参见 [PEP 561 支持](./installed_packages.md)）。
    - [typeshed](https://github.com/python/typeshed) 仓库中的相关目录。

    !!! note

        你不能通过 ``MYPYPATH`` 指向一个仅包含存根的包 ([561](https://peps.python.org/pep-0561/))，它必须已安装（参见 [PEP 561 支持](./installed_packages.md)）。

    其次，mypy 除了常规的 Python 文件和包，还会搜索存根文件。搜索模块 ``foo`` 的规则如下：

    - 搜索会在搜索路径中的每个目录中进行，直到找到匹配项。
    - 如果找到名为 ``foo`` 的包（即包含 ``__init__.py`` 或 ``__init__.pyi`` 文件的目录 ``foo``），这就是匹配项。
    - 如果找到名为 ``foo.pyi`` 的存根文件，这也是匹配项。
    - 如果找到名为 ``foo.py`` 的 Python 模块，这也是匹配项。

    这些匹配项会按顺序尝试，因此如果在搜索路径中的同一目录中找到多个匹配项（例如一个包和一个 Python 文件，或一个存根文件和一个 Python 文件），则上述列表中的第一个匹配项优先。

    特别地，如果同一目录中既有 Python 文件又有存根文件，则只使用存根文件。（不过，如果这些文件在不同的目录中，则使用在较早目录中找到的文件。）

    设置 [mypy_path](./config_file.md#mypy_path)/``MYPYPATH`` 在你希望尝试运行 mypy 对多个不同的文件集进行检查，而这些文件集恰好共享一些公共依赖项的情况下特别有用。

    例如，如果你有多个项目恰好使用相同的一组正在开发中的存根，方便的做法可能是让你的 ``MYPYPATH`` 指向包含这些存根的单一目录。

=== "英文"

    When mypy encounters an ``import`` statement or receives module names from the command line via the [--module](./command_line.md#m) or [--package](./command_line.md#p) flags, mypy tries to find the module on the file system similar to the way Python finds it. However, there are some differences.

    First, mypy has its own search path. This is computed from the following items:

    - The ``MYPYPATH`` environment variable (a list of directories, colon-separated on UNIX systems, semicolon-separated on Windows).
    - The [mypy_path](./config_file.md#mypy_path) config file option.
    - The directories containing the sources given on the command line (see [Mapping file paths to modules](#映射文件路径到模块)).
    - The installed packages marked as safe for type checking (see [PEP 561 support](./installed_packages.md))
    - The relevant directories of the [typeshed](https://github.com/python/typeshed) repo.

    !!! note

        You cannot point to a stub-only package ([561](https://peps.python.org/pep-0561/)) via the ``MYPYPATH``, it must be installed (see [PEP 561 support](./installed_packages.md))

    Second, mypy searches for stub files in addition to regular Python files and packages. The rules for searching for a module ``foo`` are as follows:

    - The search looks in each of the directories in the search path (see above) until a match is found.
    - If a package named ``foo`` is found (i.e. a directory ``foo`` containing an ``__init__.py`` or ``__init__.pyi`` file) that's a match.
    - If a stub file named ``foo.pyi`` is found, that's a match.
    - If a Python module named ``foo.py`` is found, that's a match.

    These matches are tried in order, so that if multiple matches are found in the same directory on the search path (e.g. a package and a Python file, or a stub file and a Python file) the first one in the above list wins.

    In particular, if a Python file and a stub file are both present in the same directory on the search path, only the stub file is used. (However, if the files are in different directories, the one found in the earlier directory is used.)

    Setting [mypy_path](./config_file.md#mypy_path)/``MYPYPATH`` is mostly useful in the case where you want to try running mypy against multiple distinct sets of files that happen to share some common dependencies.

    For example, if you have multiple projects that happen to be using the same set of work-in-progress stubs, it could be convenient to just have your ``MYPYPATH`` point to a single directory containing the stubs.

## 跟随导入

**Following imports**

=== "中文"

    Mypy 设计为[坚定地跟随所有导入](#如何找到导入)，即使导入的模块不是你明确希望 mypy 检查的文件。

    例如，假设我们有两个模块 ``mycode.foo`` 和 ``mycode.bar``：前者有类型提示，而后者没有。我们运行 [mypy -m mycode.foo](./command_line.md#m)，mypy 发现 ``mycode.foo`` 导入了 ``mycode.bar``。

    我们希望 mypy 如何检查 ``mycode.bar`` 呢？mypy 在这里的行为是可配置的——尽管我们 **强烈推荐** 使用默认设置——通过 [--follow-imports](./command_line.md#follow-imports) 标志。这个标志接受以下四种字符串值之一：

    - ``normal``（默认推荐）会正常跟随所有导入并类型检查所有顶级代码（以及所有函数和方法中具有至少一个类型注解的代码体）。

    - ``silent`` 的行为与 ``normal`` 相同，但会额外 *抑制* 任何错误消息。

    - ``skip`` 将 *不* 跟随导入，而是静默地用类型 ``Any`` 替换模块（以及 *从中导入的任何东西*）。

    - ``error`` 的行为与 ``skip`` 相同，但不完全是静默的——它会将导入标记为错误，如下所示：

        ```text
        main.py:1: note: Import of "mycode.bar" ignored
        main.py:1: note: (Using --follow-imports=error, module not passed on command line)
        ```

    如果你正在开始一个新的代码库并计划从一开始就使用类型提示，我们建议你使用 [--follow-imports](./command_line.md#follow-imports)（默认）或 [--follow-imports=error](./command_line.md#follow-imports)。这两种选项都可以帮助确保你不会意外地跳过代码库的任何部分。

    如果你计划将类型提示添加到一个大型的现有代码库，我们建议你首先尝试使整个代码库（包括不使用类型提示的文件）在 [follow-imports=normal](./command_line.md#follow-imports) 下通过检查。这通常不难做到：mypy 设计为在查看未注解代码时报告尽可能少的错误消息。

    只有在这样做不可行时，我们建议你仅传递你希望类型检查的文件，并使用 [follow-imports=silent](./command_line.md#follow-imports)。即使 mypy 无法完美地检查一个文件，它仍然可以通过解析文件获得一些有用的信息（例如，了解一个给定对象具有哪些方法）。有关更多建议，请参见 [现有代码](../mypy/existing_code.md)。

    除非你知道自己在做什么，否则我们不推荐使用 ``skip``：虽然这个选项可能非常强大，但也可能导致许多难以调试的错误。

    调整导入跟随行为通常在限制于特定模块时最有用。这可以通过设置每个模块的 [follow_imports](./config_file.md#follow_imports) 配置选项来实现。

=== "英文"

    Mypy is designed to [doggedly follow all imports](#如何找到导入), even if the imported module is not a file you explicitly wanted mypy to check.

    For example, suppose we have two modules ``mycode.foo`` and ``mycode.bar``: the former has type hints and the latter does not. We run [mypy -m mycode.foo](./command_line.md#m) and mypy discovers that ``mycode.foo`` imports ``mycode.bar``.

    How do we want mypy to type check ``mycode.bar``? Mypy's behaviour here is configurable -- although we **strongly recommend** using the default -- by using the [--follow-imports](./command_line.md#follow-imports) flag. This flag accepts one of four string values:

    - ``normal`` (the default, recommended) follows all imports normally and type checks all top level code (as well as the bodies of all functions and methods with at least one type annotation in the signature).

    - ``silent`` behaves in the same way as ``normal`` but will additionally *suppress* any error messages.

    - ``skip`` will *not* follow imports and instead will silently replace the module (and *anything imported from it*) with an object of type ``Any``.

    - ``error`` behaves in the same way as ``skip`` but is not quite as silent -- it will flag the import as an error, like this

        ```text
        main.py:1: note: Import of "mycode.bar" ignored
        main.py:1: note: (Using --follow-imports=error, module not passed on command line)
        ```

    If you are starting a new codebase and plan on using type hints from the start, we recommend you use either [--follow-imports](./command_line.md#follow-imports) (the default) or [--follow-imports=error](./command_line.md#follow-imports). Either option will help make sure you are not skipping checking any part of your codebase by accident.

    If you are planning on adding type hints to a large, existing code base, we recommend you start by trying to make your entire codebase (including files that do not use type hints) pass under [follow-imports=normal](./command_line.md#follow-imports). This is usually not too difficult to do: mypy is designed to report as few error messages as possible when it is looking at unannotated code.

    Only if doing this is intractable, we recommend passing mypy just the files you want to type check and use [follow-imports=silent](./command_line.md#follow-imports). Even if mypy is unable to perfectly type check a file, it can still glean some useful information by parsing it (for example, understanding what methods a given object has). See [existing-code](../mypy/existing_code.md) for more recommendations.

    We do not recommend using ``skip`` unless you know what you are doing: while this option can be quite powerful, it can also cause many hard-to-debug errors.

    Adjusting import following behaviour is often most useful when restricted to specific modules. This can be accomplished by setting a per-module [follow_imports](./config_file.md#follow_imports) config option.


