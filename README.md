[![C++ 核心指南](cpp_core_guidelines_logo_text.png)](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)

>"在 C++ 内部，有一个更小、更简单、更安全的语言努力着想要显现。"
>-- <cite>Bjarne Stroustrup</cite>

由 Bjarne Stroustrup 领导的协作努力，正如 C++ 语言本身，[C++ 核心指南](CppCoreGuidelines.md)是众多组织多年讨论和设计的成果。它们的设计鼓励普遍适用性和广泛采纳，但您可以自由复制和修改这些准则，以满足您组织的需求。

## 入门

指南本身可以在 [C++ 核心指南](CppCoreGuidelines.md) 找到。该文档使用 [GH-flavored MarkDown](https://github.github.com/gfm/) 编写。它故意保持简单，主要使用 ASCII 字符，以便进行自动后处理，如语言翻译和重新格式化。编辑们维护了一个[为浏览而格式化的版本](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)。请注意，这是手动集成的，可能比 master 分支中的版本略旧。

C++ 核心指南是一个不断发展的文档，没有严格的“发布”节奏。Bjarne Stroustrup 定期审查文档，并在引言中增加版本号。[增加版本号的提交](https://github.com/isocpp/CppCoreGuidelines/releases)在 git 中被标记。

许多准则使用了仅包含头文件的指南支持库。其中一个实现可在 [GSL: 指南支持库](https://github.com/Microsoft/GSL) 找到。

## 背景与范围

这些准则的目的是帮助人们有效地使用现代 C++。所谓的“现代 C++”是指 C++11 及更高版本。换句话说，考虑到您现在就可以开始，您希望您的代码在5年后、10年后会是什么样子？

这些准则专注于相对较高层次的问题，如接口、资源管理、内存管理和并发。这些规则影响应用架构和库设计。遵循这些规则将导致代码在静态类型安全上有保障，不会有资源泄漏，并能捕捉到比现今代码中更多的编程逻辑错误。而且它将运行得非常快——你可以负担得起正确做事。

我们对低层次的问题（如命名约定和缩进风格）关注较少。然而，任何能帮助程序员的主题都不是禁区。

我们最初的一组规则强调安全（各种形式）和简单。它们可能过于严格。我们预计将不得不引入更多的例外，以更好地适应现实世界的需求。我们还需要更多的规则。

您会发现某些规则与您的预期或甚至您的经验相反。如果我们没有建议您以任何方式改变您的编码风格，那我们就失败了！请尝试验证或反驳规则！特别是，我们非常希望我们的一些规则能够得到测量数据或更好的例子的支持。

您会发现某些规则显而易见或甚至微不足道。请记住，指南的一个目的是帮助那些经验较少或来自不同背景或语言的人更快地达到速度。

这些规则旨在得到分析工具的支持。违反规则的情况将以引用（或链接）到相关规则的形式被标记出来。我们不期望您在尝试编写代码之前记住所有的规则。

这些规则是为了逐步引入到代码库中准备的。我们计划构建支持这一点的工具，并希望其他人也会这样做。

## 贡献和许可证

我们非常欢迎评论和改进建议。随着我们对语言的理解和语言及可用库集的改进，我们计划修改和扩展这份文档。更多细节可以在 [CONTRIBUTING](./CONTRIBUTING.md) 和 [LICENSE](./LICENSE) 找到。

感谢 [DigitalOcean](https://www.digitalocean.com/?refcode=32f291566cf7&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=CopyPaste) 托管了标准 C++ 基金会网站。
