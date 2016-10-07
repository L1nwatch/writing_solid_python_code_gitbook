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

