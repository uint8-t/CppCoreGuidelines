# <a name="S-performance"></a>Per: 性能

??? 这一节应该放在主指南中吗 ???

本章节所包含的规则对于需要高性能和低延迟的人们很有价值。
就是说，这些规则是有关如何在可预测的短时段中尽可能使用更短时间和更少资源来完成任务的。
本章节中的规则要比（绝大）多数应用程序所需要的规则更多限制并更有侵入性。
请勿在一般的代码中盲目地尝试遵循这些规则：要达成低延迟的目标是需要进行一些额外工作的。

性能规则概览：

* [Per.1: 请勿进行无理由的优化](S-performance.md#Rper-reason)
* [Per.2: 请勿进行不成熟的优化](S-performance.md#Rper-Knuth)
* [Per.3: 请勿对非性能关键的代码进行优化](S-performance.md#Rper-critical)
* [Per.4: 不能假定复杂代码一定比简单代码更快](S-performance.md#Rper-simple)
* [Per.5: 不能假定低级代码一定比高级代码更快](S-performance.md#Rper-low)
* [Per.6: 请勿不进行测量就作出性能评断](S-performance.md#Rper-measure)
* [Per.7: 设计应当允许优化](S-performance.md#Rper-efficiency)
* [Per.10: 依赖静态类型系统](S-performance.md#Rper-type)
* [Per.11: 把计算从运行时转移到编译期](S-performance.md#Rper-Comp)
* [Per.12: 消除多余的别名](S-performance.md#Rper-alias)
* [Per.13: 消除多余的间接](S-performance.md#Rper-indirect)
* [Per.14: 最小化分配和回收的次数](S-performance.md#Rper-alloc)
* [Per.15: 请勿在关键逻辑分支中进行分配](S-performance.md#Rper-alloc0)
* [Per.16: 使用紧凑的数据结构](S-performance.md#Rper-compact)
* [Per.17: 在时间关键的结构中应当先声明最常用的成员](S-performance.md#Rper-struct)
* [Per.18: 空间即时间](S-performance.md#Rper-space)
* [Per.19: 进行可预测的内存访问](S-performance.md#Rper-access)
* [Per.30: 避免在关键路径中进行上下文切换](S-performance.md#Rper-context)

### <a name="Rper-reason"></a>Per.1: 请勿进行无理由的优化

##### 理由

如果没有必要优化的话，这样做的结果就是更多的错误和更高的维护成本。

##### 注解

一些人作出优化只是出于习惯或者因为感觉这很有趣。

???

### <a name="Rper-Knuth"></a>Per.2: 请勿进行不成熟的优化

##### 理由

经过精心优化的代码通常比未优化的代码更大而且更难修改。

???

### <a name="Rper-critical"></a>Per.3: 请勿对非性能关键的代码进行优化

##### 理由

对程序中并非性能关键的部分进行的优化，对于系统性能是没有效果的。

##### 注解

如果你的程序要耗费大量时间来等待 Web 或人的操作的话，对内存中的计算进行优化可能是没什么用处的。

换个角度来说：如果你的程序花费处理时间的 4% 来
计算 A 而花费 40% 的时间来计算 B，那对 A 的 50% 的改进
其影响只能和 B 的 5% 的改进相比。（如果你甚至不知道
A 或 B 到底花费了多少时间，参见 <a href="#Rper-reason">Per.1</a> 和 <a
href="#Rper-Knuth">Per.2</a>。）

### <a name="Rper-simple"></a>Per.4: 不能假定复杂代码一定比简单代码更快

##### 理由

简单的代码可能会非常快。优化器在简单代码上有时候会发生奇迹。

##### 示例，好

```cpp
// 清晰表达意图，快速执行

vector<uint8_t> v(100000);

for (auto& c : v)
    c = ~c;
```

##### 示例，不好

```cpp
// 试图更快，但通常会更慢

vector<uint8_t> v(100000);

for (size_t i = 0; i < v.size(); i += sizeof(uint64_t)) {
    uint64_t& quad_word = *reinterpret_cast<uint64_t*>(&v[i]);
    quad_word = ~quad_word;
}
```

##### 注解

???

???

### <a name="Rper-low"></a>Per.5: 不能假定低级代码一定比高级代码更快

##### 理由

低级代码有时候会妨碍优化。优化器在高级代码上有时候会发生奇迹。

##### 注解

???

???

### <a name="Rper-measure"></a>Per.6: 请勿不进行测量就作出性能评断

##### 理由

性能领域充斥各种错误认识和伪习俗。
现代的硬件和优化器并不遵循这些幼稚的假设；即便是专家也会经常感觉意外。

##### 注解

要进行高质量的性能测量是很难的，而且需要采用专门的工具。

##### 注解

有些使用了 Unix 的 `time` 或者标准库的 `<chrono>` 的简单的微基准测量，有助于打破大多数明显的错误认识。
如果确实无法精确地测量完整系统的话，至少也要尝试对一些关键操作和算法进行测量。
性能剖析可以帮你发现系统的哪些部分是性能关键的。
你很可能会感觉意外。

???

### <a name="Rper-efficiency"></a>Per.7: 设计应当允许优化

##### 理由

因为我们经常需要对最初的设计进行优化。
因为忽略后续改进的可能性的设计是很难修改的。

##### 示例

来自 C（以及 C++）的标准：

```cpp
void qsort (void* base, size_t num, size_t size, int (*compar)(const void*, const void*));
```

什么情况会需要对内存进行排序呢？
时即上，我们需要对元素序列进行排序，通常它们存储于容器之中。
对 `qsort` 的调用抛弃了许多有用的信息（比如元素的类型），强制用户对其已知的信息
进行重复（比如元素的大小），并强制用户编写额外的代码（比如用于比较 `double` 的函数）。
这蕴含了程序员工作量的增加，易错，并剥夺了编译器为优化所需的信息。

```cpp
double data[100];
// ... 填充 a ...

// 对从地址 data 开始的 100 块 sizeof(double) 大小
// 的内存，用由 compare_doubles 所定义的顺序进行排序
qsort(data, 100, sizeof(double), compare_doubles);
```

从接口设计的观点来看，`qsort` 抛弃了有用的信息。

这样做可以更好（C++98）：

```cpp
template<typename Iter>
    void sort(Iter b, Iter e);  // sort [b:e)

sort(data, data + 100);
```

这里，我们利用了编译器关于数组大小，元素类型，以及如何对 `double` 进行比较的知识。

而以 C++20 的话，我们还可以做得更好：

```cpp
// sortable 指定了 c 必须是一个
// 可以用 < 进行比较的元素的随机访问序列
void sort(sortable auto& c);

sort(c);
```

其中的关键在于传递充分的信息以便能够选择一个好的实现。
这里给出的几个 `sort` 接口仍然有一个缺憾：
它们隐含地依赖于元素类型定义了小于（`<`）运算符。
为使接口完整，我们需要另一个接受比较准则的版本：

```cpp
// 用 r 比较 c 的元素
template<random_access_range R, class C> requires sortable<R, C>
void sort(R&& r, C c);
```

`sort` 的标准库规范提供了这两个版本和其他版本。

##### 注解

不成熟的优化被称为[一切罪恶之源](S-performance.md#Rper-Knuth)，但这并不是轻视性能的理由。
考虑如何使设计可以为改进而进行修正绝不是不成熟的，而性能改进则是一种常见的改进要求。
我们的目标是建立一组习惯，使缺省情况就能得到高效，可维护，且可优化的代码。
特别是，当你编写并非一次性实现细节的函数时，应当考虑

* 信息传递：
优先采用能够为后续的实现改进带来充分信息的简洁[接口](S-interfaces.md#S-interfaces)。
要注意信息会通过我们所提供的接口来流入和流出一个实现。
* 紧凑的数据：默认情况[使用紧凑的数据](S-performance.md#Rper-compact)，比如 `std::vector`，并[系统化地进行访问](S-performance.md#Rper-access)。
如果你觉得需要一种有链接的结构的话，应尝试构造接口使这个结构不会被用户所看到。
* 函数的参数传递和返回：
对可改变和不可变的数据加以区分。
不要把资源管理的负担强加给用户。
不要把假性的间接强加给用户。
对通过接口传递信息采用[符合惯例的方式](S-functions.md#Rf-conventional)；
不合惯例的，以及“优化过的”数据传递方式可能会严重影响后续的重新实现。
* 抽象：
不要过度泛化；视图提供每一种可能用法（和误用），并把每个设计决策都（通过编译时或运行时间接）
推迟到后面处理的设计，通常是复杂的，膨胀的，难于理解的混乱体。
应当从具体的例子进行泛化，泛化时要保持性能。
不要仅仅基于关于未来需求的推测而进行泛化。
理想情况是零开销泛化。
* 程序库：
使用带有良好接口的程序库。
当没有可用的程序库时，就构建自己的，并模仿一个好程序库的接口风格。
[标准库](#???)是寻找模仿的一个好的第一来源。
* 隔离：
通过将你所选择的接口提供给你的代码来将其和杂乱和老旧风格的代码之间进行隔离。
这有时候称为为有用或必须但杂乱的代码“提供包装”。
不要让不良设计“渗入”你的代码中。

##### 示例

考虑：

```cpp
template<class ForwardIterator, class T>
bool binary_search(ForwardIterator first, ForwardIterator last, const T& val);
```

`binary_search(begin(c), end(c), 7)` 能够得出 `7` 是否在 `c` 之中。
不过，它无法得出 `7` 在何处，或者是否有多于一个 `7`。

有时候仅把最小数量的信息传递回来（如这里的 `true` 或 `false`）是足够的，但一个好的接口会
向调用方传递其所需的信息。因此，标准库还提供了

```cpp
template<class ForwardIterator, class T>
ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T& val);
```

`lower_bound` 返回第一个匹配元素（如果有）的迭代器，否则返回第一个大于 `val` 的元素的迭代器，找不到这样的元素时，返回 `last`。

不过 `lower_bound` 还是无法为所有用法返回足够的信息，因此标准库还提供了

```cpp
template<class ForwardIterator, class T>
pair<ForwardIterator, ForwardIterator>
equal_range(ForwardIterator first, ForwardIterator last, const T& val);
```

`equal_range` 返回迭代器的 `pair`，指定匹配的第一个和最后一个之后的元素。

```cpp
auto r = equal_range(begin(c), end(c), 7);
for (auto p = r.first; p != r.second; ++p)
    cout << *p << '\n';
```

显然，这三个接口都是以相同的基本代码实现的。
它们不过是将基本的二叉搜索算法表现给用户的三种方式，
包含从最简单（“让简单的事情简单！”）
到返回完整但不总是必要的信息（“不要隐藏有用的信息”）。
自然，构造这样一组接口是需要经验和领域知识的。

##### 注解

接口的构建不要仅匹配你能想到的第一种实现和第一种用例。
一旦第一个初始实现完成后，应当进行复审；一旦它被部署出去，就很难对错误进行补救了。

##### 注解

对效率的需求并不意味着对[底层代码](S-performance.md#Rper-low)的需求。
高层代码并不必然缓慢或膨胀。

##### 注解

事物都是有成本的。
不要对成本过于偏执（当代计算机真的非常快），
但需要对你所使用的东西的成本的数量级有大致的概念。
例如，应当对
一次内存访问，
一次函数调用，
一次字符串比较，
一次系统调用，
一次磁盘访问，
以及一个通过网络的消息的成本有大致的概念。

##### 注解

如果你只能想到一种实现的话，可能你并没有某种能够设计一个稳定接口的东西。
有可能它只不过是实现细节——不是每段代码都需要一个稳定接口——停下来想一想。
一个有用的问题是
“如果这个操作需要用多个线程实现的话，需要什么样的接口呢？向量化？”

##### 注解

这条规则并不抵触[不要进行不成熟的优化](S-performance.md#Rper-Knuth)规则。
它对其进行了补充，鼓励程序员在有必要的时候，使后续的——适当并且成熟的——优化能够进行。

##### 强制实施

很麻烦。
也许查找 `void*` 函数参数能够找到妨碍后续优化的接口的例子。

### <a name="Rper-type"></a>Per.10: 依赖静态类型系统

##### 理由

类型违规，弱类型（比如 `void*`），以及低级代码（比如把序列当作独立字节进行操作）等会让优化器的工作变得困难很多。简单的代码通常比手工打造的复杂代码能够更好地优化。

???

### <a name="Rper-Comp"></a>Per.11: 把计算从运行时转移到编译期

##### 理由

减少代码大小和运行时间。
通过使用常量来避免数据竞争。
编译时捕获错误（并因而消除错误处理代码）。

##### 示例

```cpp
double square(double d) { return d*d; }
static double s2 = square(2);    // 旧式代码：动态初始化

constexpr double ntimes(double d, int n)   // 假定 0 <= n
{
        double m = 1;
        while (n--) m *= d;
        return m;
}
constexpr double s3 {ntimes(2, 3)};  // 现代代码：编译期初始化
```

像 `s2` 的初始化这样的代码并不少见，尤其是比 `square()` 更复杂一些的初始化更是如此。
不过，与 `s3` 的初始化相比，它有两个问题：

* 我们得忍受运行时的一次函数调用的开销
* `s2` 在初始化开始前可能就被某个别的线程访问了。

注意：常量是不可能发生数据竞争的。

##### 示例

考虑一种流行的提供包装类的技术，在包装类自身之中存储小型对象，而把大型对象保存到堆上。

```cpp
constexpr int on_stack_max = 20;

template<typename T>
struct Scoped {     // 在 Scoped 中存储一个 T
        // ...
    T obj;
};

template<typename T>
struct On_heap {    // 在自由存储中存储一个 T
        // ...
        T* objp;
};

template<typename T>
using Handle = typename std::conditional<(sizeof(T) <= on_stack_max),
                    Scoped<T>,      // 第一种候选
                    On_heap<T>      // 第二种候选
               >::type;

void f()
{
    Handle<double> v1;                   // double 在栈中
    Handle<std::array<double, 200>> v2;  // array 保存到自由存储里
    // ...
}
```

假定 `Scoped` 和 `On_heap` 均提供了兼容的用户接口。
这里我们在编译时计算出了最优的类型。
对于选择所要调用的最优函数，也有类似的技术。

##### 注解

理想情况是，*不*试图在编译期执行所有的代码。
显然，大多数的运算都依赖于输入，因而它们没办法挪到编译期进行，
而除了这种逻辑限制外，实际情况是，复杂的编译期运算会严重增加编译时间，
并使调试变得复杂。
甚至编译期运算也可能使得代码变慢。
这种情况确实罕见，但当把一种通用运算分解为一组优化的子运算时，可能会导致指令高速缓存的效率变差。

##### 强制实施

* 找出可以（但尚不）是 constexpr 的简单函数。
* 找出调用时其全部实参均为常量表达式的函数。
* 找出可以为 constexpr 的宏。

### <a name="Rper-alias"></a>Per.12: 消除多余的别名

???

### <a name="Rper-indirect"></a>Per.13: 消除多余的间接

???

### <a name="Rper-alloc"></a>Per.14: 最小化分配和回收的次数

???

### <a name="Rper-alloc0"></a>Per.15: 请勿在关键逻辑分支中进行分配

???

### <a name="Rper-compact"></a>Per.16: 使用紧凑的数据结构

##### 理由

性能通常都是由内存访问次数所决定的。

???

### <a name="Rper-struct"></a>Per.17: 在时间关键的结构中应当先声明最常用的成员

???

### <a name="Rper-space"></a>Per.18: 空间即时间

##### 理由

性能通常都是由内存访问次数所决定的。

???

### <a name="Rper-access"></a>Per.19: 进行可预测的内存访问

##### 理由

性能对于 Cache 的性能非常敏感，而 Cache 算法则更喜欢对相邻数据进行的（通常是线性的）简单访问行为。

##### 示例

```cpp
int matrix[rows][cols];

// 不好
for (int c = 0; c < cols; ++c)
    for (int r = 0; r < rows; ++r)
        sum += matrix[r][c];

// 好
for (int r = 0; r < rows; ++r)
    for (int c = 0; c < cols; ++c)
        sum += matrix[r][c];
```

### <a name="Rper-context"></a>Per.30: 避免在关键路径中进行上下文切换

???

