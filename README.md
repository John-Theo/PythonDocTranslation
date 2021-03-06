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
```python
Old: print "The answer is", 2*2
New: print("The answer is", 2*2)

Old: print x,           # 结尾的逗号使输出结束后不换行
New: print(x, end=" ")  # 每行结束后连接上end关键字传入的字符（译者注：本例中是一个空格）

Old: print              # 输出新的一行
New: print()            # 你必须调用函数！

# 译者注：输出重定向
Old: print >>sys.stderr, "fatal error"
New: print("fatal error", file=sys.stderr)

Old: print (x, y)       # 输出 repr((x, y))（译者注：__repr__是魔术功能）
New: print((x, y))      # 和 print(x, y)不完全相同！（译者注：新旧表达作用一致）
```

您也可以自定义元素之间的分隔符，例如：
```
print("There are <", 2**32, "> possibilities!", sep="")
```
__输出：__
```
There are <4294967296> possibilities!
```
__注意：__

- [print()](https://docs.python.org/3/library/functions.html#print) 函数不支持旧 `print` 申明的 `softspace` 特征。例如在 Python 2.x 中，`print "A\n", "B"` 会输出 `"A\nB\n"`；但是，在 Python 3.x 中，`print "A\n", "B"` 输出 `"A\n B\n"`。
- 刚开始在交互模式中，你会发现自己码上很多旧的 `print x` 表达。是时候训练你的手指猛戳新的 `print(x)` 了！
- 在使用 `2to3` source-to-source 转换工具的时候，所有的 `print` 表达会自动转化成 [print()](https://docs.python.org/3/library/functions.html#print) 函数调用，所以这对比较大的项目不是特别大的麻烦。


### 返回 Views 对象 和 迭代器 而不再是 Lists

一些著名的 API 接口不再返回列表（list）：
- [dict](https://docs.python.org/3/library/stdtypes.html#dict) 方法 [dict.keys()](https://docs.python.org/3/library/stdtypes.html#dict.keys)，[dict.items()](https://docs.python.org/3/library/stdtypes.html#dict.items) 和 [dict.values()](https://docs.python.org/3/library/stdtypes.html#dict.values) 不再返回 list，而返回 view 对象 \[[1](#q1)\]。例如，这样的表达不再生效：`k = d.keys(); k.sort()`。需要使用 `k = sorted(d)` 来代替（这在 Python 2.5 中一样适用，且一样快捷）。
- 同时，`dict.iterkeys()`，`dict.iteritems()` 和 `dict.itervalues()` 方法也不再受支持。
- [map()](https://docs.python.org/3/library/functions.html#map) 和 [filter()](https://docs.python.org/3/library/functions.html#filter) 返回迭代器（iterators）。如果你确实需要 list 对象，且传入序列全部等长，一个便捷的解决方案是把 [map()](https://docs.python.org/3/library/functions.html#map) 嵌套在 [list()](https://docs.python.org/3/library/stdtypes.html#list) 里，例如 `list(map(...))`，但是，一个更好的解决方案是尽量使用 list comprehension（尤其当原始代码使用了 [lambda](https://docs.python.org/3/reference/expressions.html#lambda) 时），或者重写代码，使其不需要用到 list（译者注：耿直了老铁）。一个特别的小技巧： [map()](https://docs.python.org/3/library/functions.html#map) 作为函数的 side effects 调用；正确的变形是使用普通的 [for](https://docs.python.org/3/reference/compound_stmts.html#for) 循环（因为创建一个 list 也是浪费）。
如果传入序列并不等长，[map()](https://docs.python.org/3/library/functions.html#map) 会在最短序列的末端停止 \[[2](#q2)\]。若希望和 Python 2.x 中的 [map()](https://docs.python.org/3/library/functions.html#map) 完全兼容，同样可以把序列嵌套在 `itertools.zip_longest()` 中，例如：`map(func, *sequences)` 将写为 `list(map(func, itertools.zip_longest(*sequences)))`。
- 现在，[range()](https://docs.python.org/3/library/stdtypes.html#range) 函数的使用和 `xrange()` 函数曾经的用法类似，但是前者可以处理任意大小的值，后者不再存在。
- [zip()](https://docs.python.org/3/library/functions.html#zip) 现在也返回迭代器。

<div class="q1"></div>

> [1] 译者注：关于更好理解 views 的概念，请看下例：

```python
>>> dishes = {'eggs': 2, 'sausage': 1, 'bacon': 1, 'spam': 500}
>>> keys = dishes.keys()
>>> values = dishes.values()  

>>> # iteration（译者注：更像是枚举？）
>>> n = 0  
>>> for val in values:  
...     n += val  
>>> print(n)  
504  

>>> # keys 和 values 以相同的顺序被遍历
>>> list(keys)
['eggs', 'bacon', 'sausage', 'spam']  
>>> list(values)  
[2, 1, 1, 500]  

>>> # view 对象动态反映 dict 的变化（译者注：从这个定义来看，view 也许可以翻译为“快照”）  
>>> del dishes['eggs']  
>>> del dishes['sausage']  
>>> list(keys)  
['spam', 'bacon']  

>>> # 集合操作
>>> keys & {'eggs', 'bacon', 'salad'}  
{'bacon'}  
```

<div class="q2"></div>

> [2] 译者注：原文对“会在最短序列的末端停止”的解释是：
With multiple iterables, the iterator stops when the shortest iterable is exhausted. For cases where the function inputs are already arranged into argument tuples, see [itertools.starmap()](https://docs.python.org/3/library/itertools.html#itertools.starmap).


### 大小比较

Python 3.0 简化了大小比较的流程：

- 当表达式没有天然可比的标度时，顺序比较操作符 (`<`, `<=`, `>=`, `>`) 会抛出一个 TypeError exception。因此，诸如 `1 < ''`，`0 > None` 或 `len <= len` 这样的表达式不再合法，而对于比如 `None < None` 这样的表达，则会抛出一个 [TypeError](https://docs.python.org/3/library/exceptions.html#TypeError) 而不是返回 `False`。这就意味着对一个异质的 list 排序不再可行——所有元素必须互相之间可比。值得注意的是，这一限制对 `==` 和 `!=` 操作符并不存在：不同类型的对象比较结果总是不同。
- builtin.sorted() 和 [list.sort()](https://docs.python.org/3/library/stdtypes.html#list.sort) 不再接受 _cmp_ 参数指定的比较函数，而使用键值对指定。注意：_key_ 和 _reverse_ 参数现在只能通过键值对指定。
- cmp() 函数应该被视为不存在了，而 \_\_cmp\_\_() 魔术方法也不再受支持。现在使用 [\_\_lt\_\_()](https://docs.python.org/3/reference/datamodel.html#object.__lt__) 来排序， [\_\_eq\_\_()](https://docs.python.org/3/reference/datamodel.html#object.__eq__) 配合 [\_\_hash\_\_()](https://docs.python.org/3/reference/datamodel.html#object.__hash__) 使用，以及其他所需的比较操作符。（如果你真的很想实现 cmp() 的功能，你可以使用如下表达式 `(a > b) - (a < b)` ，它的效果和 `cmp(a, b)` 相同。）

> \_\_cmp\_\_() 魔术方法在目前的 python 3.6.4 中依然受到支持，但 cmp() 函数确实不见了。

### 整数

- [__PEP 237__](https://www.python.org/dev/peps/pep-0237)：事实上，long （译者注：长整型）被重命名为 [int](https://docs.python.org/3/library/functions.html#int)。这就意味着，只有一个內建（built-in）的叫做 [int](https://docs.python.org/3/library/functions.html#int) 的整形，但是从表现来看，它和原先的 long 差不多。
- [__PEP 238__](https://www.python.org/dev/peps/pep-0238)：一个诸如 `1/2` 的表达式返回一个 float。而使用 `1//2` 来得到截尾（取整）的结果。（后面的表达形式已经存在了数年，至少从 Python 2.2 开始就有了。）
- 因为对整形的值不再有限制， `sys.maxint` 常量被取消。但是 `sys.maxsize` 可以被用作一个大于任何 practical list 和 string index（译者注：字符串索引？） 的整数。它符合现实的“自然”整数的大小，并且和上一次同一平台中发布版本中的 `sys.maxint` 相同（如果编译环境完全相同的话）。
- 长整数的 [repr()](https://docs.python.org/3/library/functions.html#repr) 函数不需要再尾随一个 `L` 了，所以无条件 strip 该对象的代码将会删去最后一位数字。（使用 [str()](https://docs.python.org/3/library/stdtypes.html#str) 替代。）
- 八进制表示不再写作 `0720` 这样的形式；而使用例如 `0o720` 的形式来代替。

### Text Vs. Data 取代 Unicode Vs. 8-bit

你此前关于 二进制数据 和 Unicode 的 认识被全部改变了。

- Python 3.0 使用 _text_ 和 （二进制）数据 的概念来取代 Unicode 字符串 和 8位字符串。所有的 text 都是 Unicode；而已编码的 Unicode（数据）则被作为 二进制数据。用来存放 text 的类型是 [str](https://docs.python.org/3/library/stdtypes.html#str)，用来存放数据（data）的数据类型是字节（byte）。Python 3.0 和 2.x 之间最大的区别是任何企图混淆 text 和 data 的做法都会引发 [TypeError](https://docs.python.org/3/library/exceptions.html#TypeError)；而如果你要在 Python 2.x 中将 Unicode 和 8位字符串 混合使用，当 8位字符串 正好只有7位（ASCII）时，是没问题的；但当其中包含非ASCII值时，就会引发 [UnicodeDecodeError](https://docs.python.org/3/library/exceptions.html#UnicodeDecodeError) 错误。这样针对数值的做法在近几年引起了众多的不满。
- 因为这一点原则性的变化，几乎所有使用Unicode，编码或二进制数据的代码都需要更改。这项改变是有建设性的，因为在 2.x 版本中，由编码和非编码字符混合使用导致了非常多的bug。 要在 Python 2.x 中准备好，请对所有未编码的 text 使用 Unicode，并且对二进制或已编码数据使用 [str](https://docs.python.org/3/library/stdtypes.html#str)。 然后 `2to3` 工具将为您完成大部分工作。
- 以后，您不需要再对 Unicode 文本使用 `u"..."` 这样的表达了。然而，您仍然需要对二进制数据使用 `b"..."` 这样的表达。
- 因为 [str](https://docs.python.org/3/library/stdtypes.html#str) 和 [bytes](https://docs.python.org/3/library/stdtypes.html#bytes) 类型不能混用，您必须明确声明两者之间的转换。使用 [str.encode()](https://docs.python.org/3/library/stdtypes.html#str.encode) 来将 [str](https://docs.python.org/3/library/stdtypes.html#str) 转换为 [bytes](https://docs.python.org/3/library/stdtypes.html#bytes)，[bytes.decode()](https://docs.python.org/3/library/stdtypes.html#bytes.decode) 反之。你也可以分别使用 `bytes(s, encoding=...)` 和 `str(b, encoding=...)`。
- 和 [str](https://docs.python.org/3/library/stdtypes.html#str) 一样，[bytes](https://docs.python.org/3/library/stdtypes.html#bytes) 类型也是不可变的。有一个单独的 _可变_ 类型来保存缓冲（buffered）的二进制数据，[bytearray](https://docs.python.org/3/library/stdtypes.html#bytearray)。几乎所有接受 [bytes](https://docs.python.org/3/library/stdtypes.html#bytes) 的接口也接受 [bytearray](https://docs.python.org/3/library/stdtypes.html#bytearray)。可变的 API 基于 collections.MutableSequence。
- 所有在原始（raw）字符串里的反斜杠都会被编译为它字面的意思。这意味着 `'\U'` 和 `'\u'` 在原始（raw）字符串中的转义不会被专门处理。比如，`r'\u20ac'` 在 Python 3.0 中是一个含有6个字符的字符串；而在 2.6 版本中，`ur'\u20ac'` 则表示一个单独的“欧元”符号。（当然，这项改变仅仅影响原始字符串书面形式，在 Python 3.0 中，欧元符号用 `'\u20ac'` 表示。）
- 內建（built-in）的 basestring 抽象类型被移去了。取而代之的是 [str](https://docs.python.org/3/library/stdtypes.html#str)。[str](https://docs.python.org/3/library/stdtypes.html#str) 和 [bytes](https://docs.python.org/3/library/stdtypes.html#bytes) 类型没有足够多的共同功能来共享同一个基类。 `2to3` 工具（见下文）会将所有出现的 basestring 替换成 [str](https://docs.python.org/3/library/stdtypes.html#str)。
- 以文本文件（text files）打开的文件（目前作为 [open()](https://docs.python.org/3/library/functions.html#open) 函数的默认模式）通常使用一种编码方式将（内存中的）字符串和（硬盘上的）字节进行映射。以二进制文件（binary files）打开的文件（打开时在 mode 参数处为 `'b'`）通常在内存中使用字节存储。这意味着，如果一个文件以错误的模式或编码被打开，I/O 会报错，而不是安静地对错误数据进行处理。这也意味着即使是 Unix 的用户也需要在打开文件时制定正确的模式（text 或 binary）。有一种依赖平台（操作系统）的默认编码，在 Unix 系的平台上可以通过环境变量 `LANG` 设置（个别情况下需要通过对平台特异性本地环境相关的环境变量进行设置）。在大多数情况下，但不是全部情形，系统的默认编码是 UTF-8；你永远不要指望这个默认值派大用场（译者注：谁说的！明明很多乱码问题设成 utf-8 就好了，你骗人）。任何读写非纯 ASCII 文本的应用推荐覆写编码。所以再也没有必要在 codecs 模块里，使用需要关注编码的数据流了。
- The initial values of sys.stdin, sys.stdout and sys.stderr are now unicode-only text files (i.e., they are instances of io.TextIOBase). To read and write bytes data with these streams, you need to use their io.TextIOBase.buffer attribute.
- Filenames are passed to and returned from APIs as (Unicode) strings. This can present platform-specific problems because on some platforms filenames are arbitrary byte strings. (On the other hand, on Windows filenames are natively stored as Unicode.) As a work-around, most APIs (e.g. open() and many functions in the os module) that take filenames accept bytes objects as well as strings, and a few APIs have a way to ask for a bytes return value. Thus, os.listdir() returns a list of bytes instances if the argument is a bytes instance, and os.getcwdb() returns the current working directory as a bytes instance. Note that when os.listdir() returns a list of strings, filenames that cannot be decoded properly are omitted rather than raising UnicodeError.
Some system APIs like os.environ and sys.argv can also present problems when the bytes made available by the system is not interpretable using the default encoding. Setting the LANG variable and rerunning the program is probably the best approach.
- PEP 3138: The repr() of a string no longer escapes non-ASCII characters. It still escapes control characters and code points with non-printable status in the Unicode standard, however.
- PEP 3120: The default source encoding is now UTF-8.
- PEP 3131: Non-ASCII letters are now allowed in identifiers. (However, the standard library remains ASCII-only with the exception of contributor names in comments.)
- The StringIO and cStringIO modules are gone. Instead, import the io module and use io.StringIO or io.BytesIO for text and data respectively.
- See also the Unicode HOWTO, which was updated for Python 3.0.
