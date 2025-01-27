# <a name="S-interfaces"></a>I: 接口

接口是程序中的两个部分之间的契约。严格地规定服务提供者和该服务使用者的预期是必要的。
在代码的组织中，良好的接口（易于理解，促进高效的使用方式，不易出错，支持进行测试，等等）可能是最重要的单个方面了。

接口规则概览：

* [I.1: 使接口明确](S-interfaces.md#Ri-explicit)
* [I.2: 避免非 `const` 全局变量](S-interfaces.md#Ri-global)
* [I.3: 避免使用单例](S-interfaces.md#Ri-singleton)
* [I.4: 使接口严格和强类型化](S-interfaces.md#Ri-typed)
* [I.5: 说明前条件（如果有）](S-interfaces.md#Ri-pre)
* [I.6: 优先使用 `Expects()` 来表达前条件](S-interfaces.md#Ri-expects)
* [I.7: 说明后条件](S-interfaces.md#Ri-post)
* [I.8: 优先使用 `Ensures()` 来表达后条件](S-interfaces.md#Ri-ensures)
* [I.9: 当接口是模板时，用概念来文档化其参数](S-interfaces.md#Ri-concepts)
* [I.10: 使用异常来表明无法实施所要求的任务](S-interfaces.md#Ri-except)
* [I.11: 决不以原始指针（`T*`）或引用（`T&`）来传递所有权](S-interfaces.md#Ri-raw)
* [I.12: 把不能为空的指针声明为 `not_null`](S-interfaces.md#Ri-nullptr)
* [I.13: 不要只用一个指针来传递数组](S-interfaces.md#Ri-array)
* [I.22: 避免全局对象之间进行复杂的初始化](S-interfaces.md#Ri-global-init)
* [I.23: 保持较少的函数参数数量](S-interfaces.md#Ri-nargs)
* [I.24: 避免可以由同一组实参以不同顺序调用造成不同含义的相邻形参](S-interfaces.md#Ri-unrelated)
* [I.25: 优先以空抽象类作为类层次的接口](S-interfaces.md#Ri-abstract)
* [I.26: 当想要跨编译器的 ABI 时，使用一个 C 风格的语言子集](S-interfaces.md#Ri-abi)
* [I.27: 对于稳定的程序库 ABI，考虑使用 Pimpl 手法](S-interfaces.md#Ri-pimpl)
* [I.30: 将有违规则的部分封装](S-interfaces.md#Ri-encapsulate)

**参见**

* [F: 函数](S-functions.md#S-functions)
* [C.concrete: 具体类型](S-class.md#SS-concrete)
* [C.hier: 类层次](S-class.md#SS-hier)
* [C.over: 函数重载和重载运算符](S-class.md#SS-overload)
* [C.con: 容器和其他资源封装类](S-class.md#SS-containers)
* [E: 错误处理](S-errors.md#S-errors)
* [T: 模板和泛型编程](S-templates.md#S-templates)

### <a name="Ri-explicit"></a>I.1: 使接口明确

##### 理由

正确性。未在接口中规定的假设很容易被忽视而且难于测试。

##### 示例，不好

通过全局（命名空间作用域）变量（调用模式）来控制函数的行为，是隐含的，而且潜在会造成困惑。例如：

```cpp
int round(double d)
{
    return (round_up) ? ceil(d) : d;    // 请勿：“不可见的”依赖
}
```

两次调用 `round(7.2)` 的含义可能给出不同的结果，这对于调用者来说是不明显的。

##### 例外

我们有时候会通过环境变量来控制一组操作的细节，比如常规/详细的输出，或者调试/优化版本。
使用非局部的控制方式可能带来困惑，但可以只用来控制实现的细节，否则就只有固定的语义了。

##### 示例，不好

通过非局部变量（比如 `errno`）进行的报告经常被忽略。例如：

```cpp
// 请勿如此：fprintf 的返回值未进行检查
fprintf(connection, "logging: %d %d %d\n", x, y, s);
```

要是连接已经关闭而导致没有产生日志输出的话会怎么样？参见 I.???。

**替代方案**: 抛出异常。异常是无法被忽略的。

**其他形式**: 避免通过非局部或者隐含的状态来跨越接口传递信息。
注意，非 `const` 的成员函数会通过对象的状态来向其他成员函数传递信息。

**其他形式**: 接口应当是函数或者一组函数集合。
函数可以是函数模板，而函数集合可以是类或者类模板。

##### 强制实施

* 【简单】 函数不能基于声明于命名空间作用域的变量来作出影响控制流的决定。
* 【简单】 函数不能对声明于命名空间作用域的变量进行写入操作。

### <a name="Ri-global"></a>I.2: 避免非 `const` 全局变量

##### 理由

非 `const` 全局变量能够隐藏依赖关系，并使这些依赖项可能出现无法预测的变动。

##### 示例

```cpp
struct Data {
    // ... 大量成员 ...
} data;            //  非 const 数据

void compute()     // 请勿这样做
{
    // ... 使用 data ...
}

void output()     // 请勿这样做
{
    // ... 使用 data ...
}
```

哪个可能会修改 `data` 呢？

**警告**: 全局对象的初始化并不是完全有序的。
当使用全局对象时，应当用常量为之初始化。
还要注意，即便对于 `const` 对象，也可能发生未定义的初始化顺序。

##### 例外

全局对象通常优于单例。

##### 注解

全局常量是有益的。

##### 注解

针对全局变量的规则同样适用于命名空间作用域的变量。

**替代方案**: 如果你用全局（或者更一般地说命名空间作用域）数据来避免复制操作的话，请考虑把数据以 `const` 引用的形式进行传递的方案。
另一种方案是把数据定义为某个对象的状态，而把操作定义为其成员函数。

**警告**: 请关注数据竞争：当一个线程能够访问非局部数据（或以引用传递的数据），而另一个线程执行被调用的函数时，就可能带来数据竞争。
指向可变数据的每个指针或引用都是潜在的数据竞争。

使用全局指针或引用来访问和修改非 const 且非局部的数据，并非是比非 const 全局变量更好的替代方案，
这是因为它并不能解决隐藏依赖性或潜在竞争条件的问题。

##### 注解

不可变数据是不会带来数据竞争条件的。

**参见**: 另见[关于调用函数的规则](S-functions.md#SS-call)。

#### 注解

这条规则是“避免”，而不是“不要用”。当然是有（罕见）例外的，比如 `cin`、`cout` 和 `cerr`。

##### 强制实施

【简单】 报告所有在命名空间作用域中声明的非 `const` 变量和全局的指向非 const 数据的指针/引用。


### <a name="Ri-singleton"></a>I.3: 避免使用单例

##### 理由

单例基本上就是经过伪装的更复杂的全局对象。

##### 示例

```cpp
class Singleton {
    // ... 大量代码，用于确保只创建一个 Singleton，
    // 进行正确地初始化，等等
};
```

单例的想法有许多变种。
这也是问题的一方面。

##### 注解

如果不想让全局对象被改变，请将其声明为 `const` 或 `constexpr`。

##### 例外

你可以使用最简单的“单例”形式（简单到通常不被当作单例）来获得首次使用时进行初始化的效果：

```cpp
X& myX()
{
    static X my_x {3};
    return my_x;
}
```

这是解决初始化顺序相关问题的最有效方案之一。
在多线程环境中，静态对象的初始化并不会引入数据竞争条件
（除非你不小心在其构造函数中访问了某个共享对象）。

注意局部的 `static` 对象初始化并不会蕴含竞争条件。
不过，如果 `X` 的销毁中涉及了需要进行同步的操作的话，我们就得用一个不那么简单的方案。
例如：

```cpp
X& myX()
{
    static auto p = new X {3};
    return *p;  // 有可能泄漏
}
```

这样就必须有人以某种适当的线程安全方式来 `delete` 这个对象了。
这是容易出错的，因此除了以下情况外我们并不使用这种技巧：

* `myX` 是在多线程代码中，
* 这个 `X` 对象需要销毁（比如由于它要释放某个资源），而且
* `X` 的析构函数的代码需要进行同步。

如果你和许多人一样把单例定义为只能创建一个对象的类的话，像 `myX` 这样的函数并非单例，而且这种好用的技巧并不算无单例规则的例外。

##### 强制实施

通常非常困难。

* 查找名字中包含 `singleton` 的类。
* 查找只创建一个对象的类（通过对对象计数或者检查其构造函数）。
* 如果某个类 X 具有公开的静态函数，并且它包含具有该类 X 类型的函数级局部静态变量并返回指向它的指针或者引用，就禁止它。

### <a name="Ri-typed"></a>I.4: 使接口严格和强类型化

##### 理由

类型是最简单和最好的文档，它们有定义明确的含义并因而提高了易读性，并且是在编译期进行检查的。
而且，严格类型化的代码通常也能更好地进行优化。

##### 示例，请勿这样做

考虑：

```cpp
void pass(void* data);    // 使用弱的并且缺乏明确性的类型 void* 是有问题的
```

调用者无法确定它允许使用哪些类型，而且因为它并没有指定 `const`，
也不确定其数据是否会被改动。注意，任何指针类型都可以隐式转换为 `void*`，
因此调用者很容易提供这样的值给它。

被调用方必须以 `static_cast` 将数据强制转换为某个无验证的类型以使用它。
这样做易于犯错，而且啰嗦。

应当仅在设计中无法以 C++ 来予以描述的数据的传递时才使用 `const void*`。请考虑使用 `variant` 或指向基类的指针来代替它。

**替代方案**: 通常，利用模板形参可以把 `void*` 排除而改为 `T*` 或者 `T&`。
对于泛型代码，这些个 `T` 可以是一般模板参数或者是概念约束的模板参数。

##### 示例，不好

考虑：

```cpp
draw_rect(100, 200, 100, 500); // 这些数值什么意思？

draw_rect(p.x, p.y, 10, 20); // 10 和 20 的单位是什么？
```

很明显调用者在描述一个矩形，不明确的是它们都和其哪些部分相关。而且 `int` 可以表示任何形式的信息，包括各种不同单位的值，因此我们必须得猜测这四个 `int` 的含义。前两个很可能代表坐标对偶 `x` 和 `y`，但后两个是什么呢？

注释和参数的名字可以有所帮助，但我们可以直截了当：

```cpp
void draw_rectangle(Point top_left, Point bottom_right);
void draw_rectangle(Point top_left, Size height_width);

draw_rectangle(p, Point{10, 20});  // 两个角点
draw_rectangle(p, Size{10, 20});   // 一个角和一对 (height, width)
```

显然，我们是无法利用静态类型系统识别所有的错误的，
例如，假定第一个参数是左上角这一点就依赖于约定（命名或者注释）。

##### 示例，不好

考虑：

```cpp
set_settings(true, false, 42); // 这些数值什么意思？
```

各参数类型及其值并不能表明其所指定的设置项是什么以及它们的值所代表的含义。

下面的设计则更加明确，安全且易读：

```cpp
alarm_settings s{};
s.enabled = true;
s.displayMode = alarm_settings::mode::spinning_light;
s.frequency = alarm_settings::every_10_seconds;
set_settings(s);
```

对于一组布尔值的情况，可以考虑使用某种标记 `enum`；这是一种用于表示一组布尔值的模式。

```cpp
enable_lamp_options(lamp_option::on | lamp_option::animate_state_transitions);
```

##### 示例，不好

下例中，接口中并未明确给出 `time_to_blink` 的含义：按秒还是按毫秒算？

```cpp
void blink_led(int time_to_blink) // 不好 -- 在单位上含糊
{
    // ...
    // 对 time_to_blink 做一些事
    // ...
}

void use()
{
    blink_led(2);
}
```

##### 示例，好

`std::chrono::duration` 类型可以让时间段的单位明确下来。

```cpp
void blink_led(milliseconds time_to_blink) // 好 -- 单位明确
{
    // ...
    // 对 time_to_blink 做一些事
    // ...
}

void use()
{
    blink_led(1500ms);
}
```

这个函数还可以写成使其接受任何时间段单位的形式。

```cpp
template<class rep, class period>
void blink_led(duration<rep, period> time_to_blink) // 好 -- 接受任何单位
{
    // 假设最小的有意义单位是毫秒
    auto milliseconds_to_blink = duration_cast<milliseconds>(time_to_blink);
    // ...
    // 对 milliseconds_to_blink 做一些事
    // ...
}

void use()
{
    blink_led(2s);
    blink_led(1500ms);
}
```

##### 强制实施

* 【简单】 报告将 `void*` 用作参数或返回类型的情况
* 【简单】 报告使用了多个 `bool` 参数的情况
* 【难于做好】 查找使用了过多基础类型的参数的函数。

### <a name="Ri-pre"></a>I.5: 说明前条件（如果有）

##### 理由

在参数上蕴含着使它们在被调用方中能够恰当使用的约束关系。

##### 示例

考虑：

```cpp
double sqrt(double x);
```

这里 `x` 必须是非负数。类型系统是无法（简洁并自然地）表达这点的，因而我们得用别的方法。例如：

```cpp
double sqrt(double x); // x 必须是非负数
```

一些前条件可以表示为断言。例如：

```cpp
double sqrt(double x) { Expects(x >= 0); /* ... */ }
```

理想情况下，这个 `Expects(x >= 0)` 应当是 `sqrt()` 的接口的一部分，但我们无法轻易做到这点。当前，我们将之放入定义式（函数体）之中。

**参考**: `Expects()` 在 [GSL](#???) 中有说明。

##### 注解

优先使用正式的必要条件说明，比如 `Expects(!p);`。
如果这样不可行，就在注释中使用文字来说明，比如 `// 序列 [p:q) 根据 < 排序`。

##### 注解

许多成员函数都以某个类所保持的不变式作为一项前条件。
这个不变式是由构造函数所建立的，且必须在被从类之外所调用的每个成员函数的退出时重新建立。
我们并不需要对每个成员函数都说明这个不变式。

##### 强制实施

【无法强制实施】

**参见**: 有关传递指针的规则。???

### <a name="Ri-expects"></a>I.6: 优先使用 `Expects()` 来表达前条件

##### 理由

清晰地表明这个条件是一个前条件，并便于工具的利用。

##### 示例

```cpp
int area(int height, int width)
{
    Expects(height > 0 && width > 0);            // 好
    if (height <= 0 || width <= 0) my_error();   // 隐晦的
    // ...
}
```

##### 注解

前条件是可以用许多方式来说明的，包括代码注释，`if` 语句，以及 `assert()`。
这些方式使其难于与普通代码之间进行区分，难于进行更新，难于利用工具来操作，而且可能具有错误的语义（你真的总是想要在调试模式中止程序而在生产运行中不做任何检查吗？）

##### 注解

前条件应当是接口的一部分，而不是实现的一部分，
但我们至今还没有能够做到这点的语言设施。
一旦语言支持变为可用（例如，参见[契约提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)），我们就将会采用前条件，后条件和断言的标准版本。

##### 注解

`Expects()` 还可以用于在算法的中部来检查某个条件。

##### 注解

使用 `unsigned` 并不是回避[确保非负数值](S-expr.md#Res-nonnegative)问题的好方法。

##### 强制实施

【无法强制实施】 要把各种对前条件进行断言的方式都找出来是不可行的。对那些易于识别的（如 `assert()`）实例给出警告的做法，其意义在缺少语言设施的前提下是有问题的。

### <a name="Ri-post"></a>I.7: 说明后条件

##### 理由

以检测到对返回结果的误解，还可能发现实现中存在错误。

##### 示例，不好

考虑：

```cpp
int area(int height, int width) { return height * width; }  // 不好
```

这里，我们（粗心大意地）遗漏了前条件的说明，因此高度和宽度必须是正数这点是不明确的。
我们也遗漏了后条件的说明，因此算法（`height * width`）对于大于最大整数的面积来说是错误的这点是不明显的。
可能会有溢出。
应该考虑使用：

```cpp
int area(int height, int width)
{
    auto res = height * width;
    Ensures(res > 0);
    return res;
}
```

##### 示例，不好

考虑一个著名的安全性 BUG：

```cpp
void f()    // 有问题的
{
    char buffer[MAX];
    // ...
    memset(buffer, 0, sizeof(buffer));
}
```

由于没有后条件来说明缓冲区应当被清零，优化器可能会将这个看似多余的 `memset()` 调用给清除掉：

```cpp
void f()    // 有改进
{
    char buffer[MAX];
    // ...
    memset(buffer, 0, sizeof(buffer));
    Ensures(buffer[0] == 0);
}
```

##### 注解

后条件通常是在说明函数目的的代码注释中非正式地进行说明的；用 `Ensures()` 可以使之更加系统化，更加明显，并且更容易检查。

##### 注解

后条件对于那些无法在所返回的结果中直接体现的东西来说尤其重要，比如要说明所用的数据结构。

##### 示例

考虑一个操作 `Record` 的函数，它使用 `mutex` 来避免数据竞争条件：

```cpp
mutex m;

void manipulate(Record& r)    // 请勿这样做
{
    m.lock();
    // ... 没有 m.unlock() ...
}
```

这里，我们“忘记”说明应当释放 `mutex`，因此我们搞不清楚这里 `mutex` 释放的缺失是一个 BUG 还是一种功能特性。
把后条件说明将使其更加明确：

```cpp
void manipulate(Record& r)    // 后条件: m 在退出后是未锁定的
{
    m.lock();
    // ... 没有 m.unlock() ...
}
```

现在这个 BUG 就明显了（但仅对阅读了代码注释的人类来说）。

更好的做法是使用 [RAII](S-resource.md#Rr-raii) 来在代码中保证后条件（“锁必须进行释放”）的实施：

```cpp
void manipulate(Record& r)    // 最好这样
{
    lock_guard<mutex> _ {m};
    // ...
}
```

##### 注解

理想情况下，后条件应当在接口或声明式中说明，让使用者易于见到它们。
只有那些与使用者有关的后条件才应当在接口中说明。
仅与内部状态相关的后条件应当属于定义式或实现。

##### 强制实施

【无法强制实施】 这是一条理念性的指导方针，一般情况下进行直接的
检查是不可行的。不过许多工具链中都有适用于特定领域的检查器，
比如针对锁定持有情况的检查器。

### <a name="Ri-ensures"></a>I.8: 优先使用 `Ensures()` 来表达后条件

##### 理由

清晰地表明这个条件是一个后条件，并便于工具的利用。

##### 示例

```cpp
void f()
{
    char buffer[MAX];
    // ...
    memset(buffer, 0, MAX);
    Ensures(buffer[0] == 0);
}
```

##### 注解

后条件是可以用许多方式来说明的，包括代码注释，`if` 语句，以及 `assert()`。
这些方式使其难于与普通代码之间进行区分，难于进行更新，难于利用工具来操作，而且可能具有错误的语义。

**替代方案**: 如“这个资源必须被释放”这样形式的后条件最好以 [RAII](S-resource.md#Rr-raii) 的方式来表达。

##### 注释

理想情况下，`Ensures` 应当是接口的一部分，但我们无法轻易做到这点。
当前，我们将之放入定义式（函数体）之中。
一旦语言支持变为可用（例如，参见[契约提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)），我们就将会采用前条件，后条件和断言的标准版本。

##### 强制实施

【无法强制实施】 要把各种对后条件进行断言的方式都找出来是不可行的。对那些易于识别的（如 `assert()`）实例给出警告的做法，其意义在缺少语言设施的前提下是有问题的。

### <a name="Ri-concepts"></a>I.9: 当接口是模板时，用概念来文档化其参数

##### 理由

更严谨地说明接口，并使其在（不远的）将来可以在编译时进行检查。

##### 示例

使用 C++20 风格的必要条件说明。例如：

```cpp
template<typename Iter, typename Val>
  requires input_iterator<Iter> && equality_comparable_with<iter_value_t<Iter>, Val>
Iter find(Iter first, Iter last, Val v)
{
    // ...
}
```

**参见**: [泛型编程](S-templates.md#SS-GP)和[概念](S-templates.md#SS-concepts)。

##### 强制实施

对未被概念所约束（在其声明式之中或者在一个 `requires` 子句中所给出）的并非可变数量的模板形参作出警告。

### <a name="Ri-except"></a>I.10: 使用异常来表明无法实施所要求的任务

##### 理由

不应该让错误可以被忽略，因为这将导致系统或者一次运算进入未定义的（或者预料之外的）状态。
这是错误的一个主要来源。

##### 示例

```cpp
int printf(const char* ...);    // 不好: 当输出失败时返回负值

template<class F, class ...Args>
// 好: 当无法启动一个新的线程时抛出 system_error
explicit thread(F&& f, Args&&... args);
```

##### 注解

错误是什么？

错误的含义是函数无法达成其所宣称的目标（这包括后条件的建立）。
把错误忽略掉的调用方代码将导致错误的结果，或者未定义的系统状态。
例如，无法连接一个远程服务器本身并不是错误：
这个服务器可以因为各种原因而拒绝连接，因此合乎常理的方式是让其返回一个其调用者必然要检查的结果。
不过，如果无法连接本身就是被当作一种错误的话，这个失败时应当抛出一个异常。

##### 例外

许多传统的接口函数（比如 UNIX 的信号处理器）都使用错误代码（就是 `errno`）来报告其实是状态代码而不是错误的东西。你没有更好的选择只能用它，因此对其调用并不违反本条规则。

##### 替代方案

如果你不能使用异常（比如说由于你的代码全都是老式的原始指针用法，或者由于你有硬实时性的约束），请考虑使用返回一对值的代码风格：

```cpp
int val;
int error_code;
tie(val, error_code) = do_something();
if (error_code) {
    // ... 处理错误或者退出 ...
}
// ... 使用 val ...
```

这种风格不幸地会导致未初始化的变量。
从 C++17 开始，可以使用 "结构化绑定" 功能特性来从返回值直接对多个变量初始化。

```cpp
auto [val, error_code] = do_something();
if (error_code) {
    // ... 处理错误或者退出 ...
}
// ... 使用 val ...
```

##### 注解

我们并不认为“性能”是一种不使用异常的合理理由。

* 通常，显式的错误检查和处理会消耗掉和异常处理一样多的时间和空间。
* 通常，使用异常的更清晰的代码会带来更好的性能（简化了对程序执行路径的追踪和其优化）。
* 一条对性能关键代码的好规则是，把检查从代码的[关键](S-performance.md#Rper-critical)部分中移出去。
* 长期来看，更规整的代码会得到更好的优化。
* 在做出性能相关的声明前一定要小心地[进行测量](S-performance.md#Rper-measure)。

**参见**: [I.5](S-interfaces.md#Ri-pre) 和 [I.7](S-interfaces.md#Ri-post) 有关报告前条件和后条件的违反。

##### 强制实施

* 【无法强制实施】 这是一条理念性的指导方针，进行直接的检查是不可行的。
* 查找 `errno`。

### <a name="Ri-raw"></a>I.11: 决不以原始指针（`T*`）或引用（`T&`）来传递所有权

##### 理由

如果对调用者和被调用方哪一个拥有对象有疑问，那就会造成泄漏或者发生提早的析构。

##### 示例

考虑：

```cpp
X* compute(args)    // 请勿这样做
{
    X* res = new X{};
    // ...
    return res;
}
```

应当由谁来删除返回的这个 `X` 呢？如果 `compute` 返回引用的话这个问题将更难发现。
应该考虑按值来返回结果（如果结果比较大的话就用移动语义）：

```cpp
vector<double> compute(args)  // 好的
{
    vector<double> res(10000);
    // ...
    return res;
}
```

**替代方案**: 用“智能指针”来[传递所有权](S-resource.md#Rr-smartptrparam)，比如 `unique_ptr`（专有所有权）和 `shared_ptr`（共享所有权）。
这样做比返回对象自身来说并没有那么简炼，而且通常也不那么高效，
因此，仅当需要引用语义时再使用智能指针。

**替代方案**: 有时候因为 ABI 兼容性的要求或者缺少资源，是无法对老代码进行修改的。
这种情况下，请用[指导方针支持库](#???)的 `owner` 来标记拥有对象的指针：

```cpp
owner<X*> compute(args)    // 现在就明确传递了所有权这一点
{
    owner<X*> res = new X{};
    // ...
    return res;
}
```

这告诉了分析工具 `res` 是一个所有者。
就是说，它的值必须被 `delete`，或者被传递给另一个所有者，正如这里的 `return` 所做。

在资源包装类的实现中也同样使用了 `owner`。

##### 注解

以原始指针（或迭代器）的形式传递的对象，都假定是由调用方
所有的，因此其生存期也由调用方来处理。换种方式来看：
传递所有权的 API 相对于传递指针的 API 来说比较少见，
因此缺省情况就是“不传递所有权”。

**参见**: [实参传递](S-functions.md#Rf-conventional)，[使用智能指针参数](S-resource.md#Rr-smartptrparam)，以及[返回值](S-functions.md#Rf-value-return)。

##### 强制实施

* 【简单】 当对并非 `owner<T>` 的原始指针进行 `delete` 就发出警告。建议使用标准库的资源包装或者使用 `owner<T>`。
* 【简单】 当任何代码路径上遗漏了对 `owner` 指针的 `reset` 或者显式的 `delete` 时就发出警告。
* 【简单】 当把 `new` 或者返回值为 `owner` 的函数的返回值赋值给原始指针或非 `ower` 的引用时就发出警告。

### <a name="Ri-nullptr"></a>I.12: 把不能为空的指针声明为 `not_null`

##### 理由

帮助避免对 `nullptr` 解引用的错误。
通过避免多余的 `nullptr` 检查来提高性能。

##### 示例

```cpp
int length(const char* p);            // 不清楚 length(nullptr) 是否有效

length(nullptr);                      // OK?

int length(not_null<const char*> p);  // 有改善：可以假定 p 不可能为 nullptr

int length(const char* p);            // 只好假定 p 可以为 nullptr
```

通过在源代码中说明意图，实现者和工具就可以提供更好的诊断能力，比如通过静态分析来找出某些种类的错误，还可以实施优化，比如移除分支和空值测试。

##### 注解

`not_null` 在[指导方针支持库](#???)中定义。

##### 注解

指向 `char` 的指针将指向 C 风格的字符串（以零终结的字符的连续串）这一点仍然是潜规则，并且也是混乱和错误的潜在来源。请使用 `czstring` 来代替 `const char*`。

```cpp
// 可以假定 p 不能为 nullptr
// 可以假定 p 指向以零终结的字符数组
int length(not_null<zstring> p);
```

注意： `length()` 显然是经过伪装的 `std::strlen()`。

##### 强制实施

* 【简单】〔基础〕 如果有函数在所有控制流路径上访问指针参数之前检查它是否是 `nullptr`，则给出警告称其应当被声明为 `not_null`。
* 【复杂】 如果有指针返回值的函数在所有返回路径上都保证其不是 `nullptr`，则给出警告称返回类型应当被声明为 `not_null`。

### <a name="Ri-array"></a>I.13: 不要只用一个指针来传递数组

##### 理由

 (pointer, size) 式的接口是易于出错的。同样，（指向数组的）普通指针还必须依赖某种约定以使被调用方来确定其大小。

##### 示例

考虑：

```cpp
void copy_n(const T* p, T* q, int n); // 从 [p:p+n) 复制到 [q:q+n)
```

当由 `q` 所指向的数组少于 `n` 个元素会怎么样？此时我们将覆写一些可能无关的内存。
当由 `p` 所指向的数组少于 `n` 个元素会怎么样？此时我们将读取一些可能无关的内存。
此二者都是未定义的行为，而且可能是非常恶劣的 BUG。

##### 替代方案

考虑使用明确的 `span`：

```cpp
void copy(span<const T> r, span<T> r2); // 将 r 复制给 r2
```

##### 示例，不好

考虑：

```cpp
void draw(Shape* p, int n);  // 糟糕的接口；糟糕的代码
Circle arr[10];
// ...
draw(arr, 10);
```

把 `10` 作为参数 `n` 传递可能是错误的：虽然最常见的约定是假定有 `[0:n)`，但这点并未不是明确的。更糟糕的是，`draw()` 的调用通过编译了：这里有一次从数组到指针的隐式转换（数组退化），然后又进行了从 `Circle` 到 `Shape` 的另一次隐式转换。`draw()` 是不可能安全地迭代这个数组的：它无法知道元素的大小。

**替代方案**: 使用一个辅助类来确保元素的数量正确，并避免进行危险的隐式转换。例如：

```cpp
void draw2(span<Circle>);
Circle arr[10];
// ...
draw2(span<Circle>(arr));  // 推断出元素的数量
draw2(arr);    // 推断出元素的类型和数组大小

void draw3(span<Shape>);
draw3(arr);    // 错误: 无法将 Circle[10] 转换为 span<Shape>
```

这个 `draw2()` 传递了与 `draw()` 同样数量的信息，但明确指定了它接受的是 `Circle` 的范围。参见 ???.

##### 例外

使用 `zstring` 和 `czstring` 来表示 C 风格的以零终结字符串。
但这样做时，应当使用 `std::string_view` 或 [GSL](#???) 中的 `span<char>` 以避免范围错误。

##### 强制实施

* 【简单】〔边界〕 对任何依赖于从数组类型向指针类型的隐式转换的表达式给出警告。允许 zstring/czstring 指针类型的例外。
* 【简单】〔边界〕 对任何指针类型表达式进行且结果为指针类型的值的运算操作给出警告。允许 zstring/czstring 指针类型的例外。

### <a name="Ri-global-init"></a>I.22: 避免全局对象之间进行复杂的初始化

##### 理由

复杂的初始化可能导致未定义的执行顺序。

##### 示例

```cpp
// file1.c

extern const X x;

const Y y = f(x);   // 读取 x; 写入 y

// file2.c

extern const Y y;

const X x = g(y);   // 读取 y; 写入 x
```

由于 `x` 和 `y` 是处于不同翻译单元之内的，调用 `f()` 和 `g()` 的顺序就是未定义的；
我们可能会访问到还未初始化的 `const` 对象。
这里展示的是，全局（命名空间作用域）对象的初始化顺序难题并不仅限于全局*变量*而已。

##### 注解

并发代码中的初始化顺序问题是更加难于处理的。
所以通常最好完全避免使用全局（命名空间作用域）的对象。

##### 强制实施

* 标记调用了非 `constexpr` 函数的全局初始化式
* 标记访问了 `extern` 对象的全局初始化式

### <a name="Ri-nargs"></a>I.23: 保持较少的函数参数数量

##### 理由

大量参数会带来更大的出现混乱的机会。大量传递参数与其他替代方案相比也通常是代价比较大的。

##### 讨论

两个最常见的使得函数具有过多参数的原因是：

1. *缺乏抽象*
   缺少一种抽象，使得一个组合值被以
   一组独立的元素的方式进行传递，而不是以一个单独的保证了不变式的对象来传递。
   这不仅使其参数列表变长，而且会导致错误，
   因为各个成分值无法再被某种获得保证的不变式进行保护。

2. *违反了“函数单一职责”原则*
   这个函数试图完成多项任务，它可能应当被重构。

##### 示例

标准库的 `merge()` 函数达到了我们可以自如处理的界限：

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                     InputIterator2 first2, InputIterator2 last2,
                     OutputIterator result, Compare comp);
```

注意，这属于上面的第一种问题：缺乏抽象。STL 传递的不是范围（抽象），而是一对迭代器（未封装的成分值）。

其中有四个模板参数和六个函数参数。
为简化最常用和最简单的用法，比较器参数可以缺省使用 `<`：

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator merge(InputIterator1 first1, InputIterator1 last1,
                     InputIterator2 first2, InputIterator2 last2,
                     OutputIterator result);
```

这实际上不会减低其整体复杂性，但它减少了对于许多使用者的表面复杂性。
为了真正地减少参数的数量，我们得把参数归拢到更高层的抽象之中：

```cpp
template<class InputRange1, class InputRange2, class OutputIterator>
OutputIterator merge(InputRange1 r1, InputRange2 r2, OutputIterator result);
```

把参数成“批”进行组合是减少参数数量和增加进行检查的机会的一般性技巧。

或者，我们也可以用标准库概念来定义“三个类型必须可以用于归并”：

```cpp
template<class In1, class In2, class Out>
  requires mergeable<In1, In2, Out>
Out merge(In1 r1, In2 r2, Out result);
```

##### 示例

安全性剖面配置中建议将以下代码

```cpp
void f(int* some_ints, int some_ints_length);  // 不好：C 风格，不安全
```

替换为

```cpp
void f(gsl::span<int> some_ints);              // 好：安全，有边界检查
```

这样，使用一种抽象可以获得安全性和健壮性的好处，而且自然地减少了参数的数量。

##### 注解

多少参数算很多？请使用少于四个参数。
有些函数确实最好表现为四个独立的参数，但这样的函数并不多。

**替代方案**: 使用更好的抽象：把参数归集为由意义的对象，然后（按值或按引用）传递这些对象。

**替代方案**: 利用默认实参或者重载来让最常见的调用方式可以用比较少的实参来进行。

##### 强制实施

* 当函数声明了两个类型相同的迭代器（也包括指针）而不是一个范围或视图，就给出警告。
* 【无法强制实施】 这是一条理念性的指导方针，进行直接的检查是不可行的。

### <a name="Ri-unrelated"></a>I.24: 避免可以由同一组实参以不同顺序调用造成不同含义的相邻形参

##### 理由

相同类型的相邻参数很容易被不小心互换掉。

##### 示例，不好

考虑：

```cpp
void copy_n(T* p, T* q, int n);  // 从 [p:p + n) 复制到 [q:q + n)
```

这是个 K&R C 风格接口的一种恶劣的变种。它导致很容易把“目标”和“来源”参数搞反。

可以在“来源”参数上使用 `const`：

```cpp
void copy_n(const T* p, T* q, int n);  // 从 [p:p + n) 复制到 [q:q + n)
```

##### 例外

当参数的顺序不重要时，不会造成问题：

```cpp
int max(int a, int b);
```

##### 替代方案

不要以指针来传递数组，而要传递用来表示一个范围的对象（比如一个 `span`）：

```cpp
void copy_n(span<const T> p, span<T> q);  // 从 p 复制到 q
```

##### 替代方案

定义一个 `struct` 来作为参数类型，并依照各个参数来命名它的各字段：

```cpp
struct SystemParams {
    string config_file;
    string output_path;
    seconds timeout;
};
void initialize(SystemParams p);
```

这样做带来一种使其调用代码对于以后的读者变得明晰的倾向，因为这种参数
在调用点通常都要按名字来进行填充。

##### 注解

只有接口设计者才能胜任处理违反本条指导方针的源头问题。

##### 强制实施策略

【简单】 当两个连续的参数具有相同的类型时就给出警告。

我们仍在寻找不这么简单的强制实施方式。

### <a name="Ri-abstract"></a>I.25: 优先以空抽象类作为类层次的接口

##### 理由

空的（没有非静态成员数据）抽象类要比带有状态的基类更倾向于保持稳定。

##### 示例，不好

你知道 `Shape` 总会冒出来的 :-)

```cpp
class Shape {  // 不好: 接口类中加载了数据
public:
    Point center() const { return c; }
    virtual void draw() const;
    virtual void rotate(int);
    // ...
private:
    Point c;
    vector<Point> outline;
    Color col;
};
```

这将强制性要求每个派生类都要计算出一个中心点——即使这并不容易，而且这个中心点从不会被用到。相似地说，不是每个 `Shape` 都有一个 `Color`，而许多 `Shape` 也最好别用一个定义成一系列 `Point` 的轮廓来进行表示。使用抽象类要更好：

```cpp
class Shape {    // 有改进: Shape 是一个纯接口
public:
    virtual Point center() const = 0;   // 纯虚函数
    virtual void draw() const = 0;
    virtual void rotate(int) = 0;
    // ...
    // ... 没有数据成员 ...
    // ...
    virtual ~Shape() = default;        
};
```

##### 强制实施

【简单】 当把类 `C` 的指针/引用赋值给 `C` 的某个基类的指针/引用，而这个基类包含数据成员时，就给出警告。

### <a name="Ri-abi"></a>I.26: 当想要跨编译器的 ABI 时，使用一个 C 风格的语言子集

##### 理由

不同的编译器会实现不同的类的二进制布局，异常处理，函数名字，以及其他的实现细节。

##### 例外

在一些平台上正有公共的 ABI 兴起，这可以使你从更加苛刻的限制中摆脱出来。

##### 注解

如果你只用一种编译器，你也可以在接口上使用完全的 C++。但当升级到新的编译器版本之后，可能需要进行重新编译。

##### 强制实施

【无法强制实施】 要可靠地识别某个接口是否是构成 ABI 的一部分是很困难的。

### <a name="Ri-pimpl"></a>I.27: 对于稳定的程序库 ABI，考虑使用 Pimpl 手法

##### 理由

由于私有数据成员参与类的内存布局，而私有成员函数参与重载决议，
对这些实现细节的改动都要求使用了这类的所有用户全部重新编译。而持有指向实现的指针（Pimpl）的
非多态的接口类，则可以将类的用户从其实现的改变隔离开来，其代价是一层间接。

##### 示例

接口（widget.h）

```cpp
class widget {
    class impl;
    std::unique_ptr<impl> pimpl;
public:
    void draw(); // 公开 API 转发给实现
    widget(int); // 定义于实现文件中
    ~widget();   // 定义于实现文件中，其中 impl 将为完整类型
    widget(widget&&) noexcept; // 定义于实现文件中
    widget(const widget&) = delete;
    widget& operator=(widget&&) noexcept; // 定义于实现文件中
    widget& operator=(const widget&) = delete;
};

```

实现（widget.cpp）

```cpp
class widget::impl {
    int n; // private data
public:
    void draw(const widget& w) { /* ... */ }
    impl(int n) : n(n) {}
};
void widget::draw() { pimpl->draw(*this); }
widget::widget(int n) : pimpl{std::make_unique<impl>(n)} {}
widget::widget(widget&&) noexcept = default;
widget::~widget() = default;
widget& widget::operator=(widget&&) noexcept = default;
```

##### 注解

参见 [GOTW #100](https://herbsutter.com/gotw/_100/) 和 [cppreference](http://en.cppreference.com/w/cpp/language/pimpl) 有关这个手法相关的权衡和其他实现细节。

##### 强制实施

【无法强制】 很难可靠地识别出哪个接口属于 ABI 的一部分。

### <a name="Ri-encapsulate"></a>I.30: 将有违规则的部分封装

##### 理由

维持代码简单且安全。
有时候因为逻辑的或者性能的原因，需要使用难看的，不安全的或者易错的技术。
此时，将它们局部化，而不是使其“感染”接口，可以避免更多的程序员团队必须当心其
细节和微妙之处。
如果可能的话，实现复杂度不能通过接口渗透到用户代码之中。

##### 示例

考虑一个程序，其基于某种形式的输入（比如 `main` 的实参）来决定
从文件，从命令行，还是从标准输入来获得输入数据。
我们可能会将其写成

```cpp
bool owned;
owner<istream*> inp;
switch (source) {
case std_in:        owned = false; inp = &cin;                       break;
case command_line:  owned = true;  inp = new istringstream{argv[2]}; break;
case file:          owned = true;  inp = new ifstream{argv[2]};      break;
}
istream& in = *inp;
```

这违反了[避免未初始化变量](S-expr.md#Res-always)，
[避免忽略所有权](S-interfaces.md#Ri-raw)，
和[避免魔法常量](S-expr.md#Res-magic)等规则。
尤其是，人们必须记得找地方写

```cpp
if (owned) delete inp;
```

我们可以通过使用带有一个特殊的删除器（对 `cin` 不做任何事）的 `unique_ptr` 来处理这个特定的例子，
但这对于新手来说较复杂（他们很容易遇到这种问题），并且这个例子其实是一个更一般的问题的特例：
我们希望将其当做静态的某种属性（此处为所有权），需要在运行时进行
偶尔的处理。
一般的，更常见的，且更安全的例子可以被静态处理，因而我们并不希望为它们添加开销和复杂性。
然而我们还是不得不处理那些不常见的，较不安全的，而且更为昂贵的情况。
[[Str15]](http://www.stroustrup.com/resource-model.pdf) 中对这种例子有所探讨。

由此，我们编写这样的类

```cpp
class Istream { [[gsl::suppress(lifetime)]]
public:
    enum Opt { from_line = 1 };
    Istream() { }
    Istream(zstring p) : owned{true}, inp{new ifstream{p}} {}            // 从文件读取
    Istream(zstring p, Opt) : owned{true}, inp{new istringstream{p}} {}  // 从命令行读取
    ~Istream() { if (owned) delete inp; }
    operator istream& () { return *inp; }
private:
    bool owned = false;
    istream* inp = &cin;
};
```

这样，`istream` 的所有权的动态本质就被封装起来。
大体上，在现实的代码中还是需要针对潜在的错误添加一些检查。

##### 强制实施

* 很难，判断那种违背规则的代码是基本的是很难做到的
* 对允许规则违背的部分跨越接口的规则抑制进行标记

