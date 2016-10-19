## 第 7 章 使用工具辅助开发

Python 项目的开发过程，其实就是一个或多个包的开发过程，而这个开发过程又由包的安装、管理、测试和发布等多个节点构成，所以这是一个复杂的过程，使用工具进行辅助开发有利于减少流程损耗，提升生产力。比如 `setuptools、pip、paster、nose` 和 `Flask-PyPI-Proxy` 等。

### 建议 70：从 PyPI 安装包

PyPI 全称 Python Package Index，直译过来就是“Python 包索引”，它是 Python 编程语言的软件仓库，类似 Perl 的 CPAN 或 Ruby 的 Gems。可以通过包的名字查找、下载、安装 PyPI 上的包。对于包的作者，在 PyPI 上注册账号后，还可以登记、更新、上传包等。

> #### 注意
>
> PyPI 有良好的镜像机制，可以方便地在全球各地架设自有镜像，目前可用的几个镜像列出在 PyPI Mirrors 页面上，而各个镜像的同步情况可以在专门的网站上看到。豆瓣网架设了一个镜像，地址是 http://pypi.douban.com，访问速度很快，使用 pip 的 `--index` 参数。

> #### 注意
>
> 因为包在 PyPI 上的主页的 URL 都是 https://pypi.python.org/pypi/{package} 的形式，所以在知道包名的情况下，可以直接手动输入 URL

`setuptools` ，在 Ubuntu Linux 上，可以使用 apt 安装这个包：

`sudo aptitude install python-setuptools`

其他操作系统大同小异，运行其相应的包管理软件就可以安装。但如果使用 MS Windows，则需要去它的主页下载然后手动安装。

操作系统对应的软件仓库中的 `setuptools` 版本通常比较低，所以安装完成以后，最好执行以下命令将其更新到最新版本：

`easy_install -U setuptools`

`setuptools` 是来自 PEAK（Python Enterprise Application Kit，一个致力于提供 Python 开发企业级应用工具包的项目），由一组发布工具组成，方便程序员下载、构建、安装、升级和卸载 Python 包，它可以自动处理包的依赖关系。

> #### 注意
>
> 因为 PEAK 发展停滞，累及 setuptools 有好几年没有更新，所以有了一个分支项目，称为 distribute，很长一段时间里，运行 `easy_install -U setuptools` 更新安装的其实是 distribute。在 2013 年年初，distribute 合并到 setuptools，回归主分支了，而 distribute 项目也就不再维护了。

安装 `setuptools` 之后，就可以运行 `easy_install` 命令了。

### 建议 71：使用 pip 和 yolk 安装、管理包

`setuptools` 有几个缺点，比如功能缺失（不能查看已经安装的包、不能删除已经安装的包），也缺乏对 git、hg 等版本控制系统的原生支持。

pip 使用子命令形式的 CLI 接口。

【坑爹第 201 页缺失了， yolk 介绍没了】

下面介绍的是 pip 还不具备的功能：

`yolk --entry-map <package>` 可以显示包注册的所有入口点，这样可以了解到安装的包都提供了哪些命令行工具，或者支持哪些基于 `entry-point` 的插件系统。可以使用 `--entry-points` 参数查看哪些包实现了某一个包的插件协议。

如果使用的是桌面版的操作系统，利用 `yolk -H <package>` 可以打开一个浏览器，并将你指定的包显示在 PyPI 上的主页，从此告别手动拼接 URL 的历史。

### 建议 72：做 paster 创建包

个人编写的库，最好是像第三方库一样，可以方便地下载、安装、升级、卸载，也就是说能够放到 PyPI 上面，也能够很好地跟 pip 或者 yolk 这样的工具集成。Python 提供了这方面的支持，那即是 distutils 标准库，至少提供了以下几方面的内容：

* 支持包的构建、安装、发布（打包）
* 支持 PyPI 的登记、上传
* 定义了扩展命令的协议，包括 `distutils.cmd.Command 基类`、`distutils.commands` 和 `distutils.key_words` 等入口点，为 `setuptools` 和 `pip` 等提供了基础设施。


要使用 distutils，按习惯需要编写一个 `setup.py` 文件，作为后续操作的入口点。它的内容如下：

```python
from distutils.core import setup
setup(name="arithmetic",
     version='1.0',
     py_modules=["your_script_name"],
     )
```

`setup.py` 文件的意义是执行时调用 `distutils.core.setup()` 函数，而实参是通过命名参数指定的。name 参数指定的是包名；version 指定的是版本；而 `py_modules` 参数是一个序列类型，里面包含需要安装的 Python 文件。

编写好 `setup.py` 文件以后，就可以使用 `python setup.py install` 进行安装了。

安装成功后可以使用 yolk 查看一下：

`yolk -l | grep your_script_name`

`distutils` 还带有其他命令，可以通过 `python setup.py --help-commands` 进行查询。

若要把包提交到 PyPI，还要遵循 PEP241，给出足够多的元数据才行，比如对包的简短描述、详细描述、作者、作者邮箱、主页和授权方式等。包含太多内容了，如果每一个项目都手写很困难，最好找一个工具可以自动创建项目的 `setup.py` 文件以及相关的配置、目录等。Python 中做这种事的工具有好几个，做得最好的是 `pastescript`。`pastescript` 是一个有着良好插件机制的命令行工具，安装以后就可以使用 `paster` 命令，创建适用于 `setuptools` 的包文件结构。

安装好 `pastescript` 以后可以看到它注册了一个命令行入口 paster（还有许多插件协议的实现）：`yolk --entry-map pastescript`

#### 接下来讨论一下 paster 怎么用

`paster --help`，paster 支持多个命令，首先要学习的是 create。create 命令用于根据模板创建一个 Python 包项目，创建的项目文件结构可以使用 `setuptools` 直接构建，并且支持 `setuptools` 扩展的 `distutils` 命令，如 `develop` 命令等。

使用 `paster help create` 查看帮助，可以看到 `--template` 参数用以指定使用的模板，可以使用 `--list-templates` 查询一下目前安装的模板了。如果一个全新的环境只安装好了 `paster`，那么一般只有两个模板：`basic_package` 和 `paste_deploy`。

执行命令示例：`paster create -o arithmetic-2 -t basic_package arithmetic`，可以看到，简单地填写几个问题以后，paster 就在 arithmetic-2 目录生成了名为 arithmetic 的包项目。可以看到 `setup.py` 文件中所有的参数都帮我们填好了。不过第一次使用 paster 的话，回答问题时可能会输入错误，而 paster 又不能回退删除输入，所以就只好重来一次。要解决这个问题，可以用上 `--config` 参数，它是一个类似 ini 文件格式的配置文件，可以在里面填好各个模板变量的值（查询模板有哪些变量用 `--list-variables` 参数），然后就可以使用了。示例如下：

```ini
[pastescript]
description = corp-prj
license_name = 
keywords = Python
long_description = corp-prj
author = xxx corp
author_email = xxx@example.com
url = http://example.com
version = 0.0.1
```

然后这样使用：`paster create -t basic_package --config="corp-prj-setup.cfg" arithmetic`

### 建议 73：理解单元测试概念

单元测试用来验证程序单元的正确性，一般由开发人员完成，是测试过程的第一个环节，以确保缩写的代码符合软件需求和遵循开发目标。好的单元测试有以下好处：

* 减少了潜在 bug
* 大大缩减软件修复的成本
* 为集成测试提供基本保障

有效的单元测试应该从以下几个方面考虑：

* 测试先行，遵循单元测试步骤：
  * 创建测试计划（Test Plan）
  * 编写测试用例，准备测试数据
  * 编写测试脚本
  * 编写被测代码，在代码完成之后执行测试脚本
  * 修正代码缺陷，重新测试直到代码可接受为止
* 遵循单元测试基本原则：
  * 一致性
  * 原子性
  * 单一职责：测试应该基于情景（scenario）和行为，而不是方法。如果一个方法对应着多种行为，应该有多个测试用例；而一个行为即使对应多个方法也只能有一个测试用例
  * 隔离性：不能依赖于具体的环境设置，如数据库的访问、环境变量的设置、系统的时间等；也不能依赖于其他的测试用例以及测试执行的顺序，并且无条件逻辑依赖。单元测试的所有输入应该是确定的，方法的行为和结构应是可以预测的。
* 使用单元测试框架，在单元测试方面常见的测试框架有 PyUnit 等，它是 JUnit 的 Python 版本，在 Python2.1 之前需要单独安装，在 Python2.1 之后它成为了一个标准库，名为 unittest。它支持单元测试自动化，可以共享地进行测试环境的设置和清理，支持测试用例的聚集以及独立的测试报告框架。unittest 相关的概念主要有以下 4 个：
  * 测试固件（test fixtures）：测试相关的准备工作和清理工作，基于类 TestCase 创建测试固件的时候通常需要重新实现 `setUp()` 和 `tearDown()` 方法。当定义了这些方法的时候，测试运行器会在运行测试之前和之后分别调用这两个方法
  * 测试用例（test case）：最小的测试单元，通常基于 TestCase 构建
  * 测试用例集（test suite）：测试用例的集合，使用 TestSuite 类来实现，除了可以包含 TestCase 外，也可以包含 TestSuite
  * 测试运行器（test runner）：控制和驱动整个单元测试过程，一般使用 TestRunner 类作为测试用例的基本执行环境，常用的运行器为 TextTestRunner，它是 TestRunner 的子类，以文字方式运行测试并报告结果。


用 TestSuite 类来组织 TestCase，TestSuite 类可以看成是 TestCase 类的一个容器，用来对多个测试用例进行组织，这样多个测试用例可以自动在一次测试中全部完成。

```python
suite = unittest.TestSuite()
suite.addTest(MyCalTest("testAdd"))
suite.addTest(MyCalTest("testSub"))
runner = unittest.TextTestRunner()
runner.run(suite)
```

运行命令：`python -m unittest -v MyCalTest`

### 建议 74：为包编写单元测试

实际项目中的测试有不少麻烦：

* 程序员希望测试更加自动化

* 一个测试用例往往在测试之前需要进行打桩或做一些准备工作，在测试之后要清理现场，最好有一个框架可以自动完成这些工作

* 对于大项目，大量的测试用例需要分门分类地放置，而测试之后，分别产生相应的测试报告

unittest 框架，除了 自动匹配调用以 test 开头的方法之外，`unittest.TestCase` 还有模板方法 `setUp()` 和 `tearDown()`，通过覆盖这两个方法，能够实现在测试之前执行一些准备工作，并在测试之后清理现场。

可以使用 `unittest` 测试发现（test discover）功能：`python -m unittest discover`，unittest 将递归地查找当前目录下匹配 `test*.py` 模式的文件，并将其中 `unittest.TestCase` 的所有子类都实例化，然后调用相应的测试方法进行测试。

`unittest` 的测试发现功能是 Python2.7 版本中才有的，如果使用更旧的版本，需要安装 `unittest2`

除此之外，`setuptools` 对 `distutils.command` 进行了扩展，增加了命令 test。这个命令执行的时候，先运行 `egg_info` 和 `build_ext` 子命令构建项目，然后把项目路径加到 `sys.path` 中，再搜寻所有的测试套件（`test suite`，通常指多个测试用例或测试套件的组合），并运行之。要使用这个扩展命令，需要在调用 `setup()` 函数的时候向它传递 `test_suite` 元数据，例如：

```python
# setup.py
...
setup(name = "arithmetic",
     ...
     test_suite = "test_arithmetic",
     ...
```

`test_suite` 元数据的值可以指向一个包、模块、类或函数，比如在 flask 项目中，是 `test_suite = "flask.testsuite.suite"`，其中 `flask.testsuite.suite` 是一个函数；而在 `arithmetic` 项目中，`test_arithmetic` 是一个模块。

使用 `setuptools` 的测试发现功能，可以给开发人员更一致的开发体验，就像使用 `build`、`install` 命令一样。但是来自 unittest 本身的缺陷让开发人员想要找到一个更好的测试框架：

* unittest 并不够 Pythonic，比如从 JUnit 中继承而来的首字母小写的骆驼命名法；所有的测试用例都需要从 TestCase 集成
* unittest 的 `setUp()` 和 `tearDown()` 只是在 TestCase 的层面上提供，即每一个测试用例执行的时候都会运行一遍，如果有多个模块需要测试，那么创建环境和清理现场操作都会带来大量工作
* unittest 没有插件机制进行功能扩展，比如想要增加测试覆盖统计特性就非常困难。

`nose` 就是作为更好的测试框架进入视线的，而它更是一个具有更强大的测试发现运行的程序。此外，nose 定义了插件机制，使得扩展 nose 的功能成为可能（默认自带 coverage 插件）。使用 pip 安装后，就多了一个 `nosetests` 命令可以使用：`nosetests -v`

可以看到 nose 能够自动发现测试用例，并调用执行，由于它与原有的 unittest 测试用例兼容，所以可以随时将它引入到项目中来。其实 nose 的测试发现机制更进一步，它抛弃了 unittest 中 测试用例必须放在 TestCase 子类中的限制，只要命名符合 `(?:^|[b_.-])[Tt]est` 正则表达式的类和函数都可作为测试用例运行。

此外，nose 作为一个测试框架，也提供了与 `unittest.TestCase` 类似的断言函数，但它抛弃了 `unittest` 的那种 Java 风格的命令方式，使用的是符合 PEP8 的命名方式。

针对 `unittest` 中 `setUp()` 和 `tearDown()` 只能放在 TestCase 中的问题，`nose` 提供了 3 个级别的解决方案，这些配置和清理函数，可以放在包（`__init__.py` 文件中）、模块和测试用例中，非常完全地解决了不同层次的测试需要的配置和清理需求。

最后，`nose` 与 `setuptools` 的集成更加友好，提供了 `nose.collector` 作为通过的测试套件，让开发人员无须针对不同项目编写不同的套件。需要对 `setup.py` 作如下修改：

```python
# setup.py
setup(name = "arithmetic",
     ...
     # test_suite = "test_arithmetic",
     test_suite = "nose.collector",
     ...
```

然后运行 `python setup.py test`，得到的结果是一样的。因为使用 `nose.collector` 之后，`test_siuite` 元数据就确定不变了，所以它也非常适合写入 `paster` 的模板中去，在构建目录的时候自动生成。

### 建议 75：利用测试驱动开发提高代码的可测性

测试驱动开发（Test Driven Development，TDD）是敏捷开发中一个非常重要的理念，它提倡在真正开始编码之前测试先行，先编写测试代码，再在其基础上通过基本迭代完成编码，并不断完善。一般来说，遵循以下过程：

* 编写部分测试用例，并运行测试
* 如果测试通过，则回到测试用例编写的步骤，继续添加新的测试用例
* 如果测试失败，则修改代码直到通过测试
* 当所有测试用例编写完成并通过测试之后，再来考虑对代码进行重构

关于测试驱动开发和提高代码可测性方面有几点需要说明：

* TDD 只是手段而不是目的，因此在实践中尽量只验证正确的事情，并且每次仅仅验证一件事。当遇到问题时不要局限于 TDD 本身所涉及的一些概念，而应该回头想想采用 TDD 原本的出发点和目的是什么
* 测试驱动开发本身就是一门学问
* 代码的不可测性可以从以下几个方面考量：实践 TDD 困难；外部依赖太多；需要写很多模拟代码才能完成测试；职责太多导致功能模糊；内部状态过多且没有办法去操作和维护这些状态；函数没有明显返回或者参数过多；低内聚高耦合等等。

### 建议 76：使用 Pylint 检查代码风格

