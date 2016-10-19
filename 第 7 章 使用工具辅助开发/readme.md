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

如果团队遵循 PEP8 编码风格，Pylint 是个不错的选择（还有其他选择，比如 pychecker、pep8 等）。Pylint 始于 2003 年，是一个代码分析工具，用于检查 Python 代码中的错误，查找不符合代码编码规范以及潜在的问题。支持不同的 OS 平台，如 Windows、Linux、OSX 等，特性如下：

* 代码风格审查。它以 Guido van Rossum 的 PEP8 为标准，能够检查代码的行长度，不符合规范的变量名以及不恰当的模块导入等不符合编码规范的代码
* 代码错误检查。如未被实现的接口，方法缺少对应参数，访问模块中未定义的变量等
* 发现重复以及设计不合理的代码，帮助重构
* 高度的可配置化和可定制化，通过 pylintrc 文件的修改可以定义自己适合的规范
* 支持各种 IDE 和编辑器集成。如 Emacs、Eclipse、WingIDE、VIM、Spyder 等
* 能够基于 Python 代码生成 UML 图，Pylint0.15 中就集成了 Pyreverse，能够轻易生成 UML 图形
* 能够与 Hudson、Jenkins 等持续集成工具相结合支持自动代码审查

使用 Pylint 分析代码，输出分为两部分：一部分为源代码分析结果，第二部分为统计报告。报告部分主要是一些统计信息，总体来说有以下6 类：

* `Statistics by type`：检查的模块、函数、类等数量，以及它们中存在文档注释以及不良命名的比例
* `Raw metrics`：代码、注释、文档、空行等占模块代码量的百分比统计
* `Duplication`：重复代码的统计百分比
* `Messages by category`：按照消息类别分类统计的信息以及和上一次运行结果的对比
* `Messages`：具体的消息 ID 以及它们出现的次数
* `Global evaluation`：根据公式计算出的分数统计：`10.0 - ((float(5 * error + warning + refactor + convention) / statement) * 10)`

源代码分析主要以消息的形式显示代码中存在的问题，消息以 `MESSAGE_TYPE:LINE_NUM:[OBJECT:]MESSAGE` 的形式输出，主要分为以下 5 类：

* （C）惯例，违反了编码风格标准
* （R）重构，写得非常糟糕的代码
* （W）警告，某些 Python 特定的问题
* （E）错误，很可能是代码中的 bug
* （F）致命错误，阻止 Pylint 进一步运行的错误

如果信息输出 `trailing-whitespace` 信息，可以使用命令 `pylint --help-msg="trailing-whitespace"` 来查看，这个是行尾存在空格。

如果不希望对这类代码风格进行检查，可以使用命令行过滤掉这些类别的信息，比如 `pylint -d C0303,W0312 BalancePoint.py`

`Pylint` 支持可配置化，如果在项目中希望使用统一的代码规范而不是默认的风格来进行代码检查，可以指定 `--generate-rcfile` 来生成配置文件。默认的 `Pylintrc` 可以在 `Pylint` 的目录 examples 中找到。如默认支持的变量名的正则表达式为：`variable-rgx=[a-z_][a-z0-9_]{2,30}$`，可以根据自己需要进行相应修改。其他配置如 reports 用于控制是否输出统计报告；`max-module-lines` 用于设置模块最大代码行数；`max-line-length` 用于设置代码行最大长度；`max-args` 用于设置函数的参数个数等。

### 建议 77：进行高效的代码审查

有效的代码审查流程非常必要，它能够使得大量 bug 在该阶段被发现，并且大幅度降低 bug 修复的总体代价，因为代码审查阶段 bug 的修复代价非常小。团队应该以这样的态度去看待代码审查：

* 不要错误地理解代码审查会的目的，代码审查会的首要目的是提高代码质量，找出 defect 或者设计上的不足而非修复 defect。
* 代码审查过程不应该有 KPI（Key Performance Indicator） 评价的成分
* 对管理层的建议：除非经理或者组长真正参与到具体的技术问题，否则应该尽量避免这些人员参与代码审查会，因为这些人直接与员工 KPI 挂钩
* 对开发者的建议：把代码审查当做一个学习的机会

#### 定位角色

一般来说代码审查会上有 4 类角色：仲裁者、会议记录者、被评审开发人员和评审者。

* 仲裁者，一般由技术专家或者资深技术人员担任，主要职责是控制会议流程和时间，保证会议流程的高效性，同时能够在必要的时候给予评审技术指导
* 会议记录者：及时记录评审过程中发现的问题，包括问题的提出者、问题描述、问题产生的位置等
* 被评审开发人员：一般被评审开发人员在评审开始的时候应该对其代码有综合性的介绍，在评审者提出问题或者质疑的时候应该及时给出解释
* 评审者：除了以上 3 类人员外，其余与会的人都称为评审者

#### 充分准备

在代码提交审查之前，代码作者需要对代码进行自我修正等

#### 合理使用技术和工具

* 检查表（checklist）：检查表有利于有针对性地发现代码中存在的问题，如变量是否初始化，函数调用的参数，命名是否一致，字符串是否正确解码，有没有 import 未使用的 lib，逻辑操作符是否正确，（）、{} 对是否一致等
* 台面检查（Desk Checking）：适合在编码早期对顺序执行的代码进行检查，手工模拟代码的执行过程来检查程序中潜在的问题
* IRT（`Interleaving Review Technique`）：适合于并发性的代码或者容错性系统
* 代码审查工具，如 `Rietveld、review board、Collaborative Code Review Tool（CCRT）` 等

#### 控制评审时间和评审内容

为了保证效率，一般来说一次评审时间要尽量控制在 45 分钟到 1 个小时。一次评审的代码行数应控制在 200 行以内，最好不超过 400 行。

#### 关注技术层面，对事不对人

要把重点放在技术问题以及如何解决上，而不是诸如代码风格、时间进度之类的非技术层面的问题上。

#### 记录问题，追踪进一步行动

#### 不要忽视附加的培训作用

### 建议 78：将包发布到 PyPI

建立项目之后，添加了相应的业务代码，并通过测试之后，就可以考虑发布给下游用户了。如果是项目内部协作，把项目打一个 zip 包或者 tar ball 发出去，最简单不过了。另外 `setuptools` 仍然提供了完善的支持：

`sudo python setup.py sdist --formats=zip, gztar`

`setuptools` 的 `sdist` 命令的意思是构建一个源代码发行包，它将根据调用 `setup()` 函数时给定的实参将整个项目打包（和压缩）。根据当前的平台（操作系统）不同，产生的文件也是不一样的。一般在 MS Windows 系统下，产生 `.zip` 格式的压缩包，而在 GNU Linux 或 macOS 系统下，产生 `.tar.gz` 格式的压缩包。可以使用 `--formats` 参数，指定产生 `.zip` 格式和 `.tar.gz` 格式。最终产生的包文件放在 `./dist` 目录下。

产生这两个包以后，就可以发布给项目的下游合作者了。发布方式可以是邮件、FTP，或者直接使用 IM 传送。下游开发者收到后有两种安装方式：一种是解压缩，然后进入 `setup.py` 进行安装，另一种是使用 pip 安装。

如果是较大的团队一起协作一个项目，最好把包发布到 PyPI 上面。可以是 pypi.python.org 这个官方的 PyPI，也可以是团队架设的私有 PyPI。

标准库 distutils 自身已经带有发布到 PyPI 的功能，那就是 register 和 upload 命令。register 命令用以在 PyPI 上面注册一个包，这个包名必须是尚未使用的。在注册包名之前，先在 PyPI 上注册一个用户，可以通过 PyPI 网页注册，也可以直接使用 register 命令提供的选项注册。

验证通过以后，distutils 向 PyPI 申请注册包名，一般都能够成功。但如果这个包名已经被别的用户使用过了，那会引发一个 403 错误。

涉及到的命令如下：

```python
python setup.py --help-commands
python setup.py register
python setup.py register -n
python setup.py sdist upload
```

