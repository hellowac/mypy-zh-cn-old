# 使用已安装的包

**Using installed packages**

=== "中文"

    使用 pip 安装的包可以声明它们支持类型检查。例如，[aiohttp](https://docs.aiohttp.org/en/stable/) 包内置支持类型检查。

    包还可以为某个库提供存根。例如，``types-requests`` 是一个仅包含存根的包，为 [requests](https://requests.readthedocs.io/en/master/) 包提供存根。存根包通常来自 [typeshed](https://github.com/python/typeshed)，这是一个共享的 Python 库存根仓库，存根包的名称通常为 ``types-<library>`` 的形式。请注意，许多存根包并不是由原包的维护者维护的。

    以下部分解释了 mypy 如何使用这些包，以及如何创建这些包。

    !!! note 

        [PEP 561](https://peps.python.org/pep-0561/) 规定了一个包如何声明它支持类型检查。

    !!! note 

        存根包的新版本通常使用旧版本甚至相当近期版本的 mypy 不支持的类型系统特性。如果你将 mypy 固定到旧版本（例如，通过 ``requirements.txt``），建议你也固定所有存根包依赖的版本。

    !!! note 

        从 mypy 0.900 开始，大多数第三方包的存根必须显式安装。这使得 mypy 与存根版本解耦，允许在不更新 mypy 的情况下更新存根。这也允许安装最初未包含在 mypy 中的存根。早期的 mypy 版本包含了一组固定的第三方包存根。

=== "英文"


    Packages installed with pip can declare that they support type checking. For example, the [aiohttp](https://docs.aiohttp.org/en/stable/) package has built-in support for type checking.

    Packages can also provide stubs for a library. For example, ``types-requests`` is a stub-only package that provides stubs for the [requests](https://requests.readthedocs.io/en/master/) package. Stub packages are usually published from [typeshed](https://github.com/python/typeshed), a shared repository for Python library stubs, and have a name of form ``types-<library>``. Note that many stub packages are not maintained by the original maintainers of the package.

    The sections below explain how mypy can use these packages, and how you can create such packages.

    !!! note 

        [PEP 561](https://peps.python.org/pep-0561/) specifies how a package can declare that it supports type checking.

    !!! note 

        New versions of stub packages often use type system features not supported by older, and even fairly recent mypy versions. If you pin to an older version of mypy (using ``requirements.txt``, for example), it is recommended that you also pin the versions of all your stub package dependencies.

    !!! note 

        Starting in mypy 0.900, most third-party package stubs must be installed explicitly. This decouples mypy and stub versioning, allowing stubs to updated without updating mypy. This also allows stubs not originally included with mypy to be installed. Earlier mypy versions included a fixed set of stubs for third-party packages.

## 使用 mypy 检查已安装的包（PEP 561）

**Using installed packages with mypy (PEP 561)**

=== "中文"

    通常，mypy 会自动找到并使用已安装的支持类型检查或提供存根的包。这要求你在运行 mypy 的 Python 环境中安装这些包。由于许多包尚不支持类型检查，你可能还需要安装一个单独的存根包，通常命名为 ``types-<library>``。（有关如何处理不支持类型检查且缺少存根的库，请参见 [缺失的导入](./running_mypy.md#缺失的导入)）

    如果你在另一个 Python 安装或环境中安装了带类型信息的包，mypy 不会自动找到它们。一种选择是在安装 mypy 的环境中再安装一份这些包。或者，你可以使用 [--python-executable](./command_line.md#python-executable) 标志来指向另一个环境中的 Python 可执行文件，这样 mypy 就会找到为该 Python 可执行文件安装的包。

    请注意，mypy 不支持一些更高级的导入特性，如 zip 导入和自定义导入钩子。

    如果你不想使用提供类型信息的已安装包，可以使用 [--no-site-packages](./command_line.md#no-site-packages) 标志来禁用对已安装包的搜索。

    请注意，存根-only 包不能与 ``MYPYPATH`` 一起使用。如果你希望 mypy 找到该包，它必须已安装。对于包 ``foo``，存根-only 包的名称（``foo-stubs``）不是合法的包名，因此 mypy 将不会找到它，除非它已安装（有关更多信息，请参见 [PEP 561: 存根-only 包](https://peps.python.org/pep-0561/#stub-only-packages)）。

=== "英文"

    Typically mypy will automatically find and use installed packages that support type checking or provide stubs. This requires that you install the packages in the Python environment that you use to run mypy.  As many packages don't support type checking yet, you may also have to install a separate stub package, usually named ``types-<library>``. (See [Missing imports](./running_mypy.md#缺失的导入) for how to deal with libraries that don't support type checking and are also missing stubs.)

    If you have installed typed packages in another Python installation or environment, mypy won't automatically find them. One option is to install another copy of those packages in the environment in which you installed mypy. Alternatively, you can use the [--python-executable](./command_line.md#python-executable) flag to point to the Python executable for another environment, and mypy will find packages installed for that Python executable.

    Note that mypy does not support some more advanced import features, such as zip imports and custom import hooks.

    If you don't want to use installed packages that provide type information at all, use the [--no-site-packages](./command_line.md#no-site-packages) flag to disable searching for installed packages.

    Note that stub-only packages cannot be used with ``MYPYPATH``. If you want mypy to find the package, it must be installed. For a package ``foo``, the name of the stub-only package (``foo-stubs``) is not a legal package name, so mypy will not find it, unless it is installed (see [PEP 561: Stub-only Packages](https://peps.python.org/pep-0561/#stub-only-packages) for more information).

## 创建兼容 PEP 561 的包

**Creating PEP 561 compatible packages**

=== "中文"

    !!! note

        除非你维护一个 PyPI 上的包，或者想要为现有的 PyPI 包发布类型信息，否则你通常可以忽略这一部分。

    [PEP 561] 描述了三种主要的分发类型信息的方法：

    1. 包内含有 Python 实现中的内联类型注解。

    2. 包随 Python 实现一起提供 [存根文件](../mypy/stub_files.md) 以包含类型信息。

    3. 包单独提供另一包的类型信息作为存根文件（也称为“存根-only 包”）。

    如果你想为现有库创建一个存根-only 包，最简单的方法是向 [typeshed] 仓库贡献存根，存根包将自动上传到 PyPI。

    如果你想将库包自己发布到包仓库（例如 PyPI），无论是用于内部还是外部的类型检查，提供类型信息的包（通过类型注释或代码中的注解）应在其包目录中放置一个 ``py.typed`` 文件。例如，这里是一个典型的目录结构：

    ```text
    setup.py
    package_a/
        __init__.py
        lib.py
        py.typed
    ```

    ``setup.py`` 文件可以如下所示：

    ```python
    from setuptools import setup

    setup(
        name="SuperPackageA",
        author="Me",
        version="0.1",
        package_data={"package_a": ["py.typed"]},
        packages=["package_a"]
    )
    ```

    一些包同时包含存根文件和运行时文件。这些包也需要一个 ``py.typed`` 文件。下面是一个示例：

    ```text
    setup.py
    package_b/
        __init__.py
        lib.py
        lib.pyi
        py.typed
    ```

    ``setup.py`` 文件可能如下所示：

    ```python
    from setuptools import setup

    setup(
        name="SuperPackageB",
        author="Me",
        version="0.1",
        package_data={"package_b": ["py.typed", "lib.pyi"]},
        packages=["package_b"]
    )
    ```

    在这个示例中，``lib.py`` 和 ``lib.pyi`` 存根文件都存在。在运行时，Python 解释器将使用 ``lib.py``，但 mypy 将使用 ``lib.pyi``。

    如果包是存根-only（在运行时不被导入），该包应以运行时包名为前缀，并以 ``-stubs`` 为后缀。存根-only 包不需要 ``py.typed`` 文件。例如，如果我们有 ``package_c`` 的存根，可能会这样做：

    ```text
    setup.py
    package_c-stubs/
        __init__.pyi
        lib.pyi
    ```

    ``setup.py`` 文件可能如下所示：

    ```python
    from setuptools import setup

    setup(
        name="SuperPackageC",
        author="Me",
        version="0.1",
        package_data={"package_c-stubs": ["__init__.pyi", "lib.pyi"]},
        packages=["package_c-stubs"]
    )
    ```

    上述说明足以确保构建的 wheel 包含适当的文件。然而，为了确保在 ``sdist``（``.tar.gz`` 存档）中包含这些文件，你可能还需要修改 ``MANIFEST.in`` 中的包含规则：

    ```text
    global-include *.pyi
    global-include *.typed
    ```

=== "英文"

    !!! note 

        You can generally ignore this section unless you maintain a package on PyPI, or want to publish type information for an existing PyPI package.

    [PEP 561] describes three main ways to distribute type information:

    1. A package has inline type annotations in the Python implementation.

    2. A package ships [stub files](../mypy/stub_files.md) with type information alongside the Python implementation.

    3. A package ships type information for another package separately as stub files (also known as a "stub-only package").

    If you want to create a stub-only package for an existing library, the simplest way is to contribute stubs to the [typeshed] repository, and a stub package will automatically be uploaded to PyPI.

    If you would like to publish a library package to a package repository yourself (e.g. on PyPI) for either internal or external use in type checking, packages that supply type information via type comments or annotations in the code should put a ``py.typed`` file in their package directory. For example, here is a typical directory structure:

    ```text
    setup.py
    package_a/
        __init__.py
        lib.py
        py.typed
    ```

    The ``setup.py`` file could look like this:

    ```python

        from setuptools import setup

        setup(
            name="SuperPackageA",
            author="Me",
            version="0.1",
            package_data={"package_a": ["py.typed"]},
            packages=["package_a"]
        )
    ```

    Some packages have a mix of stub files and runtime files. These packages also require a ``py.typed`` file. An example can be seen below:

    ```text
    setup.py
    package_b/
        __init__.py
        lib.py
        lib.pyi
        py.typed
    ```

    The ``setup.py`` file might look like this:

    ```python
    from setuptools import setup

    setup(
        name="SuperPackageB",
        author="Me",
        version="0.1",
        package_data={"package_b": ["py.typed", "lib.pyi"]},
        packages=["package_b"]
    )
    ```

    In this example, both ``lib.py`` and the ``lib.pyi`` stub file exist. At runtime, the Python interpreter will use ``lib.py``, but mypy will use ``lib.pyi`` instead.

    If the package is stub-only (not imported at runtime), the package should have a prefix of the runtime package name and a suffix of ``-stubs``. A ``py.typed`` file is not needed for stub-only packages. For example, if we had stubs for ``package_c``, we might do the following:

    ```text
    setup.py
    package_c-stubs/
        __init__.pyi
        lib.pyi
    ```

    The ``setup.py`` might look like this:

    ```python
    from setuptools import setup

    setup(
        name="SuperPackageC",
        author="Me",
        version="0.1",
        package_data={"package_c-stubs": ["__init__.pyi", "lib.pyi"]},
        packages=["package_c-stubs"]
    )
    ```

    The instructions above are enough to ensure that the built wheels contain the appropriate files. However, to ensure inclusion inside the ``sdist`` (``.tar.gz`` archive), you may also need to modify the inclusion rules in your ``MANIFEST.in``:

    ```text
    global-include *.pyi
    global-include *.typed
    ```

[PEP 561]: https://peps.python.org/pep-0561/
[typeshed]: https://github.com/python/typeshed