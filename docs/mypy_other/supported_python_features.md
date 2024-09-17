# 支持的python特性

**Supported Python features**

=== "中文"

    不支持的 Python 特性列表维护在 mypy 的 Wiki 页面中：

    - [不支持的 Python 特性](https://github.com/python/mypy/wiki/Unsupported-Python-Features)

=== "英文"

    A list of unsupported Python features is maintained in the mypy wiki:

    - [Unsupported Python features](https://github.com/python/mypy/wiki/Unsupported-Python-Features)

## 方法和函数的运行时定义


**Runtime definition of methods and functions**

=== "中文"

    默认情况下，如果您在类或模块定义之外添加一个函数，mypy 会发出警告——但仅当这种情况对类型检查器可见时。这仅影响静态检查，因为 mypy 在运行时不执行额外的类型检查。您可以轻松绕过这一点。例如，您可以使用动态类型的代码或具有 ``Any`` 类型的值，或者可以使用 [setattr] 或其他反射功能。然而，如果您决定这样做，需要小心。如果不加选择地使用，您可能会发现静态类型检查的有效性受到影响，因为类型检查器无法看到运行时定义的函数。

=== "英文"

    By default, mypy will complain if you add a function to a class or module outside its definition -- but only if this is visible to the type checker. This only affects static checking, as mypy performs no additional type checking at runtime. You can easily work around this. For example, you can use dynamically typed code or values with ``Any`` types, or you can use [setattr] or other introspection features. However, you need to be careful if you decide to do this. If used indiscriminately, you may have difficulty using static typing effectively, since the type checker cannot see functions defined at runtime.


[setattr]: https://docs.python.org/3/library/functions.html#setattr