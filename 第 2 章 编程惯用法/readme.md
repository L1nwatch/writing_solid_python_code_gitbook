## 第 2 章 编程惯用法
### 建议 8：利用 assert 语句来发现问题

断言（assert）在很多语言中都存在，它主要为调试程序服务，能够快速方便地检查程序的异常或者发现不恰当的输入等，可防止意想不到的情况出现。

其基本语法如下：

`assert expression1, ["," expression2]`，使用的例子如下：

```python
x, y = 1, 2
assert x == y, "not equal"
```

在执行过程中它实际相当于如下代码：

```python
x, y = 1, 2
if __debug__ and not x == y:
    raise AssertionError("not equals")
```

对 Python 中使用断言需要说明如下：

* `__debug__` 的值默认设置为 True，且是只读的，在 Python 2.7 中还无法修改该值。
* 断言是有代价的，它会对性能产生一定的影响，对于编译型的语言这也许并不那么重要，因为断言只在调试模式下启用。但 Python 并没有严格定义调试和发布模式之间的区别，通常禁用断言的方法是在运行脚本的时候加上 `-O` 标志，这种方式带来的影响是它并不优化字节码，而是忽略与断言相关的语句。

断言实际是被设计用来捕获用户所定义的约束的，而不是用来捕获程序本身错误的，因此使用断言需要注意以下几点：

* 不要滥用。若由于断言引发了异常，通常代表程序中存在 bug。因此断言应该使用在正常逻辑不可到达的地方或正常情况下总是为真的场合。

* 如果 Python 本身的异常能够处理就不要再使用断言。如对于类似于数组越界、类型不匹配、除数为 0 之类的错误，不建议使用断言来进行处理。

  ```python
  # 下面的断言是多余的， 如果类型不匹配本身就会抛出异常
  def stradd(x, y):
      assert isinstance(x, basestring)
      assert isinstance(y, basestring)
      return x + y
  ```

* 不要使用断言来检查用户的输入。如对于一个数字类型，如果根据用户的设计该值的范围应该是 2~10，较好的做法是使用条件判断，并在不符合条件的时候输出错误提示信息。

* 在函数调用后，当需要确认返回值是否合理时可以使用断言

* 当条件是业务逻辑继续下去的先决条件时可以使用断言，比如 `list1` 和其副本 `list2`，如果由于某些不可控的因素，如使用了浅拷贝而 `list1` 中含有可变对象等，就可以使用断言来判断这两者的关系。

### 建议 9：数据交换值的时候不推荐使用中间变量

交换两个变量的值，熟悉的代码如下：

```python
temp = x
x = y
y = temp
```

实际上，在 Python 中有 Pythonic 的实现方式，代码如下：

`x, y = y, x`

上面的实现方式不需要借助任何中间变量并且能够获取更好的性能（可以用 `timeit` 测试）。

之所以更优，是因为 Python 表达式计算的顺序。一般情况下 Python 表达式的计算顺序是从左到右，但遇到表达式赋值的时候表达式右边的操作数先于左边的操作数计算，其在内存中执行的顺序如下：

* 先计算右边的表达式 y, x，因此先在内存中创建元组（y, x），其标示符合值分别为 y、x 及其对应的值，其中 y 和 x 是在初始化时已经存在于内存中的对象。
* 计算表达式左边的值并进行赋值，元组被依次分配给左边的标示符，通过解压缩（unpacking），元组第一标识符（为 y）分配给左边第一个元素（此时为 x），元组第二个标识符（为 x）分配给第二个元素（为 y），从而达到实现 x、y 值交换的目的。

Python 的字节码是一种类似汇编指令的中间语言，但是一个字节码指令并不是对应一个机器指令。通过 dis 模块可以进行分析：

```python
import dis
def swap1():
    x, y = 2, 3
    x, y = y, x

def swap2():
    x, y = 2, 3
    temp = x
    x = y
    y = temp
    
dis.dis(swap1)
dis.dis(swap2)
```

通过字节码可以看出，swpa1 对应的字节码中有 2 个 `LOAD_FAST` 指令、2 个 `STORE_FAST` 指令和 1 个 `ROT_TWO` 指令，而 swap2 函数对应的共生成了 3 个 `LOAD_FAST` 指令和 3 个 `STORE_FAST` 指令。而指令 `ROT_TWO` 的主要作用是交换两个栈的最顶层元素，它比执行一个 `LOAD_FAST + STORE_FAST` 指令更快。

### 建议 10：充分利用 Lazy evaluation 的特性

Lazy evaluation 常被译为“延迟计算”或“惰性计算”，指的是仅仅在真正需要执行的时候才计算表达式的值。充分利用 Lazy evaluation 的特性带来的好处主要体现在以下两个方面：

* 避免不必要的计算，带来性能上的提升。对于 Python 中的条件表达式 `if x and y`，在 x 为 false 的情况下 y 表达式的值将不再计算。而对于 `if x or y`，当 x 的值为 true 的时候将直接返回，不再计算 y 的值。因此编程中应该充分利用该特性。例如：

  ```python
  from time import time
  t = time()
  abbreviations = ["cf.", "e.g.", "ex.", "etc.", "flg."]
  for i in xrange(100000):
      for w in ("Mr.", "Hat", "is", "chasing", "."):
          if w in abbreviations and w[-1]=='.': # 这句性能较差
          # if w[-1] == '.' and w in abbreviations: # 性能好
              pass
  print time() - t
  ```

  如果使用注释的那一条 if 语句，运行的时间大约会节省 10%。总结来说，对于 or 条件表达式应该将值为真可能性较高的变量写在 or 的前面，而 and 则应该推后。

* 节省空间，使得无限循环的数据结构成为可能。Python 中最典型的使用延迟计算的例子就是生成器表达式了。比如斐波那契：

  ```python
  def fib():
      a, b = 0, 1
      while True:
          yield a
          a, b = b, a + b
  from itertools import islice
  print list(islice(fib(), 5))
  ```

### 建议 11：理解枚举替代实现的缺陷

枚举最经典的例子是季节和星期，它能够以更接近自然语言的方式来表达数据，使得程序的可读性和可维护性大大提高。但是枚举类型在 Python 3.4 以前却并不提供。人们充分利用 Python 的动态性这个特征，想出了枚举的各种替代实现方式：

* 使用类属性

  ```python
  class Seasons:
      Spring, Summer, Autumn, Winter = 0, 1, 2, 3
  # 或者可以简化为：
  class Seasons:
      Spring, Summer, Autumn, Winter = range(4)
  ```

* 借助函数

  ```python
  def enum(*posarg, **keysarg):
      return type("Enum", (object,), dict(zip(posarg, xrange(len(posarg))), **keysarg))

  Seasons = enum("Spring", "Summer", "Autumn", Winter=1)
  ```

* 使用 `collections.namedtuple` 

  ```python
  Seasons = namedtuple("Seasons", "Spring Summer Autumn Winter")._make(range(4))
  ```

显然，这些替代实现有其不合理的地方：

* 允许枚举值重复，比如在 `collections.namedtuple` 中，使得枚举值 Spring 和 Autumn 相等，却不会提示任何错误：`Seasons._replace(Spring = 2)`
* 支持无意义的操作，比如 `Seasons.Summer + Seasons.Autumn == Season.Winter`

实际上 Python 2.7 以后的版本还有另外一种替代选择——使用第三方模块 `flufl.enum`，它包含两种枚举类：一种是 `Enum`，只要保证枚举值唯一即可，对值的类型没限制；还有一种是 `IntEnum`，其枚举值是 int 型。

```python
from flufl.enum import Enum
class Seasons(Enum): # 继承自 Enum 定义枚举
    Spring = "Spring"
    Summer = 2
    Autumn = 3
    Winter = 4
Seasons = Enum("Seasons", "Spring Sumter Autumn Winter")
```

`flufl.enum` 提供了 `__members__` 属性，可以对枚举名称进行迭代。

```python
for member in Seasons.__members__:
    print member
```

可以直接使用 value 属性获取枚举元素的值，比如：

`print Seasons.Summer.value`

`flufl.enum` 不支持枚举元素的比较。比如不支持 `Seasons.Summer < Seasons.Autumn`

Python3.4 中根据 PEP435 加入了枚举 Enum，其实现主要参考 `flufl.enum`，但两者之间还是存在一些差别，如 `flufl.enum` 允许枚举继承，而 `Enum` 仅在父类没有任何枚举成员的时候才允许继承等。如果要在 Python3.4 之前的版本中使用枚举 Enum，可以安装 Enum 的向后兼容包 enum34。

### 建议 12：不推荐使用 type 来进行类型检查

作为动态性的强类型脚本语言，Python 中的变量在定义的时候并不会指明具体类型，Python 解释器会在运行时自动进行类型检查并根据需要进行隐式类型转换。按照 Python 的理念，为了充分利用其动态性的特征是不推荐进行类型检查的。解释器能够根据变量类型的不同而调用合适的内部方法进行处理，而当 a、b 类型不同而两者之间又不能进行隐式类型转换时便抛出 TypeError 异常。

不刻意进行类型检查，而是在出错的情况下通过抛出异常来进行处理，这是较为常见的方式。但实际应用中为了提高程序的健壮性，仍然会面临需要进行类型检查的情景。

内建函数 `type(object)` 用于返回当前对象的类型，因此可以通过与 Python 自带模块 types 中所定义的名称进行比较，根据其返回值确定变量类型是否符合要求。

所有基本类型对应的名称都可以在 `types` 模块中找到，然而使用 `type()` 函数并不适合用来进行变量类型检查。

* 基于内建类型扩展的用户自定义类型，type 函数并不能准确返回结果
* 在古典类中，所有类的实例的 type 值都相等

对于内建的基本类型来说，使用 `type()` 进行类型检查问题不大，但在某些特殊场合 `type()` 方法并不可靠。解决方法是，如果类型有对应的工厂函数，可以使用工厂函数对类型做相应转换，否则可以使用 `isinstance()` 函数来检测。

```python
isinstance(object, classinfo)
# 其中，classinfo 可以为直接或间接类名、基本类型名称或者由它们组成的元组，该函数在 classinfo 参数错误的情况下会抛出 TypeError 异常。
# isinstance 基本用法举例如下：
>>> isinstance(2, float)
False
>>> isinstance("a", (str, unicode))
True
>>> isinstance((2, 3), (str, list, tuple)) # 支持多种类型列表
True
```

###建议 13：尽量转换为浮点类型后再做除法

* 【PS：这应该是 Python2 中存在的问题】

Python 在最初的设计过程中借鉴了 C 语言的一些规则，比如选择 C 的 long 类型作为 Python 的整数类型，double 作为浮点类型等。同时标准的算术运算，包括除法，返回值总是和操作数类型相同。作为静态类型语言，C 语言中这一规则问题不大，因为变量都会预先申明类型，当类型不符的时候，编译器也会尽可能进行强制类型转换，否则编译会报错。但 Python 作为一门高级动态语言并没有类型申明这一说。

Python 中除了除法运算之外，整数和浮点数的其他操作行为还是一致的，因此这容易让人产生一种误解，数值的计算与具体操作数的类型无关，但事实上对于整数除法这是编程过程中潜在的一个危险。推荐的做法之一是当涉及除法运算的时候尽量先将操作数转换为浮点类型再做运算。

在 Python3 中这个问题已经不存在了。Python3 之前的版本可以通过 `from __future__ import division` 机制使整数除法不再截断，这样即使不进行浮点类型转换，输出结果也是正确的。

还需要说明一点，浮点数可能是不准确的，比如：

```python
i = 1
while i != 1.5:
    i = i + 0.1
    print i
```

这段代码会导致无限循环，在内存中根据浮点数位数规定，多余部分直接截断。对于浮点数的处理，要记住其运算结果可能并不是完全准确的。如果计算对精度要求较高，可以使用 Decimal 来进行处理或者将浮点数尽量扩大为整数，计算完毕之后再转换回去。而对于在 while 中使用 `i != 1.5` 这种条件表达式更是要避免的，浮点数的比较同样最好能够指明精度。

### 建议 14：警惕 `eval()` 的安全漏洞

Python 中`eval()` 函数将字符串 str 当成有效的表达式来求值并返回计算结果。其函数声明如下：`eval(expression[, globals[, locals]])`。

其中，参数 globals 为字典形式，locals 为任何映射对象，它们分别表示全局和局部命名空间。如果传入 globals 参数的字典中缺少 `__builtins__` 的时候，当前的全局命名空间将作为 globals 参数输入并且在表达式计算之前被解析。locals 参数默认与 globals 相同，如果两者都省略的话，表达式将在 `eval()` 调用的环境中执行。

eval 存在安全漏洞，一个简单的例子：

```python
import sys
from math import *
def ExpCalcBot(string):
    try:
        print "Your answer is", eval(user_func) # 计算输入的值
    except NameError:
        print "The expression you enter is not valid"
print 'Hi, I am ExpCalcBot. please input your expression or enter e to end'
inputstr = ''
while True:
    print 'Please enter a number or operation. Enter c to complete. :'
    inputstr = raw_input()
    if inputstr == str('e'): # 遇到输入为 e 的时候退出
        sys.exit()
    elif repr(inputstr) != repr(''):
        ExpCalcBot(inputstr)
        inputstr = ''
```

由于网络环境下运行它的用户并非都是可信任的，比如输入 `__import__("os").system("dir")` ，它会显示当前目录下的所有文件列表；输入 `__import__("os").system("del * /Q")`，会导致当前目录下的所有文件都被删除了，而这一切没有任何提示。

如果在 globals 参数中禁止全局命名空间的访问：

```python
def ExpCalcBot(string):
    try:
        math_fun_list = ["acos", "asin", "atan", "cos", "e", "log", "log10", "pi", "pow", "sin", "sqrt", "tan"]
        math_fun_dict = dict([(k, globals().get(k)) for k in math_fun_list]) # 形成可以访问的函数的字典
        print "Your name is", eval(string, {"__builtins__": None}, math_fun_dict)
    except NameError:
        print "The expression you enter is not valid"
```

再次进行恶意输入：`[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == "Quitter"][0](0)()`，`# ().__class__.__bases__[0].__subclasses__()` 用来显示 object 类的所有子类。类 Quitter 与 "quit" 功能绑定，因此上面的输入会导致程序退出。

注：可以在 Python 的安装目录下的 `Lib\site.py` 中找到其类的定义。也可以在解释器中输入查看输出结果。

对于有经验的侵入者来说，他可能会有一系列强大的手段，使得 eval 可以解释和调用这些方法，从而带来更大的破坏。此外，`eval()` 函数也给程序的调试带来一定困难，要查看 `eval()` 里面表达式具体的执行过程很难。因此在实际应用过程中如果使用对象不是信任源，应该避免使用 eval，在需要使用 eval 的地方可用安全性更好的 `ast.literal_eval` 替代。

### 建议 15：使用 `enumerate()` 获取序列迭代的索引和值

有 N 种实现方法，举例如下：

```python
# 方法一：在每次循环中对索引变量进行自增
li = ['a', 'b', 'c', 'd']
index = 0
for i in li:
    print "index:", index, "element:", i
    index += 1

# 方法二：使用 range() 和 len() 方法结合
li = ['a', 'b', 'c', 'd', 'e']
for i in range(len(li)):
    print "index:", i, "element:", li[i]
    
# 方法三：使用 while 循环，用 len() 获取循环次数
li = ['a', 'b', 'c', 'd', 'e']
index = 0
while index < len(li):
    print "index:", index, "element:", li[index]
    index += 1

# 方法四：使用 zip() 方法
li = ['a', 'b', 'c', 'd', 'e']
for i, e in zip(range(len(li)), li):
    print "index:", i, "element:", e
    
# 方法五：使用 enumerate() 获取序列迭代的索引和值
li = ['a', 'b', 'c', 'd', 'e']
for i, e in enumerate(li):
    print "index:", i, "element:", e
```

推荐使用函数 `enumerate()`，主要是为了解决在循环中获取索引以及对应值的问题。它具有一定的惰性（lazy），每次仅在需要的时候才会产生一个（index, item）对。其函数签名如下：`enumerate(sequence, start=0)`

其中，sequence 可以为序列，如 list、set 等，也可以为一个 iterator 或者任何可以迭代的对象，默认的 start 为 0，函数返回本质上为一个迭代器，可以使用 next() 方法获取下一个迭代元素。

`enumerate()` 函数的内部实现非常简单，`enumerate(sequence, start=0)` 实际相当于如下代码：

```python
def enumerate(sequence, start=0):
    n = start
    for elem in sequence:
        yield n, elem
        n += 1
```

因此利用这个特性用户还可以实现自己的 `enumerate()` 函数。比如，`myenumerate()` 以反序的方式获取序列的索引和值。

```python
def my_enumerate(sequence):
    n = -1
    for elem in reversed(sequence):
        yield len(sequence) + n, elem
        n -= 1
```

需要提醒的是，对于字典的迭代循环，`enumerate()` 函数并不适合，虽然在使用上并不会提示错误，但输出的结果与期望的大相径庭，这是因为字典默认被转换成了序列进行处理。要获取迭代过程中字典的 key 和 value，应该使用 `iteritems` 方法。

### 建议 16：分清 == 与 is 的适用场景

可以通过 `id()` 函数来看看变量在内存中具体的存储空间。

is 表示的是对象标示符（object identity），而 == 表示的意思是相等（equal）。is 的作用是用来检查对象的标示符是否一致的，也就是比较两个对象在内存中是否拥有同一块内存空间，它并不适合用来判断两个字符串是否相等。`x is y` 仅当 x 和 y 是同一个对象的时候才返回 True，`x is b` 基本相当于 `id(x) == id(y)` 。而 == 才是用来检验两个对象的值是否相等的，它实际调用内部 `__eq__()` 方法，因此 `a == b` 相当于 `a.__eq__(b)`，所以 == 操作符也是可以被重载的，而 is 不能被重载。一般情况下，如果 `x is y` 为 True 的话 `x == y` 的值也为 True（特殊情况除外，如 NaN，`a = floag('NaN')`，`a is a` 为 True，`a == a` 为 false）。

Python 中存在 `string interning` （字符串驻留）机制，对于较小的字符串，为了提高系统性能会保留其值的一个副本，当创建新的字符串的时候直接指向该副本即可。

### 建议 17：考虑兼容性，尽可能使用 Unicode

Python 内建的字符串有两种类型：`str` 和 `Unicode`，它们拥有共同的祖先 `basestring`。其中，`Unicode` 是 Python2.0 中引入的一种新的数据类型，所有的 `Unicode` 字符串都是 `Unicode` 类型的实例。

```python
# 创建一个 Unicode 字符
str_unicode = u"unicode" # 前面加 u 表示 Unicode
```

Unicode 编码系统可以分为编码方式和实现方式两个层次。在编码方式上，分为 `UCS-2` 和 `UCS-4` 两种方式，`UCS-2` 用两个字节编码，`UCS-4` 用 4 个字节编码。目前实际应用的统一码对应于 `UCS-2`，使用 16 位的编码空间。一个字符的 `Unicode` 编码是确定的，但是在实际传输过程中，由于系统平台的不同以及出于节省空间的目的，实现方式有所差异。Unicode 的实现方式称为 Unicode 转换格式（Unicode Transformation Format），简称为 UTF，包括 `UTF-7`、`UTF-16`、`UTF-32`、`UTF-8` 等，其中较为常见的为 `UTF-8`。`UTF-8` 的特点是对不同范围的字符使用不同长度的编码，其中 `0x00 ~ 0x7F` 的字符的 `UTF-8` 编码与 ASCII 编码完全相同。`UTF-8` 编码的最大长度是 4 个字节，从 Unicode 到 `UTF-8` 的编码方式如下所示：

| Unicode 编码（十六进制） | UTF-8 字节流（二进制）                      |
| ---------------- | ----------------------------------- |
| 000000 ~ 00007F  | 0xxxxxxx                            |
| 000080 ~ 0007FF  | 110xxxxx 10xxxxxx                   |
| 000800 ~ 00FFFF  | 1110xxxx 10xxxxxx 10xxxxxx          |
| 010000 ~ 10FFFF  | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

Python 中处理中文字符经常会遇到的几个问题：

* 读出文件的内容显示为乱码
* 当 Python 源文件中包含中文字符的时候抛出 SyntaxError 异常
* 普通字符和 Unicode 进行字符串连接的时候抛出 UnicodeDecodeError 异常

对字符串进行解码和编码，其中 `decode()` 方法将其他编码对应的字符串解码成 Unicode，而 `encode()` 方法将 Unicode 编码转换为另一种编码，Unicode 作为转换过程中的中间编码。`decode()` 和 `encode()` 方法的函数形式如下：

```python
# str.decode([编码参数 [, 错误处理]])
# str.encode([编码参数 [, 错误处理]])
```

常见的编码参数：

| 编码参数                  | 描述                           |
| --------------------- | ---------------------------- |
| ascii                 | 7 位 ASCII 码                  |
| laten-1 or iso-8859-1 | ISO 8859-1，Latin-1           |
| utf-8                 | 8 位可变长度编码                    |
| utf-16                | 16 位可变长度编码                   |
| utf-16-le             | UTF-16，little-endian 编码      |
| utf-16-be             | UTF-16，big-endian 编码         |
| unicode-escapse       | 与 unicode 文字 u'string' 相同    |
| raw-unicode-escape    | 与原始 Unicode 文字 ur'string' 相同 |

错误处理参数有以下 3 种常用方式：

* strict：默认处理方式，编码错误抛出 UnicodeError 异常
* ignore：忽略不可转换字符
* replace：将不可转换字符用 ? 代替

有些软件在保存 `UTF-8` 编码的时候，会在文件最开始的地方插入不可见的字符 BOM（`0xEF 0xBB 0xBF`，即 BOM），这些不可见字符无法被正确的解析，而利用 codecs 模块可以方便地处理这种问题。

```python
import codecs
content = open("test.txt", "r").read()
filehandle.close()
if content[:3] == codecs.BOM_UTF8: # 如果存在 BOM 字符则去掉
    content = content[3:]
print content.decode("utf-8")
```

#### 关于 BOM：

Unicode 存储有字节序的问题，`UTF-16` 以两个字节为编码单元，在字符的传送过程中，为了标明字节的顺序，Unicode 规范中推荐使用 BOM（Byte Order Mark）：即在 UCS 编码中用一个叫做 `ZERO WIDTH NO-BREAK SPACE` 的字符，它的编码是 `FEFF` （该编码在 UCS 中不存在对应的字符），UCS 规范建议在传输字节流前，先传输字符 `ZERO WIDTH NO-BREAK SPACE`。这样如果接收者收到 FEFF，就标明这个字节流是 `Big-Endian` 的；如果收到 FFFE，就表明这个字节流是 `Little-Endian` 的。`UTF-8` 使用字节来编码，一般不需要 BOM 来表明字节顺序，但可以用 BOM 来表明编码方式。字符 `ZERO WIDTH NO-BREAK SPACE` 的 `UTF-8` 编码是 `EF BB BF`。所以如果接收者收到以 `EF BB BF` 开头的字节流，就知道这是 `UTF-8` 编码了。

***

Python 中的默认编码，可以通过 `sys.getdefaultencoding()` 来验证）。当调用 print 方法输出的时候会隐式地进行从 ASCII 到系统默认编码（Windows 上为 CP936）的转换。要避免这种错误需要在源文件中进行编码声明，声明可用正则表达式：`coding=[:=]\s*([-\w.]+)` 表示。一般来说进行源文件编码声明有以下三种方式：

* `# coding=<encoding name>`，比如 `#coding=utf-8`

* 第二种

```python
#!/usr/bin/python
# -*- coding: <encoding name> -*-
```

* 第三种

```python
#!/usr/bin/python
# vim: set fileencoding=<encoding name> :
```

Python2.6 之后可以通过 `import unicode_literals` 自动将定义的普通字符识别为 Unicode 字符串，这样字符串的行为将和 Python3 中保持一致。

### 建议 18：构建合理的包层次来管理 module

本质上每一个 Python 文件都是一个模块，使用模块可以增强代码的可维护性和可重用性。但在大的项目中将所有的 Python 文件放在一个目录下并不是一个值得推荐的做法，需要合理地组织项目的层次来管理模块，这就是包（Package）发挥功效的地方了。

简单说包即是目录，但与普通目录不同，它除了包含常规的 Python 文件（也就是模块）以外，还包含一个 `__init__.py` 文件，同时它允许嵌套。

包中的模块可以通过"."访问符进行访问，即"包名.模块名"。有以下几种导入方法：

* 直接导入一个包：`import Package`

* 导入子模块或子包，包嵌套的情况下可以进行嵌套导入，具体如下：

  ```python
  from Package import Module1
  import Package.Module1
  from Package import Subpackage
  import Package.Subpackage
  from Package.Subpackage import Module1
  import Package.Subpackage.Module1
  ```

`__init__.py` 最明显的作用就是使包和普通目录区分；其次可以在该文件中申明模块级别的 `import` 语句从而使其变成包级别可见。如果 `__init__.py` 文件为空，当意图使用 `from Package import *` 将包 Package 中所有的模块导入当前名字空间时并不能使得导入的模块生效，这是因为不同平台间的文件的命名规则不同，Python 解释器并不能正确判定模块在对应的平台该如何导入，因此它仅仅执行 `__init__.py` 文件，如果要控制模块的导入，则需要对 `__init__.py` 文件做修改。

`__init__.py` 文件还有一个作用就是通过在该文件中定义 `__all__` 变量，控制需要导入的子包或者模块。之后再运行 `from ... import *`，可以看到 `__all__` 变量中定义的模块和包被导入当前名字空间。

包的使用能够带来以下便利：

* 合理组织代码，便于维护和使用
* 能够有效地避免名称空间冲突

如果模块包含的属性和方法存在同名冲突，使用 `import module` 可以有效地避免名称冲突。在嵌套的包结构中，每一个模块都以其所在的完整路径作为其前缀，因此，即使名称一样，但由于模块所对应的其前缀不同，因此不会产生冲突。
