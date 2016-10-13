## 第 4 章 库

### 建议 36：掌握字符串的基本用法

利用 Python 遇到未闭合的小括号时会自动将多行代码拼接为一行和把相邻的两个字符串字面量拼接在一起。相比使用 3 个连续的单（双）引号，这种方式不会把换行符和前导空格当作字符串的一部分，则更加符合用户的思维习惯。

Python 中的字符串其实有 str 和 unicode 两种，虽然在 Python3 中已经简化为一种，但如果还在编写运行 Python2 上的程序，当需要判断变量是否为字符串时，应该使用 `isinstance(s, basestring)`，这里的参数是 `basestring` 而不是 `str`。因为 `basestring` 才是 `str` 和 `unicode` 的基类，包含了普通字符串和 `unicode` 类型。

需要注意的是 `*with()` 函数族可以接受可选的 `start`、`end` 参数，善加利用，可以优化性能。另外，自 Python2.5 版本起，`*wtih()` 函数族的 `prefix` 参数可以接受 `tuple` 类型的实参，当实参中的某个元素能够匹配时，即返回 `True`。

查找与替换，`count(sub[, start[, end]])、find(sub[, start[, end]])、index(sub[, start[, end]])、rfind(sub[, start[, end]])、rindex(sub[, start[, end]])` 这些方法都接受 `start`、`end` 参数，善加利用，可以优化性能。其中 `count()` 能够查找子串 `sub` 在字符串中出现的次数，这个数值在调用 `replace` 方法的时候用得着。此外，需要注意 `find()` 和 `index()` 方法的不同：`find()` 函数族找不到时返回 -1，`index()` 函数族则抛出 `ValueError` 异常。但对于判定是否包含子串的判定并不推荐调用这些方法，而是推荐使用 `in` 和 `not in` 操作符。

`replace(old, new[, count])` 用以替换字符串的某些子串，如果指定 `count` 参数的话，就最多替换 `count` 次，如果不指定，就全部替换。

`partition(sep)、rpartition(sep)、splitlines([keepends])、split([sep[, maxsplit]])、rsplit([sep[, maxsplit]])`，只要弄清楚 `partition()` 和 `split()` 就可以了。`*partition()` 函数族是 Python2.5 新增的方法，它接受一个字符串参数，并返回一个 3 个元素的元组对象。如果 sep 没有出现在母串中，返回值是 `(sep, '', '')`；否则，返回值的第一个元素是 sep 左端的部分，第二个元素是 sep 自身，第三个元素是 sep 右端的部分。而 `split()` 的参数 maxsplit 是分切的次数，即最大的分切次数，所以返回值最多有 `maxsplit + 1` 个元素。

但 `split()` 有不少小陷阱，比如对于字符串 s，`s.split()` 和 `s.split(' ')` 的返回值是不相同的。产生差异的原因在于：当忽略 sep 参数或 sep 参数为 None 时与明确给 sep 赋予字符串值时，`split()` 采用两种不同的算法。对于前者，`split()` 先去除字符串两端的空白符，然后以任意长度的空白符串作为界定符分切字符串（即连续的空白符串被当做单一的空白符看待）；对于后者则认为两个连续的 `sep` 之间存在一个空字符串。

因为 `title()` 函数并不去除字符串两端的空白符也不会把连续的空白符替换为一个空格，所以不能把 `title()` 理解先以空白符分切字符串，然后调用 `capitalize()` 处理每个字词以使其首字母大写，再用空格将它们连接在一起。使用 `string` 模块中的 `capwords(s)` 函数，它能够去除两端的空白符，再将连续的空白符用一个空格代替。

删除：如果 `strip([chars])`、`lstrip([chars])`、`rstrip([chars])` 中的 chars 参数没有指定，就是删除空白符，空白符由 `string.whitespace` 常量定义。填充则常用于字符串的输出，比如 `center(width, [fillchar])、ljust(width[, fillchar])、rjust(width[, fillchar])、zfill(width)、expandtabs([tabsize])`，包括居中、左对齐、右对齐等，这些方法中的 `fillchar` 参数是指用以填充的字符，默认是空格。而 `zfill()` 中的 z 是指 zero，`zfill()` 即是以字符 0 进行填充，在输出数值时比较常用。`expandtabs()` 的 `tabsize` 参数默认为 8，它的功能是把字符串中的制表符（tab）转换为适当数量的空格。

### 建议 37：按需选择 `sort()` 或者 `sorted()`

Python 中的排序，常用的函数有 `sort()` 和 `sorted()` 两种。这两种函数并不完全相同，各有各的用武之地。

* 相比于 `sort()`，`sorted()` 使用的范围更为广泛，两者的函数形式分别如下：

  ```python
  sorted(iterable[, cmp[, key[, reverse]]])
  s.sort([cmp[, key[, reverse]]])
  ```

  这两个方法有以下 3 个共同的参数：

  * cmp 为用户定义的任何比较函数，函数的参数为两个可比较的元素（来自 `iterable` 或者 `list` ），函数根据第一个参数与第二个参数的关系依次返回 -1、0 或者 +1（第一个参数小于第二个参数则返回负数）。该参数默认值为 None。
  * key 是一个带参数的函数，用来为每个元素提取比较值，默认为 None（即直接比较每个元素）
  * reverse 表示排序结果是否反转

  `sorted()` 作用于任何可迭代的对象，而 `sort()` 一般作用于列表。

* 当排序对象为列表的时候两者适合的场景不同。`sorted()` 函数是在 Python2.4 版本中引入的，在这之前只有 `sort()` 函数。`sorted()` 函数会返回一个排序后的列表，原有列表保持不变；而 `sort()` 函数会直接修改原有列表，函数返回为 None。

  * 实际应用过程中需要保留原有列表，使用 `sorted()` 函数较为合适，否则可以选择 `sort()` 函数，因为 `sort()` 函数不需要复制原有列表，消耗的内存较少，效率也较高。

* 无论是 `sort()` 还是 `sorted()` 函数，传入参数 `key` 比传入参数 `cmp` 效率要高。`cmp` 传入的函数在整个排序过程中会调用多次，函数开销较大；而 `key` 针对每个元素仅做一次处理，因此使用 `key` 比使用 `cmp` 效率要高。

* `sorted()` 函数功能非常强大，使用它可以方便地针对不同的数据结构进行排序，从而满足不同需求。

  * 对字典进行排序

    ```python
    >>> phone_book = {"Linda": "7750", "Bob": "9345", "Carol": "5834"}
    >>> from operator import itemgetter
    >>> sorted_pb = sorted(phone_book.items(), key=itemgetter(1))
    >>> print(sorted_pb)
    [('Carol', '5834'), ('Linda', '7750'), ('Bob', '9345')]
    ```

  * 多维 list 排序：实际情况下也会碰到需要对多个字段进行排序的情况，这在 DB 里面用 SQL 语句很容易做到，但使用多维列表联合 `sorted()` 函数也可以轻易达到。

    ```python
    >>> import operator
    >>> game_result = [["Bob",95,"A"],["Alan",86,"C"],["Mandy",82.5,"A"],["Rob",86,"E"]]
    >>> sorted(game_result,key=operator.itemgetter(2, 1))
    [['Mandy', 82.5, 'A'], ['Bob', 95, 'A'], ['Alan', 86, 'C'], ['Rob', 86, 'E']]
    ```

  * 字典中混合 list 排序：如果字典中的 key 或者值为列表，需要对列表中的某一个位置的元素排序也是可以做到的。

    ```python
    >>> my_dict = {"Li":["M",7],"Zhang":["E",2],"Wang":["P",3],"Du":["C",2],"Ma":["C",9],"Zhe":["H",7]}
    >>> import operator
    >>> sorted(my_dict.items(),key=lambda item:operator.itemgetter(1)(item[1]))
    [('Du', ['C', 2]), ('Zhang', ['E', 2]), ('Wang', ['P', 3]), ('Zhe', ['H', 7]), ('Li', ['M', 7]), ('Ma', ['C', 9])]
    ```

  * List 中混合字典排序：如果列表中的每一个元素为字典形式，需要针对字典的多个 key 值进行排序也不难实现。

    ```python
    >>> import operator
    >>> game_result = [{"name":"Bob","wins":10,"losses":3,"rating":75},{"name":"David","wins":3,"losses":5,"rating":57},{"name":"Carol","wins":4,"losses":5,"rating":57},{"name":"Patty","wins":9,"losses":3,"rating":71.48}]
    >>> sorted(game_result,key=operator.itemgetter("rating","name"))
    [{'losses': 5, 'name': 'Carol', 'rating': 57, 'wins': 4}, {'losses': 5, 'name': 'David', 'rating': 57, 'wins': 3}, {'losses': 3, 'name': 'Patty', 'rating': 71.48, 'wins': 9}, {'losses': 3, 'name': 'Bob', 'rating': 75, 'wins': 10}]
    ```

### 建议 38：使用 copy 模块深拷贝对象

* 浅拷贝（shallow copy）：构造一个新的复合对象并将从原对象中发现的引用插入该对象中。浅拷贝的实现方式有多种，如工厂函数、切片操作、copy 模块中的 copy 操作等。
* 深拷贝（deep copy）：也构造一个新的复合对象，但是遇到引用会继续递归拷贝其所指向的具体内容，也就是说它会针对引用所指向的对象继续执行拷贝，因此产生的对象不受其他引用对象操作的影响。深拷贝的实现需要依赖 copy 模块的 `deepcopy()` 操作。

实际上在包含引用的数据结构中，浅拷贝并不能进行彻底的拷贝，当存在列表、字典等不可变对象的时候，它仅仅拷贝其引用地址。要解决上述问题需要用到深拷贝，深拷贝不仅拷贝引用也拷贝引用所指向的对象，因此深拷贝得到的对象和原对象是相互独立的。

Python `copy` 模块提供了与浅拷贝和深拷贝对应的两种方法的实现，通过名字便可以轻易进行区分，模块在拷贝出现异常的时候会抛出 `copy.error`。关于对象拷贝可以参考[资料](http://en.wikipedia.org/wiki/Object_copy)。

### 建议 39：使用 Counter 进行计数统计

可以使用不同数据结构来进行实现：

* 使用 dict

* 使用 defaultdict

  ```python
  from collections import defaultdict
  some_data = ["a", "2", 2, 4, 5, "2", "b", 4, 7, "a", 5, "d", "a", "z"]
  count_frq = defaultdict(int)
  for item in some_data:
      count_frq[item] += 1
  ```

* 使用 set 和 list

  ```python
  some_data = ["a", "2", 2, 4, 5, "2", "b", 4, 7, "a", 5, "d", "z", "a"]
  count_set = set(some_data)
  count_list = []
  for item in count_set:
      count_list.append((item, some_data.count(item)))
  ```

* 更优雅，更 Pythonic 的解决方法是使用 `collections.Counter`：

  ```python
  from collections import Counter
  some_data = ["a", "2", 2, 4, 5, "2", "b", 4, 7, "a", 5, "d", "z", "a"]
  print(Counter(some_data))
  ```

  `Counter` 类是自 Python2.7 起增加的，属于字典类的子类，是一个容器对象，主要用来统计散列对象，支持集合操作 `+`、`-`、`&`、`|`，其中 `&` 和 `|` 操作分别返回两个 `Counter` 对象各元素的最小值和最大值。它提供了 3 种不同的方式来初始化：

  ```python
  Counter("success") # 可迭代对象
  Counter(s=3, c=2, e=1, u=1) # 关键字参数
  Counter({"s":3, "c":2, "u":1, "e":1}) # 字典
  ```

   可以使用 `elements()` 方法来获取 `Counter` 中的 `key` 值。`list(Counter(some_data).elements())`。

  利用 `most_common()` 方法可以找出前 N 个出现频率最高的元素以及它们对应的次数。

  当访问不存在的元素时，默认返回为 0 而不是抛出 `KeyError` 异常。

  `update()` 方法用于被统计对象元素的更新，原有 `Counter` 计数器对象与新增元素的统计计数值相加而不是直接替换它们。

  `subtract()` 方法用于实现计数器对象中元素统计值相减，输入和输出的统计值允许为 0 或者负数。

### 建议 40：深入掌握 `ConfigParser`

比如 `pylint` 就带有一个参数 `--rcfile` 用以指定配置文件，实现对自定义代码风格的检测。常见的配置文件格式有 XML 和 ini 等，其中在 MS Windows 系统上，ini 文件格式用得尤其多，甚至操作系统的 API 也都提供了相关的接口函数来支持它。类似 ini 的文件格式，在 Linux 等操作系统中也是极常用的，比如 `pylint` 的配置文件就是这个格式。Python 有个标准库来支持它，也就是 `ConfigParser`。

`ConfigParser` 的基本用法通过手册可以掌握，但是仍然有几个知识点值得注意。首先就是 `getboolean()` 这个函数。`getboolean()` 根据一定的规则将配置项的值转换为布尔值，如以下的配置：

```ini
[section1]
option1=0
```

当调用 `getboolean("section1", "option1")` 时，将返回 `False`。不过 `getboolean()` 的真值规则值得一说：除了 0 以外，no、false 和 off 都会被转义为 `False`，而对应的 1、yes、true 和 on 则都被转义为 `True`，其他值都会导致抛出 `ValueError` 异常。

除了 `getboolean()` 之外，还需要注意的是配置项的查找规则。首先，在 `ConfigParser` 支持的配置文件格式里，有一个 `[DEFAULT]` 节，当读取的配置项不在指定的节里时，`ConfigParser` 将会到 `[DEFAULT]` 节中查找。

除此之外，还有一些机制导致项目对配置项的查找更复杂，这就是 `class ConfigParser` 构造函数中的 `defaults` 形参以及其 `get(section, option[, raw[, vars]])` 中的全名参数 `vars`。如果把这些机制全部用上，那么配置项值的查找规则如下：

* 如果找不到节名，就抛出 `NoSectionError`
* 如果给定的配置项出现在 `get()` 方法的 `var` 参数中，则返回 `var` 参数中的值
* 如果在指定的节中含有给定的配置项，则返回其值
* 如果在 【DEFAULT】中有指定的配置项，则返回其值
* 如果在构造函数的 defaults 参数中有指定的配置项，则返回其值
* 抛出 NoOptionError

Python 中字符串格式化可以使用以下语法：

```python
>>> "%(protocol)s://%(server)s:%(port)s/" % {"protocol": "http", "server": "example.com", "port": 1080}
"http://example.com:1080/"
```

其实 `ConfigParser` 支持类似的用法，所以在配置文件中可以使用。如，有如下配置选项：

```ini
# format.conf
[DEFAULT]
conn_str = %(dbn)s://%(user)s:%(pw)s@%(host)s:%(port)s/%(db)s
dbn = mysql
user = root
host = localhost
port = 3306
[db1]
user = aaa
pw = ppp
db = example
[db2]
host = 192.168.0.110
pw = www
db = example
```

这是一个 `SQLAlchemy` 应用程序的配置文件，通过这个配置文件能够获取不同的数据库配置相应的连接字符串，即 `conn_str`。它通过不同的节名来获取格式化后的值时，根据不同配置，得到不同的值：

```python
import ConfigParser
conf = ConfigParser.ConfigParser()
conf.read("format.conf")
print(conf.get("db1", "conn_str"))
print(conf.get("db2", "conn_str"))
```

### 建议 41：使用 `argparse` 处理命令行参数

尽管应用程序通常能够通过配置文件在不修改代码的情况下改变行为，但提供灵活易用的命令行参数仍然非常有意义，比如：减轻用户的学习成本，通常命令行参数的用法只需要在应用程序名后面加 `--help` 参数就能获得，而配置文件的配置方法通常需要通读手册才能掌握。同一个运行环境中有多个配置文件存在，那么需要通过命令行参数指定当前使用哪一个配置文件，如 `pylint` 的 `--rcfile` 参数。

关于命令行处理，标准库中留下的 `getopt`、`optparse`、`argparse` 等库。其中 `getopt` 是类似 UNIX 系统中 `getopt()` 这个 C 函数的实现，可以处理长短配置项和参数。如有命令行参数 `-a -b -cfoo -d bar a1 a2`，在处理之后的结果是两个列表，其中一个是配置项列表：`[('-a', '')、('-b', '')、('-c', 'foo')、('-d', 'bar')]`，每一个元素都由配置项名和其值（默认为空字符串）组成；另一个是参数列表 `['a1', 'a2']`，每一个元素都是一个参数值。

`getopt` 的问题在于两点：一个是长短配置项需要分开处理，二是对非法参数和必填参数的处理需要手动。这种处理非常原始和不便。

`optparse` 比 `getopt` 要更加方便、强劲，与 C 风格的 `getopt` 不同，它采用的是声明式风格，此外，它还能够自动生成应用程序的帮助信息。

```python
from optparse import OptionParser
parser = OptionParser()
parser.add_option("-f", "--file", dest="filename", help="write report to FILE", metavar="FILE")
parser.add_option("-q", "--quiet", action="store_false", dest="verbose", default=True, help="don't print status messages to stdout")
(options, args) = parser.parse_args()
```

可以看到 `add_option()` 方法非常强大，同时支持长短配置项，还有默认值、帮助信息等，简单的几行代码，可以支持非常丰富的命令行接口。

除此之外，虽然没有声明帮助信息，但默认给加上了 `-h` 或 `--help` 支持，通过这两个参数调用应用程序，可以看到自动生成的帮助信息。

不过 `optparse` 虽然很好，但是后来出现的 `argparse` 在继承了它声明式风格的优点之外，又多了更丰富的功能，所以现阶段最好用的参数处理标准库是 `argparse`，使 `optparse` 成为了一个被弃用的库。

与 `optparse` 中的 `add_option()` 类似，`add_argument()` 方法用以增加一个参数声明。与 `add_option()` 相比，它有几个方面的改进，其中之一就是支持类型增多，而且语法更加直观。表现在 `type` 参数不再是一个字符串，而是一个可调用对象，比如在 `add_option()` 调用时是 `type="int"`，而在 `add_argument()` 调用时直接写 `type=int` 就可以了。除了支持常规的 `int/float` 等基本数值类型外，`argparse` 还支持文件类型，只要参数合法，程序就能够使用相应的文件描述符。

```python
parser = argparse.ArgumentParser()
parser.add_argument("bar", type=argparse.FileType("w"))
parser.parse_args(["out.txt"])
```

另外，扩展类型也变得更加容易，任何可调用对象，比如函数，都可以作为 `type` 的实参。另外 `choices` 参数也支持更多的类型，而不是像 `add_option` 那样只有字符串。比如：`parser.add_argument("door", type=int, choices=range(1, 4))`。

此外，`add_argument()` 提供了对必填参数的支持，只要把 `required` 参数设置为 `True` 传递进去，当缺失这一参数时，`argparse` 就会自动退出程序，并提示用户。

`ArgumentParser` 还支持参数分组。`add_argument_group()` 可以在输出帮助信息时更加清晰，这在用法复杂的 `CLI` 应用程序中非常有帮助：

```python
parser = argparse.ArgumentParser(prog="PROG", add_help=False)
group1 = parser.add_argument_group("group1", "group1 description")
group1.add_argument("foo", help="foo help")
group2 = parser.add_argument_group("group2", "group2 description")
group2.add_argument("--bar", help="bar help")
parser.print_help()
```

另外还有 `add_mutually_exclusive_group(required=False)` 非常实用：它确保组中的参数至少有一个或者只有一个（`required=True`）。

`argparse` 也支持子命令，比如 `pip` 就有 `install/uninstall/freeze/list/show` 等子命令，这些子命令又接受不同的参数，使用 `ArgumentParser.add_subparsers()` 就可以实现类似的功能。

```python
import argparse
parser = argparse.ArgumentParser(prog="PROG")
subparsers = parser.add_subparsers(help="sub-command help")
parser_a = subparsers.add_parser("a", help="a help")
parser_a.add_argument("--bar", type=int, help="bar help")
parser.parse_args(["a", "--bar", "1"])
```

除了参数处理之外，当出现非法参数时，用户还需要做一些处理，处理完成后，一般是输出提示信息并退出应用程序。`ArgumentParser` 提供了两个方法函数，分别是 `exit(status=0, message=None)` 和 `error(message)`，可以省了 `import sys` 再调用 `sys.exit()` 的步骤。

> 注意：虽然 argparse 已经非常好用，但又出现了 `docopt`，它是比 argparse 更先进更易用的命令行参数处理器。它甚至不需要编写代码，只要编写类似 argparse 输出的帮助信息即可。这是因为它根据常见的帮助信息定义了一套领域特定语言（DSL），通过这个 DSL Parser 参数生成处理命令行参数的代码，从而实现对命令行参数的解释。`docopt` 现在还不是标准库。

### 建议 42：使用 `pandas` 处理大型 CSV 文件

CSV（Comma Separated Values）作为一种逗号分隔型值的纯文本格式文件，在实际应用中经常用到，如数据库数据的导入导出、数据分析中记录的存储等。很多语言都提供了对 CSV 文件处理的模块，Python 模块 csv 提供了一系列与 CSV 处理相关的 API。

* `reader(csvfile[, dialect="excel"][, fmtparam])`，主要用于 CSV 文件的读取，返回一个 `reader` 对象用于在 CSV 文件内容上进行行迭代。

  参数 `csvfile`，需要是支持迭代（Iterator）的对象，通常对文件（file）对象或者列表（list）对象都是适用的，并且每次调用 `next()` 方法的返回值是字符串（string）；参数 `dialect` 的默认值为 excel，与 excel 兼容；fmtparam 是一系列参数列表，主要用于需要覆盖默认的 `Dialect` 设置的情形。当 `dialect` 设置为 exdel 的时候，默认 `Dialect` 的值如下：

  ```python
  class excel(Dialect):
      delimiter = ','	# 单个字符，用于分隔字段
      quotechar = '"'	# 用于对特殊符号加引号，常见的引号为 ''
      doublequote = True	# 用于控制 quotechar 符号出现的时候的表现形式
      skipinitialspace = False	# 设置为 true 的时候 delimiter 后面的空格将会省略
      lineterminator = '\r\n'	# 行结束符
      quoting = QUOTE_MINIMAL	# 是否在字段前加引号，QUOTE_MINIMAL 表示仅当一个字段包含引号或者定义符号的时候才加引号
  ```

  ​

* `csv.write(csvfile, dialect="excel", **fmtparams)`，用于写入 CSV 文件。参数同上。例子：

  ```python
  with open("data.csv", "wb") as csvfile:
      csvwriter = csv.writer(csvfile, dialect="excel", delimiter="|", quotechar='"', quoting=csv.QUOTE_MINIMAL)
      csvwriter.writerow(["1/3/09 14:44", "'Product1'", "1200''", "Visa", "Gouya"]) # 写入行
  ```

* `csv.DictReader(csvfile, filenames=None, restkey=None, restval=None, dialect="excel", *args, **kwargs)`，同 `reader()` 方法类似，不同的是将读入的信息映射到一个字典中去，其中字典的 key 由 `fieldnames` 指定，该值省略的话将使用 CSV 文件第一行的数据作为 key 值。如果读入行的字典的个数大于 fieldnames 中指定的个数，多余的字段名将会存放在 restkey 中，而 restval 主要用于当读取行的域的个数小于 fieldnames 的时候，它的值将会被用作剩下的 key 对应的值。

* `csv.DictWriter(csvfile, fieldnames, restval='', extrasaction='raise', dialect='excel', *args, **kwargs)`，用于支持字典的写入。

  ```python
  import csv
  # DictWriter
  with open("test.csv", "wb") as csv_file:
      # 设置列名称
      FIELDS = ["Transaction_date", "Product", "Price", "Payment_Type"]
      writer = csv.DictWriter(csv_file, fieldnames=FIELDS)
      # 写入列名称
      writer.writerow(dict(zip(FIELDS, FIELDS)))
      d = {"Tranaction_date": "1/2/09 6:17", "Product": "Product1", "Price": "1200", "Payment_Type": "Mastercard"}
      # 写入一行 Writer.writerow(d)
  with open("test.csv", "rb") as csv_file:
      for d in csv.DictReader(csv_file):
          print(d)
      # output d is:{"Product": "Product1", "Transaction_date": "1/2/09 6:17", "Price": "1200", "Payment_Type": "Mastercard"}
  ```

csv 模块使用非常简单，基本可以满足大部分需求。但是 csv 模块对于大型 CSV 文件的处理无能为力。这种情况下就需要考虑其他解决方案了，pandas 模块便是较好的选择。

Pandas 即 Python Data Analysis Library，是为了解决数据分析而创建的第三方工具，它不仅提供了丰富的数据模型，而且支持多种文件格式处理，包括 CSV、HDF5、HTML 等，能够提供高效的大型数据处理。其支持的两种数据结构——Series 和 DataFrame ——是数据处理的基础。

* Series：它是一种类似数组的带索引的一维数据结构，支持的类型与 NumPy 兼容。如果不指定索引，默认为 0 到 N - 1。通过 `obj.values()` 和 `obj.index()` 可以分别获取值和索引。当给 Series 传递一个字典的时候，Series 的索引将根据字典中的键排序。如果传入字典的时候同时指定了 index 参数，当 index 与字典中的键不匹配的时候，会出现数据丢失的情况，标记为 NaN。

  在 pandas 中用函数 `isnull()` 和 `notnull()` 来检测数据是否丢失。

  ```python
  >>> obj1 = Series([1, 'a', (1, 2), 3], index=['a', 'b', 'c', 'd'])
  >>> obj1 # value 和 index 一一匹配
  a	1
  b	a
  c	(1, 2)
  d	3
  dtype: object
  >>> obj2 = Series({"Book": "Python", "Author": "Dan", "ISBN": "011334", "Price": 25}, index=["book", "Author", "ISBM", "Price"])
  >>> obj2.isnull()
  book	True	# 指定的 index 与字典的键不匹配，发生数据丢失
  Author	False
  ISBM	True	# 指定的 index 与字典的键不匹配，发生数据丢失
  Price	False
  dtype: bool
  ```

* DataFrame：类似于电子表格，其数据为排好序的数据列的集合，每一列都可以是不同的数据类型，它类似于一个二维数据结构，支持行和列的索引。和 Series 一样，索引会自动分配并且能根据指定的列进行排序。使用最多的方式是通过一个长度相等的列表的字典来构建。构建一个 DataFrame 最常用的方式是用一个相等长度列表的字典或 NumPy 数组。DataFrame 也可以通过 columns 指定序列的顺序进行排序。

  ```python
  >>> data = {"OrderDate": ["1-6-10", "1-23-10", "2-9-10", "2-26-10", "3-15-10"], "Region": ["East", "Central", "Central", "West", "East"], "Rep": ["Jones", "Kivell", "Jardine", "Gill", "Sorvino"]}
  >>> DataFrame(data, columns=["OracleDate", "Region", "Rep"]) # 通过字典构建，按照 columns 指定的顺序排序
  	OrderDate	Region	Rep
  0	1-6-10		East	Jones
  1	1-23-10		Central	Kivell
  2	2-9-10		Central	Jardine
  3	2-26-10		West	Gill
  4	3-15-10		East	Sorvino
  ```

Pandas 中处理 CSV 文件的函数主要为 `read_csv()` 和 `to_csv()` 这两个，其中 `read_csv()` 读取 CSV 文件的内容并返回 DataFram，`to_csv()` 则是其逆过程。两个函数都支持多个参数，其参数众多且过于复杂。

* 指定读取部分列和文件的行数

  ```python
  >>> df = pd.read_csv("SampleData.csv", nrows=5, usecols=["OrderDate", "Item", "Total"])
  ```


  方法 `read_csv()` 的参数 `nrows` 指定读取文件的行数，`usecols` 指定所要读取的列的列名，如果没有列名，可直接使用索引 0、1、...、n - 1。上述两个参数对大文件处理非常有用，可以避免读入整个文件而只选取所需要部分进行读取。

* 设置 CSV 文件与 excel 兼容。`dialect` 参数可以是 string 也可以是 `csv.Dialect` 的实例。如果文件格式改为使用 "|" 分隔符，则需要设置 `dialect` 相关的参数。`error_bad_lines` 设置为 `False`，当记录不符合要求的时候，如记录所包含的列数与文件列设置不相等时可以直接忽略这些列。

  ```python
  >>> dia = csv.excel()
  >>> dia.delimiter = "|"	# 设置分隔符
  >>> pd.read_csv("SD.csv")
  ```

* 对文件进行分块处理并返回一个可迭代的对象。分块处理可以避免将所有的文件载入内容，仅在使用的时候读入所需内容。参数 `chunksize` 设置分块的文件行数，10 表示每一块包含 10 个记录。将参数 `iterator` 设置为 `True` 时，返回值为 `TextFileReader`，它是一个迭代对对象。

  ```python
  >>> reader = pd.read_table("SampleData.csv", chunksize=10, iterator=True)
  >>> iter(reader).next()
  ```

* 当文件格式相似的时候，支持多个文件合并处理。

  ```python
  >>> file1st = os.listdir("test")
  >>> print(file1st)	# 同时存在 3 个格式相同的文件
  >>> os.chdir("test")
  >>> dfs = [pd.read_csv(f) for f in file1st]	# 将文件合并
  >>> total_df = pd.concat(dfs)
  >>> total_df
  ```

在处理 CSV 文件上，特别是大型 CSV 文件，pandas 不仅能够做到与 csv 模块兼容，更重要的是其 CSV 文件以 DateFrame 的格式返回，pandas 对这种数据结构提供了非常丰富的处理方法，同时 pandas 支持文件的分块和合并处理，非常灵活，其底层很多算法采用 Cython 实现运行速度较快。

### 建议 43：一般情况使用 `ElementTree` 解析 XML

`xml.dom.minidom` 和 `xml.sax` 大概是 Python 中解析 XML 文件最广为人知的两个模块了，原因一是这两个模块自 Python2.0 以来就成为 Python 的标准库；二是网上关于这两个模块的使用方面的资料最多。作为主要解析 XML 方法的两种实现，DOM 需要将整个 XML 文件加载到内存中并解析为一棵树，虽然使用较为简单，但占用内存较多，性能方面不占优势，并且不够 Pythonic；而 SAX 是基于事件驱动的，虽不需要全部装入 XML 文件，但其处理过程却较为复杂。实际上 Python 中对 XML 的处理还有更好地选择，ElementTree 便是其中一个，一般情况下使用 ElementTree 便已足够。它从 Python2.5 开始成为标准模块，cElementTree 是 ElementTree 的 Cython 实现，速度更快，消耗内存更少，性能上更占优势，在实际使用过程中应该尽量优先使用 cElementTree。两者使用方式上完全兼容。ElementTree 在解析 XML 文件上具有以下特性：

* 使用简单。它将整个 XML 文件以树的形式展示，每一个元素的属性以字典的形式表示，非常方便处理。
* 内存上消耗明显低于 DOM 解析。由于 ElementTree 底层进行了一定的优化，并且它的 iterparse 解析工具支持 SAX 事件驱动，能够以迭代的形式返回 XML 部分数据结构，从而避免将整个 XML 文件加载到内存中，因此性能上更优化，相比于 SAX 使用起来更为简单明了。
* 支持 XPath 查询，非常方便获取任意节点的值。

一般情况指的是：XML 文件大小适中，对性能要求并非非常严格。如果在实际过程中需要处理的 XML 文件大小在 GB 或近似 GB 级别，第三方模块 lxml 会获得较优的处理结果。[”使用由 Python 编写的 lxml 实现高性能 XML 解析“](http://www.ibm.com/developerworks/cn/xml/x-hiperfparse/)。

模块 ElementTree 主要存在两种类型 ElementTree 和 Element，它们支持的方法以及对应的使用示例如下所示：

* `getroot()`：返回 xml 文档的根节点

  ```python
  >>> import xml.etree.ElementTree as ET
  >>> tree = ET.ElementTree(file = "test.xml")
  >>> root = tree.getroot()
  >>> print(root)
  >>> print(root.tag)
  ```

* `find(match)、findall(match)、findtext(match, default=None)`：同 Element 相关的方法类似，只是从根节点开始搜索

  ```python
  >>> for i in root.findall("system/purpose"):
      print(i.text)
  >>> print(root.findtext("system/purpose"))
  >>> print(root.find("systempurpose")
  ```

* `iter(tag=None)`：从