# <a name="S-gsl"></a>GSL: 指导方针支持库

GSL 是一个小型的程序库，其中的设施被设计用于支持本指导方针。
不使用这些设施的话，这些指导方针不得不变得对语言细节过于限制。

核心指导方针支持库是定义在 `gsl` 命名空间中的，其中的名字可能是对标准库和其他著名程序库的名字的别名。通过 `gsl` 命名空间进行的（编译期）间接，使我们可以进行试验，以及对这些支持设施提供局部变体。

GSL 只有头文件，可以在 [GSL: 指导方针支持库](https://github.com/Microsoft/GSL)找到。
支持库中的设施被设计为极为轻量化（零开销），它们相比于传统方案并不会带来任何开销。
当需要时，它们还可以用其他功能“工具化”（比如一些检查）来帮助进行诸如调试等任务。

各指导方针中，除了使用 GSL 中的类型之外，还使用了标准程序库（如 C++17）中的类型。
例如，我们假定有一个 `variant` 类型，但它当前尚未在 GSL 中。
总之，请使用[通过表决进入 C++17 的版本](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0088r3.html)。

由于诸如当前 C++ 版本的限制等技术原因，您所使用的程序库中可能不支持下面列出的某些 GSL 类型。
请查阅您的 GSL 文档以获得更多信息。

对于以下的每个 GSL 类型，我们都为该类型给出了不变式。只要用户代码仅使用类型所提供的成员或自由函数（就是说，用户代码不会以违反任何其他指南规则的方式，绕过类型的接口来改动对象的值或位），该不变式均有效。

GSL 组件概览：

* [GSL.view: 视图](S-gsl.md#SS-views)
* [GSL.owner](S-gsl.md#SS-ownership)
* [GSL.assert: 断言](S-gsl.md#SS-assertions)
* [GSL.util: 工具](S-gsl.md#SS-utilities)
* [GSL.concept: 概念](S-gsl.md#SS-gsl-concepts)

我们计划提供一个“ISO C++ 标准风格的”半正式的 GSL 规范。

我们依赖于 ISO C++ 标准库，并希望 GSL 的一些部分能够被吸收到标准库之中。

## <a name="SS-views"></a>GSL.view: 视图

这些类型使用户可以区分带有和没有所有权的指针，并区分指向单个对象的指针和指向序列的第一个元素的指针。

“视图”都不是所有者。

引用都不是所有者（参见 [R.4](S-resource.md#Rr-ref)）。注意：有许多机会能让引用存活超过其所指代的对象，如按引用返回局部变量，持有 vector 的某个元素的引用然后进行 `push_back`，绑定到  `std::max(x, y + 1)`，等等。生存期安全性剖面配置的目标就是处理这些事情，但即便如此 `owner<T&>` 也没有意义且不建议使用。

它们的名字基本上遵循 ISO 标准库风格（小写字母和下划线）：

* `T*`      // `T*` 不是所有者，可能为 null；假定为指向单个元素。
* `T&`      // `T&` 不是所有者，不可能为“null 引用”；引用总是绑定到对象上。

“原生指针”写法（如 `int*`）假定为具有其最常见的含义；亦即指向一个对象的指针，但并不拥有它。
所有者应当被转换为资源包装（如 `unique_ptr` 或 `vector<T>`），或标为 `owner<T*>`。

* `owner<T*>`   // `T*`，拥有所指向/指代的对象；可能为 `nullptr`。

`owner` 用于对代码中有所有权的指针进行标记，它们无法更改为使用适当的资源包装。
其原因可能包括：

* 转换的成本。
* 需要为某个 ABI 使用指针。
* 这个指针时某种资源包装的实现的一部分。

`owner<T>` 和 `T` 的某种资源包装的区别在于它仍然需要明确进行 `delete`。

`owner<T>` 假定为指代自由存储（堆）上的某个对象。

当某个东西不应当为 `nullptr` 时，可以这样做：

* `not_null<T>`   // `T` 通常是某个指针类型（例如 `not_null<int*>` 和 `not_null<owner<Foo*>>`），且不能为 `nullptr`。
  `T` 可以是 `==nullptr` 有意义的任何类型。

* `span<T>`       // `[p:p+n)`，构造函数接受 `{p, q}` 和 `{p, n}`；`T` 为指针类型
* `span_p<T>`     // `{p, predicate}` `[p:q)`，其中 `q` 为首个使 `predicate(*p)` 为真的元素

`span<T>` 指代零或更多可改动的 `T`，除非 `T` 为 `const` 类型。对 `span` 元素的所有访问，尤其是通过 `operator[]` 进行的访问，默认保证进行边界检查。

> 注：有提案将 GSL 的 `span`（起初叫做 `array_view`）加入 C++ 标准库且已被采纳（名字和接口有改动），唯一不同是 `std::span` 不提供边界检查保证。因此，GSL 修改了 `span` 的名字和接口以跟踪 `std::span`，并应当与 `std::span` 完全相同，而其仅有差别应当为，GSL 的 `span` 默认是完全边界安全的。如果边界检查影响其接口，那么应当通过 ISO C++ 委员会带回改动的提案，以保持 `gsl::span` 和 `std::span` 接口兼容。如果 `std::span` 未来的演化添加了边界检查，则 `gsl::span` 即可移除。

“指针算术”最好在 `span` 之内进行。
指向多个 `char` 但并非 C 风格字符串的 `char*`（比如指向某个输入缓冲区的指针）应当表示为一个 `span`。

* `zstring`    // `char*`，假定为 C 风格字符串；亦即以零结尾的 `char` 的序列或者是 `nullptr`
* `czstring`   // `const char*`，假定为 C 风格字符串；亦即以零结尾的 `const` `char` 的序列或者是 `nullptr`

逻辑上来说，最后两种别名是没有必要的，但我们并不总是依照逻辑的，它们可以在指向单个 `char` 的指针和指向 C 风格字符串的指针之间明确地进行区分。
并未假定为零结尾的字符序列应当是 `span<char>`，或当因 ABI 问题而不可能时是 `char*`，而不是 `zstring`。


对于不能为 `nullptr` 的 C 风格字符串，应使用 `not_null<zstring>`。 ??? 我们需要为 `not_null<zstring>` 命名吗？还是说它的难看是有用的？

## <a name="SS-ownership"></a>GSL.owner: 所有权指针

* `unique_ptr<T>`     // 唯一所有权：`std::unique_ptr<T>`
* `shared_ptr<T>`     // 共享所有权：`std::shared_ptr<T>`（引用计数指针）
* `stack_array<T>`    // 栈分配数组。元素的数量在构造时确定并固定下来。其元素可改变，除非 `T` 为 `const` 类型。
* `dyn_array<T>`      // ??? 有必要吗 ??? 堆分配数组。元素的数量在构造时确定并固定下来。
  其元素可改变，除非 `T` 为 `const` 类型。基本上这是一个进行分配并拥有其元素的 `span`。

## <a name="SS-assertions"></a>GSL.assert: 断言

* `Expects`     // 前条件断言。当前放置于函数体内。今后应当移动到声明中。
```cpp
            // `Expects(p)` 当不满足 `p == true` 时会终止程序
            // `Expects` 处于一组选项的控制之下（强制，错误消息，对终止程序的替代）
```

* `Ensures`     // 后条件断言。当前放置于函数体内。今后应当移动到声明中。

现在这些断言还是宏（天呐！）而且必须（只）被用在函数定义式之内。
等待标准委员会对于契约和断言语法的确定。
参见使用属性语法的[契约提案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r1.pdf)，
比如说，`Expects(p)` 将变为 `[[expects: p]]`。

## <a name="SS-utilities"></a>GSL.util: 工具

* `finally`        // `finally(f)` 创建一个 `final_action{f}`，其析构函数将执行 `f`
* `narrow_cast`    // `narrow_cast<T>(x)` 就是 `static_cast<T>(x)`
* `narrow`         // `narrow<T>(x)` 在满足无符号提升下的 `static_cast<T>(x) == x` 时为 `static_cast<T>(x)`，否则抛出 `narrowing_error`（例如，`narrow<unsigned>(-42)` 会抛出异常）
* `[[implicit]]`   // 放在单参数构造函数上的“记号”，以明确说明它们并非显式构造函数。
* `move_owner`     // `p = move_owner(q)` 含义为 `p = q` 但 ???
* `joining_thread` // RAII 风格版本的进行联结的 `std::thread`
* `index`          // 用于进行所有的容器和数组索引的类型（当前是 `ptrdiff_t` 的别名）

## <a name="SS-gsl-concepts"></a>GSL.concept: 概念

这些概念（类型谓词）借用于
Andrew Sutton 的 Origin 程序库，
Range 提案，
以及 ISO WG21 的 Palo Alto TR。
其中许多都与已在 C++20 中成为 ISO C++ 标准的概念十分相似。

* `String`
* `Number`
* `Boolean`
* `Range`              // C++20 中为 `std::ranges::range`
* `Sortable`           // C++20 中为 `std::sortable`
* `EqualityComparable` // C++20 中为 `std::equality_comparable`
* `Convertible`        // C++20 中为 `std::convertible_to`
* `Common`             // C++20 中为 `std::common_with`
* `Integral`           // C++20 中为 `std::integral`
* `SignedIntegral`     // C++20 中为 `std::signed_integral`
* `SemiRegular`        // C++20 中为 `std::semiregular`
* `Regular`            // C++20 中为 `std::regular`
* `TotallyOrdered`     // C++20 中为 `std::totally_ordered`
* `Function`           // C++20 中为 `std::invocable`
* `RegularFunction`    // C++20 中为 `std::regular_invocable`
* `Predicate`          // C++20 中为 `std::predicate`
* `Relation`           // C++20 中为 `std::relation`
* ...

### <a name="SS-gsl-smartptrconcepts"></a>GSL.ptr: 智能指针概念

* `Pointer`  // 带有 `*`，`->`，`==`，以及默认构造的类型（默认构造被假定为设值为唯一的“null”值）
* `Unique_pointer`  // 符合 `Pointer` 的类型，可移动但不可复制
* `Shared_pointer`   // 符合 `Pointer` 的类型，可复制

