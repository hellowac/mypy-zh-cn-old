# 将 mypy 与现有代码库结合使用

**Using mypy with an existing codebase**

=== "中文"

    本节解释了如何在一个现有的、规模较大的代码库中使用 mypy，而该代码库几乎没有或完全没有类型注解。如果你是初学者，可以跳过本节。

=== "英文"

    This section explains how to get started using mypy with an existing, significant codebase that has little or no type annotations. If you are a beginner, you can skip this section.



## 从小事做起

**Start small**

=== "中文"

    如果你的代码库很大，建议先选择一个子集（例如 5,000 到 50,000 行代码），并让 mypy 首先成功运行在这个子集上，*在添加注解之前*。这应该在一两天内就能完成。越早让 mypy 在你的代码库上运行通过，你就能越早从中受益。
    
    你可能需要修复一些 mypy 报告的错误，方法是插入 mypy 要求的注解，或者通过添加 `# type: ignore` 注释来忽略你现在不想修复的错误。
    
    我们将在下面的各个部分中提到一些让 mypy 通过代码库的技巧。



=== "英文"

    If your codebase is large, pick a subset of your codebase (say, 5,000 to 50,000 lines) and get mypy to run successfully only on this subset at first, *before adding annotations*. This should be doable in a day or two. The sooner you get some form of mypy passing on your codebase, the sooner you benefit.
    
    You’ll likely need to fix some mypy errors, either by inserting annotations requested by mypy or by adding `# type: ignore` comments to silence errors you don’t want to fix now.
    
    We’ll mention some tips for getting mypy passing on your codebase in various sections below.


## 持续运行 mypy 并防止回归

**Run mypy consistently and prevent regressions**

=== "中文"

    确保你代码库中的所有开发人员以相同的方式运行 mypy。有一种方法是将 mypy 的调用命令添加到代码库中的一个小脚本中，或者将 mypy 的调用命令添加到你用来运行测试的现有工具中，例如 tox。
    
    - 确保所有人都使用相同的选项运行 mypy。将 mypy 的[配置文件](https://mypy.readthedocs.io/en/stable/config_file.html#config-file)检查到代码库中是最简单的做法。
    
    - 确保所有人检查相同的文件集。有关详细信息，请参阅“[指定要检查的代码](https://mypy.readthedocs.io/en/stable/running_mypy.html#specifying-code-to-be-checked)”。
    
    - 确保所有人使用相同版本的 mypy，例如将 mypy 与其他开发需求一起固定版本。
    
    特别是，你需要确保尽早将 mypy 集成到你的持续集成（CI）系统中。这将防止新的类型错误被引入到代码库中。
    
    一个简单的 CI 脚本可能如下所示：
    
    ```shell
    python3 -m pip install mypy==1.8
    # 运行标准化的 mypy 调用命令，例如：
    mypy my_project
    # 这也可能类似于 `scripts/run_mypy.sh`、`tox run -e mypy`、`make mypy` 等
    ```

=== "英文"

    Make sure all developers on your codebase run mypy the same way. One way to ensure this is adding a small script with your mypy invocation to your codebase, or adding your mypy invocation to existing tools you use to run tests, like tox.
    
    - Make sure everyone runs mypy with the same options. Checking a mypy [configuration file](https://mypy.readthedocs.io/en/stable/config_file.html#config-file) into your codebase is the easiest way to do this.
    
    - Make sure everyone type checks the same set of files. See [Specifying code to be checked](https://mypy.readthedocs.io/en/stable/running_mypy.html#specifying-code-to-be-checked) for details.
    
    - Make sure everyone runs mypy with the same version of mypy, for instance by pinning mypy with the rest of your dev requirements.
    
    In particular, you’ll want to make sure to run mypy as part of your Continuous Integration (CI) system as soon as possible. This will prevent new type errors from being introduced into your codebase.
    
    A simple CI script could look something like this:
    
    ```shell 
    python3 -m pip install mypy==1.8
    # Run your standardised mypy invocation, e.g.
    mypy my_project
    # This could also look like `scripts/run_mypy.sh`, `tox run -e mypy`, `make mypy`, etc
    ```

## 忽略某些模块的错误

**Ignoring errors from certain modules**

=== "中文"

    默认情况下，mypy 会跟随代码中的导入并尝试检查所有内容。这意味着即使你只传入几个文件，mypy 仍可能处理大量导入的文件，从而可能导致许多你当前不想处理的错误。
    
    一种处理这种情况的方法是忽略那些你还没准备好进行类型检查的模块中的错误。对于这种情况，[ignore_errors](https://mypy.readthedocs.io/en/stable/config_file.html#confval-ignore_errors) 选项非常有用。例如，如果你还不准备处理来自 `package_to_fix_later` 模块的错误：
    
    ```ini
    [mypy-package_to_fix_later.*]
    ignore_errors = True
    ```
    
    你甚至可以反过来操作，在全局配置中设置 `ignore_errors = True`，然后只对你准备好进行类型检查的模块启用错误报告，即设置 `ignore_errors = False`。
    
    mypy 的配置文件允许的按模块配置非常有用。许多配置选项都可以仅针对特定模块启用或禁用。特别是，你还可以按模块启用或禁用各种错误代码，详情请参阅 [Error codes](https://mypy.readthedocs.io/en/stable/error_codes.html#error-codes)。

=== "英文"

    By default mypy will follow imports in your code and try to check everything. This means even if you only pass in a few files to mypy, it may still process a large number of imported files. This could potentially result in lots of errors you don’t want to deal with at the moment.
    
    One way to deal with this is to ignore errors in modules you aren’t yet ready to type check. The [ignore_errors](https://mypy.readthedocs.io/en/stable/config_file.html#confval-ignore_errors) option is useful for this, for instance, if you aren’t yet ready to deal with errors from `package_to_fix_later`:
    
    ```ini
    [mypy-package_to_fix_later.*]
    ignore_errors = True
    ```
    
    You could even invert this, by setting `ignore_errors = True` in your global config section and only enabling error reporting with `ignore_errors = False` for the set of modules you are ready to type check.
    
    The per-module configuration that mypy’s configuration file allows can be extremely useful. Many configuration options can be enabled or disabled only for specific modules. In particular, you can also enable or disable various error codes on a per-module basis, see [Error codes](https://mypy.readthedocs.io/en/stable/error_codes.html#error-codes).

## 修复与导入相关的错误

**Fixing errors related to imports**

=== "中文"

    你可能会遇到的一类常见错误是 mypy 关于找不到模块、模块没有类型或没有存根文件的错误：
    
    ```text
    core/config.py:7: error: Cannot find implementation or library stub for module named 'frobnicate'
    core/model.py:9: error: Cannot find implementation or library stub for module named 'acme'
    ...
    ```
    
    有时，这些问题可以通过在运行 `mypy` 的环境中安装相关的包或存根库来解决。
    
    有关这些错误以及修复方法的完整参考，请参阅 [Missing imports](https://mypy.readthedocs.io/en/stable/running_mypy.html#fix-missing-imports)。
    
    你可能会发现需要抑制所有来自没有类型的模块的导入错误。如果你只在一两个地方导入了该模块，可以使用 `# type: ignore` 注释。例如，在这里，我们忽略了一个关于没有存根的第三方模块 `frobnicate` 的错误，使用了 `# type: ignore`：
    
    ```python
    import frobnicate  # type: ignore
    ...
    frobnicate.initialize()  # OK (但不会被检查)
    ```
    
    但如果你在多个地方导入该模块，这种方法就显得笨拙了。在这种情况下，我们建议使用[配置文件](https://mypy.readthedocs.io/en/stable/config_file.html#config-file)。例如，要在整个代码库中禁用关于导入 `frobnicate` 和 `acme` 的错误，可以使用如下配置：
    
    ```ini
    [mypy-frobnicate.*]
    ignore_missing_imports = True
    
    [mypy-acme.*]
    ignore_missing_imports = True
    ```
    
    如果你遇到大量错误，可能会想要忽略所有关于缺少导入的错误，例如通过设置 [--disable-error-code=import-untyped](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-ignore-missing-imports) 或全局设置 [ignore_missing_imports](https://mypy.readthedocs.io/en/stable/config_file.html#confval-ignore_missing_imports) 为 true。 这可能会隐藏后续的错误，因此如果可能，我们建议避免这样做。
    
    最后，mypy 允许对特定导入行为进行精细控制。当你尝试调整这些设置时，很容易在不知不觉中出错，因此这应该是最后的选择。更多详情，请参考[此处](https://mypy.readthedocs.io/en/stable/running_mypy.html#follow-imports)。

=== "英文"

    A common class of error you will encounter is errors from mypy about modules that it can’t find, that don’t have types, or don’t have stub files:
    
    ```text
    core/config.py:7: error: Cannot find implementation or library stub for module named 'frobnicate'
    core/model.py:9: error: Cannot find implementation or library stub for module named 'acme'
    ...
    ```
    
    Sometimes these can be fixed by installing the relevant packages or stub libraries in the environment you’re running `mypy` in.
    
    See [Missing imports](https://mypy.readthedocs.io/en/stable/running_mypy.html#fix-missing-imports) for a complete reference on these errors and the ways in which you can fix them.
    
    You’ll likely find that you want to suppress all errors from importing a given module that doesn’t have types. If you only import that module in one or two places, you can use `# type: ignore` comments. For example, here we ignore an error about a third-party module `frobnicate` that doesn’t have stubs using `# type: ignore`:
    
    ```python
    import frobnicate  # type: ignore
    ...
    frobnicate.initialize()  # OK (but not checked)
    ```
    
    But if you import the module in many places, this becomes unwieldy. In this case, we recommend using a [configuration file](https://mypy.readthedocs.io/en/stable/config_file.html#config-file). For example, to disable errors about importing frobnicate and acme everywhere in your codebase, use a config like this:
    
    ```ini
    [mypy-frobnicate.*]
    ignore_missing_imports = True
    
    [mypy-acme.*]
    ignore_missing_imports = True
    ```
    
    If you get a large number of errors, you may want to ignore all errors about missing imports, for instance by setting [--disable-error-code=import-untyped](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-ignore-missing-imports). or setting [ignore_missing_imports](https://mypy.readthedocs.io/en/stable/config_file.html#confval-ignore_missing_imports) to true globally. This can hide errors later on, so we recommend avoiding this if possible.
    
    Finally, mypy allows fine-grained control over specific import following behaviour. It’s very easy to silently shoot yourself in the foot when playing around with these, so this should be a last resort. For more details, look [here](https://mypy.readthedocs.io/en/stable/running_mypy.html#follow-imports).


## 优先注释广泛导入的模块

**Prioritise annotating widely imported modules**

=== "中文"

    大多数项目都有一些广泛导入的模块，比如工具类或模型类。尽早对这些模块进行注释是个不错的主意，因为这能让使用这些模块的代码更有效地进行类型检查。
    
    Mypy 旨在支持渐进式类型检查，也就是说，你可以按照自己的节奏逐步添加注释，因此可以放心地将某些模块暂时不做注释。注释得越多，mypy 的作用就越大，但即使只有少量注释覆盖也会有帮助。

=== "英文"

    Most projects have some widely imported modules, such as utilities or model classes. It’s a good idea to annotate these pretty early on, since this allows code using these modules to be type checked more effectively.
    
    Mypy is designed to support gradual typing, i.e. letting you add annotations at your own pace, so it’s okay to leave some of these modules unannotated. The more you annotate, the more useful mypy will be, but even a little annotation coverage is useful.

## 随时写注解

**Write annotations as you go**

=== "中文"

    可以考虑将以下内容加入到你的代码风格规范中：
    
    1. 开发人员应为任何新代码添加类型注释。
    
    2. 修改现有代码时，也鼓励添加类型注释。
    
    通过这种方式，你可以在不费太多力气的情况下逐步增加代码库中的类型注释覆盖率。

=== "英文"

    Consider adding something like these in your code style conventions:
    
    1. Developers should add annotations for any new code.
    
    2. It’s also encouraged to write annotations when you modify existing code.
    
    This way you’ll gradually increase annotation coverage in your codebase without much effort.

## 自动注解遗留代码

**Automate annotation of legacy code**

=== "中文"

    有一些工具可以基于简单的静态分析或在运行时收集的类型信息自动添加初步的类型注释。这些工具包括 [MonkeyType](https://monkeytype.readthedocs.io/en/latest/index.html)、[autotyping](https://github.com/JelleZijlstra/autotyping) 和 [PyAnnotate](https://github.com/dropbox/pyannotate)。
    
    一种简单的方法是通过测试运行来收集类型信息。如果你的测试覆盖率较好（且测试速度不太慢），这种方法可能效果不错。
    
    另一种方法是为一小部分随机选择的生产网络请求启用类型信息收集。这显然需要更加谨慎，因为类型收集可能会影响服务的可靠性或性能。

=== "英文"

    There are tools for automatically adding draft annotations based on simple static analysis or on type profiles collected at runtime. Tools include [MonkeyType](https://monkeytype.readthedocs.io/en/latest/index.html), [autotyping](https://github.com/JelleZijlstra/autotyping) and [PyAnnotate](https://github.com/dropbox/pyannotate).
    
    A simple approach is to collect types from test runs. This may work well if your test coverage is good (and if your tests aren’t very slow).
    
    Another approach is to enable type collection for a small, random fraction of production network requests. This clearly requires more care, as type collection could impact the reliability or the performance of your service.

## 引入更严格的选项

**Introduce stricter options**

=== "中文"

    Mypy 是高度可配置的。一旦你开始使用静态类型检查，可能会希望探索 mypy 提供的各种严格性选项，以捕捉更多的错误。例如，你可以要求 mypy 对某些模块的所有函数进行类型注释，以避免无意中引入不会被类型检查的代码，可以使用 [disallow_untyped_defs](https://mypy.readthedocs.io/en/stable/config_file.html#confval-disallow_untyped_defs) 实现这一点。有关详细信息，请参阅 [Mypy 配置文件](https://mypy.readthedocs.io/en/stable/config_file.html#config-file)。
    
    一个优秀的目标是让你的代码库在运行 `mypy --strict` 时能通过检查。这样基本上可以确保你不会有任何与类型相关的错误，除非有明确的规避（例如 `# type: ignore` 注释）。
    
    以下配置等效于 `--strict`（基于 mypy 1.0 版本）：
    
    ```ini
    # 先从这些开始
    warn_unused_configs = True
    warn_redundant_casts = True
    warn_unused_ignores = True
    
    # 这些选项通常容易通过
    strict_equality = True
    strict_concatenate = True
    
    # 强烈建议尽早启用此选项
    check_untyped_defs = True
    
    # 这些选项不算太复杂，但如果使用了大量无类型注释的库，可能会比较棘手
    disallow_subclassing_any = True
    disallow_untyped_decorators = True
    disallow_any_generics = True
    
    # 以下几项是各种强制使用类型注释的方式
    disallow_untyped_calls = True
    disallow_incomplete_defs = True
    disallow_untyped_defs = True
    
    # 这一项不难通过，但投资回报率较低
    no_implicit_reexport = True
    
    # 如果使用了大量无类型注释的库，这一项可能比较棘手
    warn_return_any = True
    ```
    
    注意，你也可以从 `--strict` 开始，并根据需要减去某些选项，例如：
    
    ```ini
    strict = True
    warn_return_any = False
    ```
    
    记住，许多这些选项可以按模块启用。例如，你可能希望对已经完成类型注释的模块启用 `disallow_untyped_defs`，以防止新代码在没有注释的情况下被添加。
    
    此外，如果你愿意，严格性选项不止于 `--strict`。Mypy 还有一些不在 `--strict` 范围内但依然有用的检查。请参阅完整的 [Mypy 命令行](https://mypy.readthedocs.io/en/stable/command_line.html#command-line)参考和 [可选检查的错误代码](https://mypy.readthedocs.io/en/stable/error_code_list2.html#error-codes-optional)。

=== "英文"

    Mypy is very configurable. Once you get started with static typing, you may want to explore the various strictness options mypy provides to catch more bugs. For example, you can ask mypy to require annotations for all functions in certain modules to avoid accidentally introducing code that won’t be type checked using [disallow_untyped_defs](https://mypy.readthedocs.io/en/stable/config_file.html#confval-disallow_untyped_defs). Refer to [The mypy configuration file](https://mypy.readthedocs.io/en/stable/config_file.html#config-file) for the details.
    
    An excellent goal to aim for is to have your codebase pass when run against `mypy --strict`. This basically ensures that you will never have a type related error without an explicit circumvention somewhere (such as a `# type: ignore` comment).
    
    The following config is equivalent to `--strict` (as of mypy 1.0):
    
    ```ini
    # Start off with these
    warn_unused_configs = True
    warn_redundant_casts = True
    warn_unused_ignores = True
    
    # Getting these passing should be easy
    strict_equality = True
    strict_concatenate = True
    
    # Strongly recommend enabling this one as soon as you can
    check_untyped_defs = True
    
    # These shouldn't be too much additional work, but may be tricky to
    # get passing if you use a lot of untyped libraries
    disallow_subclassing_any = True
    disallow_untyped_decorators = True
    disallow_any_generics = True
    
    # These next few are various gradations of forcing use of type annotations
    disallow_untyped_calls = True
    disallow_incomplete_defs = True
    disallow_untyped_defs = True
    
    # This one isn't too hard to get passing, but return on investment is lower
    no_implicit_reexport = True
    
    # This one can be tricky to get passing if you use a lot of untyped libraries
    warn_return_any = True
    ```
    
    Note that you can also start with --strict and subtract, for instance:
    
    ```ini
    strict = True
    warn_return_any = False
    ```
    
    Remember that many of these options can be enabled on a per-module basis. For instance, you may want to enable `disallow_untyped_defs` for modules which you’ve completed annotations for, in order to prevent new code from being added without annotations.
    
    And if you want, it doesn’t stop at `--strict`. Mypy has additional checks that are not part of `--strict` that can be useful. See the complete [The mypy command line](https://mypy.readthedocs.io/en/stable/command_line.html#command-line) reference and [Error codes for optional checks](https://mypy.readthedocs.io/en/stable/error_code_list2.html#error-codes-optional).

## 加速 mypy 运行

**Speed up mypy runs**

=== "中文"
    
    你可以使用 [mypy daemon](https://mypy.readthedocs.io/en/stable/mypy_daemon.html#mypy-daemon) 来实现更快的增量式 mypy 运行。项目越大，这个工具就越有用。如果你的项目至少有 100,000 行代码，你可能还想设置 [远程缓存](https://mypy.readthedocs.io/en/stable/additional_features.html#remote-cache) 以进一步加快速度。

=== "英文"
    
    You can use [mypy daemon](https://mypy.readthedocs.io/en/stable/mypy_daemon.html#mypy-daemon) to get much faster incremental mypy runs. The larger your project is, the more useful this will be. If your project has at least 100,000 lines of code or so, you may also want to set up [remote caching](https://mypy.readthedocs.io/en/stable/additional_features.html#remote-cache) for further speedups.