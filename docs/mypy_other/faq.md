# 常见问题

**Frequently Asked Questions**

## 为什么既有动态类型又有静态类型？

**Why have both dynamic and static typing?**

=== "中文"

    动态类型可以灵活、强大、方便且易于使用。但它并不总是最佳的选择；许多开发者选择使用静态类型语言或在 Python 中使用静态类型有其充分的理由。

    以下是 mypy 风格的静态类型的一些潜在好处：

    - 静态类型可以使程序更易于理解和维护。类型声明可以作为机器检查的文档。这一点很重要，因为代码通常比修改的频率要高得多，这对于大型和复杂的程序尤为重要。

    - 静态类型可以帮助您更早发现错误，并减少测试和调试的工作量。尤其是在大型和复杂的项目中，这可以节省大量时间。

    - 静态类型可以帮助您在代码投入生产之前发现难以发现的错误。这可以提高代码的可靠性并减少安全问题的数量。

    - 静态类型使得构建非常有用的开发工具成为可能，这些工具可以提高编程生产力或软件质量，包括具有精确和可靠代码补全的 IDE、静态分析工具等。

    - 您可以在单一语言中同时获得动态和静态类型的好处。例如，动态类型非常适合小型项目或编写程序的 UI。随着程序的增长，您可以将复杂的应用逻辑适应静态类型，以帮助维护。

    另见 mypy 网站的 [首页](https://www.mypy-lang.org)。

=== "英文"

    Dynamic typing can be flexible, powerful, convenient and easy. But it's not always the best approach; there are good reasons why many developers choose to use statically typed languages or static typing for Python.

    Here are some potential benefits of mypy-style static typing:

    - Static typing can make programs easier to understand and maintain. Type declarations can serve as machine-checked documentation. This is important as code is typically read much more often than modified, and this is especially important for large and complex programs.

    - Static typing can help you find bugs earlier and with less testing and debugging. Especially in large and complex projects this can be a major time-saver.

    - Static typing can help you find difficult-to-find bugs before your code goes into production. This can improve reliability and reduce the number of security issues.

    - Static typing makes it practical to build very useful development tools that can improve programming productivity or software quality, including IDEs with precise and reliable code completion, static analysis tools, etc.

    - You can get the benefits of both dynamic and static typing in a single language. Dynamic typing can be perfect for a small project or for writing the UI of your program, for example. As your program grows, you can adapt tricky application logic to static typing to help maintenance.

    See also the [front page](https://www.mypy-lang.org) of the mypy web site.

## 我的项目会从静态类型中受益吗？

**Would my project benefit from static typing?**

=== "中文"

    对于许多项目，动态类型是完全合适的（我们认为 Python 是一门很棒的语言）。但有时您的项目需要更强大的工具，这时 mypy 可能会派上用场。

    如果您的项目中有以下一些情况，mypy（和静态类型）可能会很有用：

    - 您的项目很大或很复杂。

    - 您的代码库需要长期维护。

    - 多个开发者在同一代码上工作。

    - 运行测试需要大量时间或工作（类型检查可以帮助您在开发早期快速发现错误，从而减少测试迭代的次数）。

    - 一些项目成员（开发者或管理人员）不喜欢动态类型，但其他人更喜欢动态类型和 Python 语法。Mypy 可能是一个大家都容易接受的解决方案。

    - 即使目前以上情况都不适用，您仍然希望为项目做好未来的准备。越早开始，采用静态类型将会越容易。

=== "英文"

    For many projects dynamic typing is perfectly fine (we think that Python is a great language). But sometimes your projects demand bigger guns, and that's when mypy may come in handy.

    If some of these ring true for your projects, mypy (and static typing) may be useful:

    - Your project is large or complex.

    - Your codebase must be maintained for a long time.

    - Multiple developers are working on the same code.

    - Running tests takes a lot of time or work (type checking helps you find errors quickly early in development, reducing the number of testing iterations).

    - Some project members (devs or management) don't like dynamic typing, but others prefer dynamic typing and Python syntax. Mypy could be a solution that everybody finds easy to accept.

    - You want to future-proof your project even if currently none of the above really apply. The earlier you start, the easier it will be to adopt static typing.

## 我可以使用 mypy 对现有的 Python 代码进行类型检查吗？

**Can I use mypy to type check my existing Python code?**

=== "中文"

    Mypy 支持大多数 Python 特性和习惯用法，许多大型 Python 项目都在成功使用 mypy。使用复杂的反射或元编程的代码可能不适合进行类型检查，但在代码库的其他较少动态的部分使用静态类型仍然是可能的。

=== "英文"

    Mypy supports most Python features and idioms, and many large Python projects are using mypy successfully. Code that uses complex introspection or metaprogramming may be impractical to type check, but it should still be possible to use static typing in other parts of a codebase that are less dynamic.

## 静态类型会让我的程序运行得更快吗？

**Will static typing make my programs run faster?**

=== "中文"

    Mypy 仅进行静态类型检查，不会提高性能。它对性能的影响极小。未来可能会有其他工具能够将静态类型的 mypy 代码编译为 C 模块或高效的 JVM 字节码，但这超出了 mypy 项目的范围。

=== "英文"

    Mypy only does static type checking and it does not improve performance. It has a minimal performance impact. In the future, there could be other tools that can compile statically typed mypy code to C modules or to efficient JVM bytecode, for example, but this is outside the scope of the mypy project.

## mypy 是免费的么？

**Is mypy free?**

=== "中文"

    是的。Mypy 是自由软件，也可以用于商业和专有项目。Mypy 采用 MIT 许可证。

=== "英文"

    Yes. Mypy is free software, and it can also be used for commercial and proprietary projects. Mypy is available under the MIT license.

## 我可以在 mypy 中使用鸭子类型吗？

**Can I use duck typing with mypy?**

=== "中文"

    Mypy 支持 [名义子类型](https://en.wikipedia.org/wiki/Nominative_type_system) 和 [结构子类型](https://en.wikipedia.org/wiki/Structural_type_system)。结构子类型可以被视为“静态鸭子类型”。有人认为，结构子类型更适合像 Python 这样的鸭子类型语言。然而，mypy 主要使用名义子类型，将结构子类型主要作为可选特性（除了始终支持结构子类型的内置协议，如 [Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable)）。原因如下：

    1. 使用名义类型系统时，生成简短而信息丰富的错误消息较为容易。这在使用类型推断时尤为重要。

    2. Python 内置了对名义 [isinstance()] 测试的支持，并且在程序中广泛使用。对结构 [isinstance()] 的支持有限，而且比名义类型测试的类型安全性差。

    3. 许多程序员已经熟悉静态的名义子类型，并且这种类型系统在 Java、C++ 和 C# 等语言中得到了成功应用。使用结构子类型的语言较少。

    然而，结构子类型也可以很有用。例如，如果一个“公共 API”使用协议类型，它可能会更灵活。此外，使用协议类型可以省去明确声明 ABC 实现的必要性。作为经验法则，我们建议在可能的情况下使用名义类，在必要时使用协议。有关协议类型和结构子类型的更多细节，请参见 [协议和结构子类型](../mypy/protocol_and_struct_subtyping.md) 和 [PEP 544](https://peps.python.org/pep-0544/)。

=== "英文"

    Mypy provides support for both [nominal subtyping](https://en.wikipedia.org/wiki/Nominative_type_system) and [structural subtyping](https://en.wikipedia.org/wiki/Structural_type_system). Structural subtyping can be thought of as "static duck typing". Some argue that structural subtyping is better suited for languages with duck typing such as Python. Mypy however primarily uses nominal subtyping, leaving structural subtyping mostly opt-in (except for built-in protocols such as [Iterable](https://docs.python.org/3/library/typing.html#typing.Iterable) that always support structural subtyping). Here are some reasons why:

    1. It is easy to generate short and informative error messages when using a nominal type system. This is especially important when using type inference.

    2. Python provides built-in support for nominal [isinstance()] tests and they are widely used in programs. Only limited support for structural [isinstance()] is available, and it's less type safe than nominal type tests.

    3. Many programmers are already familiar with static, nominal subtyping and it has been successfully used in languages such as Java, C++ and C#. Fewer languages use structural subtyping.

    However, structural subtyping can also be useful. For example, a "public API" may be more flexible if it is typed with protocols. Also, using protocol types removes the necessity to explicitly declare implementations of ABCs. As a rule of thumb, we recommend using nominal classes where possible, and protocols where necessary. For more details about protocol types and structural subtyping see [Protocols and structural subtyping](../mypy/protocol_and_struct_subtyping.md) and [PEP 544](https://peps.python.org/pep-0544/).

## 我喜欢 Python，并且不需要静态类型

**I like Python and I have no need for static typing**

=== "中文"

    mypy 的目标不是说服每个人编写静态类型的 Python —— 静态类型现在和未来都是完全可选的。目标是为 Python 程序员提供更多的选择，使 Python 在大型项目中成为其他静态类型语言的更具竞争力的替代方案，提高程序员的生产力，并提升软件质量。

=== "英文"

    The aim of mypy is not to convince everybody to write statically typed Python -- static typing is entirely optional, now and in the future. The goal is to give more options for Python programmers, to make Python a more competitive alternative to other statically typed languages in large projects, to improve programmer productivity, and to improve software quality.

## mypy 程序与普通 Python 有什么不同？

**How are mypy programs different from normal Python?**

=== "中文"

    由于您使用的是标准的 Python 实现来运行 mypy 程序，mypy 程序也是 Python 程序。类型检查器可能会对一些有效的 Python 代码发出警告，但代码始终可以运行。此外，mypy 仍然不支持一些 Python 特性和语法，但这方面的支持正在逐步改善。

    明显的区别在于静态类型检查的可用性。章节 [常见问题和解决方案](./common_issues.md) 提到了为了使代码能够无错误地通过类型检查可能需要对 Python 代码进行的一些修改。此外，您的代码必须明确声明属性。

    Mypy 支持模块化、高效的类型检查，这似乎排除了对某些语言特性（例如任意的猴子补丁方法）的类型检查。

=== "英文"

    Since you use a vanilla Python implementation to run mypy programs, mypy programs are also Python programs. The type checker may give warnings for some valid Python code, but the code is still always runnable. Also, some Python features and syntax are still not supported by mypy, but this is gradually improving.

    The obvious difference is the availability of static type checking. The section [Common issues and solutions](./common_issues.md) mentions some modifications to Python code that may be required to make code type check without errors. Also, your code must make attributes explicit.

    Mypy supports modular, efficient type checking, and this seems to rule out type checking some language features, such as arbitrary monkey patching of methods.

## mypy 与 Cython 有何不同？

**How is mypy different from Cython?**

=== "中文"

    [Cython](https://docs.cython.org/en/latest/index.html) 是一种支持编译为 CPython C 模块的 Python 变体。与 CPython 相比，它可以显著提高某些类型程序的速度，并提供静态类型（尽管这与 mypy 的实现有所不同）。mypy 在以下方面有所不同：

    - Cython 更加关注性能，而 mypy 只涉及静态类型检查，提升性能不是直接目标。

    - mypy 的语法可以说更简单、更“Pythonic”（没有 cdef/cpdef 等）用于静态类型代码。

    - mypy 的语法与 Python 兼容。mypy 程序是可以通过任何 Python 实现运行的正常 Python 程序。Cython 有许多与 Python 语法不兼容的扩展，Cython 程序通常不能在不首先编译为 CPython 扩展模块的情况下运行。Cython 也有一个纯 Python 模式，但它似乎仅支持 Cython 功能的一个子集，并且语法相当冗长。

    - mypy 具有不同的类型系统特性。例如，mypy 支持泛型（参数多态）、函数类型和双向类型推断，而 Cython 不支持这些特性。（Cython 有与 mypy 泛型相关但不同的融合类型。mypy 也有类似的特性作为泛型的扩展。）

    - mypy 类型检查器了解许多 Python 标准库模块的静态类型，并可以有效地对使用这些模块的代码进行类型检查。

    - Cython 支持直接访问 C 函数，并且许多特性都是通过将它们翻译成 C 或 C++ 来定义的。mypy 只使用 Python 语义，mypy 不处理访问 C 库功能的事宜。

=== "英文"

    [Cython](https://docs.cython.org/en/latest/index.html) is a variant of Python that supports compilation to CPython C modules. It can give major speedups to certain classes of programs compared to CPython, and it provides static typing (though this is different from mypy). Mypy differs in the following aspects, among others:

    - Cython is much more focused on performance than mypy. Mypy is only about static type checking, and increasing performance is not a direct goal.

    - The mypy syntax is arguably simpler and more "Pythonic" (no cdef/cpdef, etc.) for statically typed code.

    - The mypy syntax is compatible with Python. Mypy programs are normal Python programs that can be run using any Python implementation. Cython has many incompatible extensions to Python syntax, and Cython programs generally cannot be run without first compiling them to CPython extension modules via C. Cython also has a pure Python mode, but it seems to support only a subset of Cython functionality, and the syntax is quite verbose.

    - Mypy has a different set of type system features. For example, mypy has genericity (parametric polymorphism), function types and bidirectional type inference, which are not supported by Cython. (Cython has fused types that are different but related to mypy generics. Mypy also has a similar feature as an extension of generics.)

    - The mypy type checker knows about the static types of many Python stdlib modules and can effectively type check code that uses them.

    - Cython supports accessing C functions directly and many features are defined in terms of translating them to C or C++. Mypy just uses Python semantics, and mypy does not deal with accessing C library functionality.

## 它可以在 PyPy 上运行吗？

**Does it run on PyPy?**

=== "中文"

    在一定程度上是的。使用 PyPy 3.8，mypy 至少能够进行自身的类型检查。对于较旧版本的 PyPy，mypy 依赖于 [typed-ast](https://github.com/python/typed_ast)，它使用了一些 PyPy 不支持的 API（包括一些 CPython 的内部 API）。

=== "英文"

    Somewhat. With PyPy 3.8, mypy is at least able to type check itself. With older versions of PyPy, mypy relies on [typed-ast](https://github.com/python/typed_ast), which uses several APIs that PyPy does not support (including some internal CPython APIs).

## mypy 是一个很棒的项目。我可以帮助吗？

**Mypy is a cool project. Can I help?**

=== "中文"

    任何帮助都非常感谢！如果您希望做出贡献，请 [联系](https://www.mypy-lang.org/contact.html) 开发者。任何与开发、设计、宣传、文档、测试、网站维护、融资等相关的帮助都是有用的。通过贡献您可以学到很多东西，任何人都可以帮助，甚至是初学者！不过，如果您希望参与 mypy 内部的工作，了解编译器和/或类型系统的知识是必不可少的。

=== "英文"

    Any help is much appreciated! [Contact](https://www.mypy-lang.org/contact.html) the developers if you would like to contribute. Any help related to development, design, publicity, documentation, testing, web site maintenance, financing, etc. can be helpful. You can learn a lot by contributing, and anybody can help, even beginners! However, some knowledge of compilers and/or type systems is essential if you want to work on mypy internals.

[isinstance()]: https://docs.python.org/3/library/functions.html#isinstance