# <a name="S-naming"></a>NL: 命名和代码布局建议

维持一致的命名和代码布局是很有用的。
即便不为其他原因，也可以减少“我的代码风格比你的好”这类的纷争。
然而，人们使用许多许多的不同代码风格，并狂热地坚持它们（的优缺点）。
而且，大多数的现实项目都包含来自于许多来源的代码，因而通常不可能把所有的代码都标准化为某个单一的代码风格。
经过许多的用户请求给予指导后，我们给出一组规则，当你没有更好的选择时可以使用它们，但真正的目标在于一致性，而不是任何一组特定的规则。
IDE 和工具可以提供辅助（当然也可能造成妨碍）。

命名和代码布局规则：

* [NL.1: 不要在代码注释中说明可以由代码来清晰表达的东西](S-naming.md#Rl-comments)
* [NL.2: 在代码注释中说明意图](S-naming.md#Rl-comments-intent)
* [NL.3: 保持代码注释简明干脆](S-naming.md#Rl-comments-crisp)
* [NL.4: 保持一种统一的缩进风格](S-naming.md#Rl-indent)
* [NL.5: 避免在名字中编码类型信息](S-naming.md#Rl-name-type)
* [NL.7: 使名字的长度大约正比于其作用域的长度](S-naming.md#Rl-name-length)
* [NL.8: 使用一种统一的命名风格](S-naming.md#Rl-name)
* [NL.9: 将 `ALL_CAPS`（全大写）仅用于宏的名字](S-naming.md#Rl-all-caps)
* [NL.10: 优先采用 `underscore_style`（下划线风格）的名字](S-naming.md#Rl-camel)
* [NL.11: 使字面量可阅读](S-naming.md#Rl-literals)
* [NL.15: 节制地使用空格](S-naming.md#Rl-space)
* [NL.16: 使用一种常规的类成员声明次序](S-naming.md#Rl-order)
* [NL.17: 使用从 K&R 衍生出的代码布局](S-naming.md#Rl-knr)
* [NL.18: 使用 C++ 风格的声明符布局](S-naming.md#Rl-ptr)
* [NL.19: 避免使用容易误读的名字](S-naming.md#Rl-misread)
* [NL.20: 不要把两个语句放在同一行中](S-naming.md#Rl-stmt)
* [NL.21: 每个声明式（仅）声明一个名字](S-naming.md#Rl-dcl)
* [NL.25: 请勿将 `void` 用作参数类型](S-naming.md#Rl-void)
* [NL.26: 采用符合惯例的 `const` 写法](S-naming.md#Rl-const)
* [NL.27: 为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`](S-naming.md#Rl-file-suffix)

这些问题的大部分都是审美问题，程序员都有很强的个人倾向。
IDE 也都会提供某些默认方案和一组替代方案。
这些规则是作为缺省建议的，如果没有别的理由，请采用它们。

我们收到一些意见称命名和代码布局非常个人化和任意性，我们不应该试图为之“立法”。
我们并不是在“立法”（参见前一个段落）。
不过，我们也收到了大量的针对某些命名和代码布局约定的请求，要求当没有外来限制的时候应当采用它们。

更专门和详细的规则更加易于强制实施。

这些规则恐怕会和 [PPP Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf) 中的建议有很强的相似性，
它是为支持 Stroustrup 的 [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html) 而编制的。

### <a name="Rl-comments"></a>NL.1: 不要在代码注释中说明可以由代码来清晰表达的东西

##### 理由

编译器不会读注释。
注释没有代码那么精确。
注释不会像代码那样进行一致地更新。

##### 示例，不好

```cpp
auto x = m * v1 + vv;   // 将 m 乘以 v1 并将其结果加上 vv
```

##### 强制实施

构建一个 AI 程序来解释口语英文文字，看看它所说的是否可以用 C++ 来更好地表达。

### <a name="Rl-comments-intent"></a>NL.2: 在代码注释中说明意图

##### 理由

代码表示的是做了什么，而不是想要做成什么。通常来说意图比实现能够更清晰简明地进行说明。

##### 示例

```cpp
void stable_sort(Sortable& c)
    // 对 c 根据由 < 决定的顺序进行排序，保持相等元素（由 == 定义）的
    // 原始相对顺序
{
    // ... 相当多的不平常的代码行 ...
}
```

##### 注解

如果代码注释和代码有冲突，则它们都可能是错的。

### <a name="Rl-comments-crisp"></a>NL.3: 保持代码注释简明干脆

##### 理由

冗长啰嗦会拖慢理解速度，而且到处散布在代码文件里也会让代码难于阅读。

##### 注解

使用明白易懂的英文。
也许我可以流利使用丹麦语，但大多数程序员不行；我的代码的维护者也不行。
避免使用网络用语，注意你的文法，标点，以及大小写。
目标是专业性，而不是“够酷”。

##### 强制实施

不可能。

### <a name="Rl-indent"></a>NL.4: 保持一种统一的缩进风格

##### 理由

可读性。避免“微妙的错误”。

##### 示例，不好

```cpp
int i;
for (i = 0; i < max; ++i); // 可能出现的 BUG
if (i == j)
    return i;
```

##### 注解

总是把 `if (...)`，`for (...)`，以及 `while (...)` 之后的语句进行缩进是一个好主意：

```cpp
if (i < 0) error("negative argument");

if (i < 0)
    error("negative argument");
```

##### 强制实施

使用一种工具。

### <a name="Rl-name-type"></a>NL.5: 避免在名字中编码类型信息

##### 原理

当名字反映类型而不是功能时，它将变得难于为提供其功能而改变其所使用的类型。
而且，当改变变量的类型时，使用它的代码也得修改。
最小化无意进行的转换。

##### 示例，不好

```cpp
void print_int(int i);
void print_string(const char*);

print_int(1);          // 重复，人工进行类型匹配
print_string("xyzzy"); // 重复，人工进行类型匹配
```

##### 示例，好

```cpp
void print(int i);
void print(string_view);    // 对任意字符串式的序列都能工作

print(1);              // 简洁，自动类型匹配
print("xyzzy");        // 简洁，自动类型匹配
```

##### 注解

带有类型编码的名字要么啰嗦要么难懂。

```cpp
printS  // 打印一个 std::string
prints  // 打印一个 C 风格字符串
printi  // 打印一个 int
```

在无类型语言中曾经采用过像匈牙利记法这样的技巧来在名字中编码类型，但在像 C++ 这样的强静态类型语言中，这通常是不必要而且实际上是有害的，因为这些标注会过时（这些累赘和注释类似，而且和它们一样会烂掉），而且它们干扰了语言的恰当用法（应当代之以使用相同的名字和重载决议）。

##### 注解

一些代码风格会使用非常一般性的（而不是特定于类型的）前缀来代表变量的一般用法。

```cpp
auto p = new User();
auto p = make_unique<User>();
// 注："p" 并非是说“User 类型的原始指针”，
//     而只是一般性的“这是一次间接访问”

auto cntHits = calc_total_of_hits(/*...*/);
// 注："cnt" 并非用于编码某个类型，
//     而只是一般性的“这是某种东西的一个计数”
```

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

##### 注解

一些代码风格会对成员和局部变量，以及全局变量之间进行区分。

```cpp
struct S {
    int m_;
    S(int m) : m_{abs(m)} { }
};
```

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

##### 注解

像 C++ 这样，一些代码风格对类型和非类型之间进行区分。
例如，对类型名字首字母大写，而函数和变量名字则不这样做。

```cpp
typename<typename T>
class HashTable {   // 将 string 映射为 T
    // ...
};

HashTable<int> index;
```

这样做是没有害处的，且并不属于本条指导方针，因为其并未编码类型信息。

### <a name="Rl-name-length"></a>NL.7: 使名字的长度大约正比于其作用域的长度

**原理**: 作用域越大，搞混的机会和意外的名字冲突的机会就越大。

##### 示例

```cpp
double sqrt(double x);   // 返回 x 的平方根；x 必须是非负数

int length(const char* p);  // 返回零结尾的 C 风格字符串的字符数量

int length_of_string(const char zero_terminated_array_of_char[])    // 不好: 啰嗦

int g;      // 不好: 全局变量具有密秘的名字

int open;   // 不好: 全局变量使用短小且常用的名字
```

为指针使用 `p`，以及为浮点变量使用 `x` 是符合惯例的，在受限的作用域中不会造成混乱。

##### 强制实施

???

### <a name="Rl-name"></a>NL.8: 使用一种统一的命名风格

**原理**: 命名和命名风格的一致性会提高可读性。

##### 注解

命名风格有好多，当你使用多个程序库时，你无法遵循所有它们不同的命名约定。
应当选用一种“自有风格”，但保持“导入”的程序为其原有风格不变。

##### 示例

ISO 标准仅使用小写字母和数字，并用下划线进行词的连接。

* `int`
* `vector`
* `my_map`

避免使用双下划线 `__`。

##### 示例

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf)：
采用 ISO 标准，但在自己的类型和概念上采用大写字母：

* `int`
* `vector`
* `My_map`

##### 示例

CamelCase：多词标识符的每个词首字母大写：

* `int`
* `vector`
* `MyMap`
* `myMap`

一些命名约定会将首字母大写，而另一些不会。

##### 注解

应当试图在对缩略词的使用和标识符长度上保持一致风格。

```cpp
int mtbf {12};
int mean_time_between_failures {12}; // 你自己决定
```

##### 强制实施

除使用具有不同命名约定的程序库之外应当是可能做到的。

### <a name="Rl-all-caps"></a>NL.9: 将 `ALL_CAPS`（全大写）仅用于宏的名字

##### 理由

避免在宏和遵循作用域和类型规则的名字之间造成混乱。

##### 示例

```cpp
void f()
{
    const int SIZE{1000};  // 不好，应代之以 'size'
    int v[SIZE];
}
```

##### 注解

这条规则适用于非宏的符号常量：

```cpp
enum bad { BAD, WORSE, HORRIBLE }; // 不好
```

##### 强制实施

* 对带有小写字母的宏进行标记
* 对 `ALL_CAPS` 非宏名字进行标记

### <a name="Rl-camel"></a>NL.10: 优先采用 `underscore_style`（下划线风格）的名字

##### 理由

用下划线来分隔名字的各部分就是 C 和 C++ 的原始风格，并被用于 C++ 标准库中。

##### 注解

这条规则仅作为当你有选择权时的缺省方案。
通常你是没有什么选择权的，而只能遵循某个已经设立的风格以维持[一致性](S-naming.md#Rl-name)。
对一致性的需要优先于个人喜好。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

[Stroustrup](http://www.stroustrup.com/Programming/PPP-style.pdf)：
采用 ISO 标准，但在自己的类型和概念上采用大写字母：

* `int`
* `vector`
* `My_map`

##### 强制实施

不可能。

### <a name="Rl-literals"></a>NL.11: 使字面量可阅读

##### 理由

可读性。

##### 示例

用数字分隔符来避免长串的数字

```cpp
auto c = 299'792'458; // m/s2
auto q2 = 0b0000'1111'0000'0000;
auto ss_number = 123'456'7890;
```

##### 示例

需要清晰性时使用字面量后缀

```cpp
auto hello = "Hello!"s; // std::string
auto world = "world";   // C 风格字符串
auto interval = 100ms;  // 使用 <chrono>
```

##### 注解

不能在代码中到处当做[“魔法常量”](S-expr.md#Res-magic)一样乱用字面量，
但当定义它们时使它们更可读仍是个好主意。
在较长的整数串中很容易出现拼写错误。

##### 强制实施

标记长数字串。麻烦的是“长”的定义；也许应当是 7。

### <a name="Rl-space"></a>NL.15: 节制地使用空格

##### 理由

太多的空格会让文本更大更分散。

##### 示例，不好

```cpp
#include < map >

int main(int argc, char * argv [ ])
{
    // ...
}
```

##### 示例

```cpp
#include <map>

int main(int argc, char* argv[])
{
    // ...
}
```

##### 注解

一些 IDE 有其自己的看法，并会添加分散的空格。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 注解

我们将恰当放置的空白评价为能够明显有助于可读性。但请勿过度。

### <a name="Rl-order"></a>NL.16: 使用一种常规的类成员声明次序

##### 理由

一种常规的成员次序会提高可读性。

以如下次序声明类

* 类型：类，枚举，别名（`using`）
* 构造函数，赋值，析构函数
* 函数
* 数据

采用先是 `public`，然后是 `protected`，之后是 `private` 的次序。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

```cpp
class X {
public:
    // 接口
protected:
    // 供派生类实现使用的不带检查的函数
private:
    // 实现细节
};
```

##### 示例

有时候，成员的默认顺序，与将公开接口从实现细节中分离出来的需求之间有冲突。
这种情况下，私有类型和函数可以和私有数据放在一起。

```cpp
class X {
public:
    // 接口
protected:
    // 供派生类实现使用的不带检查的函数
private:
    // 实现细节（类型，函数和数据）
};
```

##### 示例，不好

避免让具有某一种访问（如 `public`）的多个声明块被具有不同访问（如 `private`）的其他声明块分隔开。

```cpp
class X {
public:
    void f();
public:
    int g();
    // ...
};
```

用宏来声明成员组的做法通常会导致违反所有的次序规则。
不过，宏的使用掩盖了其所表达的东西。

##### 强制实施

对背离上述建议次序的代码进行标记。将会有大量的老代码不符合这条规则。

### <a name="Rl-knr"></a>NL.17: 使用从 K&R 衍生出的代码布局

##### 理由

这正是 C 和 C++ 的原始代码布局。它很好地保持了纵向空间。它对不同语言构造（如函数和类）进行了很好的区分。

##### 注解

在 C++ 的语境中，这种风格通常被称为“Stroustrup”。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 示例

```cpp
struct Cable {
    int x;
    // ...
};

double foo(int x)
{
    if (0 < x) {
        // ...
    }

    switch (x) {
    case 0:
        // ...
        break;
    case amazing:
        // ...
        break;
    default:
        // ...
        break;
    }

    if (0 < x)
        ++x;

    if (x < 0)
        something();
    else
        something_else();

    return some_value;
}
```

注意 `if` 和 `(` 之间有一个空格

##### 注解

每个语句，`if` 的分支，以及 `for` 的代码体都使用单独的代码行。

##### 注解

`class` 和 `struct` 的 `{` *并不*在单独的代码行上，但函数的 `{` 在单独的代码行上。

##### 注解

对你自定义的类型的名字进行首字母大写，以将其与标准库类型相区分。

##### 注解

不要对函数名大写。

##### 强制实施

如果想要强制实施的话，请使用某个 IDE 进行格式化。

### <a name="Rl-ptr"></a>NL.18: 使用 C++ 风格的声明符布局

##### 理由

C 风格的布局强调其在表达式中的用法和文法，而 C++ 风格强调的是类型。
对表达式用法的说辞并不适用于引用。

##### 示例

```cpp
T& operator[](size_t);   // OK
T &operator[](size_t);   // 奇怪
T & operator[](size_t);   // 不确定
```

##### 注解

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 强制实施

由于历史原因而不可能。


### <a name="Rl-misread"></a>NL.19: 避免使用容易误读的名字

##### 理由

可读性。
并非每个人都有能将字符轻易区分开的屏幕和打印机。
我们很容易搞混拼写相似和略微拼错的单词。

##### 示例

```cpp
int oO01lL = 6; // 不好

int splunk = 7;
int splonk = 8; // 不好：splunk 和 splonk 很容易搞混
```

##### 强制实施

???

### <a name="Rl-stmt"></a>NL.20: 不要把两个语句放在同一行中

##### 理由

可读性。
当一行里有多个语句时，相当容易忽视某个语句。

##### 示例

```cpp
int x = 7; char* p = 29;    // 请勿如此
int x = 7; f(x);  ++x;      // 请勿如此
```

##### 强制实施

容易。

### <a name="Rl-dcl"></a>NL.21: 每个声明式（仅）声明一个名字

##### 理由

可读性。
最小化声明符语法造成的混乱。

##### 注解

相关细节，参见 [ES.10](S-expr.md#Res-name-one)、


### <a name="Rl-void"></a>NL.25: 请勿将 `void` 用作参数类型

##### 理由

这很啰嗦，而且仅在考虑 C 兼容性是才有必要。

##### 示例

```cpp
void f(void);   // 不好

void g();       // 好多了
```

##### 注解

即便是 Dennis Ritchie 自己都认为 `void f(void)` 很讨厌。
你可以反驳称，在 C 中当函数原型很少见时禁止这样的代码：

```cpp
int f();
f(1, 2, "weird but valid C89");   // 希望 f() 被定义为 int f(a, b, c) char* c; { /* ... */ }
```

可能造成很大的问题，但这并不适于 21 世纪和 C++。

### <a name="Rl-const"></a>NL.26: 采用符合惯例的 `const` 写法

##### 理由

更多程序员更加熟悉惯例写法。
大型代码库中的一致性。

##### 示例

```cpp
const int x = 7;    // OK
int const y = 9;    // 不好

const int *const p = nullptr;   // OK, 指向常量 int 的常量指针
int const *const p = nullptr;   // 不好，指向常量 int 的常量指针
```

##### 注解

我们知道你可能会说“不好”的例子比标有“OK”的更符合逻辑，
但它们会让更多人搞混，尤其是那些依赖于采用了远为常用，符合惯例的 OK 风格的教学材料的新手们。

一如往常，请记住这些命名和代码布局规则的目标在于一致性，而审美则会有广泛的变化。

这个推荐适用于[当你没有约束条件或者没有更好的想法时](S-naming.md#S-naming)的情况。
经过很多要求给予指导后，添加这个规则。

##### 强制实施

标记用作类型的后缀的 `const`。

### <a name="Rl-file-suffix"></a>NL.27: 为代码文件使用后缀 `.cpp`，而对接口文件使用后缀 `.h`

##### 理由

这是一条历史悠久的约定。
不过一致性更加重要，因此如果你的项目用了别的约定的话，应当遵守它。

##### 注解

这项约定反应了一种常见使用模式：
头文件更容易和 C 语言共享使用，并可以作为 C++ 和 C 编译，它们通常用 `.h` 后缀，
并且对于有意要和 C 共用的头文件来说，让所有头文件都使用 `.h` 而不是别的扩展名要更加容易。
另一方面，实现文件则很少会和 C 共用，通常应当和 `.c` 文件相区别，
因此一般最好为所有的 C++ 实现文件用别的扩展名（如 `.cpp`）来命名。

特定的名字 `.h` 和 `.cpp` 并不是必要的（只是作为缺省建议），其他的名字也被广泛采用。
例子包括 `.hh`，`.C`，和 `.cxx` 等。请以类似方式使用这些名字。
本文档中我们把 `.h` 和 `.cpp` 作为头文件和实现文件的简便提法，
虽然实际上的扩展名可能是不同的。

也许你的 IDE（如果你使用的话）对后缀有较强的倾向。

##### 示例

```cpp
// foo.h:
extern int a;   // 声明
extern void foo();

// foo.cpp:
int a;   // 定义
void foo() { ++a; }
```

`foo.h` 提供了 `foo.cpp` 的接口。最好避免全局变量。

##### 示例，不好

```cpp
// foo.h:
int a;   // 定义
void foo() { ++a; }
```

一个程序中两次 `#include <foo.h>` 将导致因为对唯一定义规则的两次违反而出现一个连接错误。

##### 强制实施

* 对不符合约定的文件名进行标记。
* 检查 `.h` 和 `.cpp`（或等价文件）遵循下列各规则。
