# Python3.0 有什么新东西？

## 常见的绊脚石
如果你使用 Python 2.5，这一部分列出了部分最有可能绊倒你的这些变化。

### Print 是函数
`print` 语句被`print()`函数替代，

```python
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

你也可以自定义元素之间的分隔符。例如：
`print("There are <", 2**32, "> possibilities!", sep="")`
 将输出 `There are <4294967296> possibilities!`
 * 注意：
    * print 函数不会支持旧的 print 语句的 softspace 特性。例如，在 Python 2.X ，print "A\n", "B" 会输出 "A\nB\n"；  
    但是在 Python 3.0  print("A\n", "B") 会输出 "A\n B\n"。
    * 首先，你会发现你自己在交互模式的会输入旧的 print x。是时候训练你的手指打 print(x) 代替了。  
    * 当使用 2to3 代码转换工具，所有的 print 语句会被自动转换为 print() 调用，所以对大型项目这可能是最不容易出现 issue 的地方。

### 用 Views 和迭代器替代列表
很多众所周知的 API 将不会返回列表：
    * dict 的方法：`dict.keys(), dict.items()` 和 `dict.values()` 将返回 views 代替列表。  
    例如：以下代码将不能运行：`k = d.keys(); k.sort()`。使用`k = sorted(d)` 代替。（在 Python2.5 也能运行，一样有效）
    * 当然， dict.iterkeys(), dict.iteritems() and dict.itervalues() 也不会在支持。
    * map() 和 filter() 返回迭代器。如果你真的需要列表，而且输入的每个序列的长度都相等，  
    一个很快的解决方案是用 list() 包装 map()，例如：list(map(...))，  
    但是一个更好的解决办法是使用列表解析（特别是原代码使用了 lambda ），  
    或者重写代码知道不在需要列表。Particularly tricky is map() invoked for the side effects of the function; the correct transformation 
    is to use a regular for loop (since creating a list would just be wasteful).  
    
    如果输出的多个序列的长度不相等，map() 将在短的序列结束时停止。为了 map() 和 Python 2.7 更好的兼容性，可以使用 itertools.zip_longest()  
    包装列表，例如：map(func, *sequences) 变为 list(map(func, itertools.zip_longest(*sequences)))
    * range() 现在和之前的 xrange() 一样使用了，except it works with values of arbitrary size 。后者将不存在。
    * zip() 现在返回的是迭代器。

### 排序比较
Python 3.0 在排序比较有简化了的规则：

*  当操作数没有有意义的自然序，排序比较操作符（<, <=, >=, >）将抛出类型错误异常。  
    也就是说，表达式或者将不在有效，而且将抛出类型错误而不是返回 False。一个推论是  
    对一个异构的列表进行排序将不在起作用-所有的元素和其他元素都可比。注意。这个规则  
    不适用于 == 和 != 操作符：不同类型的对象相比总是不相等的。
* builtin.sorted() 和 list.sort() 将不在接受 cmp 参数用来作为比较函数。使用 key 参数代替。  
    key 参数和 reverse 参数现在是"keyword-only"。
* The cmp() function should be treated as gone, and the `__cmp__()` special method is no longer supported. Use __lt__() for sorting, __eq__() with __hash__(), and other rich comparisons as needed. (If you really need the cmp() functionality, you could use the expression (a > b) - (a < b) as the equivalent for cmp(a, b).)

### 整型数

* PEP0237：本质上来说，long 被重命名为 int，也就是说，现在只有一种内建类型叫做 int，但是它的表现和之前的 long 类型类似。
* PEP0238：像 1/2 这样的表达式返回浮点数。使用 1//2 get the truncating behavior。（后者的语法从 Python 2.2 开始存在很久了）
* sys.maxint 常量被移除。因为整型数不在有大小限制了。然而，sys.maxsize 可以被用作一个比任何真实的列表或字符串的下标大的整型数。  
It conforms to the implementation’s “natural” integer size and is typically the same as sys.maxint in previous releases on the same platform (assuming the same build options).
* The repr() of a long integer doesn’t include the trailing L anymore, so code that unconditionally strips that character will chop off the last digit instead. (Use str() instead.)
* Octal literals are no longer of the form 0720; use 0o720 instead.

### Text Vs. Data Instead Of Unicode Vs. 8-bit

* TODO

## 语法改变概览
这部分是 Python 3.0 每个语法改变的简单概述

### 新语法
* TODO PEP3107：函数参数和返回值注解。这提供了一个注解函数参数和返回值的标准的方法。除了在运行时使用`__annotations__` 属性来访问这些注解以外，没有其他方法。  
    这个目的是为了通过元编程，装饰器和框架鼓励实验。The intent is to encourage experimentation through metaclasses, decorators or frameworks.
* PEP3102：唯关键字参数。
* TODO PEP3104：`nonlocal`语句。使用`nonlocal x`你可以
* PEP3132： 扩充的可迭代对象解包。你现在可以像这样写`a, b, *rest = some_sequence`，甚至是这样`*rest, a = stuff`。rest 对象总是列表（可能为空）；  
    等号右边可以是任何可迭代对象。例如`(a, *rest, b) = range(5)`。这会使 a=0，b=4， rest=[1,2,3]。
* PEP0274：字典推导。`{k: v for k, v in stuff}` 和`dict(stuff)` 作用相同，但是前者更灵活。
* 集合字面值，如`{1, 2}`。注意，`{}`是一个空字典，使用 `set()` 初始化空集合。现在也支持集合推导，如`{x for x in stuff}` 和`set(stuff)`效果相同，但是前者更加灵活。
* 新的八进制字面值，如：`0o720`（2.6 就存在）。旧的八进制字面值`(0720)`已经移除。
* 新的二进制字幕，如： `0b1010` 而且现在有类似的内建方法，`bin`。
* 字节字面值是以 'b' 和 'B' 作为前缀。和内建方法`bytes()`效果相同。

### 改变的语法
* TODO
* as 和 with 成为保留字。（从2.6开始）
* `True, False` 和 `None` 成为保留字。
* 将`except Exception e`改为`except Exception as e`。
* PEP3151：新的 Metaclass 语法。

    ```python
    class C:
        __metaclass__ = M
        ...
    ```

    你现在必须这样用：

    ```python
    class C(metaclass=M):
        ...
    ```
    全局变量 __metalclass__ 现在不在支持。（TODO）

* 列表推导不在支持类似于`[... for var in item1, item2, ...]`这样的语法。使用`[... for var in (item1, item2, ...)]`代替。  
    但是请注意，列表推导有不同的语义：列表解析接近于TODO
* 省略号`(...)`可以作为原子表达式在任何地方使用。（之前只能在切片里面使用）而且它现在只能是`...`（之前可以是`. . . `这样）

### 移除的语法
* 元组参数解包被移除，你不能在写`def foo(a, (b, c)): ...`这样的代码了。使用`def foo(a, b_c): b, c = b_c`代替。
* 移除重音符（用 `repr()` 代替）
* 移除了`<>`(用`!=`代替)
* 移除关键字：`exce()`不在是关键字，它被保留作为函数。（幸运的是这个语法在 2.X 里保留了）。也请注意，`exec`不在接受流式参数，使用 `exec(f.read())` 代替 `exec()`。
* 整形数字面量不在支持l 和 L 作为后缀。
* 字符串字面量不在支持 u 和 U 作为前缀。
* `from module import *`语法只允许在模块级别使用，不能在函数里面使用。
* TODO
* 老式类不在支持。


