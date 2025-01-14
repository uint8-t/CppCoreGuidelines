# <a name="S-references"></a>RF: 参考材料

已经为 C++，尤其是对 C++ 的使用编写过了许多的编码标准、规则和指导方针。
它们中许多都

* 关注的是低级问题，比如标识符的拼写
* 是由 C++ 的新手编写的
* 将“禁止程序员作出不常见行为”作为其首要目标
* 将维持许多编译器的可移植性作为目标（有些已经是 10 年前的了）
* 是为了维持好几十年的代码库而编写的
* 是仅关注单一的应用领域的
* 只会产生反效果
* 被忽略了（程序员为了完成工作不得不忽略它们）

不良的编码标准要比没有编码标准还要差。
不过一组恰当的指导方针比没有标准要好得多：“形式即解放。”

我们为什么不能有一种允许所有我们想要的同时又禁止所有我们不期望的东西的语言（“完美的语言”）呢？
本质上说，这是由于可负担的语言（及其工具链）同时也要为那些需求与你不同的人提供服务，并且要为你今后比今天更多的需求提供服务。
而且，你的需求会随时间而改变，而为此你则需要采用一种通用语言。
今日貌似理想的语言在未来可能会变得过于受限了。

编码指导方针可以使语言能够适应于特定的需求。
因此，并不存在适用于每个人的单一编码风格。
我们预计不同的组织会提供更具限制性和更严格的编码风格附加规定。

参考材料部分：

* [RF.rules: 编码规则](S-references.md#SS-rules)
* [RF.books: 带有编码指导方针的书籍](S-references.md#SS-books)
* [RF.C++: C++ 编程 (C++11/C++14/C++17)](S-references.md#SS-Cplusplus)
* [RF.web: 网站](S-references.md#SS-web)
* [RS.video: 有关“当代 C++”的视频](S-references.md#SS-vid)
* [RF.man: 手册](S-references.md#SS-man)
* [RF.core: 核心指导方针相关材料](S-references.md#SS-core)

## <a name="SS-rules"></a>RF.rules: 编码规则

* [AUTOSAR Guidelines for the use of the C++14 language in critical and safety-related systems v17.10](https://www.autosar.org/fileadmin/user_upload/standards/adaptive/17-10/AUTOSAR_RS_CPP14Guidelines.pdf)
* [Boost Library Requirements and Guidelines](http://www.boost.org/development/requirements.html).
  ???.
* [Bloomberg: BDE C++ Coding](https://github.com/bloomberg/bde/wiki/CodingStandards.pdf).
  着重强调了代码的组织和布局。
* Facebook: ???
* [GCC Coding Conventions](https://gcc.gnu.org/codingconventions.html).
  C++03 以及（相当）一部分向后兼容。
* [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
  面向 C++17 和（同样）较老的代码库。Google 的专家们现在正展开活跃的合作，以改进这里的各项指导方针，有希望能够合并这些成果，以使它们能够成为他们也同样推荐采纳的一组现代的通用指导方针。
* [JSF++: JOINT STRIKE FIGHTER AIR VEHICLE C++ CODING STANDARDS](http://www.stroustrup.com/JSF-AV-rules.pdf).
  文档编号 2RDU00001 Rev C. December 2005.
  针对飞行控制软件。
  针对硬实时。
  这意味着它需要非常多的限制（“程序如果发生故障就会有人挂掉”）。
  例如，飞机起飞后禁止进行任何自由存储的分配和回收（禁止内存溢出并禁止发生碎片化）。
  禁止使用异常（因为没有可用工具可以保证异常能够在固定的短时间段内被处理）。
  所使用的程序库必须是已被证明可以用于关键任务应用的。
  它和这个指导方针集合的相似性并不让人惊讶，因为 Bjarne Stroustrup 正是 JSF++ 的作者之一。
  建议采纳，但请注意其非常特定的关注领域。
* [MISRA C++ 2008: Guidelines for the use of the C++ language in critical systems] (https://www.misra.org.uk/Buyonline/tabid/58/Default.aspx)。
* [Using C++ in Mozilla Code](https://firefox-source-docs.mozilla.org/code-quality/coding-style/using_cxx_in_firefox_code.html).
  如其名称所示，它关注于跨许多（老）编译器的兼容性。
  因此，它是很具有限制性的。
* [Geosoft.no: C++ Programming Style Guidelines](http://geosoft.no/development/cppstyle.html).
  ???.
* [Possibility.com: C++ Coding Standard](http://www.possibility.com/Cpp/CppCodingStandard.html).
  ???.
* [SEI CERT: Secure C++ Coding Standard](https://wiki.sei.cmu.edu/confluence/x/Wnw-BQ).
  针对安全关键代码所编写的一组非常好的规则（还带有示例和原理说明）。
  它们的许多规则都广泛适用。
* [High Integrity C++ Coding Standard](http://www.codingstandard.com/).
* [llvm](http://llvm.org/docs/CodingStandards.html).
  有些简略，基于 C++14，而且是（有理由地）针对其应用领域的。
* ???

## <a name="SS-books"></a>RF.books: 带有编码指导方针的书籍

* [Meyers96](S-unclassified.md#Meyers96) Scott Meyers: *More Effective C++*. Addison-Wesley 1996.
* [Meyers97](S-unclassified.md#Meyers97) Scott Meyers: *Effective C++, Second Edition*. Addison-Wesley 1997.
* [Meyers01](S-unclassified.md#Meyers01) Scott Meyers: *Effective STL*. Addison-Wesley 2001.
* [Meyers05](S-unclassified.md#Meyers05) Scott Meyers: *Effective C++, Third Edition*. Addison-Wesley 2005.
* [Meyers15](S-unclassified.md#Meyers15) Scott Meyers: *Effective Modern C++*. O'Reilly 2015.
* [SuttAlex05](S-unclassified.md#SuttAlex05) Sutter and Alexandrescu: *C++ Coding Standards*. Addison-Wesley 2005. 与其说是一组规则，不如说是一组元规则。前 C++11 时代。
* [Stroustrup05](S-unclassified.md#Stroustrup05) Bjarne Stroustrup: [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
  LCSD05. October 2005.
* [Stroustrup14](S-unclassified.md#Stroustrup05) Stroustrup: [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
  每章的结尾都有一个包含一组建议的忠告部分。
* [Stroustrup13](S-unclassified.md#Stroustrup13) Stroustrup: [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html).
  Addison Wesley 2013.
  每章的结尾都有一个包含一组建议的忠告部分。
* Stroustrup: [Style Guide](http://www.stroustrup.com/Programming/PPP-style.pdf)
  for [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html).
  大多是一些低级的命名和代码布局规则。
  主要作为教学工具。

## <a name="SS-Cplusplus"></a>RF.C++: C++ 编程 (C++11/C++14)

* [TC++PL4](http://www.stroustrup.com/4th.html):
  面向有经验的程序员的，对 C++ 语言和标准库的全面彻底的描述。
* [Tour++](http://www.stroustrup.com/Tour.html):
  面向有经验的程序员的，对 C++ 语言和标准库的简介。
* [Programming: Principles and Practice using C++](http://www.stroustrup.com/programming.html):
  面向初学者和新手们的教材。

## <a name="SS-web"></a>RF.web: 网站

* [isocpp.org](https://isocpp.org)
* [Bjarne Stroustrup 的个人主页](http://www.stroustrup.com)
* [WG21](http://www.open-std.org/jtc1/sc22/wg21/)
* [Boost](http://www.boost.org)<a name="Boost"></a>
* [Adobe open source](https://opensource.adobe.com/)
* [Poco libraries](http://pocoproject.org/)
* Sutter's Mill?
* ???

## <a name="SS-vid"></a>RS.video: 有关“当代 C++”的视频

* Bjarne Stroustrup: [C++11?Style](http://channel9.msdn.com/Events/GoingNative/GoingNative-2012/Keynote-Bjarne-Stroustrup-Cpp11-Style). 2012.
* Bjarne Stroustrup: [The Essence of C++: With Examples in C++84, C++98, C++11, and?C++14](http://channel9.msdn.com/Events/GoingNative/2013/Opening-Keynote-Bjarne-Stroustrup). 2013
* [CppCon '14](https://isocpp.org/blog/2014/11/cppcon-videos-c9) 的全部演讲
* Bjarne Stroustrup: [The essence of C++](https://www.youtube.com/watch?v=86xWVb4XIyE) 在爱丁堡大学。2014
* Bjarne Stroustrup: [The Evolution of C++ Past, Present and Future](https://www.youtube.com/watch?v=_wzc7a3McOs). CppCon 2016 keynote.
* Bjarne Stroustrup: [Make Simple Tasks Simple!](https://www.youtube.com/watch?v=nesCaocNjtQ). CppCon 2014 keynote.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote about the Core Guidelines.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote about the Core Guidelines.
* CppCon 15
* ??? C++ Next
* ??? Meting C++
* ??? more ???

## <a name="SS-man"></a>RF.man: 手册

* ISO C++ Standard C++11.
* ISO C++ Standard C++14.
* [ISO C++ Standard C++17](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4606.pdf). 委员会草案。
* [Palo Alto "Concepts" TR](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).
* [ISO C++ Concepts TS](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4553.pdf).
* [WG21 Ranges report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/n4569.pdf). 草案。


## <a name="SS-core"></a>RF.core: 核心指导方针相关材料

这个部分包含一些用于展示核心指导方针及其背后的思想的有用材料：

* [Our documents directory](https://github.com/isocpp/CppCoreGuidelines/tree/master/docs)
* Stroustrup, Sutter, and Dos Reis: [A brief introduction to C++’s model for type- and resource-safety](http://www.stroustrup.com/resource-model.pdf). A paper with lots of examples.
* Sergey Zubkov: [a Core Guidelines talk](https://www.youtube.com/watch?v=DyLwdl_6vmU)
and here are the [slides](http://2017.cppconf.ru/talks/sergey-zubkov). In Russian. 2017.
* Neil MacIntosh: [The Guideline Support Library: One Year Later](https://www.youtube.com/watch?v=_GhNnCuaEjo). CppCon 2016.
* Bjarne Stroustrup: [Writing Good C++14](https://www.youtube.com/watch?v=1OEu9C51K2A). CppCon 2015 keynote.
* Herb Sutter: [Writing Good C++14... By Default](https://www.youtube.com/watch?v=hEx5DNLWGgA). CppCon 2015 keynote.
* Peter Sommerlad: [C++ Core Guidelines - Modernize your C++ Code Base](https://www.youtube.com/watch?v=fQ926v4ZzAM). ACCU 2017.
* Bjarne Stroustrup: [No Littering!](https://www.youtube.com/watch?v=01zI9kV4h8c). Bay Area ACCU 2016.
It gives some idea of the ambition level for the Core uidelines.

CppCon 的展示的幻灯片是可以获得的（其链接，还有上传的视频）。

极大欢迎对于这个列表的贡献。

## <a name="SS-ack"></a>鸣谢

感谢对规则、建议、支持信息和参考材料等等作出了各种贡献的许多人：

* Peter Juhl
* Neil MacIntosh
* Axel Naumann
* Andrew Pardoe
* Gabriel Dos Reis
* Zhuang, Jiangang (Jeff)
* Sergey Zubkov

请查看 github 的贡献者列表。

