# <a name="S-profile"></a>Pro: 剖面配置

理想情况，我们应当遵循所有这些指导方针。
这样能够得到最简洁，最规范，最少易错性，而且通常是最快的代码。
不幸的是这通常是不可能的，因为我们不得不让代码适合于大型的代码库并使用一些现存的程序库。
常常是，这样的代码已经编写好几十年了，并且并不遵循这些指导方针。
我们必须以[渐次采纳](S-modernizing.md#S-modernizing)为目标。

无论采用何种渐次采纳的策略，我们都应当能够首先采用一些相关指导方针的集合来
处理某些问题的集合，遗留其他的以后处理。
当发现某些而不是全部的指导方针对于代码库有关时，以及当在某个专门化的应用领域
采用一组专门化的指导方针的集合时，会出现类似的“相关指导方针”的主意。
我们称这样的相关指导方针的集合为一个“剖面配置”。
我们对这种指导方针集合的目标是其内聚性，它们一起可以有助于我们达成某个特定的目标，如“消除范围错误”
或“静态类型安全性”。
每个剖面配置都被设计用于消除一个类别的错误。
而“随意”实施一些独立的规则，相对于提供确定的改善来说，更像是对代码库的破坏。

“剖面配置”是确定的并且可移植实施的规则（限制）子集，它们是专门设计以达成某项特定保证的。
“确定性”意味着它们仅需要进行局部分析，并且可以在一台计算机中进行实现（虽然并不比如此）。
“可移植实施性”表明它们和语言规则相似，因而程序员们可以期望不同实施工具对于相同的代码给出相同的答案。

编写成在这样的语言剖面配置下仍免于警告的代码，可以认为是遵循这个剖面配置的。
而遵从的代码则可以认为对于该剖面配置的目标安全属性来说是安全的。
遵从的代码不会成为这种性质的错误的根本原因，
虽然程序中可能从其他的代码引入这样的错误。
剖面配置还可能会引入一些额外的库类型，以简化遵从性并鼓励编写正确的代码。

剖面配置概览：

* [Pro.type: 类型安全性](S-profile.md#SS-type)
* [Pro.bounds: 边界安全性](S-profile.md#SS-bounds)
* [Pro.lifetime: 生存期安全性](S-profile.md#SS-lifetime)

未来，我们打算定义更多的剖面配置，并向现有剖面配置中添加更多的检查。
候选者有：

* 窄化算术提升和转换（可能会成为一个单独的安全算术剖面配置的一部分）
* 从负浮点数向无符号整型类型进行算术强制转换（同上）
* 经选择的未定义行为：从 Gabriel Dos Reis 为 WG21 研究小组开发的 UB 列表入手
* 经选择的未指明行为：处理可移植性问题。
* `const` 违反：大多数情况已经由编译器完成，但我们可以捕捉不适当的强制转换和 `const` 的不当使用。

剖面配置的开启是由实现所定义的；典型情况下，是在分析工具之中进行的设置。

要抑制对某个剖面配置检查，可以在语言构造上放一个 `suppress` 标注。例如：

```cpp
[[suppress(bounds)]] char* raw_find(char* p, int n, char x)    // 在 p[0]..p[n - 1] 中寻找 x
{
    // ...
}
```

这样 `raw_find()` 就可以在内存中到处爬了。
显然，进行抑制应当是非常罕见的。

## <a name="SS-type"></a>Pro.safety: 类型安全性剖面配置

这个剖面配置将能够简化正确使用类型的代码编写，并避免因疏忽产生类型双关。
它是关注于移除各种主要的类型违例的因素（包括对强制转换和联合的不安全使用）而达成这点的。

针对本部分的目的而言，
类型安全性被定义为这样的性质：对变量的使用不会不遵守其所定义的类型的规则。
通过类型 `T` 所访问的内存，不应该是某个实际上包含了无关类型 `U` 的对象的有效内存。
注意，当和[边界安全性](S-profile.md#SS-bounds)、[生存期安全性](S-profile.md#SS-lifetime)组合起来时，安全性才是完整的。

这个剖面配置的实现应当在源代码中识别出下列模式，将之作为不符合并给出诊断信息。

类型安全性剖面配置概览：

* <a name="Pro-type-avoidcasts"></a>Type.1: [避免强制转换](S-expr.md#Res-casts)：

  1. <a name="Pro-type-reinterpretcast"></a>请勿使用 `reinterpret_cast`；此为[避免强制转换](S-expr.md#Res-casts)和[优先使用具名的强制转换](S-expr.md#Res-casts-named)的严格的版本。  
  2. <a name="Pro-type-arithmeticcast"></a>请勿在算术类型上使用 `static_cast`；此为[避免强制转换](S-expr.md#Res-casts)和[优先使用具名的强制转换](S-expr.md#Res-casts-named)的严格的版本。  
  3. <a name="Pro-type-identitycast"></a>当源指针类型和目标类型相同时，请勿进行指针强制转换；此为[避免强制转换](S-expr.md#Res-casts)的严格的版本。  
  4. <a name="Pro-type-implicitpointercast"></a>当指针转换可以隐式转换时，请勿使用指针强制转换；此为[避免强制转换](S-expr.md#Res-casts)的严格的版本。  
* <a name="Pro-type-downcast"></a>Type.2: 请勿使用 `static_cast` 进行向下强制转换：
[代之以使用 `dynamic_cast`](S-class.md#Rh-dynamic_cast)。
* <a name="Pro-type-constcast"></a>Type.3: 请勿使用 `const_cast` 强制掉 `const`（亦即不要这样做）：
[不要强制掉 `const`](S-expr.md#Res-casts-const)。
* <a name="Pro-type-cstylecast"></a>Type.4: 请勿使用  C 风格的强制转换 `(T)expression` 和函数式风格强制转换 `T(expression)`：
优先使用[构造语法](S-expr.md#Res-construct)，[具名的强制转换](S-expr.md#Res-casts-named)，或 `T{expression}`。
* <a name="Pro-type-init"></a>Type.5: 请勿在初始化之前使用变量：
[坚持进行初始化](S-expr.md#Res-always)。
* <a name="Pro-type-memberinit"></a>Type.6: 坚持初始化成员变量：
[坚持进行初始化](S-expr.md#Res-always)，
可以采用[默认构造函数](S-class.md#Rc-default0)或者
[默认成员初始化式](S-class.md#Rc-in-class-initializer)。
* <a name="Pro-type-union"></a>Type.7: 避免裸 union：
[代之以使用 `variant`](S-class.md#Ru-naked)。
* <a name="Pro-type-varargs"></a>Type.8: 避免 varargs：
[不要使用 `va_arg` 参数](S-functions.md#F-varargs)。

##### 影响

在类型安全性剖面配置下，你可以相信每个操作都将在有效的对象上进行。
可能抛出异常以报告无法（在编译时）被静态地检测到的错误。
要注意的是，这种类型安全性仅当我们同样具有[边界安全性](S-profile.md#SS-bounds)和[生存期安全性](S-profile.md#SS-lifetime)时才是完整的。
而没有这些保证的话，一个内存区域可能以与其所存储的单个或多个对象，或对象的一部分无关的方式被访问。


## <a name="SS-bounds"></a>Pro.bounds: 边界安全性剖面配置

这个剖面配置将能简化对于在分配的内存块的边界之中进行操作的编码工作。
它是通过关注于移除边界违例的主要根源——即指针算术和数组索引——而做到这点的。
这个剖面配置的核心功能之一就是限制指针只能指向单个对象而不是数组。

我们将边界安全性定义为这样一种性质：程序不通过一个对象来对分配给这个对象的内存范围之外的内存进行访问。
仅当边界安全性与[类型安全性](S-profile.md#SS-type)和[生存期安全性](S-profile.md#SS-lifetime)组合起来时才是完整的，
它们还会包含其他允许发生边界违例的不安全操作。

边界安全性剖面配置概览：

* <a name="Pro-bounds-arithmetic"></a>Bounds.1: 请勿使用指针算术。请使用 `span` 代替：
[（仅）传递单个对象的指针](S-interfaces.md#Ri-array)，并[保持指针算术的简单性](S-expr.md#Res-ptr)。
* <a name="Pro-bounds-arrayindex"></a>Bounds.2: 仅使用常量表达式对数组进行索引操作：
[（仅）传递单个对象的指针](S-interfaces.md#Ri-array)，并[保持指针算术的简单性](S-expr.md#Res-ptr)。
* <a name="Pro-bounds-decay"></a>Bounds.3: 避免数组向指针的退化：
[（仅）传递单个对象的指针](S-interfaces.md#Ri-array)，并[保持指针算术的简单性](S-expr.md#Res-ptr)。
* <a name="Pro-bounds-stdlib"></a>Bounds.4: 请勿使用不进行边界检查的标准库函数和类型：
[以类型安全的方式使用标准库](S-stdlib.md#Rsl-bounds)

##### 影响

边界安全性意味着，当访问对象（尤其是数组）时不会越过对象的内存分配范围。
这消除了一大类的隐伏且难于发现的错误，包括（不）著名的“缓冲区溢出”错误。
这避免了安全漏洞，以及（当越界写入时发生）内存损坏错误的大量来源。
即使越界访问“只是读取操作”，它也可能导致不变式的违反（当所访问的不是预期的类型时）
和“神秘的值”。


## <a name="SS-lifetime"></a>Pro.lifetime: 生存期安全性剖面配置

通过已经不指向任何东西的指针进行访问，是错误的一种主要来源，
而且在许多传统的 C 或 C++ 风格的编程中这很难避免。
例如，指针可能未初始化，值为 `nullptr`，指向越过指针范围，或者指向已删除的对象。

[参见此处的设计说明书的当前版本](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Lifetime.pdf)

生存期安全性剖面配置概览：

* <a name="Pro-lifetime-invalid-deref"></a>Lifetime.1: 不要解引用无效指针：
[检测或避免](S-expr.md#Res-deref)。

##### 影响

一旦强制实施了编码风格规则，静态分析，以及程序库支持的组合方案之后，本剖面配置将能

* 消除 C++ 中的恶劣错误的一种主要来源
* 消除潜在安全漏洞的一种主要来源
* 通过消除多余的“偏执”检查而改善性能
* 提升代码正确性的信心
* 通过强制遵循一种关键的 C++ 语言规则而避免未定义的行为


