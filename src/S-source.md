# <a name="S-source"></a>SF: 源文件

区分声明（用作接口）和定义（用作实现）。
用头文件来表达接口并强调逻辑结构。

源文件规则概览：

* [SF.1: 如果你的项目还未采用别的约定的话，应当为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`](S-source.md#Rs-file-suffix)
* [SF.2: 头文件不能含有对象定义或非内联的函数定义](S-source.md#Rs-inline)
* [SF.3: 对在多个源文件中使用的任何声明，都应使用头文件](S-source.md#Rs-declaration-header)
* [SF.4: 在文件中的其他所有声明之前包含头文件](S-source.md#Rs-include-order)
* [SF.5: `.cpp` 文件必须包含定义了它的接口的一个或多个头文件](S-source.md#Rs-consistency)
* [SF.6: `using namespace` 指令，（仅）可以为迁移而使用，可以为基础程序库使用（比如 `std`），或者在局部作用域中使用](S-source.md#Rs-using)
* [SF.7: 请勿在头文件中的全局作用域使用 `using namespace` 指令](S-source.md#Rs-using-directive)
* [SF.8: 为所有的头文件使用 `#include` 防卫宏](S-source.md#Rs-guards)
* [SF.9: 避免源文件的循环依赖](S-source.md#Rs-cycles)
* [SF.10: 避免依赖于隐含地 `#include` 进来的名字](S-source.md#Rs-implicit)
* [SF.11: 头文件应当是自包含的](S-source.md#Rs-contained)
* [SF.12: 对相对于包含文件的文件优先采用引号形式的 `#include`，其他情况下采用角括号形式](S-source.md#Rs-incform)

* [SF.20: 用 `namespace` 表示逻辑结构](S-source.md#Rs-namespace)
* [SF.21: 请勿在头文件中使用无名（匿名）命名空间](S-source.md#Rs-unnamed)
* [SF.22: 为所有的内部/不导出的实体使用无名（匿名）命名空间](S-source.md#Rs-unnamed2)

### <a name="Rs-file-suffix"></a>SF.1: 如果你的项目还未采用别的约定的话，应当为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`

参见 [NL.27](S-naming.md#Rl-file-suffix)

### <a name="Rs-inline"></a>SF.2: 头文件不能含有对象定义或非内联的函数定义

##### 理由

对受制于唯一定义规则的实体的包含将导致连接错误。

##### 示例

```cpp
// file.h:
namespace Foo {
    int x = 7;
    int xx() { return x+x; }
}

// file1.cpp:
#include <file.h>
// ... 更多代码 ...

 // file2.cpp:
#include <file.h>
// ... 更多代码 ...
```

当连接 `file1.cpp` 和 `file2.cpp` 时将出现两个连接器错误。

**其他形式**: 头文件必须仅包含：

* `#include` 其他的头文件（可能包括包含防卫宏）
* 模板
* 类定义
* 函数声明
* `extern` 声明
* `inline` 函数定义
* `constexpr` 定义
* `const` 定义
* `using` 别名定义
* ???

##### 强制实施

根据以上白名单来检查。

### <a name="Rs-declaration-header"></a>SF.3: 对在多个源文件中使用的任何声明，都应使用头文件

##### 理由

可维护性。可读性。

##### 示例，不好

```cpp
// bar.cpp:
void bar() { cout << "bar\n"; }

// foo.cpp:
extern void bar();
void foo() { bar(); }
```

`bar` 的维护者在需要改变 `bar` 的类型时，无法找到其全部声明。
`bar` 的使用者不知道他所使用的接口是否完整和正确。顶多会从连接器获得一些（延迟的）错误消息。

##### 强制实施

* 对并未放入 `.h` 而在其他源文件中的实体声明进行标记。

### <a name="Rs-include-order"></a>SF.4: 在文件中的其他所有声明之前包含头文件

##### 理由

最小化上下文的依赖并增加可读性。

##### 示例

```cpp
#include <vector>
#include <algorithm>
#include <string>

// ... 我自己的代码 ...
```

##### 示例，不好

```cpp
#include <vector>

// ... 我自己的代码 ...

#include <algorithm>
#include <string>
```

##### 注解

这对于 `.h` 和 `.cpp` 文件都同样适用。

##### 注解

有一种论点是通过在打算保护的代码的*后面*再 `#include` 头文件，以此将代码同头文件中的声明式和宏等之间进行隔离
（如上面例子中标为“不好”之处）。
不过，

* 这只能对单个文件（在单个层次上）工作：如果采用这个技巧的头文件被别的头文件所包含，这个威胁就会再次出现。
* 命名空间（一个“实现命名空间”）可以针对许多的上下文依赖进行保护。
* 完全的保护和灵活性需要模块。

**参见**：

* [工作草案，C++ 的模块扩展](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4592.pdf)
* [模块，组件化及其迁移](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0141r0.pdf)

##### 强制实施

容易。

### <a name="Rs-consistency"></a>SF.5: `.cpp` 文件必须包含定义了它的接口的一个或多个头文件

##### 理由

这使得编译器可以提早进行一致性检查。

##### 示例，不好

```cpp
// foo.h:
void foo(int);
int bar(long);
int foobar(int);

// foo.cpp:
void foo(int) { /* ... */ }
int bar(double) { /* ... */ }
double foobar(int);
```

这个错误直到调用了 `bar` 或 `foobar` 的程序的连接时才会被发现。

##### 示例

```cpp
// foo.h:
void foo(int);
int bar(long);
int foobar(int);

// foo.cpp:
#include "foo.h"

void foo(int) { /* ... */ }
int bar(double) { /* ... */ }
double foobar(int);   // 错误: 错误的返回类型
```

`foobar` 的返回类型错误在编译 `foo.cpp` 时立即就被发现了。
对 `bar` 的参数类型错误在连接时之前无法被发现，因为可能会有重载发生，但系统性地使用 `.h` 文件能够增加时其被程序员更早发现的可能性。

##### 强制实施

???

### <a name="Rs-using"></a>SF.6: `using namespace` 指令，（仅）可以为迁移而使用，可以为基础程序库使用（比如 `std`），或者在局部作用域中使用

##### 理由

`using namespace` 可能造成名字冲突，因而应当节制使用。
然而，将用户代码中的每个命名空间中的名字都进行限定并不总是能够做到（比如在转换过程中）
而且有时候命名空间非常基础，并且在代码库中广为使用，坚持进行限定将使其既啰嗦又分散注意力。

##### 示例

```cpp
#include <string>
#include <vector>
#include <iostream>
#include <memory>
#include <algorithm>

using namespace std;

// ...
```

显然地，大量使用了标准库，而且貌似没使用别的程序库，因此要求每一处带有使用 `std::`
会使人分散注意力。

##### 示例

使用 `using namespace std;` 导致程序员可能面临与标准库中的名字造成名字冲突

```cpp
#include <cmath>
using namespace std;

int g(int x)
{
    int sqrt = 7;
    // ...
    return sqrt(x); // 错误
}
```

不过，不大可能导致并非错误的名字解析，
假定使用 `using namespace std` 的人们都了解 `std` 以及这种风险。

##### 注解

`.cpp` 文件也是一种形式的局部作用域。
包含一条 `using namespace X` 的 N 行的 `.cpp` 文件中发生名字冲突的机会，
和包含一条 `using namespace X` 的 N 行的函数，
以及每个都包含一条 `using namespace X` 的总行数为 N 行的 M 个函数，没有多少差别。

##### 注解

[请勿在头文件全局作用域中使用 `using namespace`](S-source.md#Rs-using-directive)。

### <a name="Rs-using-directive"></a>SF.7: 请勿在头文件中的全局作用域使用 `using namespace`

##### 理由

这样做使 `#include` 一方无法有效地进行区分并使用其他方式。这还可能使所 `#include` 的头文件之间出现顺序依赖，它们以不同次序包含时可能具有不同的意义。

##### 示例

```cpp
// bad.h
#include <iostream>
using namespace std; // bad

// user.cpp
#include "bad.h"

bool copy(/*... some parameters ...*/);    // some function that happens to be named copy

int main()
{
    copy(/*...*/);    // now overloads local ::copy and std::copy, could be ambiguous
}
```

##### 注解

一个例外是 `using namespace std::literals;`。若要在头文件中使用
字符串字面量，则必须如此，而且根据[规则](http://eel.is/c++draft/over.literal)——用户必须以
`operator""_x` 来命名他们自己的 UDL——它们并不会与标准库相冲突。

##### 强制实施

标记头文件的全局作用域中的 `using namespace`。

### <a name="Rs-guards"></a>SF.8: 为所有的头文件使用 `#include` 防卫宏

##### 理由

避免文件被多次 `#include`。

为避免包含防卫宏的冲突，不要仅使用文件名来命名防卫宏。
确保还要包含一个关键词和好的区分词，比如头文件所属的程序库
或组件的名字。

##### 示例

```cpp
// file foobar.h:
#ifndef LIBRARY_FOOBAR_H
#define LIBRARY_FOOBAR_H
// ... 声明 ...
#endif // LIBRARY_FOOBAR_H
```

##### 强制实施

标记没有 `#include` 防卫的 `.h` 文件。

##### 注解

一些实现提供了如 `#pragma once` 这样的厂商扩展作为包含防卫宏的替代。
这并非标准且不可移植。它向程序中注入了宿主机器的文件系统的语义，
而且把你锁定到某个特定厂商。
我们的建议是编写 ISO C++：参见[规则 P.2](S-philosophy.md#Rp-Cplusplus)。

### <a name="Rs-cycles"></a>SF.9: 避免源文件的循环依赖

##### 理由

循环会使理解变得困难，并拖慢编译速度。
它们还会使（当其可用时）向利用语言支持的模块进行转换工作变得复杂。


##### 注解

要消除循环依赖；请勿仅仅用 `#include` 防卫宏来试图打破它们。

##### 示例，不好

```cpp
// file1.h:
#include "file2.h"

// file2.h:
#include "file3.h"

// file3.h:
#include "file1.h"
```

##### 强制实施

对任何循环依赖进行标记。


### <a name="Rs-implicit"></a>SF.10: 避免依赖于隐含地 `#include` 进来的名字

##### 理由

避免意外。
避免当 `#include` 的头文件改变时改变一条 `#include`。
避免意外地变为依赖于所包含的头文件中的实现细节和逻辑上独立的实体。

##### 示例，不好

```cpp
#include <iostream>
using namespace std;

void use()
{
    string s;
    cin >> s;               // 好
    getline(cin, s);        // 错误：getline() 未定义
    if (s == "surprise") {  // 错误：== 未定义
        // ...
    }
}
```

`<iostream>` 暴露了 `std::string` 的定义（“为什么？”是一个有趣的问题），
但其并不必然是通过传递包含整个 `<string>` 头文件而做到这一点的，
这带来了常见的新手问题“为什么 `getline(cin,s);` 不成？”，
甚至偶尔出现的“`string` 无法用 `==` 来比较”。

其解决方案是明确地 `#include <string>`：

##### 示例，好

```cpp
#include <iostream>
#include <string>
using namespace std;

void use()
{
    string s;
    cin >> s;               // 好
    getline(cin, s);        // 好
    if (s == "surprise") {  // 好
        // ...
    }
}
```

##### 注解

一些头文件正是用于从一些头文件中合并一组声明。
例如：

```cpp
// basic_std_lib.h:

#include <string>
#include <map>
#include <iostream>
#include <random>
#include <vector>
```

用户只用一条 `#include` 就可以获得整组的声明了：

```cpp
#include "basic_std_lib.h"
```

本条反对隐式包含的规则并不防止这种特意的聚集包含。

##### 强制实施

强制实施将需要一些有关头文件中哪些是“导出”给用户所用而哪些是用于实现的知识。
在我们能用到模块之前没有真正的好方案。

### <a name="Rs-contained"></a>SF.11: 头文件应当是自包含的

##### 理由

易用性，头文件应当易于使用，且单独包含即可正常工作。
头文件应当对其所提供的功能进行封装。
避免让头文件的使用方来管理它的依赖项。

##### 示例

```cpp
#include "helpers.h"
// helpers.h 依赖于 std::string 并已包含了 <string>
```

##### 注解

不遵守这条规则将导致头文件的使用方难于诊断所出现的错误。

##### 注解

头文件应当包含其所有依赖项。请小心使用相对路径，各 C++ 实现对于它们的含义是有分歧的。

##### 强制实施

以一项测试来验证头文件自身可通过编译，或者一个仅包含了该头文件的 cpp 文件可通过编译。

### <a name="Rs-incform"></a>SF.12: 对相对于包含文件的文件优先采用引号形式的 `#include`，其他情况下采用角括号形式

##### 理由

[标准](http://eel.is/c++draft/cpp.include) 向编译器提供了对于实现
使用角括号（`<>`）或引号（`""`）语法的 `#include` 的两种形式的灵活性。
各厂商利用了这点并采用了不同的搜索算法和指定包含路径的方法。

无论如何，指导方针是使用引号形式来（从同一个组件或项目中）包含那些存在于某个相对于含有这条 `#include` 语句的文件的相对路径中的文件，其他情况尽可能使用角括号形式。这样做鼓励明确表现出文件与包含它的文件之间的局部性，或当需要某种不同的搜索算法的情形。这样一眼就可以很容易明白头文件是从某个局部相对文件包含的，还是某个标准库头文件或别的搜索路径（比如另一个程序库或一组常用包含路径）中的某个头文件。

##### 示例

```cpp
// foo.cpp:
#include <string>                // 来自标准程序库，要求使用 <> 形式
#include <some_library/common.h> // 从另一个程序库中包含的，并非出于局部相对位置的文件；使用 <> 形式
#include "foo.h"                 // 处于同一项目中局部相对于 foo.cpp 的文件，使用 "" 形式
#include "foo_utils/utils.h"     // 处于同一项目中局部相对于 foo.cpp 的文件，使用 "" 形式
#include <component_b/bar.h>     // 通过搜索路径定位到的处于同一项目中的文件，使用 <> 形式
```

##### 注解

不遵守这条可能会导致很难诊断的错误：由于包含时指定的错误的范围而选择了错误的文件。例如，通常 `#include ""` 的搜索算法首先搜索存在于某个局部相对路径中的文件，因此使用这种形式来指代某个并非位于局部相对路径的文件，就一位置一旦在局部相对路径中出现了一个这样的文件（比如进行包含的文件被移动到了别的位置），它就会在原来所包含的文件之前被找到，并使包含文件集合以一种预料之外的方式被改变。

程序库作者们应当把它们的头文件放到一个文件夹中，然后让其客户使用相对路径来包含这些文件：`#include <some_library/common.h>`。

##### 强制实施

检测按 `""` 引用的头文件是否可以按 `<>` 引用。

### <a name="Rs-namespace"></a>SF.20: 用 `namespace` 表示逻辑结构

##### 理由

 ???

##### 示例

```cpp
???
```

##### 强制实施

???

### <a name="Rs-unnamed"></a>SF.21: 请勿在头文件中使用无名（匿名）命名空间

##### 理由

在头文件中使用无名命名空间差不多都是一个 BUG。

##### 示例

```cpp
// 文件 foo.h:
namespace
{
    const double x = 1.234;  // 不好

    double foo(double y)     // 不好
    {
        return y + x;
    }
}

namespace Foo
{
    const double x = 1.234; // 好

    inline double foo(double y)        // 好
    {
        return y + x;
    }
}
```

##### 强制实施

* 对头文件中所使用的任何匿名命名空间进行标记。

### <a name="Rs-unnamed2"></a>SF.22: 为所有的内部/不导出的实体使用无名（匿名）命名空间

##### 理由

外部实体无法依赖于嵌套的无名命名空间中的实体。
考虑将实现源文件中的所有定义都放入无名命名空间中，除非它定义的是一个“外部/导出”实体。

##### 示例；不好

```cpp
static int f();
int g();
static bool h();
int k();
```

##### 示例；好

```cpp
namespace {
    int f();
    bool h();
}
int g();
int k();
```

##### 示例

API 类及其成员不能放在无名命名空间中；而在实现源文件中所定义的任何的“辅助”类或函数则应当放在无名命名空间作用域之中。

```cpp
???
```

##### 强制实施

* ???

