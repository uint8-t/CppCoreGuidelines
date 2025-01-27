# <a name="S-resource"></a>R: 资源管理

本章节中包含于资源相关的各项规则。
资源，就是任何必须进行获取，并（显式或隐式）进行释放的东西，比如内存、文件句柄、Socket 和锁等等。
其必须进行释放的原因在于它们是短缺的，因而即便是采用延迟释放也许也是有害的。
基本的目标是要确保不会泄漏任何资源，而且不会持有不在需要的任何资源。
负责释放某个资源的实体被称作是其所有者。

少数情况下，发生泄漏是可接受的甚至是理想的：
如果所编写的程序只是基于输入来产生输出，而其所需的内存正比于输入的大小，那么最理想的（性能和开发便利性）策略有时候恰是不要删除任何东西。
如果有足够的内存来处理最大输入的话，让其泄漏即可，但如果并非如此，应当保证给出一条恰当的错误消息。
这里，我们将忽略这样的情况。

* 资源管理规则概览：

  * [R.1: 利用资源句柄和 RAII（资源获取即初始化）来自动管理资源](S-resource.md#Rr-raii)
  * [R.2: 接口中的原生指针（仅）代表个体对象](S-resource.md#Rr-use-ptr)
  * [R.3: 原生指针（`T*`）没有所有权](S-resource.md#Rr-ptr)
  * [R.4: 原生引用（`T&`）没有所有权](S-resource.md#Rr-ref)
  * [R.5: 优先采用有作用域的对象，避免不必要的堆分配](S-resource.md#Rr-scoped)
  * [R.6: 避免非 `const` 的全局变量](S-resource.md#Rr-global)

* 分配和回收规则概览：

  * [R.10: 避免 `malloc()` 和 `free()`](S-resource.md#Rr-mallocfree)
  * [R.11: 避免显式调用 `new` 和 `delete`](S-resource.md#Rr-newdelete)
  * [R.12: 显式资源分配的结果应当立即交给一个管理对象](S-resource.md#Rr-immediate-alloc)
  * [R.13: 单个表达式语句中至多进行一次显式资源分配](S-resource.md#Rr-single-alloc)
  * [R.14: 避免使用 `[]` 形参，优先使用 `span`](#Rr-ap)
  * [R.15: 总是同时重载相匹配的分配、回收函数对](S-resource.md#Rr-pair)

* <a name="Rr-summary-smartptrs"></a>智能指针规则概览：

  * [R.20: 用 `unique_ptr` 或 `shared_ptr` 表示所有权](S-resource.md#Rr-owner)
  * [R.21: 优先采用 `unique_ptr` 而不是 `shared_ptr`，除非需要共享所有权](S-resource.md#Rr-unique)
  * [R.22: 使用 `make_shared()` 创建 `shared_ptr`](S-resource.md#Rr-make_shared)
  * [R.23: 使用 `make_unique()` 创建 `unique_ptr`](S-resource.md#Rr-make_unique)
  * [R.24: 使用 `std::weak_ptr` 来打断 `shared_ptr` 的循环引用](S-resource.md#Rr-weak_ptr)
  * [R.30: 以智能指针为参数，仅用于明确表达生存期语义](S-resource.md#Rr-smartptrparam)
  * [R.31: 非 `std` 的智能指针，应当遵循 `std` 的行为模式](S-resource.md#Rr-smart)
  * [R.32: `unique_ptr<widget>` 参数用以表达函数假定获得 `widget` 的所有权](S-resource.md#Rr-uniqueptrparam)
  * [R.33: `unique_ptr<widget>&` 参数用以表达函数对该 `widget` 重新置位](S-resource.md#Rr-reseat)
  * [R.34: `shared_ptr<widget>` 参数用以表达函数是所有者的一份子](S-resource.md#Rr-sharedptrparam-owner)
  * [R.35: `shared_ptr<widget>&` 参数用以表达函数可能会对共享的指针重新置位](S-resource.md#Rr-sharedptrparam)
  * [R.36: `const shared_ptr<widget>&` 参数用以表达它可能将保留一个对对象的引用 ???](S-resource.md#Rr-sharedptrparam-const)
  * [R.37: 不要把来自某个智能指针别名的指针或引用传递出去](S-resource.md#Rr-smartptrget)

### <a name="Rr-raii"></a>R.1: 利用资源句柄和 RAII（资源获取即初始化）来自动管理资源

##### 理由

避免资源泄漏和人工资源管理的复杂性。
C++ 语言确保的构造函数/析构函数对称性，反映了资源的获取/释放函数对（比如 `fopen`/`fclose`，`lock`/`unlock`，以及 `new`/`delete` 等）的对称性本质。
每当需要处理某个需要成对儿的获取/释放函数调用的资源时，应当将资源封装到保证这种配对调用的对象之中——在构造函数中获取资源，并在其析构函数中释放它。

##### 示例，不好

考虑：

```cpp
void send(X* x, string_view destination)
{
    auto port = open_port(destination);
    my_mutex.lock();
    // ...
    send(port, x);
    // ...
    my_mutex.unlock();
    close_port(port);
    delete x;
}
```

这段代码中，你必须记得在所有路径中调用 `unlock`、`close_port` 和 `delete`，并且每个都恰好调用一次。
而且，一旦上面标有 `...` 的任何代码抛出了异常，`x` 就会泄漏，而 `my_mutex` 则保持锁定。

##### 示例

考虑：

```cpp
void send(unique_ptr<X> x, string_view destination)  // x 拥有这个 X
{
    Port port{destination};            // port 拥有这个 PortHandle
    lock_guard<mutex> guard{my_mutex}; // guard 拥有这个锁
    // ...
    send(port, x);
    // ...
} // 自动解锁 my_mutex 并删除 x 中的指针
```

现在所有的资源清理都是自动进行的，不管是否发生了异常，所有路径中都会执行一次。额外的好处是，该函数现在明确声称它将接过指针的所有权。

`Port` 又是什么呢？是一个封装资源的便利句柄：

```cpp
class Port {
    PortHandle port;
public:
    Port(string_view destination) : port{open_port(destination)} { }
    ~Port() { close_port(port); }
    operator PortHandle() { return port; }

    // port 句柄通常是不能克隆的，因此根据需要关闭了复制和赋值
    Port(const Port&) = delete;
    Port& operator=(const Port&) = delete;
};
```

##### 注解

一旦发现一个“表现不良”的资源并未以带有析构函数的类来表示，就用一个类来包装它，或者使用 [`finally`](S-errors.md#Re-finally)。

**参见**: [RAII](S-errors.md#Re-raii)

### <a name="Rr-use-ptr"></a>R.2: 接口中的原生指针（仅）代表个体对象

##### 理由

最好用某个容器类型（比如 `vector`，拥有数据），或者用 `span`（不拥有数据）来表示数组。
这些容器和视图都带有足够的信息来进行范围检查。

##### 示例，不好

```cpp
void f(int* p, int n)   // n 为 p[] 中的元素数量
{
    // ...
    p[2] = 7;   // 不好: 对原生指针采用下标
    // ...
}
```

编译期不会读注释，而如果不读其他代码的话你也无法指导 `p` 是否真的指向了 `n` 个元素。
应当代之以 `span`。

##### 示例

```cpp
void g(int* p, int fmt)   // 用格式 fmt 打印 *p
{
    // ... 只使用 *p 和 p[0] ...
}
```

##### 例外

C 风格的字符串是以单个指向以零结尾的字符序列的指针来传递的。
为了表明对这种约定的依赖，应当使用 `zstring` 而不是 `char*`。

##### 注解

当前许多的单元素指针的用法其实都应当用引用。
不过，如果 `nullptr` 是可能的值的话，也许引用就不是合理的替代方案了。

##### 强制实施

* 对并非来自容器、视图或迭代器的指针进行的指针算术（包括 `++`）进行标记。
  这条规则对比较老的代码库实施时，可能会产生巨量的误报。
* 对把数组名被传递为单纯的指针进行标记。

### <a name="Rr-ptr"></a>R.3: 原生指针（`T*`）没有所有权

##### 理由

对此（C++ 标准中和大多数代码中都）没有异议，大多数原生指针都是无所有权的。
我们希望将有所有权的指针标示出来，以使得可以可靠和高效地删除由有所有权指针所指向的对象。

##### 示例

```cpp
void f()
{
    int* p1 = new int{7};           // 不好: 原生指针拥有了所有权
    auto p2 = make_unique<int>(7);  // OK: int 被一个唯一指针所拥有
    // ...
}
```

`unique_ptr` 保证对它的对象进行删除（即便是发生异常时也如此），以此保护不发生泄漏。而 `T*` 做不到这点。

##### 示例

```cpp
template<typename T>
class X {
public:
    T* p;   // 不好: 不清楚 p 是不是带有所有权
    T* q;   // 不好: 不清楚 q 是不是带有所有权
    // ...
};
```

可以通过明确所有权来修正这个问题：

```cpp
template<typename T>
class X2 {
public:
    owner<T*> p;  // OK: p 具有所有权
    T* q;         // OK: q 没有所有权
    // ...
};
```

##### 例外

最主要的例外就是遗留代码，尤其是那些必须维持可以用 C 编译或者通过 ABI 来建立 C 和 C 风格的 C++ 之间的接口的代码。
亿万行的代码都违反本条规则而让 `T*` 具有所有权的现实是无法被忽略的。
我们由衷希望看到程序变换工具把这些 20 岁以上的“遗留”代码转换成光鲜的现代代码，
我们鼓励这种工具的开发、部署和使用，
我们希望这里的各项指导方针能够有助于这种工具的开发，
而且我们也在这一领域的研发工作中持续作出贡献。
但是，这是需要时间的：“遗留代码”的产生比我们能翻新的老代码还要快，因此这将会花费许多年的时间。

这些代码是无法被全部重写的（即便假定有良好的代码转换软件），尤其不会很快发生。
这个问题是不能简单通过把所有有所有权的指针都转换成 `unique_ptr` 和 `shared_ptr` 来（大规模）解决的，
这部分是因为我们确实需要在基础的资源句柄的实现中一起使用有所有权的“原生指针”和简单的指针。
例如，常见的 `vector` 实现中都有一个有所有权的指针和两个没有所有权的指针。
许多 ABI（以及基本上全部的面向 C 的接口代码）都使用 `T*`，其中不少都是有所有权的。
一些接口是无法简单地用 `owner` 来标记的，因为它们需要维持可以作为 C 来编译，
（这可能是少见的恰当的使用宏的场合，它仅在 C++ 模式中扩展为 `owner`）。

##### 注解

`owner<T*>` 并没有超出 `T*` 的默认语义。使用它可以不改动任何使用方代码，也不会影响 ABI。
它只不过是一项针对程序员和分析工具的提示。
比如说，当 `owner<T*>` 是某个类的成员时，这个类最好提供一个析构函数来 `delete` 它。

##### 示例，不好

返回（原生）指针的做法向调用方暴露了在生存期管理上的不确定性；就是说，谁应该删除其所指向的对象呢？

```cpp
Gadget* make_gadget(int n)
{
    auto p = new Gadget{n};
    // ...
    return p;
}

void caller(int n)
{
    auto p = make_gadget(n);   // 要记得 delete p
    // ...
    delete p;
}
```

除了遭受[资源泄漏](#???)的问题外，这也带来了一组假性的分配和回收操作，而这其实是不必要的。如果 Gadget 可以廉价地从函数转移出来（就是说，它很小，或者具有高效的移动操作）的话，直接“按值”返回即可（参见[输出返回值](S-functions.md#Rf-out)）：

```cpp
Gadget make_gadget(int n)
{
    Gadget g{n};
    // ...
    return g;
}
```

##### 注解

这条规则适用于工厂函数。

##### 注解

如果指针语义是必须的（比如说，因为返回类型需要指代类层次中的基类（或接口）），则可以返回“智能指针”。

##### 强制实施

* 【简单】 对在并非 `owner<T>` 的原生指针上进行的 `delete` 给出警告。
* 【中等】 对一个 `owner<T>` 指针，当并非每个代码路径中都要么进行 `reset` 要么明确 `delete`，则给出警告。
* 【简单】 当 `new` 的返回值被赋值给原生指针时，给出警告。
* 【简单】 当函数所返回的对象是在函数中所分配的，并且它具有移动构造函数时，给出警告。
  建议代之以按值返回。

### <a name="Rr-ref"></a>R.4: 原生引用（`T&`）没有所有权

##### 理由

对此（C++ 标准中和大多数代码中都）没有异议，大多数原生引用都是无所有权的。
我们希望将所有者都标示出来，以使得可以可靠和高效地删除由有所有权指针所指向的对象。

##### 示例

```cpp
void f()
{
    int& r = *new int{7};  // 不好: 原生的具有所有权的引用
    // ...
    delete &r;             // 不好: 违反了有关删除原生指针的规则
}
```

**参见**: [原生指针的规则](S-resource.md#Rr-ptr)

##### 强制实施

参见[原生指针的规则](S-resource.md#Rr-ptr)

### <a name="Rr-scoped"></a>R.5: 优先采用有作用域的对象，避免不必要的堆分配

##### 理由

有作用域的对象是局部对象、全局对象，或者成员。
它们也意味着在其所在作用域或者所在对象之外无须花费单独的分配和回收成本。
有作用域对象的成员自身也是有作用域的，有作用域对象的构造函数和析构函数负责管理其成员的生存期。

##### 示例

下面的例子效率不佳（因为无必要的分配和回收），在 `...` 中抛出异常和返回也是脆弱的（导致发生泄漏），而且也比较啰嗦：

```cpp
void f(int n)
{
    auto p = new Gadget{n};
    // ...
    delete p;
}
```

可以用局部变量来代替它：

```cpp
void f(int n)
{
    Gadget g{n};
    // ...
}
```

##### 强制实施

* 【中级】 如果分配了某个对象，又在函数内的所有路径中都进行了回收，则给出警告。建议它应当被代之以一个局部的栈对象。
* 【简单】 当局部的 `Unique_pointer` 或 `Shared_pointer` 在其生存期结束前未被移动、复制、赋值或 `reset`，则给出警告。
例外：不对指向无界数组的局部 `Unique_pointer` 产生这种警告。（见下文）

##### 例外

创建指向堆分配缓冲区的局部 `const unique_ptr<T[]>` 是没问题的，这是一种有效的表现有作用域的动态数组的方法。

##### 示例

一个局部 `const unique_ptr<T[]>` 变量的有效用例：

```cpp
int get_median_value(const std::list<int>& integers)
{
  const auto size = integers.size();

  // OK: declaring a local unique_ptr<T[]>.
  const auto local_buffer = std::make_unique_for_overwrite<int[]>(size);

  std::copy_n(begin(integers), size, local_buffer.get());
  std::nth_element(local_buffer.get(), local_buffer.get() + size/2, local_buffer.get() + size);

  return local_buffer[size/2];
}
```

### <a name="Rr-global"></a>R.6: 避免非 `const` 的全局变量

参见 [I.2](S-interfaces.md#Ri-global)

## <a name="SS-alloc"></a>R.alloc: 分配与回收

### <a name="Rr-mallocfree"></a>R.10: 避免 `malloc()` 和 `free()`

##### 理由

`malloc()` 和 `free()` 并不支持构造和销毁，而且无法同 `new` 和 `delete` 之间进行混用。

##### 示例

```cpp
class Record {
    int id;
    string name;
    // ...
};

void use()
{
    // p1 可能是 nullptr
    // *p1 并未初始化；尤其是，
    // 其中的 string 也还不是一个 string，而是一片和 string 大小相同的字节而已
    Record* p1 = static_cast<Record*>(malloc(sizeof(Record)));

    auto p2 = new Record;

    // 如果没有抛出异常的话，*p2 就经过了默认初始化
    auto p3 = new(nothrow) Record;
    // p3 可能为 nullptr；否则 *p3 就经过了默认初始化

    // ...

    delete p1;    // 错误: 不能 delete 由 malloc() 分配的对象
    free(p2);    // 错误: 不能 free() 由 new 分配的对象
}
```

在某些实现中，`delete` 和 `free()` 可能可以工作，或者也可能引发运行时的错误。

##### 例外

有些应用程序或者代码段是不能接受异常的。
它们的最佳例子就是那些性命攸关的硬实时代码。
但要注意的是，许多对异常的禁止其实是基于某种（不良的）迷信，
或者来源于对没有进行系统性的资源管理的老式代码库的关注（很不幸，但这经常是必须的）。
在这些情况下，应当考虑使用 `nothrow` 版本的 `new`。

##### 强制实施

对 `malloc` 和 `free` 的使用进行标记。

### <a name="Rr-newdelete"></a>R.11: 避免显式调用 `new` 和 `delete`

##### 理由

由 `new` 所返回的指针应当属于一个资源句柄（它将调用 `delete`）。
若由 `new` 所返回的指针被赋值给普通的裸指针，那么这个对象就可能会泄漏。

##### 注解

大型程序中，裸露的 `delete`（即出现于应用程序代码中，而不是专门进行资源管理的代码中）
很可能是一个 BUG：如果已经有了 N 个 `delete` 的话，怎么确定我们需要的不是 N+1 或者 N-1 个呢？
这种 BUG 可能会潜伏起来：它也许只会在维护过程中才暴露出来。
如果出现了裸露的 `new`，那就可能在别的什么地方需要一个裸露的 `delete`，因而可能也是一个 BUG。

##### 强制实施

【简单】 对任何 `new` 和 `delete` 的显式使用都给出警告。建议代之以 `make_unique`。

### <a name="Rr-immediate-alloc"></a>R.12: 显式资源分配的结果应当立即交给一个管理对象

##### 理由

如果不这样做的话，当发生异常或者返回时就可能造成泄露。

##### 示例，不好

```cpp
void func(const string& name)
{
    FILE* f = fopen(name, "r");            // 打开文件
    vector<char> buf(1024);
    auto _ = finally([f] { fclose(f); });  // 记得要关闭文件
    // ...
}
```

`buf` 的分配可能会失败，并导致文件句柄的泄漏。

##### 示例

```cpp
void func(const string& name)
{
    ifstream f{name};   // 打开文件
    vector<char> buf(1024);
    // ...
}
```

对文件句柄（在 `ifstream` 中）的使用是简单、高效而且安全的。

##### 强制实施

* 将用于初始化指针的显式分配标记出来。（问题：我们能识别出多少直接资源分配呢？）

### <a name="Rr-single-alloc"></a>R.13: 单个表达式语句中至多进行一次显式资源分配

##### 理由

如果在一条语句中进行两次显式资源分配的话就可能发生资源泄漏，这是因为许多的子表达式（包括函数参数）的求值顺序都是未指明的。

##### 示例

```cpp
void fun(shared_ptr<Widget> sp1, shared_ptr<Widget> sp2);
```

可以这样调用 `fun`：

```cpp
// 不好：可能会泄漏
fun(shared_ptr<Widget>(new Widget(a, b)), shared_ptr<Widget>(new Widget(c, d)));
```

这是异常不安全的，因为编译器可能会把两个用以创建函数的两个参数的表达式重新排序。
特别是，编译器是可以交错执行这两个表达式的：
它可能会首先为两个对象都（通过调用 `operator new`）进行内存分配，然后再试图调用二者的 `Widget` 构造函数。
一旦其中一个构造函数调用抛出了异常，那么另一个对象的内存可能永远不会被释放了！

这个微妙的问题其实有一种简单的解决方案：永远不要在一条表达式语句中进行多于一次的显式资源分配。
例如：

```cpp
shared_ptr<Widget> sp1(new Widget(a, b)); // 好多了，但不太干净
fun(sp1, new Widget(c, d));
```

最佳的方案是使用返回所有者对象的工厂函数，而完全避免显式的分配：

```cpp
fun(make_shared<Widget>(a, b), make_shared<Widget>(c, d)); // 最佳
```

如果还没有，请自己编写一个工厂包装。

##### 强制实施

* 将包含多次显式资源分配的表达式标记出来。（问题：我们能识别出多少直接资源分配呢？）

### <a name="Rr-ap"></a>R.14: 避免使用 `[]` 形参，优先使用 `span`

##### 理由

数组会退化为指针，因而丢失其大小信息，并留下了发生范围错误的机会。
使用 `span` 来保留大小信息。

##### 示例

```cpp
void f(int[]);          // 不建议的做法

void f(int*);           // 对多个对象不建议的做法
                        // （指针应当指向单个对象，不要进行下标运算）

void f(gsl::span<int>); // 好，建议的做法
```

##### 强制实施

标记出 `[]` 参数。代之以使用 `span`。

### <a name="Rr-pair"></a>R.15: 总是同时重载相匹配的分配、回收函数对

##### 理由

不然的话就出现不匹配的操作，并导致混乱。

##### 示例

```cpp
class X {
    // ...
    void* operator new(size_t s);
    void operator delete(void*);
    // ...
};
```

##### 注解

如果想要无法进行回收的内存的话，可以将回收操作 `=delete`。
请勿留下它而不进行声明。

##### 强制实施

标记出不完全的操作对。

## <a name="SS-smart"></a>R.smart: 智能指针

### <a name="Rr-owner"></a>R.20: 用 `unique_ptr` 或 `shared_ptr` 表示所有权

##### 理由

它们可以避免资源泄漏。

##### 示例

考虑：

```cpp
void f()
{
    X* p1 { new X };              // 不好，p1 会泄漏
    auto p4 = make_unique<X>();   // 好，唯一所有权
    auto p5 = make_shared<X>();   // 好，共享所有权
}
```

这里（只有）初始化 `p1` 的对象将会泄漏。

##### 强制实施

【简单】 如果 `new` 的返回值被赋值给了原生指针，就给出警告。
【简单】 如果返回带所有权原始指针的函数的结果被赋值给了原生指针，就给出警告。

### <a name="Rr-unique"></a>R.21: 优先采用 `unique_ptr` 而不是 `shared_ptr`，除非需要共享所有权

##### 理由

`unique_ptr` 概念上要更简单且更可预测（知道它何时会销毁），而且更快（不需要暗中维护引用计数）。

##### 示例，不好

这里并不需要维护一个引用计数。

```cpp
void f()
{
    shared_ptr<Base> base = make_shared<Derived>();
    // 局部范围中使用 base，并未进行复制——引用计数不会超过 1
} // 销毁 base
```

##### 示例

这样更加高效：

```cpp
void f()
{
    unique_ptr<Base> base = make_unique<Derived>();
    // 局部范围中使用 base
} // 销毁 base
```

##### 强制实施

【简单】 如果函数所使用的 `Shared_pointer` 的对象是函数之内所分配的，而且既不会将这个 `Shared_pointer` 返回，也不会将其传递给其他接受 `Shared_pointer&` 的函数的话，就给出警告。建议代之以 `unique_ptr`。

### <a name="Rr-make_shared"></a>R.22: 使用 `make_shared()` 创建 `shared_ptr`

##### 理由

`make_shared` 为构造提供了更精炼的语句。
它也提供了一个机会，通过把 `shared_ptr` 的使用计数和对象相邻放置，来消除为引用计数进行独立的内存分配操作。

##### 示例

考虑：

```cpp
shared_ptr<X> p1 { new X{2} }; // 不好
auto p = make_shared<X>(2);    // 好
```

`make_shared()` 版本仅提到一次 `X`，因而它通常比显式的 `new` 方式要更简短（而且更快）。

##### 强制实施

【简单】 如果 `shared_ptr` 从 `new` 的结果而不是 `make_shared` 进行构造，就给出警告。

### <a name="Rr-make_unique"></a>R.23: 使用 `make_unique()` 创建 `unique_ptr`

##### 理由

`make_unique` 为构造提供了更精炼的语句。
它也保证了复杂表达式中的异常安全性。

##### 示例

```cpp
unique_ptr<Foo> p {new Foo{7}};    // OK: 不过有重复

auto q = make_unique<Foo>(7);      // 有改善: 并未重复 Foo
```

##### 强制实施

【简单】 如果 `unique_ptr` 从 `new` 的结果而不是 `make_unique` 进行构造，就给出警告。

### <a name="Rr-weak_ptr"></a>R.24: 使用 `std::weak_ptr` 来打断 `shared_ptr` 的循环引用

##### 理由

`shared_ptr` 是基于引用计数的，而带有循环的结构中的引用计数不可能变为零，因此我们需要一种机制
来打破带有循环的结构。

##### 示例

```cpp
#include <memory>

class bar;

class foo {
public:
  explicit foo(const std::shared_ptr<bar>& forward_reference)
    : forward_reference_(forward_reference)
  { }
private:
  std::shared_ptr<bar> forward_reference_;
};

class bar {
public:
  explicit bar(const std::weak_ptr<foo>& back_reference)
    : back_reference_(back_reference)
  { }
  void do_something()
  {
    if (auto shared_back_reference = back_reference_.lock()) {
      // 使用 *shared_back_reference
    }
  }
private:
  std::weak_ptr<foo> back_reference_;
};
```

##### 注解

??? (HS: 许多人都说要“打破循环引用”，不过我觉得“临时性共享所有权”可能更关键。)
???(BS: 打破循环引用是必须达成的目标；临时性共享所有权则是达成的方式。
你也可以仅仅使用另一个 `shared_ptr` 来得到“临时性共享所有权”。)

##### 强制实施

??? 可能无法做到。如果能够静态地检测出循环引用的话，我们就不需要 `weak_ptr` 了。

### <a name="Rr-smartptrparam"></a>R.30: 以智能指针为参数，仅用于明确表达生存期语义

参见 [F.7](S-functions.md#Rf-smart)。

### <a name="Rr-smart"></a>R.31: 非 `std` 的智能指针，应当遵循 `std` 的行为模式

##### 理由

下面段落中的规则同样适用于第三方和自定义的其他种类的智能指针，而且对于诊断引发导致了性能和正确性问题的一般性的智能指针错误来说也是非常有帮助的。
你将会期望你所使用的所有智能指针都遵循这些规则。

任何重载了一元 `*` 和 `->` 的类型（无论主模板还是特化）都被当成是智能指针：

* 如果它可以复制，则将其当做一种具有引用计数的 `Shared_ptr`。
* 如果它不能复制，则将其当做一种唯一的 `Unique_ptr`。

##### 示例，不好

```cpp
// 使用 Boost 的 intrusive_ptr
#include <boost/intrusive_ptr.hpp>
void f(boost::intrusive_ptr<widget> p)  // 根据 'sharedptrparam' 规则是错误的
{
    p->foo();
}

// 使用 Microsoft 的 CComPtr
#include <atlbase.h>
void f(CComPtr<widget> p)               // 根据 'sharedptrparam' 规则是错误的
{
    p->foo();
}
```

上面两段根据 [`sharedptrparam` 指导方针](S-resource.md#Rr-smartptrparam)来说都是错误的：
`p` 是一个 `Shared_pointer`，但其共享性质完全没有被用到，而对其进行按值传递则是一种暗含的劣化；
这两个函数应当仅当它们需要参与 `widget` 的生存期管理时才接受智能指针。否则当可以为 `nullptr` 时它们就应当接受 `widget*`，否则，理想情况下，函数应当接受 `widget&`。
这些智能指针都符合 `Shared_pointer` 的概念，因此这些强制实施指导方针的规则可以直接应用，并使得这种一般性的劣化情况暴露出来。

### <a name="Rr-uniqueptrparam"></a>R.32: `unique_ptr<widget>` 参数用以表达函数假定获得 `widget` 的所有权

##### 理由

以这种方式使用 `unique_ptr` 同时说明并强制施加了函数调用时的所有权转移。

##### 示例

```cpp
void sink(unique_ptr<widget>); // 获得这个 widget 的所有权

void uses(widget*);            // 仅仅使用了这个 widget
```

##### 示例，不好

```cpp
void thinko(const unique_ptr<widget>&); // 通常不是你想要的
```

##### 强制实施

* 【简单】 如果函数以左值引用接受 `Unique_pointer<T>` 参数，但并未在至少一个代码路径中向其赋值或者对其调用 `reset()`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数以 `const` 引用接受 `Unique_pointer<T>` 参数，则给出警告。建议代之以接受 `const T*` 或 `const T&`。

### <a name="Rr-reseat"></a>R.33: `unique_ptr<widget>&` 参数用以表达函数对该 `widget` 重新置位

##### 示例

以这种方式使用 `unique_ptr` 同时说明并强制施加了函数调用时的重新置位语义。

##### 注解

所谓“重新置位（Reseat）”的含义是“让指针或智能指针指代某个不同的对象”。

##### 示例

```cpp
void reseat(unique_ptr<widget>&); // “将要”或“可能”重新置位指针
```

##### 示例，不好

```cpp
void thinko(const unique_ptr<widget>&); // 通常不是你想要的
```

##### 强制实施

* 【简单】 如果函数以左值引用接受 `Unique_pointer<T>` 参数，但并未在至少一个代码路径中向其赋值或者对其调用 `reset()`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数以 `const` 引用接受 `Unique_pointer<T>` 参数，则给出警告。建议代之以接受 `const T*` 或 `const T&`。

### <a name="Rr-sharedptrparam-owner"></a>R.34: 用 `shared_ptr<widget>` 参数表达共享所有权

##### 理由

这样做明确了函数的所有权共享语义。

##### 示例，好

```cpp
class WidgetUser
{
public:
    // WidgetUser 将会共享这个 widget 的所有权
    explicit WidgetUser(std::shared_ptr<widget> w) noexcept:
        m_widget{std::move(w)} {}
    // ...
private:
    std::shared_ptr<widget> m_widget;
};
```

##### 强制实施

* 【简单】 如果函数以左值引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中向其赋值或者对其调用 `reset()`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数按值或者以 `const` 引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中将其复制或移动给另一个 `Shared_pointer`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数以右值引用接受 `Shared_pointer<T>` 参数，则给出警告。建议代之以按值传递。

### <a name="Rr-sharedptrparam"></a>R.35: `shared_ptr<widget>&` 参数用以表达函数可能会对共享的指针重新置位

##### 理由

这样做明确了函数的重新置位语义。

##### 注解

所谓“重新置位（Reseat）”的含义是“让引用或智能指针指代某个不同的对象”。

##### 示例，好

```cpp
void ChangeWidget(std::shared_ptr<widget>& w)
{
    // 这将会改变调用方的 widget
    w = std::make_shared<widget>(widget{});
}
```

##### 强制实施

* 【简单】 如果函数以左值引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中向其赋值或者对其调用 `reset()`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数按值或者以 `const` 引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中将其复制或移动给另一个 `Shared_pointer`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数以右值引用接受 `Shared_pointer<T>` 参数，则给出警告。建议代之以按值传递。

### <a name="Rr-sharedptrparam-const"></a>R.36: `const shared_ptr<widget>&` 参数用以表达它可能将保留一个对对象的引用 ???

##### 理由

这样做明确了函数的 ??? 语义。

##### 示例，好

```cpp
void share(shared_ptr<widget>);            // 共享——“将会”保持一个引用计数

void reseat(shared_ptr<widget>&);          // “可能”重新置位指针

void may_share(const shared_ptr<widget>&); // “可能”保持一个引用计数
```

##### 强制实施

* 【简单】 如果函数以左值引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中向其赋值或者对其调用 `reset()`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数按值或者以 `const` 引用接受 `Shared_pointer<T>` 参数，但并未在至少一个代码路径中将其复制或移动给另一个 `Shared_pointer`，则给出警告。建议代之以接受 `T*` 或 `T&`。
* 【简单】〔基础〕 如果函数以右值引用接受 `Shared_pointer<T>` 参数，则给出警告。建议代之以按值传递。

### <a name="Rr-smartptrget"></a>R.37: 不要把来自某个智能指针别名的指针或引用传递出去

##### 理由

违反这条规则，是导致引用计数的丢失和出现悬挂指针的首要原因。
函数应当优先向其调用链中传递原生指针和引用。
在调用树的顶层，原生指针或引用是从用以保持对象存活的智能指针中获得的。
我们需要确保这个智能指针不会在调用树的下面被疏忽地进行重置或者重新赋值。

##### 注解

为了做到这点，有时候需要获得智能指针的一个局部副本，它可以确保在函数及其调用树的执行期间维持对象存活。

##### 示例

考虑下面的代码：

```cpp
// 全局（静态或堆）对象，或者有别名的局部对象 ...
shared_ptr<widget> g_p = ...;

void f(widget& w)
{
    g();
    use(w);  // A
}

void g()
{
    g_p = ...; // 噢，如果这就是这个 widget 的最后一个 shared_ptr 的话，这会销毁这个 widget
}
```

下面的代码是不应该通过代码评审的：

```cpp
void my_code()
{
    // 不好: 传递的是从非局部的智能指针中获得的指针或引用
    //       而这可能会在 f 或其调用的函数中的某处被不经意地重置掉
    f(*g_p);

    // 不好: 原因相同，只不过将其作为“this”指针传递
    g_p->func();
}
```

修正很简单——获取该指针的一个局部副本，为调用树“保持一个引用计数”：

```cpp
void my_code()
{
    // 很廉价: 一次增量就搞定了整个函数以及下面的所有调用树
    auto pin = g_p;

    // 好: 传递的是从局部的无别名智能指针中获得的指针或引用
    f(*pin);

    // 好: 原因相同
    pin->func();
}
```

##### 强制实施

* 【简单】 如果从非局部或局部但潜在具有别名的智能指针变量（`Unique_pointer` 或 `Shared_pointer`）中所获取的指针或引用，被用于进行函数调用，则给出警告。如果智能指针是一个 `Shared_pointer`，则建议代之以获取该智能指针的一个局部副本并从中获取指针或引用。

