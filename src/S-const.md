# <a name="S-const"></a>Con: 常量与不可变性

常量是不会出现竞争条件的。
当大量的对象不会改变它们的值时，对程序进行推理将变得更容易。
承诺“不改动”作为参数所传递对象的接口，极大地提升了可读性。

常量规则概览：

* [Con.1: 缺省情况下，对象应当是不可变的](S-const.md#Rconst-immutable)
* [Con.2: 缺省情况下，成员函数应当为 `const`](S-const.md#Rconst-fct)
* [Con.3: 缺省情况下，应当传递指向 `const` 对象的指针或引用](S-const.md#Rconst-ref)
* [Con.4: 构造之后不再改变其值的对象应当以 `const` 来定义](S-const.md#Rconst-const)
* [Con.5: 以 `constexpr` 来定义可以在编译期计算的值](S-const.md#Rconst-constexpr)

### <a name="Rconst-immutable"></a>Con.1: 缺省情况下，对象应当是不可变的

##### 理由

不可变对象更易于进行推理，应仅当需要改动对象的值时，才使之为非 `const` 对象。
避免出现意外造成的或者很难发觉的值的改变。

##### 示例

```cpp
for (const int i : c) cout << i << '\n';    // 仅进行读取: const

for (int i : c) cout << i << '\n';          // 不好: 仅进行读取
```

##### 例外

按值传递的函数参数很少被改动，但也很少被声明为 `const`。
为了避免造成混淆和大量的误报，不要对函数参数实施这条规则。。

```cpp
void f(const char* const p); // 迂腐
void g(const int i) { ... }  // 迂腐
```

注意，函数参数是局部变量，其改动也是局部的。

##### 强制实施

* 标记未发生改动的非 `const` 变量（排除参数以避免误报）

### <a name="Rconst-fct"></a>Con.2: 缺省情况下，成员函数应当为 `const`

##### 理由

除非成员函数会改变对象的可观察状态，否则它应当标记为 `const`。
这样做更精确地描述了设计意图，具有更佳的可读性，编译器可以识别更多的错误，而且有时能够带来更多的优化机会。

##### 示例，不好

```cpp
class Point {
    int x, y;
public:
    int getx() { return x; }    // 不好，应当为 const，它并不改变对象的状态
    // ...
};

void f(const Point& pt)
{
    int x = pt.getx();          // 错误，无法通过编译，因为 getx 并未标记为 const
}
```

##### 注解

传递非 `const` 的指针或引用并非天生就是不好的，
但应当只有在所调用的函数预计会修改这个对象时才这样做。
代码的读者必须假定接受“普通的” `T*` 或 `T&` 的函数都将会修改其所指代的对象。
如果它现在不会，那它可能以后会，且无需强制要求重新编译。

##### 注解

有些代码和程序库提供的函数是声明为 `T*`，
但这些函数并不会修改这个 `T`。
这对于进行代码现代化转换的人们来说是个问题。
你可以：

* 如果倾向于长期解决方案的话，将程序库更新为 `const` 正确的；
* “强制掉 `const`”（[最好避免这样做](S-expr.md#Res-casts-const)）；
* 提供包装函数。

例如：

```cpp
void f(int* p);   // 老代码：f() 并不会修改 `*p`
void f(const int* p) { f(const_cast<int*>(p)); } // 包装函数
```

注意，这种包装函数的方案是一种补丁，只能在无法修改 `f()` 的声明时才使用它，
比如当它属于某个你无法修改的程序库时。

##### 注解

`const` 成员函数可以改动 `mutable` 对象的值，或者通过某个指针成员改动对象的值。
一种常见用法是来维护一个缓存以避免重复进行复杂的运算。
例如，这里的 `Date` 缓存（记住）了其字符串表示，以简化其重复使用：

```cpp
class Date {
public:
    // ...
    const string& string_ref() const
    {
        if (string_val == "") compute_string_rep();
        return string_val;
    }
    // ...
private:
    void compute_string_rep() const;    // 计算字符串表示并将其存入 string_val
    mutable string string_val;
    // ...
};
```

另一种说法是 `const` 特性不会传递。
通过 `const` 成员函数改动 `mutable` 成员的值和通过非 `const` 指针来访问的对象的值
是有可能的。
由类负责确保这样的改动仅当根据其语义（不变式）对于其用户有意义时
才会发生。

**参见**：[PImpl](S-interfaces.md#Ri-pimpl)

##### 强制实施

* 如果未标记为 `const` 的成员函数并未对任何成员变量实施非 `const` 操作的话，对其进行标记。

### <a name="Rconst-ref"></a>Con.3: 缺省情况下，应当传递指向 `const` 对象的指针或引用

##### 理由

避免所调用的函数意外地改变了这个值。
如果被调用的函数不会改动状态的话，对程序的推理将变得容易得多。

##### 示例

```cpp
void f(char* p);        // f 会不会修改 *p?（假定它会修改）
void g(const char* p);  // g 不会修改 *p
```

##### 注解

传递指向非 `const` 对象的指针或引用并不是天生就有问题的，
不过只有当所调用的函数本就有意改动对象时才能这样做。

##### 注解

[请勿强制掉 `const`](S-expr.md#Res-casts-const)。

##### 强制实施

* 如果函数并未修改以指向非 `const` 的指针或引用传递的对象，则对其进行标记。
* 如果函数（利用强制转换）修改了以指向 `const` 的指针或引用传递的对象，则对其进行标记。

### <a name="Rconst-const"></a>Con.4: 构造之后不再改变其值的对象应当以 `const` 来定义

##### 理由

避免意外地改变对象的值。

##### 示例

```cpp
void f()
{
    int x = 7;
    const int y = 9;

    for (;;) {
        // ...
    }
    // ...
}
```

既然 `x` 并非 `const`，我们就必须假定它可能在循环中的某处会被修改。

##### 强制实施

* 标记并未被修改的非 `const` 变量。

### <a name="Rconst-constexpr"></a>Con.5: 以 `constexpr` 来定义可以在编译期计算的值

##### 理由

更好的性能，更好的编译期检查，受保证的编译期求值，竞争条件可能性为零。

##### 示例

```cpp
double x = f(2);            // 可能在运行时求值
const double y = f(2);      // 可能在运行时求值
constexpr double z = f(2);  // 除非 f(2) 可在编译期求值，否则会报错
```

##### 注解

参见 F.4。

##### 强制实施

* 对带有常量表达式初始化式的 `const` 定义进行标记。

