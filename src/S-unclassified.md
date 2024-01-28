# <a name="S-unclassified"></a>To-do: 未分类的规则原型

这是我们的未完成列表。
以下各条目最终将成为规则或者规则的一部分。
或者，我们也会决定不需要做出改动并将条目移除。

* 禁止远距离友元关系
* 应不应该处理物理设计（文件里有什么）和大规模设计（程序库，程序库的组合）？
* 命名空间
* 避免在全局作用域中使用 using 指令（但允许如 std 或其他的“基础”命名空间（如 experimental））
* 命名空间应当有什么粒度？是（如 Sutter/Alexandrescu 所定义的）所有被设计为一同工作或者一同发布的类和函数，还是应该更窄或是更宽？
* 应该用内联命名空间吗（比如 `std::literals::*_literals`）？
* 避免隐式转换
* Const 成员函数应当是线程安全的……aka, 但我并不想真的改掉变量，只是在第一次调用它的时候向它赋一个值……argh
* 始终初始化变量，为成员变量使用初始化列表。
* 无论谁编写了接受或返回 `void*` 的公开接口，都应该上火刑。我曾经好多年都以它作为自己的个人喜好来着。 :)
* 尽可能应用 `const`：成员函数，变量，以及 `const_iterators`
* 使用 `auto`
* `(size)` vs. `{initializers}` vs. `{Extent{size}}`
* 不要过度抽象
* 不要沿着调用栈向下传递指针
* 通过函数底部退出
* 应当提供在多态之间进行选择的指导方针吗？是的。经典的（虚函数，引用语义） vs. Sean Parent 风格（值语义，类型擦除，类似 `std::function`）  vs. CRTP/静态的？也许还需要 vs. 标签派发？
* 我们的指导方针是否应当在构造函数或析构函数中禁止进行虚函数调用？是的。许多人都禁止了，虽然我觉得这是 C++ 的一大优势 ??? -保留意见（D 走向 Java 之路太让我失望了）。有好的例子吗？
* 在 lambda 方面，在算法调用和其他回调场景中什么因素会影响决定使用 lambda 还是（局部？）类？
* 讨论一下 `std::bind`，Stephen T. Lavavej 对它有太多批评，使我开始觉得它是不是真的会在未来消失掉。应该建议以 lambda 代替它吗？
* 怎么处理泄漏的临时变量？ : `p = (s1 + s2).c_str();`
* 指针和迭代器的失效会导致悬挂指针：

```cpp
    void bad()
    {
        int* p = new int[700];
        int* q = &p[7];
        delete p;

        vector<int> v(700);
        int* q2 = &v[7];
        v.resize(900);

        // ... 使用 q 和 q2 ...
    }
```
* LSP
* 私有继承 vs/and 成员
* 避免静态类成员变量（竞争条件，几乎就是全局变量）

* 使用 RAII 锁定保护（`lock_guard`，`unique_lock`，`shared_lock`），绝不直接调用 `mutex.lock` 和 `mutex.unlock`（RAII）
* 优先使用非递归锁（它们通常用作不良情况的变通手段，有开销）
* 联结（join）你的每个线程！（因为如果没被联结或脱离（detach）的话，析构函数会调用 `std::terminate`……有什么好理由来脱离线程吗？） -- ??? 支持库该不该为 `std::thread` 提供一个 RAII 包装呢？
* 当必须同时获取两个或更多的互斥体时，应当使用 `std::lock`（或者别的死锁免除算法？）
* 当使用 `condition_variable` 时，始终用一个互斥体来保护它（在互斥体外面设置原子 bool 的值的做法是错误的！），并对条件变量自身使用同一个互斥体。
* 绝不对 `std::atomic<user-defined-struct>` 使用 `atomic_compare_exchange_strong`（填充位中的区别会造成影响，而在循环中使用 `compare_exchange_weak` 则能够归于稳定的填充位）
* 单独的 `shared_future` 对象不是线程安全的：两个线程不能等待同一个 `shared_future` 对象（它们可以等待指代相同共享状态的 `shared_future` 的副本）
* 单独的 `shared_ptr` 对象不是线程安全的：不同的线程可以调用指代相同共享对象的*不同* `shared_ptr` 的非 `const` 成员函数，但当一个线程访问一个 `shared_ptr` 对象时，另一个线程不能调用相同 `shared_ptr` 对象的非 `const` 成员函数（如果确实需要，考虑代之以 `atomic_shared_ptr`）

* 算术相关规则

# 参考文献

* <a name="Abrahams01"></a>
  \[Abrahams01]:  D. Abrahams. [Exception-Safety in Generic Components](http://www.boost.org/community/exception_safety.html).
* <a name="Alexandrescu01"></a>
  \[Alexandrescu01]:  A. Alexandrescu. Modern C++ Design (Addison-Wesley, 2001).
* <a name="Cplusplus03"></a>
  \[C++03]:           ISO/IEC 14882:2003(E), Programming Languages — C++ (updated ISO and ANSI C++ Standard including the contents of (C++98) plus errata corrections).
* <a name="Cargill92"></a>
  \[Cargill92]:       T. Cargill. C++ Programming Style (Addison-Wesley, 1992).
* <a name="Cline99"></a>
  \[Cline99]:         M. Cline, G. Lomow, and M. Girou. C++ FAQs (2ndEdition) (Addison-Wesley, 1999).
* <a name="Dewhurst03"></a>
  \[Dewhurst03]:      S. Dewhurst. C++ Gotchas (Addison-Wesley, 2003).
* <a name="Henricson97"></a>
  \[Henricson97]:     M. Henricson and E. Nyquist. Industrial Strength C++ (Prentice Hall, 1997).
* <a name="Koenig97"></a>
  \[Koenig97]:        A. Koenig and B. Moo. Ruminations on C++ (Addison-Wesley, 1997).
* <a name="Lakos96"></a>
  \[Lakos96]:         J. Lakos. Large-Scale C++ Software Design (Addison-Wesley, 1996).
* <a name="Meyers96"></a>
  \[Meyers96]:        S. Meyers. More Effective C++ (Addison-Wesley, 1996).
* <a name="Meyers97"></a>
  \[Meyers97]:        S. Meyers. Effective C++ (2nd Edition) (Addison-Wesley, 1997).
* <a name="Meyers01"></a>
  \[Meyers01]:        S. Meyers. Effective STL (Addison-Wesley, 2001).
* <a name="Meyers05"></a>
  \[Meyers05]:        S. Meyers. Effective C++ (3rd Edition) (Addison-Wesley, 2005).
* <a name="Meyers15"></a>
  \[Meyers15]:        S. Meyers. Effective Modern C++ (O'Reilly, 2015).
* <a name="Murray93"></a>
  \[Murray93]:        R. Murray. C++ Strategies and Tactics (Addison-Wesley, 1993).
* <a name="Stroustrup94"></a>
  \[Stroustrup94]:    B. Stroustrup. The Design and Evolution of C++ (Addison-Wesley, 1994).
* <a name="Stroustrup00"></a>
  \[Stroustrup00]:    B. Stroustrup. The C++ Programming Language (Special 3rdEdition) (Addison-Wesley, 2000).
* <a name="Stroustrup05"></a>
  \[Stroustrup05]:    B. Stroustrup. [A rationale for semantically enhanced library languages](http://www.stroustrup.com/SELLrationale.pdf).
* <a name="Stroustrup13"></a>
  \[Stroustrup13]:    B. Stroustrup. [The C++ Programming Language (4th Edition)](http://www.stroustrup.com/4th.html). Addison Wesley 2013.
* <a name="Stroustrup14"></a>
  \[Stroustrup14]:    B. Stroustrup. [A Tour of C++](http://www.stroustrup.com/Tour.html).
  Addison Wesley 2014.
* <a name="Stroustrup15></a>
  \[Stroustrup15]:    B. Stroustrup, Herb Sutter, and G. Dos Reis: [A brief introduction to C++'s model for type- and resource-safety](https://github.com/isocpp/CppCoreGuidelines/blob/master/docs/Introduction%20to%20type%20and%20resource%20safety.pdf).
* <a name="SuttHysl04b"></a>
  \[SuttHysl04b]:     H. Sutter and J. Hyslop. [Collecting Shared Objects](https://web.archive.org/web/20120926011837/http://www.drdobbs.com/collecting-shared-objects/184401839) (C/C++ Users Journal, 22(8), August 2004).
* <a name="SuttAlex05"></a>
  \[SuttAlex05]:      H. Sutter and  A. Alexandrescu. C++ Coding Standards. Addison-Wesley 2005.
* <a name="Sutter00"></a>
  \[Sutter00]:        H. Sutter. Exceptional C++ (Addison-Wesley, 2000).
* <a name="Sutter02"></a>
  \[Sutter02]:        H. Sutter. More Exceptional C++ (Addison-Wesley, 2002).
* <a name="Sutter04"></a>
  \[Sutter04]:        H. Sutter. Exceptional C++ Style (Addison-Wesley, 2004).
* <a name="Taligent94"></a>
  \[Taligent94]: Taligent's Guide to Designing Programs (Addison-Wesley, 1994).
