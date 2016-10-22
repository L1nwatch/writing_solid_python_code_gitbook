# 编写高质量代码改善 Python 程序的 91 个建议

## 第 1 章 引论

### 建议 1：理解 Pythonic 概念

对于 Pythonic 的概念，有一个更具体的指南，《The Zen of Python》（Python 之禅）。有几点非常深入人心：

* 美胜丑，显胜隐，简胜杂，杂胜乱，平胜陡，疏胜密。
* 找到简单问题的一个方法，最好是唯一的方法（正确的解决之道）
* 难以解释的实现，源自不好的注意；如果有非常棒的注意，它的实现肯定易于理解

#### Pythonic 的定义

遵循 Pythonic 的定义，看起来就像是伪代码。其实，所有的伪代码都可以轻易地转换为可执行的 Python 代码。

Pythonic 也许可以定义为：充分体现 Python 自身特色的代码风格。

#### 代码风格

在语法上，代码风格要充分表现 Python 自身特色。比如说两个变量交换，利用 Python 的 packaging/unpackaging 机制，Pythonic 的代码只需要以下一行：

`a, b = b, a`。

还有，在遍历一个容器的时候：

```python
for i in a_list:
    do_sth_with(i)
```

灵活地使用迭代器是一种 Python 风格。比如说，需要安全地关闭文件描述符，可以使用以下 with 语句：

```python
with open(path, 'r') as f:
    do_sth_with(f)
```

应当追求的是充分利用 Python 语法，但不应当过分地使用奇技淫巧，比如利用 Python 的 Slice 语法，可以写出如下代码：

```python
a = [1, 2, 3, 4]
c = 'abcdef'
print(a[::-1])
print(c[::-1])
```

实际上，这个时候更好地体现 Pythonic 的代码时充分利用 Python 库里的 `reversed()` 函数的代码。

```python
print list(reversed(a))
print list(reversed(c))
```

#### 标准库

写 Pythonic 程序需要对标准库有充分的理解，特别是内置函数和内置数据类型。比如，对于字符串格式化，推荐这样写：

```python
print '(greet) from (language).'.format(greet = "Hello world", language = 'Python')
```

`str.format()` 方法非常清晰地表明了这条语句的意图，而且模板的使用也减少了许多不必要的字符，使可读性得到了很大的提升。

#### Pythonic 的库或框架

编写应用程序的时候的要求会更高一些。因为编写应用程序一般需要团队合作，那么可能你编写的那一部分正好是团队的另一成员需要调用的接口，换言之，你可能正在编写库或框架。

程序员利用 Pythonic 的库或框架能更加容易、更加自然地完成任务。如果用 Python 编写的库或框架迫使程序员编写累赘的或不推荐的代码，那么可以说它并不 Pythonic。现在业内通常认为 Flask 这个框架是比较 Pythonic 的，它的一个 Hello world 级别用例如下：

```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return "Hello world!"

if __name__ == '__main__':
    app.run()
```

一个 Pythonic 的框架不会对已经通过惯用法完成的东西重复发明“轮子”，而且它也遵循常用的 Python 惯例。创建 Pythonic 的框架及其困难，现在认为像 generators 之类的特性尤为 Pythonic。

另一个有关新趋势的例子是，Python 的包和模块结构日益规范化。现在的库或框架跟随了一下潮流：

* 包和模块的命名采用小写、单数形式，而且短小
* 包通常仅作为命名空间，如只包含空的 `__init__.py` 文件。

## 建议 2：编写 Pythonic 代码

* 要避免劣化代码，比如不合适的变量命名等

  * 避免只用大小写来区分不同的对象

  * 避免使用容易引起混淆的名称，包括：

    * 重复使用已经存在于上下文中的变量名来表示不同的类型
    * 误用了内建名称来表示其他含义的名称而使之在当前命名空间被屏蔽
    * 没有构建新的数据类型的情况下使用类似于 `element、list、dict` 等作为变量名
    * 使用 o（字母 O 小写的形式，容易与数值 0 混淆）、1（字母 L 小写的形式，容易与数字 1 混淆）等作为变量名

  * 以下两个示例，示例二比示例一好：

    ```python
    # 示例一：
    def funA(list, num):
        for element in list:
            if num == element:
                return True
            else:
                pass

    # 示例二：
    def find_num(searchList, num):
        for listValue in searchList:
            if num == listValue:
                return True
            else:
                pass
    ```

  * 不要害怕过长的变量名，比如 `person_info` 比 `pi` 的可读性要强得多。

* 深入认识 Python 有助于编写 Pythonic 代码

  * 全面掌握 Python 提供给我们的所有特性，包括语言特性和库特性。其中最好的学习方式应该是通读官方手册中的 `Language Reference` 和 `Library Reference`。

  * 一方面 Python 语言推荐使用大量的惯用法来完成任务；另一方面，语言的进化又趋于更好地支持惯用法。

  * 深入学习业界公认的比较 Pythonic 的代码，比如 `Flask、gevent 和 requests` 等。而使用 Python 的标准库 httplib2 时，代码就非常复杂，程序员需要了解相当多的关于 `HTTP` 协议和 `Basic Auth` 的知识才能编程。

  * 最后，也可以尝试利用工具达到事半功倍的效果。比如风格检查程序 PEP8。其实一开始 PEP 8 是一篇关于 Python 编码风格的指南，它提出了保持代码一致性的细节要求。它至少包括了对代码布局、注释、命名规范等方面的要求，在代码中遵循这些原则，有利于编写 Pythonic 的代码。

    * 应用程序 PEP 8 可以用来进行检测，它是使用 Python 开发的，安装的方法：`pip install -U pep8`

    * 然后即可简单地用它检测一下自己的代码：`pep8 --first optparse.py`，如果嫌报表不够细致，可以考虑使用 `--show-source` 参数让 `PEP8` 显示每一个错误和警告对应的代码。

    * 比如，对代码的换行，不好的风格如下：

      ```python
      if foo == "blah": do_blah_thing()
      do_one(); do_two(); do_three()
      ```

      而 Pythonic 的风格则是这样的：

      ```python
      if foo == "blah":
          do_blah_thing()
      do_one()
      do_two()
      do_three()
      ```

    * `PEP8` 程序，甚至还可以给出“正确”的写法，除了针对某一个源代码文件以外，还可以直接检测一个项目的质量，并通过直观的报表给出报告。

    * `PEP8` 有优秀的插件架构，可以方便地实现特定风格的检测；它生成的报告易于处理，可以很方便地与编辑器集成。

    * `PEP8` 不是唯一的编程规范。有些公司制定的编程规范也非常有参考意义，比如 `Google Python Style Guide`。同样，`PEP8` 也不是唯一的风格检测程序，类似的应用还有 `Pychecker、Pylint、Pyflakes` 等。其中 `Pychecker` 是 `Google Python Style Guide` 推荐的工具；`Pylint` 因可以非常方便地通过编辑配置文件实现公司或团队的风格检测而受到许多人的青睐；`Pyflakes` 则因为易于集成到 vim 中，所以使用的人也非常多。

## 建议 3：理解 Python 与 C 语言的不同之处

Python 底层是用 C 语言实现的，但切忌用 C 语言的思维和风格来编写 Python 代码。

1. “缩进”与“{}”

   避免缩进带来的困扰的方法之一就是养成良好的习惯，统一缩进风格，不要混用 Tab 键和空格。

2. ‘与“

   C 语言中单引号代表一个字符，它实际对应于编译器所采用的字符集中的一个整数值。而双引号则表示字符串，默认以 '\0' 结尾。

3. 三元操作符 ”?:“

   `C?X:Y` 在 Python 中等价的形式为 `X if C else Y`

4. `switch...case`

   Python 中可以使用多种方式来实现 `switch...case` 比如下面这一种：

   ```C
   switch(n) {
     case 0:
       printf("0");
       break;
     case 1:
       printf("1");
       break;
     case 2:
       printf("2");
       break;
     default:
       printf("????");
       break;
   }
   ```

   ```python
   def f(x):
       return {
         0: "0",
         1: "1",
         2: "2"
       }.get(n, "???")
   ```

### 建议 4：在代码中适当添加注释

Python 中有 3 种形式的代码注释：块注释、行注释以及文档注释（docstring）。这 3 种形式的惯用法大概有如下几种：

* 使用块或者行注释的时候仅注释那些复杂的操作、算法，还有可能别人难以理解的技巧或者不够一目了然的代码
* 注释和代码隔开一定的距离，同时在块注释之后最好多留几行空白再写代码。
* 给外部可访问的函数和方法添加文档注释。注释要清楚地描述方法的功能，并对参数、返回值以及可能发生的异常进行说明，使得外部调用它的人员仅仅看 docstring 就能正确使用。（使用 `""""""` 进行注释）
* 推荐在头文件中包含 copyright 申明、模块描述等，如有必要，可以考虑加入作者信息以及变更记录
* 需要避免的注释：
  * 代码即注释（不写注释）
  * 注释与代码重复
  * 利用注释语法快速删除代码。对于不再需要的代码，应该将其删除，而不是将其注释掉。即使担心以后还会用到，版本控制工具也可以让你轻松找回被删除的代码。
  * 代码不断更新而注释却没有更新
  * 注释比代码本身还复杂繁琐
  * 将别处的注释和代码一起拷贝过来，但上下文的变更导致注释与代码不同步
  * 将注释当作自己的娱乐空间从而留下个性特征

### 建议 5：通过适当添加空行使代码布局更为优雅、合理

Python 代码布局有一些基本规则可以遵循：

* 在一组代码表达完一个完整的思路之后，应该用空白行进行间隔。如每个函数之间，导入声明、变量赋值等。通俗点讲就是不要在一段代码中说明几件事。推荐在函数定义或者类定义之间空两行，在类定义与第一个方法之间，或者需要进行语义分割的地方空一行。
* 尽量保持上下文语义的易理解性
* 避免过长的代码行，每行最好不要超过 80 个字符。超过的部分可以用圆括号、方括号和花括号等进行行连接，并且保持行连接的元素垂直对齐。
* 不要为了保持水平对齐而使用多余的空格，其实使阅读者尽可能容易地理解代码所要表达的意义更重要。
* 空格的使用要能够在需要强调的时候警示读者，在疏松关系的实体间起到分隔作用，而在具有紧密关系的时候不要使用空格。具体细节如下：
  * 二元运算符（赋值、比较、布尔运算的左右两边应该有空格）
  * 逗号和分号前不要使用空格
  * 函数名和左括号之间、序列索引操作时序列名和 [] 之间不需要空格，函数的默认参数两侧不需要空格
  * 强调前面的操作符的时候使用空格

### 建议 6：编写函数的几个原则

函数能够带来最大化的代码重用和最小化的代码冗余。精心设计的函数不仅可以提高程序的健壮性，还可以增强可读性、减少维护成本。

一般来说函数设计有以下基本原则可以参考：

* 原则 1：函数设计要尽量短小，嵌套层次不宜过深。最好能控制在 3 层以内。
* 原则 2：函数申明应该做到合理、简单、易于使用。参数个数不宜太多。
* 原则 3：函数参数设计应该考虑向下兼容。比如相同功能的函数不同版本的实现，唯一不同的是在更高级的版本中添加了参数导致程序中函数调用的接口发生了改变。这并不是最佳设计，更好的方法是通过加入默认参数来避免这种退化，做到向下兼容。
* 原则 4：一个函数只做一件事，尽量保证函数语句粒度的一致性。
* 原则 5：不要在函数中定义可变对象作为默认值
* 原则 6：使用异常替换返回错误
* 原则 7：保证通过单元测试

### 建议 7：将常量集中到一个文件

Python 的内建命名空间是支持一小部分常量的，例如 True、False、None 等，只是 Python 没有提供定义常量的直接方式而已。

在 Python 中应该如何使用常量？一般来说有以下两种方式：

* 通过命名风格来提醒使用者该变量代表的意义为常量，如常量名所有字母大写，用下划线连接各个单词

* 通过自定义的类实现常量功能。这要求符合”命名全部为大写“和”值一旦绑定便不可再修改“这两个条件。下面是一种较为常见的解决办法，它通过对常量对应的值进行修改时或者命名不符合规范时抛出异常来满足以上常量的两个条件。

  ```python
  class _const:
      class ConstError(TypeError):
          pass
      class ConstCaseError(ConstError):
          pass
      
      def __setattr__(self, name, value):
          if self.__dict__.has_key(name):
              raise self.ConstError, "Can't change const.{}".format(name)
          if not name.isupper():
              raise self.ConstCaseError, "const name {} is not all uppercase".format(name)
          self.__dict__[name] = value

  import sys
  sys.modules[__name__] = _const()
  ```

  如果上面的代码对应的模块名为 const，使用时候只需要 import const，便可以直接定义常量了，如以下代码：

  ```python
  import const
  const.COMPANY = "IBM"
  ```

* 无论采用哪一种方式来实现常量，都提倡将常量集中到一个文件中，因为这样有利于维护，一旦需要修改常量的值，可以集中统一而不是逐个文件去检查。采用第二种方式实现的常量可以这么做：将存放常量的文件命名为 `constant.py` ，并在其中定义一系列的常量。

  ```python
  class _const:
  	[...]
      
  import sys
  sys.modules[__name__] = _const()
  import const
  const.MY_CONSTANT = 1
  const.MY_SECOND_CONSTANT = 2
  ```

  当在其他模块中引用这些常量时，按照如下方式进行即可：

  ```python
  from constant import const
  print(const.MY_SECOND_CONSTANT)
  ```

  ​