# 附加功能

**Additional features**

=== "中文"

    这一部分讨论了在之前的部分中没有自然融入的各种特性。

=== "英文"

    This section discusses various features that did not fit in naturally in one of the previous sections.

## 数据类

**Dataclasses**

=== "中文"

    [dataclasses] 模块允许定义和自定义简单的无样板类。可以使用 [@dataclasses.dataclass] 装饰器来定义这些类：

    ```python
    from dataclasses import dataclass, field

    @dataclass
    class Application:
        name: str
        plugins: list[str] = field(default_factory=list)

    test = Application("Testing...")  # 正常
    bad = Application("Testing...", "with plugin")  # 错误：期望 list[str]
    ```

    Mypy 将根据定义 dataclasses 时使用的标志来检测特殊方法（如 [\_\_lt\_\_](https://docs.python.org/3/reference/datamodel.html#object.__lt__ )）。例如：

    ```python
    from dataclasses import dataclass

    @dataclass(order=True)
    class OrderedPoint:
        x: int
        y: int

    @dataclass(order=False)
    class UnorderedPoint:
        x: int
        y: int

    OrderedPoint(1, 2) < OrderedPoint(3, 4)  # 正常
    UnorderedPoint(1, 2) < UnorderedPoint(3, 4)  # 错误：不支持的操作数类型
    ```

    数据类可以是泛型的，并且可以像普通类一样使用：

    ```python
    from dataclasses import dataclass
    from typing import Generic, TypeVar

    T = TypeVar('T')

    @dataclass
    class BoxedData(Generic[T]):
        data: T
        label: str

    def unbox(bd: BoxedData[T]) -> T:
        ...

    val = unbox(BoxedData(42, "<important>"))  # 正常，推断类型为 int
    ```

    有关更多信息，请参见 [官方文档](https://docs.python.org/3/library/dataclasses.html) 和 [PEP 557](https://peps.python.org/pep-0557/)。

=== "英文"

    The [dataclasses] module allows defining and customizing simple boilerplate-free classes. They can be defined using the [@dataclasses.dataclass] decorator:



    ```python
    from dataclasses import dataclass, field

    @dataclass
    class Application:
        name: str
        plugins: list[str] = field(default_factory=list)

    test = Application("Testing...")  # OK
    bad = Application("Testing...", "with plugin")  # Error: list[str] expected
    ```

    Mypy will detect special methods (such as [\_\_lt\_\_](https://docs.python.org/3/reference/datamodel.html#object.__lt__)) depending on the flags used to define dataclasses. For example:

    ```python
    from dataclasses import dataclass

    @dataclass(order=True)
    class OrderedPoint:
        x: int
        y: int

    @dataclass(order=False)
    class UnorderedPoint:
        x: int
        y: int

    OrderedPoint(1, 2) < OrderedPoint(3, 4)  # OK
    UnorderedPoint(1, 2) < UnorderedPoint(3, 4)  # Error: Unsupported operand types
    ```


    Dataclasses can be generic and can be used in any other way a normal class can be used:

    ```python
    from dataclasses import dataclass
    from typing import Generic, TypeVar

    T = TypeVar('T')

    @dataclass
    class BoxedData(Generic[T]):
        data: T
        label: str

    def unbox(bd: BoxedData[T]) -> T:
        ...

    val = unbox(BoxedData(42, "<important>"))  # OK, inferred type is int
    ```


    For more information see [official docs](https://docs.python.org/3/library/dataclasses.html) and [PEP 557](https://peps.python.org/pep-0557/).

### 注意事项/已知问题

**Caveats/Known Issues**

=== "中文"

    在 [dataclasses](https://docs.python.org/3/library/dataclasses.html#module-dataclasses) 模块中，一些函数，如 [asdict()]，具有不精确（过于宽松）的类型。这将在未来的版本中修复。

    Mypy 目前尚不识别 [dataclasses.dataclass] 的别名，并且可能永远不会识别动态计算的装饰器。以下示例 **不** 能正常工作：

    ```python
    from dataclasses import dataclass

    dataclass_alias = dataclass
    def dataclass_wrapper(cls):
        return dataclass(cls)

    @dataclass_alias
    class AliasDecorated:
        """
        Mypy 不识别此类为数据类，因为它是由 `dataclass` 的别名装饰的，而不是由 `dataclass` 本身装饰的。
        """
        attribute: int

    AliasDecorated(attribute=1) # 错误：意外的关键字参数
    ```

    要使 Mypy 识别 [dataclasses.dataclass] 的包装器作为数据类装饰器，可以考虑使用 [dataclass_transform()] 装饰器：

    ```python
    from dataclasses import dataclass, Field
    from typing import TypeVar, dataclass_transform

    T = TypeVar('T')

    @dataclass_transform(field_specifiers=(Field,))
    def my_dataclass(cls: type[T]) -> type[T]:
        ...
        return dataclass(cls)
    ```

=== "英文"

    Some functions in the [dataclasses](https://docs.python.org/3/library/dataclasses.html#module-dataclasses) module, such as [asdict()], have imprecise (too permissive) types. This will be fixed in future releases.

    Mypy does not yet recognize aliases of [dataclasses.dataclass], and will probably never recognize dynamically computed decorators. The following example does **not** work:

    ```python
    from dataclasses import dataclass

    dataclass_alias = dataclass
    def dataclass_wrapper(cls):
    return dataclass(cls)

    @dataclass_alias
    class AliasDecorated:
    """
    Mypy doesn't recognize this as a dataclass because it is decorated by an
    alias of `dataclass` rather than by `dataclass` itself.
    """
    attribute: int

    AliasDecorated(attribute=1) # error: Unexpected keyword argument
    ```

    To have Mypy recognize a wrapper of [dataclasses.dataclass] as a dataclass decorator, consider using the [dataclass_transform()] decorator:

    ```python
    from dataclasses import dataclass, Field
    from typing import TypeVar, dataclass_transform

    T = TypeVar('T')

    @dataclass_transform(field_specifiers=(Field,))
    def my_dataclass(cls: type[T]) -> type[T]:
        ...
        return dataclass(cls)
    ```

## 数据类转换

**Data Class Transforms**

=== "中文"

    Mypy 支持 [dataclass_transform()] 装饰器，如 [PEP 681](https://www.python.org/dev/peps/pep-0681/#the-dataclass-transform-decorator) 中所述。

    !!! note

        实际上，Mypy 将假定这样的类具有内部属性 `__dataclass_fields__`（即使它们在运行时可能没有该属性），并将假定函数如 [dataclasses.is_dataclass()] 和 [dataclasses.fields()] 会将它们视为数据类（尽管它们在运行时可能会失败）。

=== "英文"

    Mypy supports the [dataclass_transform()] decorator as described in [PEP 681](https://www.python.org/dev/peps/pep-0681/#the-dataclass-transform-decorator).

    !!! note 

        Pragmatically, mypy will assume such classes have the internal attribute `__dataclass_fields__` (even though they might lack it in runtime) and will assume functions such as [dataclasses.is_dataclass()] and [dataclasses.fields()] treat them as if they were dataclasses (even though they may fail at runtime).

## attrs 包

**The attrs package**

=== "中文"

    [attrs](https://www.attrs.org/en/stable/index.html) 是一个可以让你定义类而无需编写样板代码的包。Mypy 可以检测到该包的使用，并将根据它找到的类型注解生成装饰类所需的方法定义。类型注解可以如下添加：

    ```python
    import attr

    @attr.define
    class A:
        one: int
        two: int = 7
        three: int = attr.field(8)
    ```

    如果你使用 ``auto_attribs=False``，则必须使用 ``attr.field``：

    ```python
    import attr

    @attr.define
    class A:
        one: int = attr.field()          # 变量注解（Python 3.6+）
        two = attr.field()  # type: int  # 类型注释
        three = attr.field(type=int)     # type= 参数
    ```

    Typeshed 有几个“善意谎言”注解，以便更容易进行类型检查。[attrs.field] 和 [attrs.Factory] 实际上返回对象，但注解说明这些返回期望被赋值的类型。这使得如下代码可以正常工作：

    ```python
    import attr

    @attr.define
    class A:
        one: int = attr.field(8)
        two: dict[str, str] = attr.Factory(dict)
        bad: str = attr.field(16)   # 错误：无法将 int 赋值给 str
    ```

=== "英文"

    [attrs](https://www.attrs.org/en/stable/index.html) is a package that lets you define classes without writing boilerplate code. Mypy can detect uses of the package and will generate the necessary method definitions for decorated classes using the type annotations it finds. Type annotations can be added as follows:

    ```python

    import attr

    @attrs.define
    class A:
        one: int
        two: int = 7
        three: int = attrs.field(8)
    ```

    If you're using ``auto_attribs=False`` you must use ``attrs.field``:

    ```python

    import attrs

    @attrs.define
    class A:
        one: int = attrs.field()          # Variable annotation (Python 3.6+)
        two = attrs.field()  # type: int  # Type comment
        three = attrs.field(type=int)     # type= argument
    ```


    Typeshed has a couple of "white lie" annotations to make type checking easier. [attrs.field] and [attrs.Factory] actually return objects, but the annotation says these return the types that they expect to be assigned to. That enables this to work:


    ```python

    import attrs

    @attrs.define
    class A:
        one: int = attrs.field(8)
        two: dict[str, str] = attrs.Factory(dict)
        bad: str = attrs.field(16)   # Error: can't assign int to str
    ```


### 注意事项/已知问题2

**Caveats/Known Issues2**

=== "中文"

    * 对 `attr` 类和属性的检测仅通过函数名进行。这意味着如果你有自己的辅助函数，例如 ``return attrs.field()``，mypy 将无法识别它们。

    * 所有 mypy 关注的布尔参数必须是字面量 ``True`` 或 ``False``。例如，下面的代码将不起作用：

    ```python
    import attr
    YES = True
    @attr.define(init=YES)
    class A:
        ...
    ```

    * 目前，``converter`` 只支持命名函数。如果 mypy 发现其他内容，它会抱怨无法理解该参数，并且 [\_\_init\_\_](https://docs.python.org/3/reference/datamodel.html#object.__init__) 中的类型注解将被替换为 ``Any``。

    * [验证器装饰器](https://www.attrs.org/en/stable/examples.html#examples-validators) 和 [默认值装饰器](https://www.attrs.org/en/stable/examples.html#defaults) 在设置/验证属性时不会进行类型检查。

    * mypy 添加的方法定义目前会覆盖任何现有的方法定义。

=== "英文"

    * The detection of attr classes and attributes works by function name only. This means that if you have your own helper functions that, for example, ``return attrs.field()`` mypy will not see them.

    * All boolean arguments that mypy cares about must be literal ``True`` or ``False``. e.g the following will not work:

    ```python

    import attrs
    YES = True
    @attrs.define(init=YES)
    class A:
        ...
    ```

    * Currently, ``converter`` only supports named functions.  If mypy finds something else it will complain about not understanding the argument and the type annotation in [\_\_init\_\_](https://docs.python.org/3/reference/datamodel.html#object.__init__) will be replaced by ``Any``.

    * [Validator decorators](https://www.attrs.org/en/stable/examples.html#examples-validators) and [default decorators](https://www.attrs.org/en/stable/examples.html#defaults) are not type-checked against the attribute they are setting/validating.

    * Method definitions added by mypy currently overwrite any existing method definitions.

## 使用远程缓存加速 mypy 运行

**Using a remote cache to speed up mypy runs**

=== "中文"

    Mypy 进行类型检查是 *递增式* 的，它重用之前运行的结果来加快后续运行的速度。如果你正在对一个大型代码库进行类型检查，mypy 有时仍可能比预期的慢。例如，如果你基于比上一次 mypy 运行目标提交更新得多的最新提交创建了一个新分支，mypy 可能需要处理几乎每个文件，因为很大一部分源文件可能已经更改。这种情况也可能在你对本地分支进行 rebase 后发生。

    Mypy 支持使用 *远程缓存* 来提高性能，尤其是在上述情况中。在大型代码库中，远程缓存有时可以将 mypy 的运行速度提高 10 倍或更多。

    Mypy 不包括设置所需的所有组件——通常你需要与 Continuous Integration (CI) 或构建系统进行一些简单的集成，以配置 mypy 使用远程缓存。以下讨论假设你已经为需要加速的 mypy 构建设置了 CI 系统，并且你正在使用中央 Git 仓库。将这些设置推广到不同的环境应该不难。

    这里是所需的主要组件：

    * 一个用于存储所有已提交的 mypy 缓存文件的共享仓库。

    * 一个 CI 构建，它在每次 CI 构建运行的提交中将 mypy 递增缓存文件上传到共享仓库。

    * 一个包装脚本，开发者用它来运行启用了远程缓存的 mypy。

    下面我们将详细讨论每个组件。

=== "英文"

    Mypy performs type checking *incrementally*, reusing results from previous runs to speed up successive runs. If you are type checking a large codebase, mypy can still be sometimes slower than desirable. For example, if you create a new branch based on a much more recent commit than the target of the previous mypy run, mypy may have to process almost every file, as a large fraction of source files may have changed. This can also happen after you've rebased a local branch.

    Mypy supports using a *remote cache* to improve performance in cases such as the above.  In a large codebase, remote caching can sometimes speed up mypy runs by a factor of 10, or more.

    Mypy doesn't include all components needed to set this up -- generally you will have to perform some simple integration with your Continuous Integration (CI) or build system to configure mypy to use a remote cache. This discussion assumes you have a CI system set up for the mypy build you want to speed up, and that you are using a central git repository. Generalizing to different environments should not be difficult.

    Here are the main components needed:

    * A shared repository for storing mypy cache files for all landed commits.

    * CI build that uploads mypy incremental cache files to the shared repository for each commit for which the CI build runs.

    * A wrapper script around mypy that developers use to run mypy with remote caching enabled.

    Below we discuss each of these components in some detail.

### 缓存文件的共享仓库

**Shared repository for cache files**

=== "中文"

    你需要一个仓库，允许你从 CI 构建上传 mypy 缓存文件，并根据提交 ID 使缓存文件可供下载。一个简单的方法是将 ``.mypy_cache`` 目录（包含 mypy 缓存数据）制作成一个可下载的 *构建工件*，从你的 CI 构建中生成（这取决于你的 CI 系统的能力）。另外，你也可以将数据上传到一个 web 服务器或 S3 等存储服务。

=== "英文"

    You need a repository that allows you to upload mypy cache files from your CI build and make the cache files available for download based on a commit id.  A simple approach would be to produce an archive of the ``.mypy_cache`` directory (which contains the mypy cache data) as a downloadable *build artifact* from your CI build (depending on the capabilities of your CI system).  Alternatively, you could upload the data to a web server or to S3, for example.

### 持续集成构建

**Continuous Integration build**

=== "中文"

    CI 构建会运行常规的 mypy 构建，并创建一个包含构建产生的 ``.mypy_cache`` 目录的归档文件。最后，它会将缓存作为构建工件生成，或上传到一个仓库，使其可以被 mypy 包装脚本访问。

    你的 CI 脚本可能如下工作：

    * 正常运行 mypy。这将生成位于 ``.mypy_cache`` 目录下的缓存数据。

    * 从 ``.mypy_cache`` 目录创建一个 tarball 归档文件。

    * 确定当前 Git 主分支的提交 ID（例如，使用 ``git rev-parse HEAD``）。

    * 将 tarball 上传到共享仓库，名称由提交 ID 派生。

=== "英文"

    The CI build would run a regular mypy build and create an archive containing the ``.mypy_cache`` directory produced by the build. Finally, it will produce the cache as a build artifact or upload it to a repository where it is accessible by the mypy wrapper script.

    Your CI script might work like this:

    * Run mypy normally. This will generate cache data under the ``.mypy_cache`` directory.

    * Create a tarball from the ``.mypy_cache`` directory.

    * Determine the current git master branch commit id (say, using ``git rev-parse HEAD``).

    * Upload the tarball to the shared repository with a name derived from the commit id.

### Mypy 包装脚本

**Mypy wrapper script**

=== "中文"

    包装脚本由开发人员在本地开发过程中使用，以替代直接调用 mypy。该包装脚本首先从共享仓库中填充本地的 ``.mypy_cache`` 目录，然后运行常规的增量构建。

    包装脚本需要一些逻辑来确定本地开发分支所基于的最新中央仓库提交（按惯例为 git 的 ``origin/master`` 分支）。在典型的 git 设置中，你可以这样做：

    ```shell
    git merge-base HEAD origin/master
    ```

    下一步是根据上面 git 命令生成的合并基点的提交 ID，从共享仓库下载缓存数据（``.mypy_cache`` 目录的内容）。脚本将解压缩数据，以便 mypy 从一个新的 ``.mypy_cache`` 开始。最后，脚本正常运行 mypy。就这样！

=== "英文"

    The wrapper script is used by developers to run mypy locally during development instead of invoking mypy directly.  The wrapper first populates the local ``.mypy_cache`` directory from the shared repository and then runs a normal incremental build.

    The wrapper script needs some logic to determine the most recent central repository commit (by convention, the ``origin/master`` branch for git) the local development branch is based on. In a typical git setup you can do it like this:

    ```shell
    git merge-base HEAD origin/master
    ```

    The next step is to download the cache data (contents of the ``.mypy_cache`` directory) from the shared repository based on the commit id of the merge base produced by the git command above. The script will decompress the data so that mypy will start with a fresh ``.mypy_cache``. Finally, the script runs mypy normally. And that's all!

### 使用 mypy 守护进程进行缓存

**Caching with mypy daemon**

=== "中文"

    你还可以使用 [mypy 守护进程](../mypy_conf/mypy_daemon.md) 进行远程缓存。远程缓存将显著加快启动或重启守护进程后的第一次 ``dmypy check`` 运行速度。

    mypy 守护进程需要缓存文件中额外的细粒度依赖数据，这些数据默认情况下不包括在内。要在 CI 构建中使用缓存与 mypy 守护进程，请使用 [--cache-fine-grained](../mypy_conf/command_line.md#cache-fine-grained) 选项：

    ```shell
    $ mypy --cache-fine-grained <args...>
    ```

    此标志会向缓存中添加额外的信息，以供守护进程使用。为了利用这些额外的信息，你还需要在 ``dmypy start`` 或 ``dmypy restart`` 时使用 ``--use-fine-grained-cache`` 选项。例如：

    ```shell
    $ dmypy start -- --use-fine-grained-cache <options...>
    ```

    现在，你的第一次 ``dmypy check`` 运行应该会快得多，因为它可以使用缓存信息来避免处理整个程序。

=== "英文"

    You can also use remote caching with the [mypy daemon](../mypy_conf/mypy_daemon.md). The remote cache will significantly speed up the first ``dmypy check`` run after starting or restarting the daemon.

    The mypy daemon requires extra fine-grained dependency data in the cache files which aren't included by default. To use caching with the mypy daemon, use the [--cache-fine-grained](../mypy_conf/command_line.md#cache-fine-grained) option in your CI build

    ```shell
    $ mypy --cache-fine-grained <args...>
    ```

    This flag adds extra information for the daemon to the cache. In order to use this extra information, you will also need to use the ``--use-fine-grained-cache`` option with ``dmypy start`` or ``dmypy restart``. Example

    ```shell
    $ dmypy start -- --use-fine-grained-cache <options...>
    ```

    Now your first ``dmypy check`` run should be much faster, as it can use cache information to avoid processing the whole program.

### 改进

**Refinements**

=== "中文"

    有几个可选的改进措施可以进一步提升性能，尤其是当你的代码库达到几十万行或更多时：

    * 如果包装脚本确定合并基点与之前的运行没有变化，那么就不需要下载缓存数据，而是最好重用现有的本地缓存数据。

    * 如果你使用 mypy 守护进程，可能需要在每次合并基点或本地分支更改后重新启动守护进程，以避免在增量构建中处理大量更改，因为这比下载缓存数据和重启守护进程要慢得多。

    * 如果当前本地分支基于非常新的 master 提交，远程缓存数据可能尚不可用，因为构建缓存文件会有一定的延迟。可以考虑查找例如最近 5 个 master 提交的缓存数据，并使用其中最可用的最新数据。

    * 如果由于某些原因（例如，从公共网络访问）远程缓存无法访问，脚本仍然可以回退到常规的增量构建。

    * 你可以为不同的本地分支设置多个本地缓存目录，使用 [--cache-dir](../mypy_conf/command_line.md#cache-dir) 选项。如果用户切换到一个已有缓存数据的分支，可以继续使用现有的缓存数据，而不是重新下载数据。

    * 你可以设置你的 CI 构建使用远程缓存来加速 CI 构建。这在每次 CI 构建从新状态开始且无法访问之前构建的缓存文件时尤其有用。仍然建议运行一次完整的非增量 mypy 构建以创建缓存数据，因为重复增量更新缓存数据可能会导致长期内的偏差（可能由于 mypy 缓存问题）。

=== "英文"

    There are several optional refinements that may improve things further at least if your codebase is hundreds of thousands of lines or more:

    * If the wrapper script determines that the merge base hasn't changed from a previous run, there's no need to download the cache data and it's better to instead reuse the existing local cache data.

    * If you use the mypy daemon, you may want to restart the daemon each time after the merge base or local branch has changed to avoid processing a potentially large number of changes in an incremental build, as this can be much slower than downloading cache data and restarting the daemon.

    * If the current local branch is based on a very recent master commit, the remote cache data may not yet be available for that commit, as there will necessarily be some latency to build the cache files. It may be a good idea to look for cache data for, say, the 5 latest master commits and use the most recent data that is available.

    * If the remote cache is not accessible for some reason (say, from a public network), the script can still fall back to a normal incremental build.

    * You can have multiple local cache directories for different local branches using the [--cache-dir](../mypy_conf/command_line.md#cache-dir) option. If the user switches to an existing branch where downloaded cache data is already available, you can continue to use the existing cache data instead of redownloading the data.

    * You can set up your CI build to use a remote cache to speed up the CI build. This would be particularly useful if each CI build starts from a fresh state without access to cache files from previous builds. It's still recommended to run a full, non-incremental mypy build to create the cache data, as repeatedly updating cache data incrementally could result in drift over a long time period (due to a mypy caching issue, perhaps).

## 扩展的 Callable 类型

**Extended Callable types**

=== "中文"

    !!! note 

        此功能已弃用。你可以使用 [回调协议](../mypy/protocol_and_struct_subtyping.md) 作为替代。

    作为一个实验性的 mypy 扩展，你可以指定支持关键字参数、可选参数等的 [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) 类型。当你指定 [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) 的参数时，你可以选择仅提供一个无名位置参数的类型，或提供一个表示更复杂参数形式的“参数说明符”。这使得你可以更接近 Python 中 ``def`` 语句所提供的全部可能性。

    例如，这里有一个复杂的函数定义及其对应的 [Callable](https://docs.python.org/3/library/typing.html#typing.Callable)：

    ```python
    from typing import Callable
    from mypy_extensions import (Arg, DefaultArg, NamedArg,
                                DefaultNamedArg, VarArg, KwArg)

    def func(__a: int,  # 这种约定用于无名参数
            b: int,
            c: int = 0,
            *args: int,
            d: int,
            e: int = 0,
            **kwargs: int) -> int:
        ...

    F = Callable[[int,  # 或 Arg(int)
                Arg(int, 'b'),
                DefaultArg(int, 'c'),
                VarArg(int),
                NamedArg(int, 'd'),
                DefaultNamedArg(int, 'e'),
                KwArg(int)],
                int]

    f: F = func
    ```

    参数说明符是特殊的函数调用，用于指定参数的以下方面：

    - 其类型（基本格式仅支持此项）

    - 其名称（如果有）

    - 是否可以省略

    - 是否可以或必须使用关键字传递

    - 是否为 ``*args`` 参数（表示剩余的位置参数）

    - 是否为 ``**kwargs`` 参数（表示剩余的关键字参数）

    以下是 ``mypy_extensions`` 中可用的函数：

    ```python
    def Arg(type=Any, name=None):
        # 一个普通的、强制的、位置参数。
        # 如果指定了名称，可以作为关键字传递。

    def DefaultArg(type=Any, name=None):
        # 一个可选的的位置参数（即有默认值）。
        # 如果指定了名称，可以作为关键字传递。

    def NamedArg(type=Any, name=None):
        # 一个强制的仅限关键字的参数。

    def DefaultNamedArg(type=Any, name=None):
        # 一个可选的仅限关键字的参数（即有默认值）。

    def VarArg(type=Any):
        # 一个 *args 风格的可变位置参数。
        # 一个 VarArg() 说明符表示所有剩余的位置参数。

    def KwArg(type=Any):
        # 一个 **kwargs 风格的可变关键字参数。
        # 一个 KwArg() 说明符表示所有剩余的关键字参数。
    ```

    在所有情况下，``type`` 参数默认为 ``Any``，如果省略 ``name`` 参数，则该参数没有名称（``NamedArg`` 和 ``DefaultNamedArg`` 需要名称）。一个基本的 [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) 如下：

    ```python
    MyFunc = Callable[[int, str, int], float]
    ```

    等同于：

    ```python
    MyFunc = Callable[[Arg(int), Arg(str), Arg(int)], float]
    ```

    一个具有未指定参数类型的 [Callable](https://docs.python.org/3/library/typing.html#typing.Callable)，如：

    ```python
    MyOtherFunc = Callable[..., int]
    ```

    （大致）等同于：

    ```python
    MyOtherFunc = Callable[[VarArg(), KwArg()], int]
    ```

    !!! note 

        上述每个函数目前在运行时仅返回其 ``type`` 参数，因此参数说明符中包含的信息在运行时不可用。这一限制是为了与 Python 3.5+ 标准库中的现有 ``typing.py`` 模块及通过 PyPI 分发的模块保持向后兼容。

=== "英文"

    !!! note 

        This feature is deprecated.  You can use
    [callback protocols](../mypy/protocol_and_struct_subtyping.md) as a replacement.

    As an experimental mypy extension, you can specify [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) types that support keyword arguments, optional arguments, and more.  When you specify the arguments of a [Callable](https://docs.python.org/3/library/typing.html#typing.Callable), you can choose to supply just the type of a nameless positional argument, or an "argument specifier" representing a more complicated form of argument.  This allows one to more closely emulate the full range of possibilities given by the ``def`` statement in Python.

    As an example, here's a complicated function definition and the corresponding [Callable](https://docs.python.org/3/library/typing.html#typing.Callable):

    ```python
    from typing import Callable
    from mypy_extensions import (Arg, DefaultArg, NamedArg,
                                DefaultNamedArg, VarArg, KwArg)

    def func(__a: int,  # This convention is for nameless arguments
            b: int,
            c: int = 0,
            *args: int,
            d: int,
            e: int = 0,
            **kwargs: int) -> int:
        ...

    F = Callable[[int,  # Or Arg(int)
                Arg(int, 'b'),
                DefaultArg(int, 'c'),
                VarArg(int),
                NamedArg(int, 'd'),
                DefaultNamedArg(int, 'e'),
                KwArg(int)],
                int]

    f: F = func
    ```


    Argument specifiers are special function calls that can specify the following aspects of an argument:

    - its type (the only thing that the basic format supports)

    - its name (if it has one)

    - whether it may be omitted

    - whether it may or must be passed using a keyword

    - whether it is a ``*args`` argument (representing the remaining positional arguments)

    - whether it is a ``**kwargs`` argument (representing the remaining keyword arguments)

    The following functions are available in ``mypy_extensions`` for this purpose:

    ```python
    def Arg(type=Any, name=None):
        # A normal, mandatory, positional argument.
        # If the name is specified it may be passed as a keyword.

    def DefaultArg(type=Any, name=None):
        # An optional positional argument (i.e. with a default value).
        # If the name is specified it may be passed as a keyword.

    def NamedArg(type=Any, name=None):
        # A mandatory keyword-only argument.

    def DefaultNamedArg(type=Any, name=None):
        # An optional keyword-only argument (i.e. with a default value).

    def VarArg(type=Any):
        # A *args-style variadic positional argument.
        # A single VarArg() specifier represents all remaining
        # positional arguments.

    def KwArg(type=Any):
        # A **kwargs-style variadic keyword argument.
        # A single KwArg() specifier represents all remaining
        # keyword arguments.
    ```


    In all cases, the ``type`` argument defaults to ``Any``, and if the ``name`` argument is omitted the argument has no name (the name is required for ``NamedArg`` and ``DefaultNamedArg``).  A basic [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) such as

    ```python
    MyFunc = Callable[[int, str, int], float]
    ```


    is equivalent to the following:

    ```python
    MyFunc = Callable[[Arg(int), Arg(str), Arg(int)], float]
    ```

    A [Callable](https://docs.python.org/3/library/typing.html#typing.Callable) with unspecified argument types, such as

    ```python
    MyOtherFunc = Callable[..., int]
    ```

    is (roughly) equivalent to

    ```python
    MyOtherFunc = Callable[[VarArg(), KwArg()], int]
    ```

    !!! note 

        Each of the functions above currently just returns its ``type`` argument at runtime, so the information contained in the argument specifiers is not available at runtime.  This limitation is necessary for backwards compatibility with the existing ``typing.py`` module as present in the Python 3.5+ standard library and distributed via PyPI.

[dataclasses]: https://docs.python.org/3/library/dataclasses.html#module-dataclasses
[@dataclasses.dataclass]: https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass
[dataclasses.dataclass]: https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass
[asdict()]: https://docs.python.org/3/library/dataclasses.html#dataclasses.asdict
[dataclass_transform()]: https://docs.python.org/3/library/typing.html#typing.dataclass_transform
[dataclasses.is_dataclass()]: https://docs.python.org/3/library/dataclasses.html#dataclasses.is_dataclass
[dataclasses.fields()]: https://docs.python.org/3/library/dataclasses.html#dataclasses.fields
[attrs.field()]: https://www.attrs.org/en/stable/api.html#attrs.field
[attrs.Factory]: https://www.attrs.org/en/stable/api.html#attrs.Factory