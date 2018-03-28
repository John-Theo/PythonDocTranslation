# What’s New In Python 3.0

> 原文链接：https://docs.python.org/3/whatsnew/3.0.html

>   原作者：Guido van Rossum

> 责任翻译：庄心昊

__！ 注意：本文对比版本为 3.0 和 2.6，如果你关心最新版本的变化，请在__ [这里](https://docs.python.org/3/whatsnew/changelog.html#changelog) __查看 change log。__

__另外，如果你发现本文的机制在最新的版本中不适用，请提交issue。__

__（以下为作者正文）__

---

本文简述了 Python 3.0 相对于 2.6 版本的新功能。 Python 3.0 也被称为 “Python 3000” 或 “Py3K”，是有史以来第一个 _特意向后兼容_ 的Python版本。比普通的发行版本有更多的变化，且对于所有Python用户来说更重要。尽管如此，在充分了解了这些变化之后，你会发现 Python 其实并没有改变太多 —— 总的来说，我们主要修复了广为提及的小问题，并去除了很多无用的设计。

本文的目的不是提供所有新功能的详细说明，而是试图给出一个简便的概述。有关完整的详细信息，请参阅 Python 3.0 的文档和/或文中引用的许多PEP。如果您想了解特定功能的完整实施和设计原理，PEP通常比常规文档具有更多的细节；但请注意，一旦功能完全实施，PEP通常不会保持在最新状态。

> __译者注：PEP (Python Enhancement Proposals) 是一份为Python社区提供各种增强功能的技术规格，也是提交新特性，以便让社区指出问题，精确化技术文档的提案。[`传送门`](https://www.python.org/dev/peps/)__

由于时间限制，这个文件并不像它应该呈现的那样完整。与新的发行版本一样，源代码中的 `Misc/NEWS` 文件包含大量有关已更改的每个小细节的详细信息。


## 常见的绊脚石

如果您已经习惯于使用 Python 2.5，本节列出了最有可能困住您的一些变化。

### print 是一个函数
`print` 语句已被 [print()](https://docs.python.org/3/library/functions.html#print) 函数替代，其中的关键字参数替换了原先 `print` 语句的大部分特殊语法（[__PEP 3105__](https://www.python.org/dev/peps/pep-3105)）。 例程：
```
Old: print "The answer is", 2*2
New: print("The answer is", 2*2)

Old: print x,           # Trailing comma suppresses newline
New: print(x, end=" ")  # Appends a space instead of a newline

Old: print              # Prints a newline
New: print()            # You must call the function!

Old: print >>sys.stderr, "fatal error"
New: print("fatal error", file=sys.stderr)

Old: print (x, y)       # prints repr((x, y))
New: print((x, y))      # Not the same as print(x, y)!
```
