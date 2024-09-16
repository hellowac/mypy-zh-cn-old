# 默认启用的错误代码

**Error codes enabled by default**

=== "中文"

    This section documents various errors codes that mypy can generate with default options. See [Error codes](./error_codes.md) for general documentation about error codes.  [Error codes for optional checks](./error_code_list2.md) documents additional error codes that you can enable.

=== "英文"

    This section documents various errors codes that mypy can generate with default options. See [Error codes](./error_codes.md) for general documentation about error codes.  [Error codes for optional checks](./error_code_list2.md) documents additional error codes that you can enable.

## 检查属性是否存在 [attr-defined]

**Check that attribute exists [attr-defined]**

=== "中文"

    Mypy checks that an attribute is defined in the target class or module when using the dot operator. This applies to both getting and setting an attribute. New attributes are defined by assignments in the class body, or assignments to ``self.x`` in methods. These assignments don't generate ``attr-defined`` errors.

    Example:

    ```python
    class Resource:
        def __init__(self, name: str) -> None:
            self.name = name

    r = Resource('x')
    print(r.name)  # OK
    print(r.id)  # Error: "Resource" has no attribute "id"  [attr-defined]
    r.id = 5  # Error: "Resource" has no attribute "id"  [attr-defined]
    ```

    This error code is also generated if an imported name is not defined in the module in a ``from ... import`` statement (as long as the target module can be found):

    ```python
    # Error: Module "os" has no attribute "non_existent"  [attr-defined]
    from os import non_existent
    ```

    A reference to a missing attribute is given the ``Any`` type. In the above example, the type of ``non_existent`` will be ``Any``, which can be important if you silence the error.

=== "英文"

    Mypy checks that an attribute is defined in the target class or module when using the dot operator. This applies to both getting and setting an attribute. New attributes are defined by assignments in the class body, or assignments to ``self.x`` in methods. These assignments don't generate ``attr-defined`` errors.

    Example:

    ```python
    class Resource:
        def __init__(self, name: str) -> None:
            self.name = name

    r = Resource('x')
    print(r.name)  # OK
    print(r.id)  # Error: "Resource" has no attribute "id"  [attr-defined]
    r.id = 5  # Error: "Resource" has no attribute "id"  [attr-defined]
    ```

    This error code is also generated if an imported name is not defined in the module in a ``from ... import`` statement (as long as the target module can be found):

    ```python
    # Error: Module "os" has no attribute "non_existent"  [attr-defined]
    from os import non_existent
    ```

    A reference to a missing attribute is given the ``Any`` type. In the above example, the type of ``non_existent`` will be ``Any``, which can be important if you silence the error.

## 检查每个联合项中是否存在属性 [union-attr]

**Check that attribute exists in each union item [union-attr]**

=== "中文"

    If you access the attribute of a value with a union type, mypy checks that the attribute is defined for *every* type in that union. Otherwise the operation can fail at runtime. This also applies to optional types.

    Example:

    ```python
    from typing import Union

    class Cat:
        def sleep(self) -> None: ...
        def miaow(self) -> None: ...

    class Dog:
        def sleep(self) -> None: ...
        def follow_me(self) -> None: ...

    def func(animal: Union[Cat, Dog]) -> None:
        # OK: 'sleep' is defined for both Cat and Dog
        animal.sleep()
        # Error: Item "Cat" of "Union[Cat, Dog]" has no attribute "follow_me"  [union-attr]
        animal.follow_me()
    ```

    You can often work around these errors by using ``assert isinstance(obj, ClassName)`` or ``assert obj is not None`` to tell mypy that you know that the type is more specific than what mypy thinks.

=== "英文"

    If you access the attribute of a value with a union type, mypy checks that the attribute is defined for *every* type in that union. Otherwise the operation can fail at runtime. This also applies to optional types.

    Example:

    ```python
    from typing import Union

    class Cat:
        def sleep(self) -> None: ...
        def miaow(self) -> None: ...

    class Dog:
        def sleep(self) -> None: ...
        def follow_me(self) -> None: ...

    def func(animal: Union[Cat, Dog]) -> None:
        # OK: 'sleep' is defined for both Cat and Dog
        animal.sleep()
        # Error: Item "Cat" of "Union[Cat, Dog]" has no attribute "follow_me"  [union-attr]
        animal.follow_me()
    ```

    You can often work around these errors by using ``assert isinstance(obj, ClassName)`` or ``assert obj is not None`` to tell mypy that you know that the type is more specific than what mypy thinks.

## 检查名称是否已定义 [name-defined]

**Check that name is defined [name-defined]**

=== "中文"

    Mypy expects that all references to names have a corresponding definition in an active scope, such as an assignment, function definition or an import. This can catch missing definitions, missing imports, and typos.

    This example accidentally calls ``sort()`` instead of [sorted()]:

    ```python
    x = sort([3, 2, 4])  # Error: Name "sort" is not defined  [name-defined]
    ```

=== "英文"

    Mypy expects that all references to names have a corresponding definition in an active scope, such as an assignment, function definition or an import. This can catch missing definitions, missing imports, and typos.

    This example accidentally calls ``sort()`` instead of [sorted()]:

    ```python
    x = sort([3, 2, 4])  # Error: Name "sort" is not defined  [name-defined]
    ```

## 检查变量是否在定义前被使用 [used-before-def]

**Check that a variable is not used before it's defined [used-before-def]**

=== "中文"

    Mypy will generate an error if a name is used before it's defined. While the name-defined check will catch issues with names that are undefined, it will not flag if a variable is used and then defined later in the scope. used-before-def check will catch such cases.

    Example:

    ```python
    print(x)  # Error: Name "x" is used before definition [used-before-def]
    x = 123
    ```

=== "英文"

    Mypy will generate an error if a name is used before it's defined. While the name-defined check will catch issues with names that are undefined, it will not flag if a variable is used and then defined later in the scope. used-before-def check will catch such cases.

    Example:

    ```python
    print(x)  # Error: Name "x" is used before definition [used-before-def]
    x = 123
    ```

## 检查调用中的参数 [call-arg]

**Check arguments in calls [call-arg]**

=== "中文"

    Mypy expects that the number and names of arguments match the called function. Note that argument type checks have a separate error code ``arg-type``.

    Example:

    ```python
    from typing import Sequence

    def greet(name: str) -> None:
            print('hello', name)

    greet('jack')  # OK
    greet('jill', 'jack')  # Error: Too many arguments for "greet"  [call-arg]
    ```

=== "英文"

    Mypy expects that the number and names of arguments match the called function. Note that argument type checks have a separate error code ``arg-type``.

    Example:

    ```python
    from typing import Sequence

    def greet(name: str) -> None:
            print('hello', name)

    greet('jack')  # OK
    greet('jill', 'jack')  # Error: Too many arguments for "greet"  [call-arg]
    ```

## 检查参数类型 [arg-type]

**Check argument types [arg-type]**

=== "中文"

    Mypy checks that argument types in a call match the declared argument types in the signature of the called function (if one exists).

    Example:

    ```python
    from typing import Optional

    def first(x: list[int]) -> Optional[int]:
        return x[0] if x else 0

    t = (5, 4)
    # Error: Argument 1 to "first" has incompatible type "tuple[int, int]";
    #        expected "list[int]"  [arg-type]
    print(first(t))
    ```

=== "英文"

    Mypy checks that argument types in a call match the declared argument types in the signature of the called function (if one exists).

    Example:

    ```python
    from typing import Optional

    def first(x: list[int]) -> Optional[int]:
        return x[0] if x else 0

    t = (5, 4)
    # Error: Argument 1 to "first" has incompatible type "tuple[int, int]";
    #        expected "list[int]"  [arg-type]
    print(first(t))
    ```

## 检查对重载函数的调用 [call-overload]

**Check calls to overloaded functions [call-overload]**

=== "中文"

    When you call an overloaded function, mypy checks that at least one of the signatures of the overload items match the argument types in the call.

    Example:

    ```python

    from typing import overload, Optional

    @overload
    def inc_maybe(x: None) -> None: ...

    @overload
    def inc_maybe(x: int) -> int: ...

    def inc_maybe(x: Optional[int]) -> Optional[int]:
        if x is None:
            return None
        else:
            return x + 1

    inc_maybe(None)  # OK
    inc_maybe(5)  # OK

    # Error: No overload variant of "inc_maybe" matches argument type "float"  [call-overload]
    inc_maybe(1.2)
    ```

=== "英文"

    When you call an overloaded function, mypy checks that at least one of the signatures of the overload items match the argument types in the call.

    Example:

    ```python

    from typing import overload, Optional

    @overload
    def inc_maybe(x: None) -> None: ...

    @overload
    def inc_maybe(x: int) -> int: ...

    def inc_maybe(x: Optional[int]) -> Optional[int]:
        if x is None:
            return None
        else:
            return x + 1

    inc_maybe(None)  # OK
    inc_maybe(5)  # OK

    # Error: No overload variant of "inc_maybe" matches argument type "float"  [call-overload]
    inc_maybe(1.2)
    ```

## 检查类型的有效性 [valid-type]

**Check validity of types [valid-type]**

=== "中文"

    Mypy checks that each type annotation and any expression that represents a type is a valid type. Examples of valid types include classes, union types, callable types, type aliases, and literal types. Examples of invalid types include bare integer literals, functions, variables, and modules.

    This example incorrectly uses the function ``log`` as a type:

    ```python
    def log(x: object) -> None:
        print('log:', repr(x))

    # Error: Function "t.log" is not valid as a type  [valid-type]
    def log_all(objs: list[object], f: log) -> None:
        for x in objs:
            f(x)
    ```

    You can use :py:data:`~typing.Callable` as the type for callable objects:

    ```python
    from typing import Callable

    # OK
    def log_all(objs: list[object], f: Callable[[object], None]) -> None:
        for x in objs:
            f(x)
    ```

=== "英文"

    Mypy checks that each type annotation and any expression that represents a type is a valid type. Examples of valid types include classes, union types, callable types, type aliases, and literal types. Examples of invalid types include bare integer literals, functions, variables, and modules.

    This example incorrectly uses the function ``log`` as a type:

    ```python
    def log(x: object) -> None:
        print('log:', repr(x))

    # Error: Function "t.log" is not valid as a type  [valid-type]
    def log_all(objs: list[object], f: log) -> None:
        for x in objs:
            f(x)
    ```

    You can use :py:data:`~typing.Callable` as the type for callable objects:

    ```python
    from typing import Callable

    # OK
    def log_all(objs: list[object], f: Callable[[object], None]) -> None:
        for x in objs:
            f(x)
    ```

## 如果变量类型不明确，则要求注解 [var-annotated]

**Require annotation if variable type is unclear [var-annotated]**

=== "中文"

    In some cases mypy can't infer the type of a variable without an explicit annotation. Mypy treats this as an error. This typically happens when you initialize a variable with an empty collection or ``None``.  If mypy can't infer the collection item type, mypy replaces any parts of the type it couldn't infer with ``Any`` and generates an error.

    Example with an error:

    ```python

    class Bundle:
        def __init__(self) -> None:
            # Error: Need type annotation for "items"
            #        (hint: "items: list[<type>] = ...")  [var-annotated]
            self.items = []

    reveal_type(Bundle().items)  # list[Any]
    ```

    To address this, we add an explicit annotation:

    ```python

    class Bundle:
        def __init__(self) -> None:
            self.items: list[str] = []  # OK

    reveal_type(Bundle().items)  # list[str]
    ```

=== "英文"

    In some cases mypy can't infer the type of a variable without an explicit annotation. Mypy treats this as an error. This typically happens when you initialize a variable with an empty collection or ``None``.  If mypy can't infer the collection item type, mypy replaces any parts of the type it couldn't infer with ``Any`` and generates an error.

    Example with an error:

    ```python

    class Bundle:
        def __init__(self) -> None:
            # Error: Need type annotation for "items"
            #        (hint: "items: list[<type>] = ...")  [var-annotated]
            self.items = []

    reveal_type(Bundle().items)  # list[Any]
    ```

    To address this, we add an explicit annotation:

    ```python

    class Bundle:
        def __init__(self) -> None:
            self.items: list[str] = []  # OK

    reveal_type(Bundle().items)  # list[str]
    ```

## 检查重写的有效性 [override]

**Check validity of overrides [override]**

=== "中文"

    Mypy checks that an overridden method or attribute is compatible with the base class.  A method in a subclass must accept all arguments that the base class method accepts, and the return type must conform to the return type in the base class (Liskov substitution principle).

    Argument types can be more general is a subclass (i.e., they can vary contravariantly).  The return type can be narrowed in a subclass (i.e., it can vary covariantly).  It's okay to define additional arguments in a subclass method, as long all extra arguments have default values or can be left out (``*args``, for example).

    Example:

    ```python

    from typing import Optional, Union

    class Base:
        def method(self,
                    arg: int) -> Optional[int]:
            ...

    class Derived(Base):
        def method(self,
                    arg: Union[int, str]) -> int:  # OK
            ...

    class DerivedBad(Base):
        # Error: Argument 1 of "method" is incompatible with "Base"  [override]
        def method(self,
                    arg: bool) -> int:
            ...
    ```

=== "英文"

    Mypy checks that an overridden method or attribute is compatible with the base class.  A method in a subclass must accept all arguments that the base class method accepts, and the return type must conform to the return type in the base class (Liskov substitution principle).

    Argument types can be more general is a subclass (i.e., they can vary contravariantly).  The return type can be narrowed in a subclass (i.e., it can vary covariantly).  It's okay to define additional arguments in a subclass method, as long all extra arguments have default values or can be left out (``*args``, for example).

    Example:

    ```python

    from typing import Optional, Union

    class Base:
        def method(self,
                    arg: int) -> Optional[int]:
            ...

    class Derived(Base):
        def method(self,
                    arg: Union[int, str]) -> int:  # OK
            ...

    class DerivedBad(Base):
        # Error: Argument 1 of "method" is incompatible with "Base"  [override]
        def method(self,
                    arg: bool) -> int:
            ...
    ```

## 检查函数是否返回值 [return]

**Check that function returns a value [return]**

=== "中文"

    If a function has a non-``None`` return type, mypy expects that the function always explicitly returns a value (or raises an exception). The function should not fall off the end of the function, since this is often a bug.

    Example:

    ```python

    # Error: Missing return statement  [return]
    def show(x: int) -> int:
        print(x)

    # Error: Missing return statement  [return]
    def pred1(x: int) -> int:
        if x > 0:
            return x - 1

    # OK
    def pred2(x: int) -> int:
        if x > 0:
            return x - 1
        else:
            raise ValueError('not defined for zero')
    ```

=== "英文"

    If a function has a non-``None`` return type, mypy expects that the function always explicitly returns a value (or raises an exception). The function should not fall off the end of the function, since this is often a bug.

    Example:

    ```python

    # Error: Missing return statement  [return]
    def show(x: int) -> int:
        print(x)

    # Error: Missing return statement  [return]
    def pred1(x: int) -> int:
        if x > 0:
            return x - 1

    # OK
    def pred2(x: int) -> int:
        if x > 0:
            return x - 1
        else:
            raise ValueError('not defined for zero')
    ```

## 检查函数体是否为空（不包括存根） [empty-body]

**Check that functions don't have empty bodies outside stubs [empty-body]**

=== "中文"

    This error code is similar to the ``[return]`` code but is emitted specifically for functions and methods with empty bodies (if they are annotated with non-trivial return type). Such a distinction exists because in some contexts an empty body can be valid, for example for an abstract method or in a stub file. Also old versions of mypy used to unconditionally allow functions with empty bodies, so having a dedicated error code simplifies cross-version compatibility.

    Note that empty bodies are allowed for methods in *protocols*, and such methods are considered implicitly abstract:

    ```python

    from abc import abstractmethod
    from typing import Protocol

    class RegularABC:
        @abstractmethod
        def foo(self) -> int:
            pass  # OK
        def bar(self) -> int:
            pass  # Error: Missing return statement  [empty-body]

    class Proto(Protocol):
        def bar(self) -> int:
            pass  # OK
    ```

=== "英文"

    This error code is similar to the ``[return]`` code but is emitted specifically for functions and methods with empty bodies (if they are annotated with non-trivial return type). Such a distinction exists because in some contexts an empty body can be valid, for example for an abstract method or in a stub file. Also old versions of mypy used to unconditionally allow functions with empty bodies, so having a dedicated error code simplifies cross-version compatibility.

    Note that empty bodies are allowed for methods in *protocols*, and such methods are considered implicitly abstract:

    ```python

    from abc import abstractmethod
    from typing import Protocol

    class RegularABC:
        @abstractmethod
        def foo(self) -> int:
            pass  # OK
        def bar(self) -> int:
            pass  # Error: Missing return statement  [empty-body]

    class Proto(Protocol):
        def bar(self) -> int:
            pass  # OK
    ```

## 检查返回值是否兼容 [return-value]

**Check that return value is compatible [return-value]**

=== "中文"

    Mypy checks that the returned value is compatible with the type signature of the function.

    Example:

    ```python
    def func(x: int) -> str:
        # Error: Incompatible return value type (got "int", expected "str")  [return-value]
        return x + 1
    ```

=== "英文"

    Mypy checks that the returned value is compatible with the type signature of the function.

    Example:

    ```python
    def func(x: int) -> str:
        # Error: Incompatible return value type (got "int", expected "str")  [return-value]
        return x + 1
    ```

## 检查赋值语句中的类型 [assignment]

**Check types in assignment statement [assignment]**

=== "中文"

    Mypy checks that the assigned expression is compatible with the assignment target (or targets).

    Example:

    ```python
    class Resource:
        def __init__(self, name: str) -> None:
            self.name = name

    r = Resource('A')

    r.name = 'B'  # OK

    # Error: Incompatible types in assignment (expression has type "int",
    #        variable has type "str")  [assignment]
    r.name = 5
    ```

=== "英文"

    Mypy checks that the assigned expression is compatible with the assignment target (or targets).

    Example:

    ```python
    class Resource:
        def __init__(self, name: str) -> None:
            self.name = name

    r = Resource('A')

    r.name = 'B'  # OK

    # Error: Incompatible types in assignment (expression has type "int",
    #        variable has type "str")  [assignment]
    r.name = 5
    ```

## 检查赋值目标是否不是方法 [method-assign]

**Check that assignment target is not a method [method-assign]**

=== "中文"

    In general, assigning to a method on class object or instance (a.k.a. monkey-patching) is ambiguous in terms of types, since Python's static type system cannot express the difference between bound and unbound callable types. Consider this example:

    ```python
    class A:
        def f(self) -> None: pass
        def g(self) -> None: pass

    def h(self: A) -> None: pass

    A.f = h  # Type of h is Callable[[A], None]
    A().f()  # This works
    A.f = A().g  # Type of A().g is Callable[[], None]
    A().f()  # ...but this also works at runtime
    ```

    To prevent the ambiguity, mypy will flag both assignments by default. If this error code is disabled, mypy will treat the assigned value in all method assignments as unbound, so only the second assignment will still generate an error.

    !!! note 

        This error code is a subcode of the more general ``[assignment]`` code.

=== "英文"

    In general, assigning to a method on class object or instance (a.k.a. monkey-patching) is ambiguous in terms of types, since Python's static type system cannot express the difference between bound and unbound callable types. Consider this example:

    ```python
    class A:
        def f(self) -> None: pass
        def g(self) -> None: pass

    def h(self: A) -> None: pass

    A.f = h  # Type of h is Callable[[A], None]
    A().f()  # This works
    A.f = A().g  # Type of A().g is Callable[[], None]
    A().f()  # ...but this also works at runtime
    ```

    To prevent the ambiguity, mypy will flag both assignments by default. If this error code is disabled, mypy will treat the assigned value in all method assignments as unbound, so only the second assignment will still generate an error.

    !!! note 

        This error code is a subcode of the more general ``[assignment]`` code.

## 检查类型变量值 [type-var]

**Check type variable values [type-var]**

=== "中文"

    Mypy checks that value of a type variable is compatible with a value restriction or the upper bound type.

    Example:

    ```python
    from typing import TypeVar

    T1 = TypeVar('T1', int, float)

    def add(x: T1, y: T1) -> T1:
        return x + y

    add(4, 5.5)  # OK

    # Error: Value of type variable "T1" of "add" cannot be "str"  [type-var]
    add('x', 'y')
    ```

=== "英文"

    Mypy checks that value of a type variable is compatible with a value restriction or the upper bound type.

    Example:

    ```python
    from typing import TypeVar

    T1 = TypeVar('T1', int, float)

    def add(x: T1, y: T1) -> T1:
        return x + y

    add(4, 5.5)  # OK

    # Error: Value of type variable "T1" of "add" cannot be "str"  [type-var]
    add('x', 'y')
    ```

## 检查各种操作符的使用 [operator]

**Check uses of various operators [operator]**

=== "中文"

    Mypy checks that operands support a binary or unary operation, such as ``+`` or ``~``. Indexing operations are so common that they have their own error code ``index`` (see below).

    Example:

    ```python
    # Error: Unsupported operand types for + ("int" and "str")  [operator]
    1 + 'x'
    ```

=== "英文"

    Mypy checks that operands support a binary or unary operation, such as ``+`` or ``~``. Indexing operations are so common that they have their own error code ``index`` (see below).

    Example:

    ```python
    # Error: Unsupported operand types for + ("int" and "str")  [operator]
    1 + 'x'
    ```

## 检查索引操作 [index]

**Check indexing operations [index]**

=== "中文"

    Mypy checks that the indexed value in indexing operation such as ``x[y]`` supports indexing, and that the index expression has a valid type.

    Example:

    ```python
    a = {'x': 1, 'y': 2}

    a['x']  # OK

    # Error: Invalid index type "int" for "dict[str, int]"; expected type "str"  [index]
    print(a[1])

    # Error: Invalid index type "bytes" for "dict[str, int]"; expected type "str"  [index]
    a[b'x'] = 4
    ```

=== "英文"

    Mypy checks that the indexed value in indexing operation such as ``x[y]`` supports indexing, and that the index expression has a valid type.

    Example:

    ```python
    a = {'x': 1, 'y': 2}

    a['x']  # OK

    # Error: Invalid index type "int" for "dict[str, int]"; expected type "str"  [index]
    print(a[1])

    # Error: Invalid index type "bytes" for "dict[str, int]"; expected type "str"  [index]
    a[b'x'] = 4
    ```

## 检查列表项 [list-item]

**Check list items [list-item]**

=== "中文"

    When constructing a list using ``[item, ...]``, mypy checks that each item is compatible with the list type that is inferred from the surrounding context.

    Example:

    ```python

    # Error: List item 0 has incompatible type "int"; expected "str"  [list-item]
    a: list[str] = [0]
    ```

=== "英文"

    When constructing a list using ``[item, ...]``, mypy checks that each item is compatible with the list type that is inferred from the surrounding context.

    Example:

    ```python

    # Error: List item 0 has incompatible type "int"; expected "str"  [list-item]
    a: list[str] = [0]
    ```

## 检查字典项 [dict-item]

**Check dict items [dict-item]**

=== "中文"

    When constructing a dictionary using ``{key: value, ...}`` or ``dict(key=value, ...)``, mypy checks that each key and value is compatible with the dictionary type that is inferred from the surrounding context.

    Example:

    ```python
    # Error: Dict entry 0 has incompatible type "str": "str"; expected "str": "int"  [dict-item]
    d: dict[str, int] = {'key': 'value'}
    ```

=== "英文"

    When constructing a dictionary using ``{key: value, ...}`` or ``dict(key=value, ...)``, mypy checks that each key and value is compatible with the dictionary type that is inferred from the surrounding context.

    Example:

    ```python
    # Error: Dict entry 0 has incompatible type "str": "str"; expected "str": "int"  [dict-item]
    d: dict[str, int] = {'key': 'value'}
    ```

## 检查 TypedDict 项 [typeddict-item]

**Check TypedDict items [typeddict-item]**

=== "中文"

    When constructing a TypedDict object, mypy checks that each key and value is compatible with the TypedDict type that is inferred from the surrounding context.

    When getting a TypedDict item, mypy checks that the key exists. When assigning to a TypedDict, mypy checks that both the key and the value are valid.

    Example:

    ```python

    from typing import TypedDict

    class Point(TypedDict):
        x: int
        y: int

    # Error: Incompatible types (expression has type "float",
    #        TypedDict item "x" has type "int")  [typeddict-item]
    p: Point = {'x': 1.2, 'y': 4}
    ```

=== "英文"

    When constructing a TypedDict object, mypy checks that each key and value is compatible with the TypedDict type that is inferred from the surrounding context.

    When getting a TypedDict item, mypy checks that the key exists. When assigning to a TypedDict, mypy checks that both the key and the value are valid.

    Example:

    ```python

    from typing import TypedDict

    class Point(TypedDict):
        x: int
        y: int

    # Error: Incompatible types (expression has type "float",
    #        TypedDict item "x" has type "int")  [typeddict-item]
    p: Point = {'x': 1.2, 'y': 4}
    ```

## 检查 TypedDict 键 [typeddict-unknown-key]

**Check TypedDict Keys [typeddict-unknown-key]**

=== "中文"

    When constructing a TypedDict object, mypy checks whether the definition contains unknown keys, to catch invalid keys and misspellings. On the other hand, mypy will not generate an error when a previously constructed TypedDict value with extra keys is passed to a function as an argument, since TypedDict values support structural subtyping ("static duck typing") and the keys are assumed to have been validated at the point of construction. Example:

    ```python

    from typing import TypedDict

    class Point(TypedDict):
        x: int
        y: int

    class Point3D(Point):
        z: int

    def add_x_coordinates(a: Point, b: Point) -> int:
        return a["x"] + b["x"]

    a: Point = {"x": 1, "y": 4}
    b: Point3D = {"x": 2, "y": 5, "z": 6}

    add_x_coordinates(a, b)  # OK

    # Error: Extra key "z" for TypedDict "Point"  [typeddict-unknown-key]
    add_x_coordinates(a, {"x": 1, "y": 4, "z": 5})
    ```

    Setting a TypedDict item using an unknown key will also generate this error, since it could be a misspelling:

    ```python
    a: Point = {"x": 1, "y": 2}
    # Error: Extra key "z" for TypedDict "Point"  [typeddict-unknown-key]
    a["z"] = 3
    ```

    Reading an unknown key will generate the more general (and serious) ``typeddict-item`` error, which is likely to result in an exception at runtime:

    ```python
    a: Point = {"x": 1, "y": 2}
    # Error: TypedDict "Point" has no key "z"  [typeddict-item]
    _ = a["z"]
    ```

    !!! note 

        This error code is a subcode of the wider ``[typeddict-item]`` code.

=== "英文"

    When constructing a TypedDict object, mypy checks whether the definition contains unknown keys, to catch invalid keys and misspellings. On the other hand, mypy will not generate an error when a previously constructed TypedDict value with extra keys is passed to a function as an argument, since TypedDict values support structural subtyping ("static duck typing") and the keys are assumed to have been validated at the point of construction. Example:

    ```python

    from typing import TypedDict

    class Point(TypedDict):
        x: int
        y: int

    class Point3D(Point):
        z: int

    def add_x_coordinates(a: Point, b: Point) -> int:
        return a["x"] + b["x"]

    a: Point = {"x": 1, "y": 4}
    b: Point3D = {"x": 2, "y": 5, "z": 6}

    add_x_coordinates(a, b)  # OK

    # Error: Extra key "z" for TypedDict "Point"  [typeddict-unknown-key]
    add_x_coordinates(a, {"x": 1, "y": 4, "z": 5})
    ```

    Setting a TypedDict item using an unknown key will also generate this error, since it could be a misspelling:

    ```python
    a: Point = {"x": 1, "y": 2}
    # Error: Extra key "z" for TypedDict "Point"  [typeddict-unknown-key]
    a["z"] = 3
    ```

    Reading an unknown key will generate the more general (and serious) ``typeddict-item`` error, which is likely to result in an exception at runtime:

    ```python
    a: Point = {"x": 1, "y": 2}
    # Error: TypedDict "Point" has no key "z"  [typeddict-item]
    _ = a["z"]
    ```

    !!! note 

        This error code is a subcode of the wider ``[typeddict-item]`` code.

## 检查目标类型是否已知 [has-type]

**Check that type of target is known [has-type]**

=== "中文"

    Mypy sometimes generates an error when it hasn't inferred any type for a variable being referenced. This can happen for references to variables that are initialized later in the source file, and for references across modules that form an import cycle. When this happens, the reference gets an implicit ``Any`` type.

    In this example the definitions of ``x`` and ``y`` are circular:

    ```python

    class Problem:
        def set_x(self) -> None:
            # Error: Cannot determine type of "y"  [has-type]
            self.x = self.y

        def set_y(self) -> None:
            self.y = self.x
    ```

    To work around this error, you can add an explicit type annotation to the target variable or attribute. Sometimes you can also reorganize the code so that the definition of the variable is placed earlier than the reference to the variable in a source file. Untangling cyclic imports may also help.

    We add an explicit annotation to the ``y`` attribute to work around the issue:

    ```python
    class Problem:
        def set_x(self) -> None:
            self.x = self.y  # OK

        def set_y(self) -> None:
            self.y: int = self.x  # Added annotation here
    ```

=== "英文"

    Mypy sometimes generates an error when it hasn't inferred any type for a variable being referenced. This can happen for references to variables that are initialized later in the source file, and for references across modules that form an import cycle. When this happens, the reference gets an implicit ``Any`` type.

    In this example the definitions of ``x`` and ``y`` are circular:

    ```python

    class Problem:
        def set_x(self) -> None:
            # Error: Cannot determine type of "y"  [has-type]
            self.x = self.y

        def set_y(self) -> None:
            self.y = self.x
    ```

    To work around this error, you can add an explicit type annotation to the target variable or attribute. Sometimes you can also reorganize the code so that the definition of the variable is placed earlier than the reference to the variable in a source file. Untangling cyclic imports may also help.

    We add an explicit annotation to the ``y`` attribute to work around the issue:

    ```python
    class Problem:
        def set_x(self) -> None:
            self.x = self.y  # OK

        def set_y(self) -> None:
            self.y: int = self.x  # Added annotation here
    ```

## 检查导入问题 [import]

**Check for an issue with imports [import]**

=== "中文"

    Mypy generates an error if it can't resolve an `import` statement. This is a parent error code of `import-not-found` and `import-untyped`

    See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for how to work around these errors.

=== "英文"

    Mypy generates an error if it can't resolve an `import` statement. This is a parent error code of `import-not-found` and `import-untyped`

    See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for how to work around these errors.

## 检查导入目标是否可以找到 [import-not-found]

**Check that import target can be found [import-not-found]**

=== "中文"

    Mypy generates an error if it can't find the source code or a stub file for an imported module.

    Example:

    ```python

    # Error: Cannot find implementation or library stub for module named "m0dule_with_typo"  [import-not-found]
    import m0dule_with_typo
    ```

    See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for how to work around these errors.

=== "英文"

    Mypy generates an error if it can't find the source code or a stub file for an imported module.

    Example:

    ```python

    # Error: Cannot find implementation or library stub for module named "m0dule_with_typo"  [import-not-found]
    import m0dule_with_typo
    ```

    See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for how to work around these errors.

## 检查导入目标是否可以找到 [import-untyped]

**Check that import target can be found [import-untyped]**

=== "中文"

    Mypy generates an error if it can find the source code for an imported module, but that module does not provide type annotations (via [PEP 561](../mypy_conf/installed_packages.md)).

    Example:

    ```python
    # Error: Library stubs not installed for "bs4"  [import-untyped]
    import bs4
    # Error: Skipping analyzing "no_py_typed": module is installed, but missing library stubs or py.typed marker  [import-untyped]
    import no_py_typed
    ```

    In some cases, these errors can be fixed by installing an appropriate stub package. See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for more details.

=== "英文"

    Mypy generates an error if it can find the source code for an imported module, but that module does not provide type annotations (via [PEP 561](../mypy_conf/installed_packages.md)).

    Example:

    ```python
    # Error: Library stubs not installed for "bs4"  [import-untyped]
    import bs4
    # Error: Skipping analyzing "no_py_typed": module is installed, but missing library stubs or py.typed marker  [import-untyped]
    import no_py_typed
    ```

    In some cases, these errors can be fixed by installing an appropriate stub package. See [Missing imports](../mypy_conf/running_mypy.md#缺失的导入) for more details.

## 检查每个名称是否仅定义一次 [no-redef]

**Check that each name is defined once [no-redef]**

=== "中文"

    Mypy may generate an error if you have multiple definitions for a name in the same namespace.  The reason is that this is often an error, as the second definition may overwrite the first one. Also, mypy often can't be able to determine whether references point to the first or the second definition, which would compromise type checking.

    If you silence this error, all references to the defined name refer to the *first* definition.

    Example:

    ```python
    class A:
        def __init__(self, x: int) -> None: ...

    class A:  # Error: Name "A" already defined on line 1  [no-redef]
        def __init__(self, x: str) -> None: ...

    # Error: Argument 1 to "A" has incompatible type "str"; expected "int"
    #        (the first definition wins!)
    A('x')
    ```

=== "英文"

    Mypy may generate an error if you have multiple definitions for a name in the same namespace.  The reason is that this is often an error, as the second definition may overwrite the first one. Also, mypy often can't be able to determine whether references point to the first or the second definition, which would compromise type checking.

    If you silence this error, all references to the defined name refer to the *first* definition.

    Example:

    ```python
    class A:
        def __init__(self, x: int) -> None: ...

    class A:  # Error: Name "A" already defined on line 1  [no-redef]
        def __init__(self, x: str) -> None: ...

    # Error: Argument 1 to "A" has incompatible type "str"; expected "int"
    #        (the first definition wins!)
    A('x')
    ```

## 检查被调用的函数是否返回值 [func-returns-value]

**Check that called function returns a value [func-returns-value]**

=== "中文"

    Mypy reports an error if you call a function with a ``None`` return type and don't ignore the return value, as this is usually (but not always) a programming error.

    In this example, the ``if f()`` check is always false since ``f`` returns ``None``:

    ```python
    def f() -> None:
        ...

    # OK: we don't do anything with the return value
    f()

    # Error: "f" does not return a value (it only ever returns None)  [func-returns-value]
    if f():
        print("not false")
    ```

=== "英文"

    Mypy reports an error if you call a function with a ``None`` return type and don't ignore the return value, as this is usually (but not always) a programming error.

    In this example, the ``if f()`` check is always false since ``f`` returns ``None``:

    ```python
    def f() -> None:
        ...

    # OK: we don't do anything with the return value
    f()

    # Error: "f" does not return a value (it only ever returns None)  [func-returns-value]
    if f():
        print("not false")
    ```

## 检查抽象类的实例化 [abstract]

**Check instantiation of abstract classes [abstract]**

=== "中文"

    Mypy generates an error if you try to instantiate an abstract base class (ABC). An abstract base class is a class with at least one abstract method or attribute. (See also [abc](https://docs.python.org/3/library/abc.html#module-abc) module documentation)

    Sometimes a class is made accidentally abstract, often due to an unimplemented abstract method. In a case like this you need to provide an implementation for the method to make the class concrete (non-abstract).

    Example:

    ```python
    from abc import ABCMeta, abstractmethod

    class Persistent(metaclass=ABCMeta):
        @abstractmethod
        def save(self) -> None: ...

    class Thing(Persistent):
        def __init__(self) -> None:
            ...

        ...  # No "save" method

    # Error: Cannot instantiate abstract class "Thing" with abstract attribute "save"  [abstract]
    t = Thing()
    ```

=== "英文"

    Mypy generates an error if you try to instantiate an abstract base class (ABC). An abstract base class is a class with at least one abstract method or attribute. (See also [abc](https://docs.python.org/3/library/abc.html#module-abc) module documentation)

    Sometimes a class is made accidentally abstract, often due to an unimplemented abstract method. In a case like this you need to provide an implementation for the method to make the class concrete (non-abstract).

    Example:

    ```python
    from abc import ABCMeta, abstractmethod

    class Persistent(metaclass=ABCMeta):
        @abstractmethod
        def save(self) -> None: ...

    class Thing(Persistent):
        def __init__(self) -> None:
            ...

        ...  # No "save" method

    # Error: Cannot instantiate abstract class "Thing" with abstract attribute "save"  [abstract]
    t = Thing()
    ```

## 安全处理抽象类型对象类型 [type-abstract]

**Safe handling of abstract type object types [type-abstract]**

=== "中文"

    Mypy always allows instantiating (calling) type objects typed as ``Type[t]``, even if it is not known that ``t`` is non-abstract, since it is a common pattern to create functions that act as object factories (custom constructors). Therefore, to prevent issues described in the above section, when an abstract type object is passed where ``Type[t]`` is expected, mypy will give an error. Example:

    ```python
    from abc import ABCMeta, abstractmethod
    from typing import List, Type, TypeVar

    class Config(metaclass=ABCMeta):
        @abstractmethod
        def get_value(self, attr: str) -> str: ...

    T = TypeVar("T")
    def make_many(typ: Type[T], n: int) -> List[T]:
        return [typ() for _ in range(n)]  # This will raise if typ is abstract

    # Error: Only concrete class can be given where "Type[Config]" is expected [type-abstract]
    make_many(Config, 5)
    ```

=== "英文"

    Mypy always allows instantiating (calling) type objects typed as ``Type[t]``, even if it is not known that ``t`` is non-abstract, since it is a common pattern to create functions that act as object factories (custom constructors). Therefore, to prevent issues described in the above section, when an abstract type object is passed where ``Type[t]`` is expected, mypy will give an error. Example:

    ```python
    from abc import ABCMeta, abstractmethod
    from typing import List, Type, TypeVar

    class Config(metaclass=ABCMeta):
        @abstractmethod
        def get_value(self, attr: str) -> str: ...

    T = TypeVar("T")
    def make_many(typ: Type[T], n: int) -> List[T]:
        return [typ() for _ in range(n)]  # This will raise if typ is abstract

    # Error: Only concrete class can be given where "Type[Config]" is expected [type-abstract]
    make_many(Config, 5)
    ```

## 检查通过 super 调用抽象方法是否有效 [safe-super]

**Check that call to an abstract method via super is valid [safe-super]**

=== "中文"

    Abstract methods often don't have any default implementation, i.e. their bodies are just empty. Calling such methods in subclasses via ``super()`` will cause runtime errors, so mypy prevents you from doing so:

    ```python

    from abc import abstractmethod
    class Base:
        @abstractmethod
        def foo(self) -> int: ...
    class Sub(Base):
        def foo(self) -> int:
            return super().foo() + 1  # error: Call to abstract method "foo" of "Base" with
                                        # trivial body via super() is unsafe  [safe-super]
    Sub().foo()  # This will crash at runtime.
    ```

    Mypy considers the following as trivial bodies: a ``pass`` statement, a literal ellipsis ``...``, a docstring, and a ``raise NotImplementedError`` statement.

=== "英文"

    Abstract methods often don't have any default implementation, i.e. their bodies are just empty. Calling such methods in subclasses via ``super()`` will cause runtime errors, so mypy prevents you from doing so:

    ```python

    from abc import abstractmethod
    class Base:
        @abstractmethod
        def foo(self) -> int: ...
    class Sub(Base):
        def foo(self) -> int:
            return super().foo() + 1  # error: Call to abstract method "foo" of "Base" with
                                        # trivial body via super() is unsafe  [safe-super]
    Sub().foo()  # This will crash at runtime.
    ```

    Mypy considers the following as trivial bodies: a ``pass`` statement, a literal ellipsis ``...``, a docstring, and a ``raise NotImplementedError`` statement.

## 检查 NewType 的目标 [valid-newtype]

**Check the target of NewType [valid-newtype]**

=== "中文"

    The target of a [NewType](https://docs.python.org/3/library/typing.html#typing.NewType) definition must be a class type. It can't be a union type, ``Any``, or various other special types.

    You can also get this error if the target has been imported from a module whose source mypy cannot find, since any such definitions are treated by mypy as values with ``Any`` types. Example:

    ```python
    from typing import NewType

    # The source for "acme" is not available for mypy
    from acme import Entity  # type: ignore

    # Error: Argument 2 to NewType(...) must be subclassable (got "Any")  [valid-newtype]
    UserEntity = NewType('UserEntity', Entity)
    ```

    To work around the issue, you can either give mypy access to the sources for ``acme`` or create a stub file for the module.  See :ref:`ignore-missing-imports` for more information.

=== "英文"

    The target of a [NewType](https://docs.python.org/3/library/typing.html#typing.NewType) definition must be a class type. It can't be a union type, ``Any``, or various other special types.

    You can also get this error if the target has been imported from a module whose source mypy cannot find, since any such definitions are treated by mypy as values with ``Any`` types. Example:

    ```python
    from typing import NewType

    # The source for "acme" is not available for mypy
    from acme import Entity  # type: ignore

    # Error: Argument 2 to NewType(...) must be subclassable (got "Any")  [valid-newtype]
    UserEntity = NewType('UserEntity', Entity)
    ```

    To work around the issue, you can either give mypy access to the sources for ``acme`` or create a stub file for the module.  See :ref:`ignore-missing-imports` for more information.

## 检查 `__exit__` 的返回类型 [exit-return]

**Check the return type of \_\_exit\_\_ [exit-return]**

=== "中文"

    If mypy can determine that [\_\_exit\_\_](https://docs.python.org/3/reference/datamodel.html#object.__exit__) always returns ``False``, mypy checks that the return type is *not* ``bool``.  The boolean value of the return type affects which lines mypy thinks are reachable after a ``with`` statement, since any [\_\_exit\_\_](https://docs.python.org/3/reference/datamodel.html#object.__exit__) method that can return ``True`` may swallow exceptions. An imprecise return type can result in mysterious errors reported near ``with`` statements.

    To fix this, use either ``typing.Literal[False]`` or ``None`` as the return type. Returning ``None`` is equivalent to returning ``False`` in this context, since both are treated as false values.

    Example:

    ```python
    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> bool:  # Error
            print('exit')
            return False
    ```

    This produces the following output from mypy:

    ```text
    example.py:3: error: "bool" is invalid as return type for "__exit__" that always returns False
    example.py:3: note: Use "typing_extensions.Literal[False]" as the return type or change it to
        "None"
    example.py:3: note: If return type of "__exit__" implies that it may return True, the context
        manager may swallow exceptions
    ```

    You can use ``Literal[False]`` to fix the error:

    ```python
    from typing import Literal

    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> Literal[False]:  # OK
            print('exit')
            return False
    ```

    You can also use ``None``:

    ```python
    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> None:  # Also OK
            print('exit')
    ```

=== "英文"

    If mypy can determine that [\_\_exit\_\_](https://docs.python.org/3/reference/datamodel.html#object.__exit__) always returns ``False``, mypy checks that the return type is *not* ``bool``.  The boolean value of the return type affects which lines mypy thinks are reachable after a ``with`` statement, since any [\_\_exit\_\_](https://docs.python.org/3/reference/datamodel.html#object.__exit__) method that can return ``True`` may swallow exceptions. An imprecise return type can result in mysterious errors reported near ``with`` statements.

    To fix this, use either ``typing.Literal[False]`` or ``None`` as the return type. Returning ``None`` is equivalent to returning ``False`` in this context, since both are treated as false values.

    Example:

    ```python
    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> bool:  # Error
            print('exit')
            return False
    ```

    This produces the following output from mypy:

    ```text
    example.py:3: error: "bool" is invalid as return type for "__exit__" that always returns False
    example.py:3: note: Use "typing_extensions.Literal[False]" as the return type or change it to
        "None"
    example.py:3: note: If return type of "__exit__" implies that it may return True, the context
        manager may swallow exceptions
    ```

    You can use ``Literal[False]`` to fix the error:

    ```python
    from typing import Literal

    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> Literal[False]:  # OK
            print('exit')
            return False
    ```

    You can also use ``None``:

    ```python
    class MyContext:
        ...
        def __exit__(self, exc, value, tb) -> None:  # Also OK
            print('exit')
    ```

## 检查命名是否一致 [name-match]

**Check that naming is consistent [name-match]**

=== "中文"

    The definition of a named tuple or a TypedDict must be named consistently when using the call-based syntax. Example:

    ```python
    from typing import NamedTuple

    # Error: First argument to namedtuple() should be "Point2D", not "Point"
    Point2D = NamedTuple("Point", [("x", int), ("y", int)])
    ```

=== "英文"

    The definition of a named tuple or a TypedDict must be named consistently when using the call-based syntax. Example:

    ```python
    from typing import NamedTuple

    # Error: First argument to namedtuple() should be "Point2D", not "Point"
    Point2D = NamedTuple("Point", [("x", int), ("y", int)])
    ```

## 检查文字是否在预期位置使用 [literal-required]

**Check that literal is used where expected [literal-required]**

=== "中文"

    There are some places where only a (string) literal value is expected for the purposes of static type checking, for example a ``TypedDict`` key, or a ``__match_args__`` item. Providing a ``str``-valued variable in such contexts will result in an error. Note that in many cases you can also use ``Final`` or ``Literal`` variables. Example:

    ```python
    from typing import Final, Literal, TypedDict

    class Point(TypedDict):
        x: int
        y: int

    def test(p: Point) -> None:
        X: Final = "x"
        p[X]  # OK

        Y: Literal["y"] = "y"
        p[Y]  # OK

        key = "x"  # Inferred type of key is `str`
        # Error: TypedDict key must be a string literal;
        #   expected one of ("x", "y")  [literal-required]
        p[key]
    ```

=== "英文"

    There are some places where only a (string) literal value is expected for the purposes of static type checking, for example a ``TypedDict`` key, or a ``__match_args__`` item. Providing a ``str``-valued variable in such contexts will result in an error. Note that in many cases you can also use ``Final`` or ``Literal`` variables. Example:

    ```python
    from typing import Final, Literal, TypedDict

    class Point(TypedDict):
        x: int
        y: int

    def test(p: Point) -> None:
        X: Final = "x"
        p[X]  # OK

        Y: Literal["y"] = "y"
        p[Y]  # OK

        key = "x"  # Inferred type of key is `str`
        # Error: TypedDict key must be a string literal;
        #   expected one of ("x", "y")  [literal-required]
        p[key]
    ```

## 检查重载函数是否有实现 [no-overload-impl]

**Check that overloaded functions have an implementation [no-overload-impl]**

=== "中文"

    Overloaded functions outside of stub files must be followed by a non overloaded implementation.

    ```python
    from typing import overload

    @overload
    def func(value: int) -> int:
        ...

    @overload
    def func(value: str) -> str:
        ...

    # presence of required function below is checked
    def func(value):
        pass  # actual implementation
    ```

=== "英文"

    Overloaded functions outside of stub files must be followed by a non overloaded implementation.

    ```python
    from typing import overload

    @overload
    def func(value: int) -> int:
        ...

    @overload
    def func(value: str) -> str:
        ...

    # presence of required function below is checked
    def func(value):
        pass  # actual implementation
    ```

## 检查协程返回值是否被使用 [unused-coroutine]

**Check that coroutine return value is used [unused-coroutine]**

=== "中文"

    Mypy ensures that return values of async def functions are not ignored, as this is usually a programming error, as the coroutine won't be executed at the call site.

    ```python
    async def f() -> None:
        ...

    async def g() -> None:
        f()  # Error: missing await
        await f()  # OK
    ```

    You can work around this error by assigning the result to a temporary, otherwise unused variable:

    ```python

        _ = f()  # No error
    ```

=== "英文"

    Mypy ensures that return values of async def functions are not ignored, as this is usually a programming error, as the coroutine won't be executed at the call site.

    ```python
    async def f() -> None:
        ...

    async def g() -> None:
        f()  # Error: missing await
        await f()  # OK
    ```

    You can work around this error by assigning the result to a temporary, otherwise unused variable:

    ```python

        _ = f()  # No error
    ```

## 警告顶层 await 表达式 [top-level-await]

**Warn about top level await expressions [top-level-await]**

=== "中文"

    This error code is separate from the general ``[syntax]`` errors, because in some environments (e.g. IPython) a top level ``await`` is allowed. In such environments a user may want to use ``--disable-error-code=top-level-await``, that allows to still have errors for other improper uses of ``await``, for example:

    ```python
    async def f() -> None:
        ...

    top = await f()  # Error: "await" outside function  [top-level-await]
    ```

=== "英文"

    This error code is separate from the general ``[syntax]`` errors, because in some environments (e.g. IPython) a top level ``await`` is allowed. In such environments a user may want to use ``--disable-error-code=top-level-await``, that allows to still have errors for other improper uses of ``await``, for example:

    ```python
    async def f() -> None:
        ...

    top = await f()  # Error: "await" outside function  [top-level-await]
    ```

## 警告在协程外部使用 await 表达式 [await-not-async]

**Warn about await expressions used outside of coroutines [await-not-async]**

=== "中文"

    ``await`` must be used inside a coroutine.

    ```python
    async def f() -> None:
        ...

    def g() -> None:
        await f()  # Error: "await" outside coroutine ("async def")  [await-not-async]
    ```

=== "英文"

    ``await`` must be used inside a coroutine.

    ```python
    async def f() -> None:
        ...

    def g() -> None:
        await f()  # Error: "await" outside coroutine ("async def")  [await-not-async]
    ```

## 检查 assert_type 中的类型 [assert-type]

**Check types in assert_type [assert-type]**

=== "中文"

    The inferred type for an expression passed to ``assert_type`` must match the provided type.

    ```python
    from typing_extensions import assert_type

    assert_type([1], list[int])  # OK

    assert_type([1], list[str])  # Error
    ```

=== "英文"

    The inferred type for an expression passed to ``assert_type`` must match the provided type.

    ```python
    from typing_extensions import assert_type

    assert_type([1], list[int])  # OK

    assert_type([1], list[str])  # Error
    ```

## 检查函数是否未在布尔上下文中使用 [truthy-function]

**Check that function isn't used in boolean context [truthy-function]**

=== "中文"

    Functions will always evaluate to true in boolean contexts.

    ```python
    def f():
        ...

    if f:  # Error: Function "Callable[[], Any]" could always be true in boolean context  [truthy-function]
        pass
    ```

=== "英文"

    Functions will always evaluate to true in boolean contexts.

    ```python
    def f():
        ...

    if f:  # Error: Function "Callable[[], Any]" could always be true in boolean context  [truthy-function]
        pass
    ```

## 检查字符串格式化/插值是否类型安全 [str-format]

**Check that string formatting/interpolation is type-safe [str-format]**

=== "中文"

    Mypy will check that f-strings, ``str.format()`` calls, and ``%`` interpolations are valid (when corresponding template is a literal string). This includes checking number and types of replacements, for example:

    ```python
    # Error: Cannot find replacement for positional format specifier 1 [str-format]
    "{} and {}".format("spam")
    "{} and {}".format("spam", "eggs")  # OK
    # Error: Not all arguments converted during string formatting [str-format]
    "{} and {}".format("spam", "eggs", "cheese")

    # Error: Incompatible types in string interpolation
    # (expression has type "float", placeholder has type "int") [str-format]
    "{:d}".format(3.14)
    ```

=== "英文"

    Mypy will check that f-strings, ``str.format()`` calls, and ``%`` interpolations are valid (when corresponding template is a literal string). This includes checking number and types of replacements, for example:

    ```python
    # Error: Cannot find replacement for positional format specifier 1 [str-format]
    "{} and {}".format("spam")
    "{} and {}".format("spam", "eggs")  # OK
    # Error: Not all arguments converted during string formatting [str-format]
    "{} and {}".format("spam", "eggs", "cheese")

    # Error: Incompatible types in string interpolation
    # (expression has type "float", placeholder has type "int") [str-format]
    "{:d}".format(3.14)
    ```

## 检查隐式字节强制转换 [str-bytes-safe]

**Check for implicit bytes coercions [str-bytes-safe]**

=== "中文"

    Warn about cases where a bytes object may be converted to a string in an unexpected manner.

    ```python
    b = b"abc"

    # Error: If x = b'abc' then f"{x}" or "{}".format(x) produces "b'abc'", not "abc".
    # If this is desired behavior, use f"{x!r}" or "{!r}".format(x).
    # Otherwise, decode the bytes [str-bytes-safe]
    print(f"The alphabet starts with {b}")

    # Okay
    print(f"The alphabet starts with {b!r}")  # The alphabet starts with b'abc'
    print(f"The alphabet starts with {b.decode('utf-8')}")  # The alphabet starts with abc
    ```

=== "英文"

    Warn about cases where a bytes object may be converted to a string in an unexpected manner.

    ```python
    b = b"abc"

    # Error: If x = b'abc' then f"{x}" or "{}".format(x) produces "b'abc'", not "abc".
    # If this is desired behavior, use f"{x!r}" or "{!r}".format(x).
    # Otherwise, decode the bytes [str-bytes-safe]
    print(f"The alphabet starts with {b}")

    # Okay
    print(f"The alphabet starts with {b!r}")  # The alphabet starts with b'abc'
    print(f"The alphabet starts with {b.decode('utf-8')}")  # The alphabet starts with abc
    ```

## 检查重载函数是否没有重叠 [overload-overlap]

**Check that overloaded functions don't overlap [overload-overlap]**

=== "中文"

    Warn if multiple ``@overload`` variants overlap in potentially unsafe ways. This guards against the following situation:

    ```python
    from typing import overload

    class A: ...
    class B(A): ...

    @overload
    def foo(x: B) -> int: ...  # Error: Overloaded function signatures 1 and 2 overlap with incompatible return types  [overload-overlap]
    @overload
    def foo(x: A) -> str: ...
    def foo(x): ...

    def takes_a(a: A) -> str:
        return foo(a)

    a: A = B()
    value = takes_a(a)
    # mypy will think that value is a str, but it could actually be an int
    reveal_type(value) # Revealed type is "builtins.str"
    ```


    Note that in cases where you ignore this error, mypy will usually still infer the types you expect.

    See [overloading](https://mypy.readthedocs.io/en/stable/more_types.html#function-overloading) for more explanation.

=== "英文"

    Warn if multiple ``@overload`` variants overlap in potentially unsafe ways. This guards against the following situation:

    ```python
    from typing import overload

    class A: ...
    class B(A): ...

    @overload
    def foo(x: B) -> int: ...  # Error: Overloaded function signatures 1 and 2 overlap with incompatible return types  [overload-overlap]
    @overload
    def foo(x: A) -> str: ...
    def foo(x): ...

    def takes_a(a: A) -> str:
        return foo(a)

    a: A = B()
    value = takes_a(a)
    # mypy will think that value is a str, but it could actually be an int
    reveal_type(value) # Revealed type is "builtins.str"
    ```


    Note that in cases where you ignore this error, mypy will usually still infer the types you expect.

    See [overloading](https://mypy.readthedocs.io/en/stable/more_types.html#function-overloading) for more explanation.

## 通知未检查函数中的注解 [annotation-unchecked]

**Notify about an annotation in an unchecked function [annotation-unchecked]**

=== "中文"

    Sometimes a user may accidentally omit an annotation for a function, and mypy will not check the body of this function (unless one uses [--check-untyped-defs](../mypy_conf/command_line.md#check-untyped-defs) or [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs) ). To avoid such situations go unnoticed, mypy will show a note, if there are any type annotations in an unchecked function:

    ```python
    def test_assignment():  # "-> None" return annotation is missing
        # Note: By default the bodies of untyped functions are not checked,
        # consider using --check-untyped-defs [annotation-unchecked]
        x: int = "no way"
    ```

    Note that mypy will still exit with return code ``0``, since such behaviour is specified by [PEP 484](https://peps.python.org/pep-0484/).

=== "英文"

    Sometimes a user may accidentally omit an annotation for a function, and mypy will not check the body of this function (unless one uses [--check-untyped-defs](../mypy_conf/command_line.md#check-untyped-defs) or [--disallow-untyped-defs](../mypy_conf/command_line.md#disallow-untyped-defs) ). To avoid such situations go unnoticed, mypy will show a note, if there are any type annotations in an unchecked function:

    ```python
    def test_assignment():  # "-> None" return annotation is missing
        # Note: By default the bodies of untyped functions are not checked,
        # consider using --check-untyped-defs [annotation-unchecked]
        x: int = "no way"
    ```

    Note that mypy will still exit with return code ``0``, since such behaviour is specified by [PEP 484](https://peps.python.org/pep-0484/).

## 装饰器在属性前不被支持 [prop-decorator]

**Decorator preceding property not supported [prop-decorator]**

=== "中文"

    Mypy does not yet support analysis of decorators that precede the property decorator. If the decorator does not preserve the declared type of the property, mypy will not infer the correct type for the declaration. If the decorator cannot be moved after the ``@property`` decorator, then you must use a type ignore comment:

    ```python
    class MyClass:
        @special  # type: ignore[prop-decorator]
        @property
        def magic(self) -> str:
            return "xyzzy"
    ```

    !!! note 

        For backward compatibility, this error code is a subcode of the generic ``[misc]`` code.

=== "英文"

    Mypy does not yet support analysis of decorators that precede the property decorator. If the decorator does not preserve the declared type of the property, mypy will not infer the correct type for the declaration. If the decorator cannot be moved after the ``@property`` decorator, then you must use a type ignore comment:

    ```python
    class MyClass:
        @special  # type: ignore[prop-decorator]
        @property
        def magic(self) -> str:
            return "xyzzy"
    ```

    !!! note 

        For backward compatibility, this error code is a subcode of the generic ``[misc]`` code.

## 报告语法错误 [syntax]

**Report syntax errors [syntax]**

=== "中文"

    If the code being checked is not syntactically valid, mypy issues a syntax error. Most, but not all, syntax errors are *blocking errors*: they can't be ignored with a ``# type: ignore`` comment.

=== "英文"

    If the code being checked is not syntactically valid, mypy issues a syntax error. Most, but not all, syntax errors are *blocking errors*: they can't be ignored with a ``# type: ignore`` comment.

## 杂项检查 [misc]

**Miscellaneous checks [misc]**

=== "中文"

    Mypy performs numerous other, less commonly failing checks that don't have specific error codes. These use the ``misc`` error code. Other than being used for multiple unrelated errors, the ``misc`` error code is not special. For example, you can ignore all errors in this category by using ``# type: ignore[misc]`` comment. Since these errors are not expected to be common, it's unlikely that you'll see two *different* errors with the ``misc`` code on a single line -- though this can certainly happen once in a while.

    !!! note 

        Future mypy versions will likely add new error codes for some errors that currently use the ``misc`` error code.

=== "英文"

    Mypy performs numerous other, less commonly failing checks that don't have specific error codes. These use the ``misc`` error code. Other than being used for multiple unrelated errors, the ``misc`` error code is not special. For example, you can ignore all errors in this category by using ``# type: ignore[misc]`` comment. Since these errors are not expected to be common, it's unlikely that you'll see two *different* errors with the ``misc`` code on a single line -- though this can certainly happen once in a while.

    !!! note 

        Future mypy versions will likely add new error codes for some errors that currently use the ``misc`` error code.

[sorted()]: https://docs.python.org/3/library/functions.html#sorted