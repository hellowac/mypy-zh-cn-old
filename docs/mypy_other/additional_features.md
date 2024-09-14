# 附加功能

**Additional features**

=== "中文"

=== "英文"

This section discusses various features that did not fit in naturally in one of the previous sections.

## 数据类

**Dataclasses**

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

=== "英文"

Mypy supports the [dataclass_transform()] decorator as described in [PEP 681](https://www.python.org/dev/peps/pep-0681/#the-dataclass-transform-decorator).

!!! note 

    Pragmatically, mypy will assume such classes have the internal attribute `__dataclass_fields__` (even though they might lack it in runtime) and will assume functions such as [dataclasses.is_dataclass()] and [dataclasses.fields()] treat them as if they were dataclasses (even though they may fail at runtime).

## attrs 包

**The attrs package**

=== "中文"

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

=== "英文"

You need a repository that allows you to upload mypy cache files from your CI build and make the cache files available for download based on a commit id.  A simple approach would be to produce an archive of the ``.mypy_cache`` directory (which contains the mypy cache data) as a downloadable *build artifact* from your CI build (depending on the capabilities of your CI system).  Alternatively, you could upload the data to a web server or to S3, for example.

### 持续集成构建

**Continuous Integration build**

=== "中文"

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