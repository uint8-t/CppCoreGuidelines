# <a name="S-faq"></a>FAQ: 常见问题及其回答

本节中包括了对关于这些指导方针的常见问题的回答。

### <a name="Faq-aims"></a>FAQ.1: 这些指导方针的想要达成什么目标？

请参见<a href="#S-abstract">本页面开头</a>。这是一个开源项目，旨在为采用当今的 C++ 标准来编写 C++ 代码而维护的一组现代的权威指导方针。这些指导方针的设计是现代的，尽可能使机器可实施的，并且是为贡献和分支保持开放，以使各种组织机构可以便于将它们整合到其自己组织的编码指导方针之中。

### <a name="Faq-announced"></a>FAQ.2: 这项工作是何时何地首次公开的？

是在 [Bjarne Stroustrup 在他为 CppCon 2015 的开场主旨演讲，“Writing Good C++14”](https://isocpp.org/blog/2015/09/stroustrup-cppcon15-keynote)。另请参见[相应的 isocpp.org 博客条目](https://isocpp.org/blog/2015/09/bjarne-stroustrup-announces-cpp-core-guidelines)，关于类型和内存安全性指导方针的原理请参见 [Herb Sutter 的后续 CppCon 2015 演讲，“Writing Good C++14 ... By Default”](https://isocpp.org/blog/2015/09/sutter-cppcon15-day2plenary)。

### <a name="Faq-maintainers"></a>FAQ.3: 谁是这些指导方针的作者和维护者？

最初的主要作者和维护者是 Bjarne Stroustrup 和 Herb Sutter，而迄今为止的指导方针则是由来自 CERN，Microsoft，Morgan Stanley，以及许多其他组织机构的专家所贡献的。指导方针发布时，其正处于 "0.6" 状态，我们欢迎人们进行贡献。正如 Stroustrup 在其声明中所说：“我们需要帮助！”

### <a name="Faq-contribute"></a>FAQ.4: 我如何进行贡献呢？

参见 [CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md)。我们感激志愿者的帮助！

### <a name="Faq-maintainer"></a>FAQ.5: 怎样成为一名编辑或维护者？

通过先进行大量贡献并使你的贡献被认可具有一致的质量。参见 [CONTRIBUTING.md](https://github.com/isocpp/CppCoreGuidelines/blob/master/CONTRIBUTING.md)。我们感激志愿者的帮助！

### <a name="Faq-iso"></a>FAQ.6: 这些指导方针被 ISO C++ 标准委员会采纳了吗？它们是否代表委员会的一致意见？

不是这样。这些指导方针不在标准之内。它们是为标准服务的，而当前维护的指导方针是为了更有效地使用当前的标准 C++的。我们的目标是使其与委员会所设计的标准保持同步。

### <a name="Faq-isocpp"></a>FAQ.7: 既然这些指导方针并不是委员会所采纳的，它们为何在 `github.com/isocpp` 之下呢？

因为 `isocpp` 是标准 C++ 基金会；而标准委员会的仓库则处于 [github.com/*cplusplus*](https://github.com/cplusplus) 之下。我们需要一个中立组织来持有版权和许可以明确其并不是由某个人或供应商所控制的。这个自然实体就是基金会，其设立是为了推进使用并持续更新对现代标准 C++ 的理解，以及推进标准委员会的工作。其所遵循的正是与 isocpp.org 为 [C++ FAQ](https://isocpp.org/faq) 所做的相同模式，它是有 Bjarne Stroustrup，Marshall Cline，和 Herb Sutter 所发起的工作，并以相同的方式贡献为了开放项目。

### <a name="Faq-cpp98"></a>FAQ.8: 会有 C++98 版本的指导方针吗？C++11 版本呢？

不会。这些指导方针的目标是更好地使用现代标准 C++，以及假定你有一个现代的遵循标准的编译器时如何进行代码编写的。

### <a name="Faq-language-extensions"></a>FAQ.9: 这些指导方针中会提出新的语言功能吗？

不会。这些指导方针的目标是更好地使用现代标准 C++，它们自我限定为仅建议使用这些功能。

### <a name="Faq-markdown"></a>FAQ.10: 这些指导方针的书写使用的是哪个版本的 Markdown？

这些编码指导方针使用的是 [CommonMark](http://commonmark.org)，以及 `<a>` HTML 锚定元素。

我们正在考虑以下这些来自 [GitHub Flavored Markdown (GFM)](https://help.github.com/articles/github-flavored-markdown/) 的扩展：

* 有围栏代码块（正在讨论是否统一使用缩进还是围栏代码块）
* 表格（我们虽然还没用到，但很需要它们，这是一种 GFM 扩展）

避免使用其他 HTML 标签和其他扩展。

注意：我们还没对这种风格达成一致。

### <a name="Faq-gsl"></a>FAQ.50: 什么是 GSL（指导方针支持程序库）？

GSL 是在指导方针中所指定的类型和别名的一个小集合。当写下本文时，对它们的说明还过于松散；我们计划添加一个 WG21 风格的接口规范来确保不同实现之间保持一致，并作为一项可能的标准化提案，按常规遵循标准委员会进行采纳、改进、修订或否决。

### <a name="Faq-msgsl"></a>FAQ.51: [github.com/Microsoft/GSL](https://github.com/Microsoft/GSL) 是 GSL 吗？

不是。它只是由 Microsoft 所贡献的第一个实现。我们鼓励其他供应商提供其他的实现，对该实现的分支和贡献也是被鼓励的。书写本文作为一项公开项目的一周中，已经出现了至少一个 GPLv3 的开源实现。我们计划制定一个 WG21 风格的接口规范来确保不同实现之间保持一致。

### <a name="Faq-gsl-implementation"></a>FAQ.52: 为何不在指导方针之中提供一个真正的 GSL 实现呢？

我们不愿去保佑某个特定的实现，因为我们不希望让人们以为只有一个实现，而疏忽大意地扼杀了其他并行的实现。而如果在指导方针中包含一个真正实现的话，无论是谁提供了它都会变得过于有影响力。我们更倾向于采用委员会的更具长期性的方案，即指定其接口而不是实现。但同时我们也需要至少存在一个实现；希望可以有很多。

### <a name="Faq-boost"></a>FAQ.53: 为什么不把 GSL 类型提交给 Boost 呢？

因为我们想要立刻使用它们，也因为我们想要在一旦标准库中出现了满足其需要的类型时立刻将它们撤销掉。

### <a name="Faq-gsl-iso"></a>FAQ.54: ISO C++ 标准委员会采纳了 GSL（指导方针支持程序库）吗？

没有。GSL 的存在只为提供少量标准库中还没有的类型和别名。如果委员会决定了（这些类型或者满足其需要的其他类型的）标准化的版本，就可以将它们从 GSL 中删除了。

### <a name="Faq-gsl-string-view"></a>FAQ.55: 既然你是尽可能使用标准类型，为什么 GSL 的 `span<char>` 同 Library Fundamentals 1 Technical Specification 和 C++17 工作文本中的 `string_view` 不同呢？为什么不使用委员会采纳的 `string_view`？

有关 C++ 标准库的视图的分类的统一观点是，“视图（view）”意味着“只读”，而“跨距（span）”意味着“可读写”。如果你只需要一组字符的不需要保证边界检查的只读视图，并且你可以用 C++17，那就使用 C++17 的 `std::string_view`。否则，如果你需要的是不需要保证边界检查的可读写视图，并且可以用 C++20，那就用 C++20 的 `std::span<char>`。否则，就用 `gsl::span<char>`。

### <a name="Faq-gsl-owner"></a>FAQ.56: `owner` 和提案的 `observer_ptr` 一样吗？

不一样。`owner` 有所有权，它是一个别名，而且适用于任何间接类型。而 `observer_ptr` 的主要意图则是明确某个*没有*所有权的指针。

### <a name="Faq-gsl-stack-array"></a>FAQ.57: `stack_array` 和标准的 `array` 一样吗？

不一样。`stack_array` 保证在栈上分配。虽然 `std::array` 直接在其自身内部包含存储，但 `array` 对象可以放在包括堆在内的任何地方。

### <a name="Faq-gsl-dyn-array"></a>FAQ.58: `dyn_array` 和 `vector` 或者提案的 `dynarray` 一样吗？

不一样。`dyn_array` 是不可改变大小的，是一种指代堆分配的固定大小数组的一种安全方式。与 `vector` 不同，它是为了取代数组 `new[]` 的。与委员会中提案的 `dynarray` 不同，它并不会参与编译器和语言的魔法，来在当它作为分配于栈上的对象的成员时也在栈上分配；它只不过指代一个“动态的”或基于堆的数组而已。

### <a name="Faq-gsl-expects"></a>FAQ.59: `Expects` 和 `assert` 一样吗？

不一样。它是一种对于契约前条件语言支持的占位符。

### <a name="Faq-gsl-ensures"></a>FAQ.60: `Ensures` 和 `assert` 一样吗？

不一样。它是一种对于契约后条件语言支持的占位符。

