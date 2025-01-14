# <a name="S-not"></a>NR: 伪规则和错误的看法

本部分包含一些在不少地方流行的规则和指导方针，但是我们慎重地建议不要采纳它们。
我们完全了解这些规则曾经在某些时间和场合是有意义的，而且我们自己也曾经采用过它们。
不过，在我们所推荐并以各项指导方针所支持的编程风格的情况中，这些“伪规则”是有害的。

即便是今天，仍有一些情况下这些规则是有意义的。
比如说，缺少合适的工具支持会导致异常在硬实时系统中的不适用，
但请不要盲目地信任“通俗智慧”（比如有关“效率”的未经数据支持的观点）；
这种“智慧”也许是基于几十年前的信息，或者是来自于与 C++ 有非常不同性质的语言的经验
（比如 C 或者 Java）。

对于这些伪规则的替代方案的正面观点都在各个规则的“替代方案”部分中给出。

伪规则概览：

* [NR.1: 请勿坚持认为声明都应当放在函数的最上面](S-not.md#Rnr-top)
* [NR.2: 请勿坚持使函数中只保留一个 `return` 语句](S-not.md#Rnr-single-return)
* [NR.3: 请勿避免使用异常](S-not.md#Rnr-no-exceptions)
* [NR.4: 请勿坚持把每个类定义放在其自己的源文件中](S-not.md#Rnr-lots-of-files)
* [NR.5: 请勿采用两阶段初始化](S-not.md#Rnr-two-phase-init)
* [NR.6: 请勿把所有清理操作放在函数末尾并使用 `goto exit`](S-not.md#Rnr-goto-exit)
* [NR.7: 请勿使所有数据成员 `protected`](S-not.md#Rnr-protected-data)
* ???

### <a name="Rnr-top"></a>NR.1: 请勿坚持认为声明都应当放在函数的最上面

##### 理由

“所有声明都在开头”的规则，是来自不允许在语句之后对变量和常量进行初始化的老编程语言的遗产。
这样做会导致更长的程序，以及更多由于未初始化的或者错误初始化的变量所导致的错误。

##### 示例，不好

```cpp
int use(int x)
{
    int i;
    char c;
    double d;

    // ... 做一些事 ...

    if (x < i) {
        // ...
        i = f(x, d);
    }
    if (i < x) {
        // ...
        i = g(x, c);
    }
    return i;
}
```

未初始化变量和其使用点的距离越长，出现 BUG 的机会就越大。
幸运的是，编译器可以发现许多“设值前使用”的错误。
不幸的是，编译器无法捕捉到所有这样的错误，而且一些 BUG 并不都像这个小例子中的这样容易发现。


##### 替代方案

* [坚持为对象进行初始化](S-expr.md#Res-always)。
* [ES.21: 不要在确实需要使用变量（或常量）之前就引入它](S-expr.md#Res-introduce)。

### <a name="Rnr-single-return"></a>NR.2: 请勿坚持使函数中只保留一个 `return` 语句

##### 理由

单返回规则会导致不必要地复杂的代码，并引入多余的状态变量。
特别是，单返回规则导致更难在函数开头集中进行错误检查。

##### 示例

```cpp
template<class T>
//  requires Number<T>
string sign(T x)
{
    if (x < 0)
        return "negative";
    if (x > 0)
        return "positive";
    return "zero";
}
```

为仅使用一个返回语句，我们得做类似这样的事：

```cpp
template<class T>
//  requires Number<T>
string sign(T x)        // 不好
{
    string res;
    if (x < 0)
        res = "negative";
    else if (x > 0)
        res = "positive";
    else
        res = "zero";
    return res;
}
```

这不仅更长，而且很可能效率更差。
越长越复杂的函数，对其进行变通就越是痛苦。
当然许多简单的函数因为它们本来就简单的逻辑都天然就只有一个 `return`。

##### 示例

```cpp
int index(const char* p)
{
    if (!p) return -1;  // 错误指标：替代方案是 "throw nullptr_error{}"
    // ... 进行查找以找出 p 的索引
    return i;
}
```

如果我们采纳这条规则的话，得做类似这样的事：

```cpp
int index2(const char* p)
{
    int i;
    if (!p)
        i = -1;  // 错误指标
    else {
        // ... 进行查找以找出 p 的索引
    }
    return i;
}
```

注意我们（故意地）违反了禁止未初始化变量的规则，因为这种风格通常都会导致这样。
而且，这种风格也会倾向于采用 [goto exit](S-not.md#Rnr-goto-exit) 伪规则。

##### 替代方案

* 保持函数短小简单。
* 随意使用多个 `return` 语句（以及抛出异常）。

### <a name="Rnr-no-exceptions"></a>NR.3: 请勿避免使用异常

##### 理由

一般有四种主要的不用异常的理由：

* 异常是低效的
* 异常会导致泄漏和错误
* 异常的性能无法预测
* 异常处理的运行时支持耗费过多空间

我们没有能够满足所有人的解决这个问题的办法。
无论如何，针对异常的讨论已经持续了四十多年了。
一些语言没有异常就无法使用，而另一些并不支持异常。
这在使用和不使用异常的方面都造成了强大的传统，并导致激烈的争论。

不过，我们可以简要说明，为什么我们认为对于通用编程以及这里的指导方针的情况来说，
异常是最佳的候选方案。
简单的论据，无论支持还是反对，都是缺乏说服力的。
确实存在一些特殊的应用，其中异常就是不合适的。
（例如，硬实时系统，且缺乏可靠的对于异常处理的耗费进行估计的支持）。

我们依次来考虑针对异常的主要反对观点：

* 异常是低效的：
和什么相比？
当进行比较时，请确保处理了同样的错误集合，并且它们都进行了等价的处理。
尤其是，不要对一个见到异常就立刻终止的程序和一个在记录错误日志之前
小心地进行资源清理的程序之间进行比较。
确实，某些系统的异常处理实现很糟糕；有时候，这样的实现迫使我们使用
其他错误处理方案，但这并不是异常的基本问题。
当使用某个有效的论据时——无论什么样的上下文——请小心你能拿出确实提供了所讨论的问题的
内部情况的健全的数据。
* 异常会导致泄漏和错误。
不会。
如果你的程序时一大堆乱糟糟的指针而没有总体的资源管理策略，
那么无论你干什么都会有问题。
如果你的系统是由上百万行这样的代码构成的，
那你可能是无法使用异常的，
不过这是一个有关过度和放纵的使用指针的问题，而不是异常的问题。
我们的观点是，你需要用 RAII 来让基于异常的错误处理变得简单且安全——比其他方案都要更简单和安全。
* 异常的性能无法预测。
如果你是在硬实时系统上，而你必须确保一个任务要在给定的时间内完成，
你需要一些工具来支撑这样的保证。
就我们所知，还没有出现这样的工具（至少对大多数程序员没有）。
* 异常处理的运行时支持耗费过多空间
小型（通常为嵌入式）系统中可能如此。
不过在放弃异常之前，请考虑采用统一的利用错误码的错误处理将耗费的空间有多少，
以及错误未被捕获将造成的损失由多少。

许多（可能是大多数）的和异常有关的问题都源自于需要和杂乱的老代码进行交互的历史性原因。

而支持使用异常的基本论点是：

* 它们把错误返回和普通返回进行了清晰的区分
* 它们无法被忘记或忽略
* 它们可以系统化地使用

请记住

* 异常是用于报告错误的（C++ 中；其他语言可能有异常的不同用法）。
* 异常不是用于可以局部处理的错误的。
* 不要试图在每个函数中捕获每一种异常（这样做是冗长的，臃肿的，而且会导致代码缓慢）。
* 异常不是用于那些当发生无法恢复的错误之后需要立即终止模块或系统的错误的。

##### 示例

```cpp
???
```

##### 替代方案

* [RAII](S-errors.md#Re-raii)
* 契约/断言：使用 GSL 的 `Expects` 和 `Ensures`（直到对契约的语言支持可以使用）

### <a name="Rnr-lots-of-files"></a>NR.4: 请勿坚持把每个类定义放在其自己的源文件中

##### 理由

将每个类都放进其自己的文件所导致的文件数量难于管理，并会拖慢编译过程。
单个的类很少是一种良好的维护和发布的逻辑单位。

##### 示例

```cpp
???
```

##### 替代方案

* 使用命名空间来包含逻辑上聚合的类和函数。

### <a name="Rnr-two-phase-init"></a>NR.5: 请勿采用两阶段初始化

##### 理由

将初始化拆分为两步会导致不变式的弱化，
更复杂的代码（必须处理半构造对象），
以及错误（当未能一致地正确处理半构造对象时）。

##### 示例，不好

```cpp
// 老式传统风格：有许多问题

class Picture
{
    int mx;
    int my;
    int * data;
public:
    // 主要问题：构造函数未进行完全构造
    Picture(int x, int y)
    {
        mx = x;         // 也不好：在构造函数体中而非
                        // 成员初始化式中进行赋值
        my = y;
        data = nullptr; // 也不好：在构造函数中而非
                        // 成员初始化式中进行常量初始化
    }

    ~Picture()
    {
        Cleanup();
    }

    // ...

    // 不好：两阶段初始化
    bool Init()
    {
        // 不变式检查
        if (mx <= 0 || my <= 0) {
            return false;
        }
        if (data) {
            return false;
        }
        data = (int*) malloc(mx*my*sizeof(int));   // 也不好：拥有原始指针，还用了 malloc
        return data != nullptr;
    }

    // 也不好：没有理由让清理操作作为单独的函数
    void Cleanup()
    {
        if (data) free(data);
        data = nullptr;
    }
};

Picture picture(100, 0); // 此时 picture 尚未就绪可用
// 这里将失败
if (!picture.Init()) {
    puts("Error, invalid picture");
}
// 现在有一个无效的 picture 对象实例。
```

##### 示例，好

```cpp
class Picture
{
    int mx;
    int my;
    vector<int> data;

    static int check_size(int size)
    {
        // 不变式检查
        Expects(size > 0);
        return size;
    }

public:
    // 更好的方式是以一个 2D 的 Size 类作为单个形参
    Picture(int x, int y)
        : mx(check_size(x))
        , my(check_size(y))
        // 现在已知 x 和 y 为有效的大小
        , data(mx * my) // 出错时将抛出 std::bad_alloc
    {
        // 图片就绪可用
    }

    // 编译器生成的析构函数会完成工作。（另见 C.21）

    // ...
};

Picture picture1(100, 100);
// picture1 已就绪可用……

// y 并非有效大小值，
// 缺省的契约违规行为将会调用 std::terminate
Picture picture2(100, 0);
// 不会抵达这里……
```

##### 替代方案

* 始终在构造函数中建立类不变式。
* 不要在需要对象之前就定义它。

### <a name="Rnr-goto-exit"></a>NR.6: 请勿把所有清理操作放在函数末尾并使用 `goto exit`

##### 理由

`goto` 是易错的。
这种技巧是进行 RAII 式的资源和错误处理的前异常时代的技巧。

##### 示例，不好

```cpp
void do_something(int n)
{
    if (n < 100) goto exit;
    // ...
    int* p = (int*) malloc(n);
    // ...
    if (some_error) goto_exit;
    // ...
exit:
    free(p);
}
```

请找出其中的 BUG。

##### 替代方案

* 使用异常和 [RAII](S-errors.md#Re-raii)
* 对于非 RAII 资源，使用 [`finally`](S-errors.md#Re-finally)。

### <a name="Rnr-protected-data"></a>NR.7: 请勿使所有数据成员 `protected`

##### 理由

`protected` 数据是一种错误来源。
`protected` 数据可以被各种地方的无界限数量的代码所操纵。
`protected` 数据是在类层次中等价于全局对象的东西。

##### 示例

```cpp
???
```

##### 替代方案

* [使成员数据 `public` 或者（更好地）`private`](S-class.md#Rh-protected)。


