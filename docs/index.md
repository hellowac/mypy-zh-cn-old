# 欢迎来到mypy文档!

**Welcome to mypy documentation!**

原文: <https://mypy.readthedocs.io/en/stable/>


=== "中文"

    Mypy 是一个用于 Python 的静态类型检查器。
    
    类型检查器帮助确保你在代码中正确地使用变量和函数。通过 mypy，你可以在 Python 程序中添加类型提示（[PEP 484](https://peps.python.org/pep-0484/)），然后 mypy 会在你错误使用这些类型时发出警告。
    
    Python 是一种动态语言，因此通常你只有在尝试运行代码时才能看到错误。而 mypy 作为一个静态检查器，它可以在不运行程序的情况下发现代码中的错误！
    
    下面是一个小例子来引起你的兴趣：
    
    ```python
    number = input("What is your favourite number?")
    print("It is", number + 1)  # error: Unsupported operand types for + ("str" and "int")
    ```
    
    为 mypy 添加类型提示不会干扰程序的正常运行。可以把类型提示当作类似注释的存在！即使 mypy 报告错误，你仍然可以使用 Python 解释器运行代码。
    
    Mypy 是为渐进类型设计的，这意味着你可以逐步向代码库中添加类型提示，并且在静态类型检查不方便时，可以随时回退到动态类型。
    
    Mypy 具有功能强大且易于使用的类型系统，支持类型推断、泛型、可调用类型、元组类型、联合类型、结构性子类型等功能。使用 mypy 会使你的程序更易于理解、调试和维护。
    
    !!! note "注意"
    
        尽管 mypy 已经准备好用于生产环境，但偶尔可能会有破坏向后兼容性的更改。Mypy 开发团队会尽量减少对用户代码的影响。如果发生重大破坏性更改，mypy 的主版本号将会提高。

=== "英文"

    Mypy is a static type checker for Python.
    
    Type checkers help ensure that you’re using variables and functions in your code correctly. With mypy, add type hints ([PEP 484](https://peps.python.org/pep-0484/)) to your Python programs, and mypy will warn you when you use those types incorrectly.
    
    Python is a dynamic language, so usually you’ll only see errors in your code when you attempt to run it. Mypy is a static checker, so it finds bugs in your programs without even running them!
    
    Here is a small example to whet your appetite:
    
    ```python
    number = input("What is your favourite number?")
    print("It is", number + 1)  # error: Unsupported operand types for + ("str" and "int")
    ```
    
    Adding type hints for mypy does not interfere with the way your program would otherwise run. Think of type hints as similar to comments! You can always use the Python interpreter to run your code, even if mypy reports errors.
    
    Mypy is designed with gradual typing in mind. This means you can add type hints to your code base slowly and that you can always fall back to dynamic typing when static typing is not convenient.
    
    Mypy has a powerful and easy-to-use type system, supporting features such as type inference, generics, callable types, tuple types, union types, structural subtyping and more. Using mypy will make your programs easier to understand, debug, and maintain.
    
    !!! note "Note"
    
        Although mypy is production ready, there may be occasional changes that break backward compatibility. The mypy development team tries to minimize the impact of changes to user code. In case of a major breaking change, mypy’s major version will be bumped.