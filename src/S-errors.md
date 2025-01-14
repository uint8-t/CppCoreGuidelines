# <a name="S-errors"></a>E: 错误处理

错误处理涉及：

* 检测某个错误
* 将有关错误的信息传递给某个处理代码
* 维持程序的某个有效状态
* 避免资源泄漏

不可能做到从所有的错误中恢复。如果从某个错误进行恢复是不可能的话，以明确定义的方式迅速“脱离”则是很重要的。错误处理的策略必须简单，否则就会成为更糟糕错误的来源。未经测试和很少被执行的错误处理代码自身也是许多 BUG 的来源。

以下规则是设计用以帮助避免几种错误的：

* 类型违规（比如对 `union` 和强制转换的误用）
* 资源泄漏（包括内存泄漏）
* 边界错误
* 生存期错误（比如在对象被 `delete` 后访问它）
* 复杂性错误（可能由于过于复杂的想法表达而导致的逻辑错误）
* 接口错误（比如通过接口传递了预期外的值）

错误处理规则概览：

* [E.1: 在设计中尽早开发错误处理策略](S-errors.md#Re-design)
* [E.2: 通过抛出异常来明示函数无法完成其所赋予的任务](S-errors.md#Re-throw)
* [E.3: 仅使用异常来进行错误处理](S-errors.md#Re-errors)
* [E.4: 围绕不变式来设计错误处理策略](S-errors.md#Re-design-invariants)
* [E.5: 让构造函数建立不变式，若其无法做到则抛出异常](S-errors.md#Re-invariant)
* [E.6: 使用 RAII 来避免泄漏](S-errors.md#Re-raii)
* [E.7: 明示前条件](S-errors.md#Re-precondition)
* [E.8: 明示后条件](S-errors.md#Re-postcondition)

* [E.12: 当函数不可能或不能接受以 `throw` 来退出时，使用 `noexcept`](S-errors.md#Re-noexcept)
* [E.13: 不要在作为某个对象的直接所有者时抛出异常](S-errors.md#Re-never-throw)
* [E.14: 应当使用为目的所设计的自定义类型（而不是内建类型）作为异常](S-errors.md#Re-exception-types)
* [E.15: 按值抛出并按引用捕获类型层次中的异常](S-errors.md#Re-exception-ref)
* [E.16: 析构函数，回收函数，`swap`，以及异常类型的复制/移动构造决不能失败](S-errors.md#Re-never-fail)
* [E.17: 不要试图在每个函数中捕获每个异常](S-errors.md#Re-not-always)
* [E.18: 最小化对 `try`/`catch` 的显式使用](S-errors.md#Re-catch)
* [E.19: 当没有合适的资源包装时，使用 `final_action` 对象来表达清理动作](S-errors.md#Re-finally)

* [E.25: 当不能抛出异常时，模拟 RAII 来进行资源管理](S-errors.md#Re-no-throw-raii)
* [E.26: 当不能抛出异常时，考虑采取快速失败](S-errors.md#Re-no-throw-crash)
* [E.27: 当不能抛出异常时，系统化地使用错误代码](S-errors.md#Re-no-throw-codes)
* [E.28: 避免基于全局状态（比如 `errno`）的错误处理](S-errors.md#Re-no-throw)

* [E.30: 请勿使用异常说明](S-errors.md#Re-specifications)
* [E.31: 恰当地对 `catch` 子句排序](S-errors.md#Re_catch)

### <a name="Re-design"></a>E.1: 在设计中尽早开发错误处理策略

##### 理由

在一个系统中改造翻新一种一致且完整的处理错误和资源泄漏的策略是很难的。

### <a name="Re-throw"></a>E.2: 通过抛出异常来明示函数无法完成其所赋予的任务

##### 理由

让错误处理有系统性，强健，而且避免重复。

##### 示例

```cpp
struct Foo {
    vector<Thing> v;
    File_handle f;
    string s;
};

void use()
{
    Foo bar {{ "{{" }}Thing{1}, Thing{2}, Thing{monkey}}, {"my_file", "r"}, "Here we go!"};
    // ...
}
```

这里，`vector` 和 `string` 的构造函数可能无法为其元素分配足够的内存，`vector` 的构造函数可能无法复制其初始化式列表中的 `Thing`，而 `File_handle` 可能无法打开所需的文件。
这些情况中，它们都会抛出异常来让 `use()` 的调用者来处理。
如果 `use()` 可以处理这种构造 `bar` 的故障的话，它可以用 `try`/`catch` 来控制。
这些情况下，`Foo` 的构造函数都会在把控制传递给试图创建 `Foo` 的任何代码之前恰当地销毁以及构造的内存。
注意它并不存在可以包含错误代码的返回值。

`File_handle` 的构造函数可以这样定义：

```cpp
File_handle::File_handle(const string& name, const string& mode)
    : f{fopen(name.c_str(), mode.c_str())}
{
    if (!f)
        throw runtime_error{"File_handle: could not open " + name + " as " + mode"}
}
```

##### 注解

人们通常说异常应当用于表明意外的事件和故障。
不过这里面存在一点循环，“什么是意外的？”
例如：

* 无法满足的前条件
* 无法构造对象的构造函数（无法建立类的[不变式](S-class.md#Rc-struct)）
* 越界错误（比如 `v[v.size()] = 7`）
* 无法获得资源（比如网络未连接）

相较而言，终止某个普通的循环则不是意外的。
除非这个循环本应当是无限循环，否则其终止就是正常且符合预期的。

##### 注解

不要用 `throw` 仅仅作为从函数中返回值的另一种方式。

##### 例外

某些系统，比如在执行开始之前就需要保证以（通常很短的）常量最大时间来执行动作的硬实时系统，这样的系统只有当存在工具可以支持精确地预测从一次 `throw` 中恢复的最大时间时才能使用异常。

**参见**: [RAII](S-errors.md#Re-raii)

**参见**: [讨论](S-discussion.md#Sd-noexcept)

##### 注解

在你决定你无法负担或者不喜欢基于异常的错误处理之前，请看一看[替代方案](S-errors.md#Re-no-throw-raii)；
它们各自都有自己的复杂性和问题。
同样地，只要可能的话，就应该进行测量之后再发表有关效率的言论。

### <a name="Re-errors"></a>E.3: 仅使用异常来进行错误处理

##### 理由

以保持错误处理和“常规代码”互相分离。
C++ 实现都倾向于基于假定异常的稀有而进行优化。

##### 示例，请勿如此

```cpp
// 请勿如此: 异常并未用于错误处理
int find_index(vector<string>& vec, const string& x)
{
    try {
        for (gsl::index i = 0; i < vec.size(); ++i)
            if (vec[i] == x) throw i;  // 找到了 x
    }
    catch (int i) {
        return i;
    }
    return -1;   // 未找到
}
```

这种代码要比显然的替代方式更加复杂，而且极可能运行慢得多。
在 `vector` 中寻找一个值是没什么意外情况的。

##### 强制实施

可能应该是启发式措施。
查找从 `catch` 子句“漏掉”的异常值。

### <a name="Re-design-invariants"></a>E.4: 围绕不变式来设计错误处理策略

##### 理由

要使用一个对象，它必须处于某个（正式或非正式通过不变式所定义的）有效的状态，而要从错误中恢复，每个还未销毁的对象也必须处于有效的状态。

##### 注解

[不变式](S-class.md#Rc-struct)是对象的成员的逻辑条件，构造函数必须进行建立，且为公开的成员函数所假定。

##### 强制实施

???

### <a name="Re-invariant"></a>E.5: 让构造函数建立不变式，若其无法做到则抛出异常

##### 理由

遗留仍未建立不变式的对象将会带来麻烦。
不是任何成员函数都可以对其进行调用。

##### 示例

```cpp
class Vector {  // 非常简单的 double 向量
    // 当 elem != nullptr 时 elem 指向 sz 个 double
public:
    Vector() : elem{nullptr}, sz{0}{}
    Vector(int s) : elem{new double[s]}, sz{s} { /* 元素的初始化 */ }
    ~Vector() { delete [] elem; }
    double& operator[](int s) { return elem[s]; }
    // ...
private:
    owner<double*> elem;
    int sz;
};
```

类不变式——这里以代码注释说明——是由构造函数建立的。
当 `new` 无法分配所需的内存时将抛出异常。
各运算符，尤其是下标运算符，都是依赖于这个不变式的。

**参见**: [当构造函数无法构造有效对象时，应当抛出异常](S-class.md#Rc-throw)

##### 强制实施

对带有 `private` 状态但没有（公开，受保护或私有的）构造函数的类进行标记。

### <a name="Re-raii"></a>E.6: 使用 RAII 来避免泄漏

##### 理由

资源泄漏通常是不可接受的。
手工的资源释放很易出错。
RAII（Resource Acquisition Is Initialization，资源获取即初始化）是最简单，最系统化的避免泄漏方案。

##### 示例

```cpp
void f1(int i)   // 不好: 可能会泄漏
{
    int* p = new int[12];
    // ...
    if (i < 17) throw Bad{"in f()", i};
    // ...
}
```

我们可以在抛出异常前小心地释放资源：

```cpp
void f2(int i)   // 笨拙且易错: 显式的释放
{
    int* p = new int[12];
    // ...
    if (i < 17) {
        delete[] p;
        throw Bad{"in f()", i};
    }
    // ...
}
```

这样很啰嗦。在更大型的可能带有多个 `throw` 的代码中，显式的释放将变得重复且易错。

```cpp
void f3(int i)   // OK: 通过资源包装来进行资源管理（请见下文）
{
    auto p = make_unique<int[]>(12);
    // ...
    if (i < 17) throw Bad{"in f()", i};
    // ...
}
```

注意即使 `throw` 是在所调用的函数中暗中发生，这也能正常工作：

```cpp
void f4(int i)   // OK: 通过资源包装来进行资源管理（请见下文）
{
    auto p = make_unique<int[]>(12);
    // ...
    helper(i);   // 可能抛出异常
    // ...
}
```

除非你确实需要指针语义，否则还是应当使用局部的资源对象：

```cpp
void f5(int i)   // OK: 通过局部对象来进行资源管理
{
    vector<int> v(12);
    // ...
    helper(i);   // 可能抛出异常
    // ...
}
```

这即简单又安全，而且通常更加高效。

##### 注解

当没有合适的资源包装，且定义一个适当的 RAII 对象/包装由于某种原因不可行时，
万不得已，可以使用 [`final_action` 对象](S-errors.md#Re-finally)来表达清理动作。

##### 注解

但是当我们所编写的程序不能使用异常时应当怎么办呢？
首先应当质疑这项假设；到处都有许多反异常的错误认识。
据我们所知，只有少量正当理由：

* 我们所在的系统太小，支持异常将会吃掉我们的 2K 内存的大部分。
* 我们所在的是硬实时系统，而且我们没有工具能保证异常会在所需时间内处理掉。
* 我们所在的系统中有成吨的遗留代码以难于理解的方式大量地使用指针
  （尤其是没有可识别的所有权策略），因此异常可能会造成泄露。
* 我们的 C++ 异常机制的实现不合理地糟糕
  （很慢，很耗内存，对于动态链接库无法正确工作，等等）。
  请向你的实现的供应商提出意见；如果没有用户提出意见，就不会出现改进。
* 如果我们质疑经理的古老智慧的话会被炒鱿鱼。

以上原因中只有第一条才是基础问题，因此一旦可能的话，还是要用异常来实现 RAII，或者设计你的 RAII 对象永不失败。
当无法使用异常时，可以模拟 RAII。
就是说，系统化地在对象构造之后检查其有效性，并且仍然在析构函数中释放所有的资源。
一种策略是为每个资源包装添加一个 `valid()` 操作：

```cpp
void f()
{
    vector<string> vs(100);   // 非 std::vector: 添加了 valid()
    if (!vs.valid()) {
        // 处理错误或退出
    }

    ifstream fs("foo");   // 非 std::ifstream: 添加了 valid()
    if (!fs.valid()) {
        // 处理错误或退出
    }

    // ...
} // 析构函数如常进行清理
```

显然这样做增加了代码大小，不允许隐式的“异常”（`valid()` 检查）传播，而且 `valid()` 检查可能被忘掉。
优先采用异常。

**参见**: [`noexcept` 的用法](S-errors.md#Re-noexcept)

##### 强制实施

???

### <a name="Re-precondition"></a>E.7: 明示前条件

##### 理由

避免接口错误。

**参见**: [前条件规则](S-interfaces.md#Ri-pre)

### <a name="Re-postcondition"></a>E.8: 明示后条件

##### 理由

避免接口错误。

**参见**: [后条件规则](S-interfaces.md#Ri-post)

### <a name="Re-noexcept"></a>E.12: 当函数不可能或不能接受以 `throw` 来退出时，使用 `noexcept`

##### 理由

使错误处理系统化，强健，且高效。

##### 示例

```cpp
double compute(double d) noexcept
{
    return log(sqrt(d <= 0 ? 1 : d));
}
```

这里，我们已知 `compute` 不会抛出异常，因为它仅由不会抛出异常的操作所组成。
通过将 `compute` 声明为 `noexcept`，让编译器和人类阅读者获得信息，使其更容易理解和操作 `compute`。

##### 注解

许多标准库函数都是 `noexcept` 的，这包括所有从 C 标准库中“继承”来的标准库函数。

##### 示例

```cpp
vector<double> munge(const vector<double>& v) noexcept
{
    vector<double> v2(v.size());
    // ... 做一些事 ...
}
```

这里的 `noexcept` 表明我不希望或无法处理无法构造局部的 `vector` 对象的情形。
也就是说，我认为内存耗尽是一种严重的设计错误（类比于硬件故障），因此我希望当其发生时让程序崩溃。

##### 注解

请勿使用传统的[异常说明](S-errors.md#Re-specifications)。

##### 参见

[讨论](S-discussion.md#Sd-noexcept)。

### <a name="Re-never-throw"></a>E.13: 不要在作为某个对象的直接所有者时抛出异常

##### 理由

这可能导致一次泄漏。

##### 示例

```cpp
void leak(int x)   // 请勿如此: 可能泄漏
{
    auto p = new int{7};
    if (x < 0) throw Get_me_out_of_here{};  // 可能泄漏 *p
    // ...
    delete p;   // 可能不会执行到这里
}
```

避免这种问题的一种方法是坚持使用资源包装：

```cpp
void no_leak(int x)
{
    auto p = make_unique<int>(7);
    if (x < 0) throw Get_me_out_of_here{};  // 将按需删除 *p
    // ...
    // 无须 delete p
}
```

另一种（通常更好）的方案是使用一个局部变量来消除指针的显式使用：

```cpp
void no_leak_simplified(int x)
{
    vector<int> v(7);
    // ...
}
```

##### 注解

如果有需要清理的某个局部“东西”，但其并未表示为带有析构函数的对象，则这样的清理
也必须在 `throw` 之前完成。
有时候，[`finally()`](S-errors.md#Re-finally) 可以把这种不系统的清理变得更加可管理一些。

### <a name="Re-exception-types"></a>E.14: 应当使用为目的所设计的自定义类型（而不是内建类型）作为异常

##### 理由

自定义类型可以把有关某个错误的信息更好地传递给处理器。
这些信息可以编码到类型自身中，而类型则不大可能会和其他人的异常造成冲突。

##### 示例

```cpp
throw 7; // 不好

throw "something bad";  // 不好

throw std::exception(); // 不好 - 未提供信息
```

从 `std::exception` 派生，能够获得选择捕获特定异常或者通过 `std::exception` 进行通盘处理的灵活性：

```cpp
class MyException: public std::runtime_error
{
public:
    MyException(const string& msg) : std::runtime_error(msg) {}
    // ...
};

// ...

throw MyException("something bad");  // 好
```

异常可以不必派生于 `std::exception`：

```cpp
class MyCustomError final {};  // 并未派生于 std::exception

// ...

throw MyCustomError{};  // 好 - 处理器必须捕获这个类型（或 ...）
```

当检测位置没有可以添加的有用信息时，可以使用派生于 `exception`
的库类型作为通用类型：

```cpp
throw std::runtime_error{"someting bad"}; // 好

// ...

throw std::invalid_argument("i is not even"); // 好
```

也可以使用 `enum` 类：

```cpp
enum class alert {RED, YELLOW, GREEN};

throw alert::RED; // 好
```

##### 强制实施

识别针对内建类型和 `std::exception` 的 `throw`。

### <a name="Re-exception-ref"></a>E.15: 按值抛出并按引用捕获类型层次中的异常

##### 理由

按值（而非指针）抛出并按引用捕获，能避免进行复制，尤其是基类子对象的切片。

##### 示例，不好

```cpp
void f()
{
    try {
        // ...
        throw new widget{}; // 请勿如此：抛出值而不要抛出原始指针
        // ...
    }
    catch (base_class e) {  // 请勿如此: 可能造成切片
        // ...
    }
}
```

可以代之以引用：

```cpp
catch (base_class& e) { /* ... */ }
```

或者（通常更好的）`const` 引用：

```cpp
catch (const base_class& e) { /* ... */ }
```

大多数处理器并不会改动异常，一般情况下我们都会[建议使用 `const`](S-expr.md#Res-const)。

##### 注解

对于如一个 `enum` 值这样的小型值类型来说，按值捕获是合适的。

##### 注解

重新抛出已捕获的异常应当使用 `throw;` 而非 `throw e;`。使用 `throw e;` 将会抛出 `e` 的一个新副本（并于异常被 `catch (const std::exception& e)` 捕获时切片成静态类型 `std::exception`），而并非重新抛出原来的 `std::runtime_error` 类型的异常。（但请关注[请勿试图在每个函数中捕获所有的异常](S-errors.md#Re-not-always)，以及[尽可能减少 `try`/`catch` 的显式使用](S-errors.md#Re-catch)。)

##### 强制实施

* 对按值捕获具有虚函数的类型进行标记。
* 对抛出原始指针进行标记。

### <a name="Re-never-fail"></a>E.16: 析构函数，回收函数，`swap`，以及异常类型的复制/移动构造决不能失败

##### 理由

如果析构函数，`swap`，内存回收，或者尝试复制/移动异常对象时会失败，就是说如果它会通过异常而退出，或者根本不会实施其所需的动作，我们就将不知道应当如何编写可靠的程序。

##### 示例，请勿如此

```cpp
class Connection {
    // ...
public:
    ~Connection()   // 请勿如此: 非常糟糕的析构函数
    {
        if (cannot_disconnect()) throw I_give_up{information};
        // ...
    }
};
```

##### 注解

许多人都曾试图编写违反这条规则的可靠代码，比如当网络连接“拒绝关闭”的情形。
尽我们所知，没人曾找到一个做到这点的通用方案。
虽然偶尔对于非常特殊的例子，你可以通过设置某个状态以便进行将来的清理的方式绕过它。
比如说，我们可能将一个不打算关闭的 socket 放入一个“故障 socket”列表之中，
让其被某个定期的系统状态清理所检查处理。
我们见过的每个这种例子都是易错的，专门的，而且通常有 BUG。

##### 注解

标准库假定析构函数，回收函数（比如 `operator delete`），和 `swap` 都不会抛出异常。当它们这样做时，基本的标准库不变式将会被打破。

##### 注解

* 回收函数，包括 `operator delete`，必须为 `noexcept`。
* `swap` 函数必须为 `noexcept`。
* 大多数的析构函数都是缺省隐含为 `noexcept` 的。
* 而且，[应该使移动操作为 `noexcept`](S-class.md#Rc-move-noexcept)。
* 当编写用作异常类型的类型时，确保其复制构造函数不为 `noexcept`。一般来说我们没法机制化地强制这一点，因为我们并不了解一个类型是否有意作为一种异常类型。
* 尝试避免抛出复制构造函数不为 `noexcept` 的类型。一般来说我们没法机制化地强制这一点，因为即便 `throw std::string(...)` 也可能抛异常，虽然实际上并不会。

##### 强制实施

* 识别会 `throw` 的析构函数，回收操作，和 `swap`。
* 识别不为 `noexcept` 的这类操作。

**参见**: [讨论](S-discussion.md#Sd-never-fail)

### <a name="Re-not-always"></a>E.17: 不要试图在每个函数中捕获每个异常

##### 理由

如果函数无法对异常进行有意义的恢复动作，其捕获这个异常就导致复杂性和浪费。
要让异常传播直到遇到一个可以处理它的函数。
要用 [RAII](S-errors.md#Re-raii) 来处理栈回溯路径上的清理操作。

##### 示例，请勿如此

```cpp
void f()   // 不好
{
    try {
        // ...
    }
    catch (...) {
        // 不做任何事
        throw;   // 传播异常
    }
}
```

##### 强制实施

* 标记嵌套的 `try` 块。
* 对带有过高的 `try` 块/函数比率的源代码文件进行标记。 (??? 问题：定义“过高”)

### <a name="Re-catch"></a>E.18: 最小化对 `try`/`catch` 的显式使用

##### 理由

`try`/`catch` 很啰嗦，而且非平凡的使用是易错的。
`try`/`catch` 可以作为对非系统化和/或低级的资源管理或错误处理的一个信号。

##### 示例，不好

```cpp
void f(zstring s)
{
    Gadget* p;
    try {
        p = new Gadget(s);
        // ...
        delete p;
    }
    catch (Gadget_construction_failure) {
        delete p;
        throw;
    }
}
```

这段代码很混乱。
可能在 `try` 块中的裸指针上发生泄漏。
不是所有的异常都被处理了。
`delete` 一个构造失败的对象几乎肯定是一个错误。
更好的是：

```cpp
void f2(zstring s)
{
    Gadget g {s};
}
```

##### 替代方案

* 合适的资源包装以及 [RAII](S-errors.md#Re-raii)
* [`finally`](S-errors.md#Re-finally)

##### 强制实施

??? 很难，需要启发式方法

### <a name="Re-finally"></a>E.19: 当没有合适的资源包装时，使用 `final_action` 对象来表达清理动作

##### 理由

[GSL](#???) 提供的 `finally` 要比 `try`/`catch` 更不啰嗦且难于搞错。

##### 示例

```cpp
void f(int n)
{
    void* p = malloc(n);
    auto _ = gsl::finally([p] { free(p); });
    // ...
}
```

##### 注解

`finally` 没有 `try`/`catch` 那样混乱，但它仍然比较专门化。
优先采用[适当的资源管理对象](S-errors.md#Re-raii)。
万不得已考虑使用 `finally`。

##### 注解

相对于老式的 [`goto exit;` 技巧](S-errors.md#Re-no-throw-codes)来说，使用 `finally` 是处理并非系统化的资源管理中的清理工作的
更加系统化并且相当简洁的方案。

##### 强制实施

启发式措施：检测 `goto exit;`。

### <a name="Re-no-throw-raii"></a>E.25: 当不能抛出异常时，模拟 RAII 来进行资源管理

##### 理由

即便没有异常，[RAII](S-errors.md#Re-raii) 通常仍然是最佳且最系统化的处理资源的方式。

##### 注解

使用异常进行错误处理，是 C++ 中唯一完整且系统化的处理非局部错误的方式。
特别是，非侵入地对对象构造的失败进行报告需要使用异常。
以无法被忽略的方式报告错误需要使用异常。
如果无法使用异常，应当尽你所能模拟它们的使用。

大量对异常的惧怕是被误导的。
当用在并非充斥指针和复杂控制结构的代码中的违例情形时，
异常处理几乎总是（在时间和空间上）可以负担，且几乎总会导致更好的代码。
当然这假定存在一个优良的异常处理机制实现，而这并非在所有系统上都存在。
存在另一些情况并不适用于上述问题，但由于其他原因而无法使用异常。
一个例子是一些硬实时系统：必须在固定的时间之内完成操作并得到错误或正确的响应。
在没有适当的时间评估工具的条件下，很难对异常作出保证。
这样的系统（比如飞控系统）通常也会禁止使用动态（堆）内存。

因此，对于错误处理的主要指导方针还是“使用异常和 [RAII](S-errors.md#Re-raii)。”
本节所处理的情况是，要么你没有高效的异常实现，
或者要面对大量的老式代码
（比如说，大量的指针，不明确定义的所有权，以及大量的不系统化的基于错误代码检查的错误处理）
而且向其引入简单且系统化的错误处理的做法不可行。

在宣称不能使用异常或者抱怨它们成本过高之前，应当考虑一下使用[错误代码](S-errors.md#Re-no-throw-codes)的例子。
请考虑使用错误代码的成本和复杂性。
如果你担心性能的话，请进行测量。

##### 示例

假定你想要编写

```cpp
void func(zstring arg)
{
    Gadget g {arg};
    // ...
}
```

当这个 `g` 并未正确构造时，`func` 将以一个异常退出。
当无法抛出异常时，我们可以通过向 `Gadget` 添加 `valid()` 成员函数来模拟 RAII 风格的资源包装：

```cpp
error_indicator func(zstring arg)
{
    Gadget g {arg};
    if (!g.valid()) return gadget_construction_error;
    // ...
    return 0;   // 零代表“正常”
}
```

显然问题现在变成了调用者必须记得测试其返回值。考虑添加 `[[nodiscard]]` 以鼓励这样的做法。

**参见**: [讨论](#???)

##### 强制实施

（仅）对于这种想法的特定版本是可能的：比如检查资源包装的构造后进行系统化的 `valid()` 测试。

### <a name="Re-no-throw-crash"></a>E.26: 当不能抛出异常时，考虑采取快速失败

##### 理由

如果你无法做好错误恢复的话，至少你可以在发生更多后续的损害之前拜托出来。

**参见**：[模拟 RAII](S-errors.md#Re-no-throw-raii)

##### 注解

当你无法系统化地进行错误处理时，考虑以“程序崩溃”作为对任何无法局部处理的错误的回应。
就是说，如果你无法在检测到错误的函数的上下文中处理它，则调用 `abort()`，`quick_exit()`，
或者相似的某个将触发某种系统重启的函数。

在具有大量进程和/或大量计算机的系统中，你总要预计到并处理这些关键程序崩溃，
比如说源于硬件故障而引发。
这种情况下，“程序崩溃”只不过把错误处理留给了系统的下一个层次。

##### 示例

```cpp
void f(int n)
{
    // ...
    p = static_cast<X*>(malloc(n * sizeof(X)));
    if (!p) abort();     // 当内存耗尽时 abort
    // ...
}
```

大多数程序都无法得体地处理内存耗尽。这大略上等价于

```cpp
void f(int n)
{
    // ...
    p = new X[n];    // 当内存耗尽时抛出异常（默认情况会调用 terminate）
    // ...
}
```

通常，在退出之前将“崩溃”的原因记录日志是个好主意。

##### 强制实施

很难对付

### <a name="Re-no-throw-codes"></a>E.27: 当不能抛出异常时，系统化地使用错误代码

##### 理由

系统化地使用任何错误处理策略都能最小化忘记处理错误的机会。

**参见**：[模拟 RAII](S-errors.md#Re-no-throw-raii)

##### 注解

需要处理几个问题：

* 如何从函数向外传递错误指示？
* 如何在错误退出函数之前释放所有资源？
* 使用什么来作为错误指示？

通常，返回错误指示意味着返回两个值：其结果以及一个错误指示。
错误指示可以是对象的一部分，例如对象可以带有 `valid()` 指示，
也可以返回一对值。

##### 示例

```cpp
Gadget make_gadget(int n)
{
    // ...
}

void user()
{
    Gadget g = make_gadget(17);
    if (!g.valid()) {
            // 错误处理
    }
    // ...
}
```

这种方案符合[模拟 RAII 资源管理](S-errors.md#Re-no-throw-raii)。
`valid()` 函数可以返回一个 `error_indicator`（比如说 `error_indicator` 枚举的某个成员）。

##### 示例

要是我们无法或者不想改动 `Gadget` 类型呢？
这种情况下，我们只能返回一对值。
例如：

```cpp
std::pair<Gadget, error_indicator> make_gadget(int n)
{
    // ...
}

void user()
{
    auto r = make_gadget(17);
    if (!r.second) {
            // 错误处理
    }
    Gadget& g = r.first;
    // ...
}
```

可见，`std::pair` 是一种可能的返回类型。
某些人则更喜欢专门的类型。
例如：

```cpp
Gval make_gadget(int n)
{
    // ...
}

void user()
{
    auto r = make_gadget(17);
    if (!r.err) {
            // 错误处理
    }
    Gadget& g = r.val;
    // ...
}
```

倾向于专门返回类型的一种原因是为其成员提供命名，而不是使用多少有些隐秘的 `first` 和 `second`,
而且可以避免与 `std::pair` 的其他使用相混淆。

##### 示例

通常，在错误退出之前必须进行清理。
这样做是很混乱的：

```cpp
std::pair<int, error_indicator> user()
{
    Gadget g1 = make_gadget(17);
    if (!g1.valid()) {
        return {0, g1_error};
    }

    Gadget g2 = make_gadget(17);
    if (!g2.valid()) {
        cleanup(g1);
        return {0, g2_error};
    }

    // ...

    if (all_foobar(g1, g2)) {
        cleanup(g2);
        cleanup(g1);
        return {0, foobar_error};
    }

    // ...

    cleanup(g2);
    cleanup(g1);
    return {res, 0};
}
```

模拟 RAII 可能不那么简单，尤其是在带有多个资源和多种可能错误的函数之中。
一种较为常见的技巧是把清理都集中到函数末尾以避免重复（注意这里本不必为 `g2` 增加一层作用域，但是编译 `goto` 版本却需要它）：

```cpp
std::pair<int, error_indicator> user()
{
    error_indicator err = 0;
    int res = 0;

    Gadget g1 = make_gadget(17);
    if (!g1.valid()) {
        err = g1_error;
        goto g1_exit;
    }

    {
        Gadget g2 = make_gadget(31);
        if (!g2.valid()) {
            err = g2_error;
            goto g2_exit;
        }

        if (all_foobar(g1, g2)) {
            err = foobar_error;
            goto exit;
        }

        // ...

    g2_exit:
        if (g2.valid()) cleanup(g2);
    }

g1_exit:
    if (g1.valid()) cleanup(g1);
    return {res,err};
}
```

函数越大，这种技巧就越有吸引力。
`finally` 可以[略微缓解这个问题](S-errors.md#Re-finally)。
而且，程序变得越大，系统化地采用一中基于错误指示的错误处理策略就越加困难。

我们[优先采用基于异常的错误处理](S-errors.md#Re-throw)，并建议[保持函数短小](S-functions.md#Rf-single)。

**参见**: [讨论](#???)

**参见**: [返回多个值](S-functions.md#Rf-out-multi)

##### 强制实施

很难对付。

### <a name="Re-no-throw"></a>E.28: 避免基于全局状态（比如 `errno`）的错误处理

##### 理由

全局状态难于管理，且易于忘记检查。
你上次检查 `printf()` 的返回值是什么时候了？

**参见**：[模拟 RAII](S-errors.md#Re-no-throw-raii)

##### 示例，不好

```cpp
int last_err;

void f(int n)
{
    // ...
    p = static_cast<X*>(malloc(n * sizeof(X)));
    if (!p) last_err = -1;     // 当内存耗尽时发生的错误
    // ...
}
```

##### 注解

C 风格的错误处理就是基于全局变量 `errno` 的，因此基本上不可能完全避免这种风格。

##### 强制实施

很难对付。


### <a name="Re-specifications"></a>E.30: 请勿使用异常说明

##### 理由

异常说明使得错误处理变得脆弱，隐含一些运行时开销，并且已经从 C++ 标准中被删除了。

##### 示例

```cpp
int use(int arg)
    throw(X, Y)
{
    // ...
    auto x = f(arg);
    // ...
}
```

当 `f()` 抛出了不同于 `X` 和 `Y` 的异常时将会执行未预期异常处理器，其默认将终止程序。
这没什么问题，但假定我们检查过着并不会发生而 `f` 则被改写为抛出某个新异常 `Z`，
这样将导致程序崩溃，除非我们改写 `use()`（并重新测试所有东西）。
障碍在于 `f()` 可能在某个我们无法控制的程序库中，而对于新的异常 `use()`
没办法对其做任何事，或者对其完全不感兴趣。
我们可以改写 `use()` 使其传递 `Z` 出去，但这样的话 `use()` 的调用方可能也需要被改写。
如此事态将很快变得无法掌控。
或者，我们可以在 `use()` 中添加一个 `try`-`catch` 以将 `Z` 映射为某种可以接受的异常。
这种方法也会很快变得无法掌控。
注意，对异常集合的改动通常都发生在系统的最底层
（比如说，当改换了网络库或者某种中间件时），因此改变将沿着冗长的调用链“冒泡上浮”。
在大型代码库中，这将意味着直到最后一个使用方也被改写之前，没人可以更新某个库到新版本。
如果 `use()` 是某个库的一部分，则也许不可能对其进行更新，因为其改动可能影响到未知的客户代码。

而让异常继续传递直到其到达某个潜在可以处理它的函数的策略，已经在多年的实践中得到了证明。

##### 注解

静态强制检查异常说明并不会带来任何好处。
相关例子请参见 [Stroustrup94](S-unclassified.md#Stroustrup94)。

##### 注解

当不会抛出异常时，请使用 [`noexcept`](S-errors.md#Re-noexcept)。

##### 强制实施

标记每个异常说明。

### <a name="Re_catch"></a>E.31: 恰当地对 `catch` 子句排序

##### 理由

`catch` 子句是以其出现顺序依次求值的，而其中一个可能会隐藏掉另一个。

##### 示例，不好

```cpp
void f()
{
    // ...
    try {
            // ...
    }
    catch (Base& b) { /* ... */ }
    catch (Derived& d) { /* ... */ }
    catch (...) { /* ... */ }
    catch (std::exception& e) { /* ... */ }
}
```

若 `Derived` 派生自 `Base` 则 `Derived` 的处理器永远不会被执行。
“捕获任何东西”的处理器保证 `std::exception` 的处理器永远不会被执行。

##### 强制实施

标记出所有的“隐藏处理器”。

