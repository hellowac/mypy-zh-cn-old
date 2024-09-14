# 支持的python特性

**Supported Python features**

=== "中文"

=== "英文"

A list of unsupported Python features is maintained in the mypy wiki:

- `Unsupported Python features <https://github.com/python/mypy/wiki/Unsupported-Python-Features>`_

## 方法和函数的运行时定义


**Runtime definition of methods and functions**

=== "中文"

=== "英文"

By default, mypy will complain if you add a function to a class or module outside its definition -- but only if this is visible to the type checker. This only affects static checking, as mypy performs no additional type checking at runtime. You can easily work around this. For example, you can use dynamically typed code or values with ``Any`` types, or you can use [setattr] or other introspection features. However, you need to be careful if you decide to do this. If used indiscriminately, you may have difficulty using static typing effectively, since the type checker cannot see functions defined at runtime.


[setattr]: https://docs.python.org/3/library/functions.html#setattr