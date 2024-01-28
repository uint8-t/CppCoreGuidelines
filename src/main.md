# <a name="main"></a>C++ 核心指导方针

2022/10/6

编辑：

* [Bjarne Stroustrup](http://www.stroustrup.com)
* [Herb Sutter](http://herbsutter.com/)

翻译

* 李一楠 (li_yinan AT 163 DOT com)

本文档是处于持续改进之中的在线文档。
本文档作为开源项目，发布版本为 0.8。
复制，使用，修改，以及创建本项目的衍生物，受到一份 MIT 风格的版权授权。
向本项目作出贡献需要同意一份贡献者授权。详情参见附属的 [LICENSE](LICENSE) 文件。
我们将本项目开放给“友好用户”进行使用，复制，修改，以及生产衍生物，并希望能够获得建设性的资源投入。

十分欢迎大家提出意见和改进建议。
随着我们的知识增长，随着语言和可用的程序库的改进，我们计划对这份文档不断进行修改和扩充。
当提出您的意见时，请关注[导言](S-introduction.md#S-introduction)部分，其中概述了我们的目标和所采用的一般方法。
贡献者的列表请参见[这里](S-references.md#SS-ack)。

已知问题：

* 仍未对规则集合的完整性、一致性和可强制实施性加以全面的检查。
* 三问号 (???) 用于标记已知的信息缺失。
* 需要更新参考部分；许多前 C++11 的源代码都过于老旧。
* [To-do: 未分类的规则原型](S-unclassified.md#S-unclassified) 是一份基本上保持最新状态的 to-do 列表。

您可以[阅读本指南的范围和结构的说明](S-abstract.md#S-abstract)，或者直接跳转到：

* [In: 导言](S-introduction.md#S-introduction)
* [P: 理念](S-philosophy.md#S-philosophy)
* [I: 接口](S-interfaces.md#S-interfaces)
* [F: 函数](S-functions.md#S-functions)
* [C: 类和类层次](S-class.md#S-class)
* [Enum: 枚举](S-enum.md#S-enum)
* [R: 资源管理](S-resource.md#S-resource)
* [ES: 表达式和语句](S-expr.md#S-expr)
* [Per: 性能](S-performance.md#S-performance)
* [CP: 并发与并行](S-concurrency.md#S-concurrency)
* [E: 错误处理](S-errors.md#S-errors)
* [Con: 常量和不可变性](S-const.md#S-const)
* [T: 模板和泛型编程](S-templates.md#S-templates)
* [CPL: C 风格的编程](S-cpl.md#S-cpl)
* [SF: 源文件](S-source.md#S-source)
* [SL: 标准库](#???)

配套章节：

* [A: 架构相关理念](S-A.md#S-A)
* [NR: 伪规则和错误的看法](S-not.md#S-not)
* [RF: 参考资料](S-references.md#S-references)
* [PRO: 剖面配置](S-profile.md#S-profile)
* [GSL: 指导方针支持库](#???)
* [NL: 命名和代码布局建议](S-naming.md#S-naming)
* [FAQ: 常见问题的解答](S-faq.md#S-faq)
* [附录 A: 程序库](S-libraries.md#S-libraries)
* [附录 B: 代码的现代化转换](S-modernizing.md#S-modernizing)
* [附录 C: 相关讨论](S-discussion.md#S-discussion)
* [附录 D: 支持工具](S-tools.md#S-tools)
* [词汇表](S-glossary.md#S-glossary)
* [To-do: 未分类的规则原型](S-unclassified.md#S-unclassified)

您可以查看有关某个具体的语言特性的一些规则：

* 赋值：
[正规类型](S-class.md#Rc-regular) --
[优先采用初始化](S-class.md#Rc-initialize) --
[复制](S-class.md#Rc-copy-semantic) --
[移动](S-class.md#Rc-move-semantic) --
[以及其他操作](S-class.md#Rc-matched) --
[缺省操作](S-class.md#Rc-eqdefault)
* `class`：
[数据](S-class.md#Rc-org) --
[不变式](S-class.md#Rc-struct) --
[成员](S-class.md#Rc-member) --
[辅助函数](S-class.md#Rc-helper) --
[具体类型](S-class.md#SS-concrete) --
[构造函数，=，和析构函数](S-class.md#S-ctor) --
[类层次](S-class.md#SS-hier) --
[运算符](S-class.md#SS-overload)
* `concept`：
[规则](S-templates.md#SS-concepts) --
[泛型编程中](S-templates.md#Rt-raise) --
[模板实参](S-templates.md#Rt-concepts) --
[语义](S-templates.md#Rt-low)
* 构造函数：
[不变式](S-class.md#Rc-struct) --
[建立不变式](S-class.md#Rc-ctor) --
[`throw`](S-class.md#Rc-throw) --
[缺省操作](S-class.md#Rc-default0) --
[不需要](S-class.md#Rc-default) --
[`explicit`](S-class.md#Rc-explicit) --
[委派](S-class.md#Rc-delegating) --
[`virtual`](S-class.md#Rc-ctor-virtual)
* 派生 `class`：
[何时使用](S-class.md#Rh-domain) --
[作为接口](S-class.md#Rh-abstract) --
[析构函数](S-class.md#Rh-dtor) --
[复制](S-class.md#Rh-copy) --
[取值和设值](S-class.md#Rh-get) --
[多继承](S-class.md#Rh-mi-interface) --
[重载](S-class.md#Rh-using) --
[分片](S-class.md#Rc-copy-virtual) --
[`dynamic_cast`](S-class.md#Rh-dynamic_cast)
* 析构函数：
[以及构造函数](S-class.md#Rc-matched) --
[何时需要？](S-class.md#Rc-dtor) --
[不可失败](S-class.md#Rc-dtor-fail)
* 异常：
[错误](S-errors.md#S-errors) --
[`throw`](S-errors.md#Re-throw) --
[仅用于错误](S-errors.md#Re-errors) --
[`noexcept`](S-errors.md#Re-noexcept) --
[最少化 `try`](S-errors.md#Re-catch) --
[无异常如何？](S-errors.md#Re-no-throw-codes)
* `for`：
[范围式 `for` 和 `for`](S-expr.md#Res-for-range) --
[`for` 和 `while`](S-expr.md#Res-for-while) --
[`for`-初始化式](S-expr.md#Res-for-init) --
[空循环体](S-expr.md#Res-empty) --
[循环变量](S-expr.md#Res-loop-counter) --
[循环变量的类型 ???](#???)
* 函数：
[命名](S-functions.md#Rf-package) --
[单操作](S-functions.md#Rf-logical) --
[不能抛出异常](S-functions.md#Rf-noexcept) --
[实参](S-functions.md#Rf-smart) --
[实参传递](S-functions.md#Rf-conventional) --
[多返回值](S-functions.md#Rf-out-multi) --
[指针](S-functions.md#Rf-return-ptr) --
[lambda](S-functions.md#Rf-capture-vs-overload)
* `inline`:
[小型函数](S-functions.md#Rf-inline) --
[头文件中](S-source.md#Rs-inline)
* 初始化：
[总是](S-expr.md#Res-always) --
[优先采用 `{}`](S-expr.md#Res-list) --
[lambda](S-expr.md#Res-lambda-init) --
[类内初始化式](S-class.md#Rc-in-class-initializer) --
[类成员](S-class.md#Rc-initialize) --
[工厂函数](S-class.md#Rc-factory)
* lambda 表达式：
[何时使用](S-class.md#SS-lambdas)
* 运算符：
[约定](S-class.md#Ro-conventional) --
[避免转换运算符](S-class.md#Ro-conversion) --
[与 lambda](S-class.md#Ro-lambda)
* `public`, `private`, 和 `protected`：
[信息隐藏](S-class.md#Rc-private) --
[一致性](S-class.md#Rh-public) --
[`protected`](S-class.md#Rh-protected)
* `static_assert`：
[编译时检查](S-philosophy.md#Rp-compile-time) --
[和概念](S-templates.md#Rt-check-class)
* `struct`：
[用于组织数据](S-class.md#Rc-org) --
[没有不变式时使用](S-class.md#Rc-struct) --
[不能有私有成员](S-class.md#Rc-class)
* `template`：
[抽象](S-templates.md#Rt-raise) --
[容器](S-templates.md#Rt-cont) --
[概念](S-templates.md#Rt-concepts)
* `unsigned`：
[和 `signed`](S-expr.md#Res-mix) --
[位操作](S-expr.md#Res-unsigned)
* `virtual`：
[接口](S-interfaces.md#Ri-abstract) --
[非 `virtual`](S-class.md#Rc-concrete) --
[析构函数](S-class.md#Rc-dtor-virtual) --
[不能失败](S-class.md#Rc-dtor-fail)

您可以查看用于表达这些规则的一些设计概念：

* 断言：???
* 错误：???
* 异常：异常保证 (???)
* 故障：???
* 不变式：???
* 泄漏：???
* 程序库：???
* 前条件：???
* 后条件：???
* 资源：???

