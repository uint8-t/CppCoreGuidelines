# <a name="S-stdlib"></a>SL: 标准库

如果只使用纯语言本身的话，任何开发任务都会变得很麻烦（无论以何种语言）。
如果使用了某个合适的程序库的话，则任何开发任务都会变得相当简单。

这些年来标准库一直在持续增长。
现在它在标准中的描述已经比语言功能特性的描述更大了。
因此，可能指导方针的库部分的规模最终将会增长等于甚至超过其他的所有部分。

<< ??? 我们需要另一个层次的规则编号 ??? >>

C++ 标准库组件概览：

* [SL.con: 容器](S-stdlib.md#SS-con)
* [SL.str: 字符串](S-stdlib.md#SS-string)
* [SL.io: I/O 流（iostream）](S-stdlib.md#SS-io)
* [SL.regex: 正则表达式](S-stdlib.md#SS-regex)
* [SL.chrono: 时间](S-stdlib.md#SS-chrono)
* [SL.C: C 标准库](S-stdlib.md#SS-clib)

标准库规则概览：

* [SL.1: 尽可能使用程序库](S-stdlib.md#Rsl-lib)
* [SL.2: 优先使用标准库而不是其他程序库](S-stdlib.md#Rsl-sl)
* [SL.3: 请勿向命名空间 `std` 中添加非标准实体](S-stdlib.md#sl-std)
* [SL.4: 以类型安全的方式使用标准库](S-stdlib.md#sl-safe)
* ???

### <a name="Rsl-lib"></a>SL.1:  尽可能使用程序库

##### 理由

节约时间。避免重复发明轮子。
避免重复他人的工作。
如果其他人的工作有了改进，则可以从中获得好处。
当你进行了改进之后可以帮助其他人。

### <a name="Rsl-sl"></a>SL.2: 优先使用标准库而不是其他程序库

##### 理由

了解标准库的人更多。
相对于你自己的代码或者大多数其他程序库来说，标准库更加倾向于稳定，进行了良好维护，而且广泛可用。


### <a name="sl-std"></a>SL.3: 请勿向命名空间 `std` 中添加非标准实体

##### 理由

向 `std` 中添加东西可能会改变本来是遵循标准的代码的含义。
添加到 `std` 的东西可能会与未来版本的标准产生冲突。

##### 示例

```cpp
namespace std { // 不好：违反标准

class My_vector {
    //     . . .
};

}

namespace Foo { // 好：允许使用用户命名空间

class My_vector {
    //     . . .
};

}
```

##### 强制实施

有可能，但很麻烦而且在一些平台上很可能导致一些问题。

### <a name="sl-safe"></a>SL.4: 以类型安全的方式使用标准库

##### 理由

因为，很显然，违反这条规则将导致未定义的行为，内存损坏，以及其他所有种类的糟糕的错误。

##### 注解

本条规则是半哲学性的元规则，需要许多具体规则予以支持。
我们需要将之作为对于更加专门的规则的总括。

更加专门的规则概览：

* [SL.4: 以类型安全的方式使用标准库](S-stdlib.md#sl-safe)


## <a name="SS-con"></a>SL.con: 容器

???

容器规则概览：

* [SL.con.1: 优先采用 STL 的 `array` 或 `vector` 而不是 C 数组](S-stdlib.md#Rsl-arrays)
* [SL.con.2: 除非有理由使用别的容器，否则默认情况应优先采用 STL 的 `vector`](S-stdlib.md#Rsl-vector)
* [SL.con.3: 避免边界错误](S-stdlib.md#Rsl-bounds)
* [SL.con.4: 请勿对非可平凡复制的实参使用 `memset` 或 `memcpy`](S-stdlib.md#Rsl-copy)

### <a name="Rsl-arrays"></a>SL.con.1: 优先采用 STL 的 `array` 或 `vector` 而不是 C 数组

##### 理由

C 数组不那么安全，而且相对于 `array` 和 `vector` 也没有什么优势。
对于定长数组，应使用 `std::array`，它传递给函数时并不会退变为指针并丢失其大小信息。
而且，和内建数组一样，栈上分配的 `std::array` 会在栈上保存它的各个元素。
对于变长数组，应使用 `std::vector`，它还可以改变大小并处理内存分配。

##### 示例

```cpp
int v[SIZE];                        // 不好

std::array<int, SIZE> w;            // ok
```

##### 示例

```cpp
int* v = new int[initial_size];     // 不好，有所有权的原生指针
delete[] v;                         // 不好，手工 delete

std::vector<int> w(initial_size);   // ok
```

##### 注解

在不拥有而引用容器中的元素时使用 `gsl::span`。

##### 注解

在栈上分配的固定大小的数组和把元素都放在自由存储上的 `vector` 之间比较性能是没什么意义的。
你同样也可以在栈上的 `std::array` 和通过指针访问 `malloc()` 的结果之间进行这样的比较。
对于大多数代码来说，即便是栈上分配和自由存储分配之间的差异也没那么重要，但 `vector` 带来的便利和安全性却是重要的。
如果有人编写的代码中这种差异确实重要，那么他显然可以在 `array` 和 `vector` 之间做出选择。

##### 强制实施

* 如果 C 数组的声明所在的函数或类也声明了 STL 的某个容器（这是为了避免在老式的非 STL 代码中的大量警告噪音），则对其进行标记。修正：最少要把 C 数组改成 `std::array`。

### <a name="Rsl-vector"></a>SL.con.2: 除非有理由使用别的容器，否则默认情况应优先采用 STL 的 `vector`

##### 理由

`vector` 和 `array` 是仅有的能够提供以下各项优势的标准容器：

* 最快的通用访问（随机访问，还包括对于向量化友好性）；
* 最快的默认访问模式（从头到尾或从尾到头方式是对预读器友好的）；
* 最少的空间耗费（连续布局中没有每个元素的开销，而且是 cache 友好的）。

通常你都需要对容器进行元素的添加和删除，因此默认应当采用 `vector`；如果并不需要改动容器的大小的话，则应采用 `array`。

即便其他容器貌似更加合适，比如 `map` 的 O(log N) 查找性能，或者 `list` 的中部高效插入，对于几个 KB 以内大小的容器来说，`vector` 仍然经常性能更好。

##### 注解

`string` 不应当用作独立字符的容器。`string` 是文本字符串；如果需要字符的容器的话，应当采用 `vector</*char_type*/>` 或者 `array</*char_type*/>`。

##### 例外

如果你有正当的理由来使用别的容器的话，就请使用它。例如：

* 若 `vector` 满足你的需求，但你并不需要容器大小可变，则应当代之以 `array`。

* 若你需要支持字典式查找的容器并保证 O(K) 或 O(log N) 的查找效率，而且容器将会比较大（超过几个 KB），你需要经常进行插入使得维护有序的 `vector` 的开销不大可行，则请代之以使用 `unordered_map` 或者 `map`。

##### 注解

使用 `()` 初始化来将 `vector` 初始化为具有特定数量的元素。
使用 `{}` 初始化来以一个元素列表来对 `vector` 进行初始化。

```cpp
vector<int> v1(20);  // v1 具有 20 个值为 0 的元素（vector<int>{}）
vector<int> v2 {20}; // v2 具有 1 个值为 20 的元素
```

[优先采用 `{}` 初始化式语法](S-expr.md#Res-list)。

##### 强制实施

* 如果 `vector` 构造之后大小不会改变（比如因为它是 `const` 或者因为没有对它调用过非 `const` 函数），则对其进行标记。修正：代之以使用 `array`。

### <a name="Rsl-bounds"></a>SL.con.3: 避免边界错误

##### 理由

越过已分配的元素的范围进行读写，通常都会导致糟糕的错误，不正确的结果，程序崩溃，以及安全漏洞。

##### 注解

应用于一组元素的范围的标准库函数，都有（或应当有）接受 `span` 的边界安全重载。
如 `vector` 这样的标准类型，在边界剖面配置下（以某种不兼容的方式，如添加契约）可以被修改为实施边界检查，或者使用 `at()`。

理想情况下，边界内保证应当可以被静态强制实行。
例如：

* 基于范围的 `for` 的循环不会越过其所针对的容器的范围
* `v.begin(),v.end()` 可以很容易确定边界安全性

这种循环和任何的等价的无检查或不安全的循环一样高效。

通常，可以用一个简单的预先检查来消除检查每个索引的需要。
例如

* 对于 `v.begin(),v.begin()+i`，`i` 可以很容易针对 `v.size()` 检查

这种循环比每次都待检查元素访问要快得多。

##### 示例，不好

```cpp
void f()
{
    array<int, 10> a, b;
    memset(a.data(), 0, 10);         // 不好，且包含长度错误（length = 10 * sizeof(int)）
    memcmp(a.data(), b.data(), 10);  // 不好，且包含长度错误（length = 10 * sizeof(int)）
}
```

而且，`std::array<>::fill()` 或 `std::fill()`，甚或是空的初始化式，都是比 `memset()` 更好的候选。

##### 示例，好

```cpp
void f()
{
    array<int, 10> a, b, c{};       // c 被初始化为零
    a.fill(0);
    fill(b.begin(), b.end(), 0);    // std::fill()
    fill(b, 0);                     // std::ranges::fill()

    if ( a == b ) {
      // ...
    }
}
```

##### Example

如果代码使用的是未修改的标准库，仍然有一些变通方案来以边界安全的方式使用 `std::array` 和 `std::vector`。代码中可以调用各个类的 `.at()` 成员函数，这将抛出 `std::out_of_range` 异常。或者，代码中可以调用 `at()` 自由函数，这将在边界违例时导致快速失败（或者某个自定义动作）。

```cpp
void f(std::vector<int>& v, std::array<int, 12> a, int i)
{
    v[0] = a[0];        // 不好
    v.at(0) = a[0];     // OK（替代方案 1）
    at(v, 0) = a[0];    // OK（替代方案 2）

    v.at(0) = a[i];     // 不好
    v.at(0) = a.at(i);  // OK（替代方案 1）
    v.at(0) = at(a, i); // OK（替代方案 2）
}
```

##### 强制实施

* 对于没有边界检查的标准库函数的任何调用都给出诊断消息。
??? 在这里添加一组禁用函数的连接列表

本条规则属于[边界剖面配置](S-profile.md#SS-bounds)。


### <a name="Rsl-copy"></a>SL.con.4: 请勿对非可平凡复制的实参使用 `memset` 或 `memcpy`

##### 理由

这样做会破坏对象语义（例如，其会覆写掉 `vptr`）。

##### 注解

`(w)memset`，`(w)memcpy`，`(w)memmove`，以及 `(w)memcmp` 与此相似。

##### 示例

```cpp
struct base {
    virtual void update() = 0;
};

struct derived : public base {
    void update() override {}
};


void f (derived& a, derived& b) // 虚表再见！
{
    memset(&a, 0, sizeof(derived));
    memcpy(&a, &b, sizeof(derived));
    memcmp(&a, &b, sizeof(derived));
}
```

应当代之以定义适当的默认初始化，复制，以及比较函数

```cpp
void g(derived& a, derived& b)
{
    a = {};    // 默认初始化
    b = a;     // 复制
    if (a == b) do_something(a,b);
}
```

##### 强制实施

* 对在不可平凡复制的类型使用这些函数进行标记

**TODO 注释**:

* 对于标准库的影响需要和 WG21 之间进行紧密的协调，即便不需要标准化也应当至少保证兼容性。
* 我们正在考虑为标准库（尤其是 C 标准库）中如 `memcmp` 这样的函数指定边界安全的重载，并在 GSL 中提供它们。
* 对于标准中没有进行完全的边界检查的现存函数和如 `vector` 这样的类型来说，我们的目标是在启用了边界剖面配置的代码中调用时，这些功能应当进行边界检查，而从遗留代码中调用时则没有检查，可能需要利用契约来实现（正由几个 WG21 成员进行提案工作）。



## <a name="SS-string"></a>SL.str: 字符串

文本处理是一个大的主题。
`std::string` 无法全部覆盖这些。
这一部分主要尝试澄清 `std::string` 和 `char*`、`zstring`、`string_view` 和 `gsl::span<char>` 之间的关系。
有关非 ASCII 字符集和编码的重要问题（比如 `wchar_t`，Unicode，以及 UTF-8 等）将在别处讨论。

**参见**：[正则表达式](S-stdlib.md#SS-regex)

在这里，我们用“字符序列”或“字符串”来代表（终将）作为文本来读取的字符序列。
We don't consider ???

字符串概览：

* [SL.str.1: 使用 `std::string` 以拥有字符序列](S-stdlib.md#Rstr-string)
* [SL.str.2: 使用 `std::string_view` 或 `gsl::span<char>` 以指代字符序列](S-stdlib.md#Rstr-view)
* [SL.str.3: 使用 `zstring` 或 `czstring` 以指代 C 风格、以零结尾的字符序列](S-stdlib.md#Rstr-zstring)
* [SL.str.4: 使用 `char*` 以指代单个字符](S-stdlib.md#Rstr-char*)
* [SL.str.5: 使用 `std::byte` 以指代并不必须表示字符的字节值](S-stdlib.md#Rstr-byte)

* [SL.str.10: 当需要实施相关于文化地域的操作时，使用 `std::string`](S-stdlib.md#Rstr-locale)
* [SL.str.11: 当需要改动字符串时，使用 `gsl::span<char>` 而不是 `std::string_view`](S-stdlib.md#Rstr-span)
* [SL.str.12: 为作为标准库的 `string` 类型的字符串字面量使用后缀 `s`](S-stdlib.md#Rstr-s)

**参见**：

* [F.24 span](S-functions.md#Rf-range)
* [F.25 zstring](S-functions.md#Rf-zstring)


### <a name="Rstr-string"></a>SL.str.1: 使用 `std::string` 以拥有字符序列

##### 理由

`string` 能够正确处理资源分配，所有权，复制，渐进扩容，并提供许多有用的操作。

##### 示例

```cpp
vector<string> read_until(const string& terminator)
{
    vector<string> res;
    for (string s; cin >> s && s != terminator; ) // 读取一个单词
        res.push_back(s);
    return res;
}
```

注意已经为 `string` 提供了 `>>` 和 `!=`（作为有用操作的例子），并且没有显示的内存分配，
回收，或者范围检查（`string` 会处理这些）。

C++17 中，我们可以使用 `string_view` 而不是 `const string&` 作为参数，以允许调用方更大的灵活性：

```cpp
vector<string> read_until(string_view terminator)   // C++17
{
    vector<string> res;
    for (string s; cin >> s && s != terminator; ) // 读取一个单词
        res.push_back(s);
    return res;
}
```

##### 示例，不好

不要使用 C 风格的字符串来进行需要不单纯的内存管理的操作：

```cpp
char* cat(const char* s1, const char* s2)   // 当心！
    // return s1 + '.' + s2
{
    int l1 = strlen(s1);
    int l2 = strlen(s2);
    char* p = (char*)malloc(l1 + l2 + 2);
    strcpy(p, s1, l1);
    p[l1] = '.';
    strcpy(p + l1 + 1, s2, l2);
    p[l1 + l2 + 1] = 0;
    return p;
}
```

我们搞对了吗？
调用者能记得要对返回的指针调用 `free()` 吗？
这段代码能通过安全性评审吗？

##### 注解

没有测量就不要假设 `string` 比底层技术慢，要记得并非所有代码都是性能攸关的。
[请勿进行不成熟的优化](S-performance.md#Rper-Knuth)

##### 强制实施

???

### <a name="Rstr-view"></a>SL.str.2: 使用 `std::string_view` 或 `gsl::span<char>` 以指代字符序列

##### 理由

`std::string_view` 或 `gsl::span<char>` 提供了简易且（潜在）安全的对字符序列的访问，并与序列的
分配和存储方式无关。

##### 示例

```cpp
vector<string> read_until(string_view terminator);

void user(zstring p, const string& s, string_view ss)
{
    auto v1 = read_until(p);
    auto v2 = read_until(s);
    auto v3 = read_until(ss);
    // ...
}
```

##### 注解

`std::string_view`（C++17）是只读的。

##### 强制实施

???

### <a name="Rstr-zstring"></a>SL.str.3: 使用 `zstring` 或 `czstring` 以指代 C 风格、以零结尾的字符序列

##### 理由

可读性。
明确意图。
普通的 `char*` 可以是指向单个字符的指针，指向字符数组的指针，指向 C 风格（零结尾）字符串的指针，甚或是指向小整数的指针。
对这些情况加以区分能够避免误解和 BUG。

##### 示例

```cpp
void f1(const char* s); // s 可能是个字符串
```

我们所知的只不过是它可能是 nullptr 或者指向至少一个字符

```cpp
void f1(zstring s);     // s 是 C 风格字符串或者 nullptr
void f1(czstring s);    // s 是 C 风格字符串常量或者 nullptr
void f1(std::byte* s);  // s 是某个字节的指针（C++17）
```

##### 注解

除非确实有理由，否则不要把 C 风格的字符串转换为 `string`。

##### 注解

与其他的“普通指针”一样，`zstring` 不能表达所有权。

##### 注解

已经存在了上亿行的 C++ 代码，它们大多使用 `char*` 和 `const char*` 却并不注明其意图。
各种不同的方式都在使用它们，包括以之表示所有权，以及（代替 `void*`）作为通用的内存指针。
很难区分这些用法，因此这条指导方针很难被遵守。
而这是 C 和 C++ 程序中的最主要的 BUG 来源之一，因此一旦可行就遵守这条指导方针是值得的。

##### 强制实施

* 标记在 `char*` 上使用的 `[]`
* 标记在 `char*` 上使用的 `delete`
* 标记在 `char*` 上使用的 `free()`

### <a name="Rstr-char*"></a>SL.str.4: 使用 `char*` 以指代单个字符

##### 示例

现存代码中对 `char*` 的各种不同用法，是一种主要的错误来源。

##### 示例，不好

```cpp
char arr[] = {'a', 'b', 'c'};

void print(const char* p)
{
    cout << p << '\n';
}

void use()
{
    print(arr);   // 运行时错误；可能非常糟糕
}
```

数组 `arr` 并非 C 风格字符串，因为它不是零结尾的。

##### 替代方案

参见 [`zstring`](S-stdlib.md#Rstr-zstring)，[`string`](S-stdlib.md#Rstr-string)，以及 [`string_view`](S-stdlib.md#Rstr-view)。

##### 强制实施

* 标记在 `char*` 上使用的 `[]`

### <a name="Rstr-byte"></a>SL.str.5: 使用 `std::byte` 以指代并不必须表示字符的字节值

##### 理由

用 `char*` 来表示指向不一定是字符的东西的指针会造成混乱，
并会妨碍有价值的优化。

##### 示例

```cpp
???
```

##### 注解

C++17

##### 强制实施

???


### <a name="Rstr-locale"></a>SL.str.10: 当需要实施相关于文化地域的操作时，使用 `std::string`

##### 理由

`std::string` 支持标准库的 [`locale` 功能](S-stdlib.md#Rstr-locale)

##### 示例

```cpp
???
```

##### 注解

???

##### 强制实施

???

### <a name="Rstr-span"></a>SL.str.11: 当需要改动字符串时，使用 `gsl::span<char>` 而不是 `std::string_view`

##### 理由

`std::string_view` 是只读的。

##### 示例

???

##### 注解

???

##### 强制实施

编译器会标记出试图写入 `string_view` 的地方。

### <a name="Rstr-s"></a>SL.str.12: 为作为标准库的 `string` 类型的字符串字面量使用后缀 `s`

##### 理由

直接表达想法能够最小化犯错机会。

##### 示例

```cpp
auto pp1 = make_pair("Tokyo", 9.00);         // {C 风格字符串,double} 有意如此？
pair<string, double> pp2 = {"Tokyo", 9.00};  // 稍微啰嗦
auto pp3 = make_pair("Tokyo"s, 9.00);        // {std::string,double}    // C++14
pair pp4 = {"Tokyo"s, 9.00};                 // {std::string,double}    // C++17


```

##### 强制实施

???


## <a name="SS-io"></a>SL.io: I/O 流（iostream）

`iostream` 是一种类型安全的，可扩展的，带格式的和无格式的流式 I/O 的 I/O 程序库。
它支持多种（且用户可扩展的）缓冲策略以及多种文化地域。
它可以用于进行便利的 I/O，内存读写（字符串流），
以及用户定义的扩展，诸如跨网络的流（asio：尚未标准化）。

I/O 流规则概览：

* [SL.io.1: 仅在必要时才使用字符层面的输入](S-stdlib.md#Rio-low)
* [SL.io.2: 当进行读取时，总要考虑非法输入](S-stdlib.md#Rio-validate)
* [SL.io.3: 优先使用 iostream 进行 I/O](S-stdlib.md#Rio-streams)
* [SL.io.10: 除非你使用了 `printf` 族函数，否则要调用 `ios_base::sync_with_stdio(false)`](S-stdlib.md#Rio-sync)
* [SL.io.50: 避免使用 `endl`](S-stdlib.md#Rio-endl)
* [???](#???)

### <a name="Rio-low"></a>SL.io.1: 仅在必要时才使用字符层面的输入

##### 理由

除非你确实仅处理单个的字符，否则使用字符级的输入将导致用户代码实施潜在易错的
且潜在低效的从字符进行标记组合的工作。

##### 示例

```cpp
char c;
char buf[128];
int i = 0;
while (cin.get(c) && !isspace(c) && i < 128)
    buf[i++] = c;
if (i == 128) {
    // ... 处理过长的字符串 ....
}
```

更好的做法（简单得多而且可能更快）：

```cpp
string s;
s.reserve(128);
cin>>s;
```

而且额能并不需要 `reserve(128)`。

##### 强制实施

???


### <a name="Rio-validate"></a>SL.io.2: 当进行读取时，总要考虑非法输入

##### 理由

错误通常最好尽快处理。
如果输入无效，所有的函数都必须编写为对付不良的数据（而这并不现实）。

##### 示例

```cpp
???
```

##### 强制实施

???

### <a name="Rio-streams"></a>SL.io.3: 优先使用 iostream 进行 I/O

##### 理由

`iosteam` 安全，灵活，并且可扩展。

##### 示例

```cpp
// 写出一个复数：
complex<double> z{ 3,4 };
cout << z << '\n';
```

`complex` 是一个用户定义的类型，而其 I/O 的定义无需改动 `iostream` 库。

##### 示例

```cpp
// 读取一系列复数：
for (complex<double> z; cin>>z)
    v.push_back(z);
```

##### 例外

??? 性能 ???

##### 讨论：`iostream` vs. `printf()` 家族

人们通常说（并且通常是正确的）`printf` 家族比 `iostream` 有两个优势：
格式化的灵活性和性能。
这需要与 `iostream` 在处理用户定义类型方面的扩展性，针对安全性的违反方面的韧性，
隐含的内存管理，以及 `locale` 处理等优势之间进行权衡。

如果需要 I/O 性能的话，你几乎总能做到比 `printf()` 更好。

`gets()`，使用 `%s` 的 `scanf()`，和使用 `%s` 的 `printf()` 在安全性方面冒风险（容易遭受缓冲区溢出问题而且通常很易错）。
C11 定义了一些“可选扩展”，它们对其实参进行一些额外检查。
如果您的 C 程序库中包含 `gets_s()`、`scanf_s()` 和 `printf_s()`，它们也许是更安全的替代方案，但仍然并非是类型安全的。

##### 强制实施

可选地标记 `<cstdio>` 和 `<stdio.h>`。

### <a name="Rio-sync"></a>SL.io.10: 除非你使用了 `printf` 族函数，否则要调用 `ios_base::sync_with_stdio(false)`

##### 理由

`iostreams` 和 `printf` 风格的 I/O 之间的同步是由代价的。
`cin` 和 `cout` 默认是与 `printf` 相同步的。

##### 示例

```cpp
int main()
{
    ios_base::sync_with_stdio(false);
    // ... 使用 iostreams ...
}
```

##### 强制实施

???

### <a name="Rio-endl"></a>SL.io.50: 避免使用 `endl`

##### 理由

`endl` 操纵符大致相当于 `'\n'` 和 `"\n"`；
其最常用的情况只不过会以添加多余的 `flush()` 的方式拖慢程序。
与 `printf` 式输出相比，这种拖慢程度是比较显著的。

##### 示例

```cpp
cout << "Hello, World!" << endl;    // 两次输出操作和一次 flush
cout << "hello, World!\n";          // 一次输出操作且没有 flush
```

##### 注解

对于 `cin`/`cout`（或同等设备）的交互来说，没什么原因必须进行冲洗；它们是自动进行的。
对于向文件写入来说，也很少需要 `flush`。

##### 注解

对于字符串流（指 `ostringstream`），插入一个 `endl` 完全等价于
插入一个 `'\n'` 字符，但正是这种情况下，`endl` 可能会明显比较慢。

`endl` *并不*关注产生平台专有的行结尾序列（比如 Windows 上的 `"\r\n"`）。
因此，字符串流的 `s << endl` 只会插入*单个* `'\n'` 字符。

##### 注解

除了（偶尔会比较重要的）性能问题外，
从 `"\\n"` 和 `endl` 之间进行选择基本上完全是审美问题。

## <a name="SS-regex"></a>SL.regex: 正则表达式

`<regex>` 是标准 C++ 的正则表达式库。
它支持许多正则表达式的模式约定。

## <a name="SS-chrono"></a>SL.chrono: 时间

`<chrono>`（在命名空间 `std::chrono` 中定义）提供了 `time_point` 和 `duration`，并同时提供了
用于以各种不同单位输出时间的函数。
它还提供了用于注册 `time_point` 的时钟。

## <a name="SS-clib"></a>SL.C: C 标准库

???

C 标准库规则概览：

* [SL.C.1: 请勿使用 setjmp/longjmp](S-stdlib.md#Rclib-jmp)
* [???](#???)
* [???](#???)

### <a name="Rclib-jmp"></a>SL.C.1: 请勿使用 setjmp/longjmp

##### 理由

`longjmp` 会忽略析构函数，由此使得依赖于 RAII 的所有资源管理策略全部失效。

##### 强制实施

标记出现的所有 `longjmp` 和 `setjmp`



