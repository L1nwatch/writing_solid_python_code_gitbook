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
  * 测试运行期（test runner）：控制和驱动整个单元测试过程，一般使用 TestRunner 类作为测试用例的基本执行环境，常用的运行器为 TextTestRunner，它是 TestRunner 的子类，以文字方式运行测试并报告结果。

