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

* 不要使用断言来检查用户的输入。如对于一个数字类型，如果根据用户的设计该值得范围应该是 2~10，较好的做法是使用条件判断，并在不符合条件的时候输出错误提示信息。

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
          if w in abbreviations: # 这句性能较差
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

实际上 Python 2.7 以后的版本还有另外一种替代选择——使用第三方模块 `flufl.enum`，它包含两种枚举类：一种是 `Enum`，只要保证枚举值唯一即可，对值得类型没限制；还有一种是 `IntEnum`，其枚举值是 int 型。

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

Python 在最初的设计过程中借鉴了 C 语言的一些