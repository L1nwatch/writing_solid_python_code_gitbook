## 第 5 章 设计模式

软件开发行业的设计模式广为人知，这是 GoF 的《设计模式——可复用面向对象软件的基础》的功劳，后来的《Head First 设计模式》则通过幽默的文风使其广泛流行于程序员之间。但是这两本分别使用 C++ 和 Java 编程语言作为载体，不能直接照搬到 Python 程序中，否则会有静态语言风格。

Python 的动态语言特性并不能完全替代设计模式。

### 建议 50：利用模块实现单例模式

在 GoF 的 23 种设计模式中，单例是最常使用的模式，通过单例模式可以保证系统中一个类只有一个实例而且该实例易于被外界访问，从而方便对实例个数的控制并节约系统资源。每当大家想要实现一个 XxxManager 的类时，往往意味着这是一个单例。

有不少现代编程语言将其加到了语言特性中，如 scala 和 falcon 语言都把 object 定义成关键词，并用其声明单例。如在 scala 中，一个单例如下：

```scala
object Singleton {
  def show = println("I am a singleton")
}
```

object 定义了一个名为 Singleton 的单例，它满足单例的 3 个需求：一是只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。对于第三点，在任何地方都可以通过调用 `Singleton.show()` 来验证。在 scala 中，单例没有显式的初始化操作，但并不是所有在语法层面支持单例模式的编程语言都如此，比如 falcon 就不一样。

```falcon
object object_name [from class1, class2 ... classN]
    property_1 = expression
    property_2 = expression
    ...
    property_N = expression
    [init block]
    function method_1([parameter_list])
        [method_body]
    end
    ...
    function method_N([parameter_list])
        [method_body]
    end
end
```

`[init block]` 能够让程序员手动控制单例的初始化代码。但是与 scala 和 falcon 相比，动态语言 Python 缺乏声明私有构造函数的语法元素，实例又带有类型信息。所以以下方法是不可行的：

```python
class _Singleton(object):
    pass
Singleton = _Singleton()
del _Singleton	# 试图删除 class 定义
another = Singleton.__class__()	# 没用，绕过！
print(type(another))
# 输出
<class '__main__._Singleton'>
```

可见虽然把 Singleton 的类定义删除了，但仍然有办法通过已有实例的 `__class__` 属性生成一个新的实例。于是许多 Pythonista 把目光聚集到真正创建实例的方法 `__new__` 上：

```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls._instance
if __name__ = '__main__':
    s1 = Singleton()
    s2 = Singleton()
    assert id(s1) == id(s2)
```

这个方法基本上可以保证“只能有一个实例”的要求了，但是在并发情况下可能会发生意外，解决办法是引入锁：

```python
class Singleton(object):
    objs = {}
    objs_locker = threading.Lock()
    def __new__(cls, *args, **kwargs):
        if cls in cls.objs:
            return cls.objs[cls]
        cls.objs_locker.acquire()
        try:
            if cls in cls.objs:	## double check locking
                return cls.objs[cls]
            cls.objs[cls] = object.__new__(cls)
        finally:
            cls.objs_locker.release()
```

利用经典的双检查锁机制，确保了在并发环境下 Singleton 的正确实现。但这个方案并不完美，比如以下两个问题：

* 如果 Singleton 的子类重载了 `__new__()` 方法，会覆盖或者干扰 Singleton 类中 `__new__()` 的执行，虽然这种情况出现的概率极小，但不容忽视。
* 如果子类有 `__init__()` 方法，那么每次实例化该 Singleton 的时候，`__init__()` 都会被调用到，这显然是不应该的，`__init__()` 只应该在创建实例的时候被调用一次。

这两个问题当然可以解决，比如通过文档告知其他程序员，子类化 Singleton 的时候，务必调用父类的 `__new__()` 方法；而第二个问题也可以通过偷偷地替换掉 `__init__()` 方法来确保它只调用一次。但是，为了实现一个单例，做大量的、水面之下的工作相当不 Pythonic。

模块采用的其实是天然的单例的实现方式：

* 所有的变量都会绑定到模块
* 模块只初始化一次
* import 机制是线程安全的（保证了在并发状态下模块也只有一个实例）

所以创建一个 world 单例时：

```python
# World.py
import Sun
def run():
    while True:
        Sun.rise()
        Sun.set()
```

然后在入口文件 `main.py` 里导入，并调用 `run()` 函数：

```python
# main.py
import World
World.run()
```

> #### 注意
>
> Alex Martelli 认为单例模式要求“实例的唯一性”本身是有问题的，实际更值得关注的是实例的状态，只要所有的实例共享状态（可以狭义地理解为属性）、行为（可以狭义地理解为方法）一致就可以了。于是有 Borg 模式（在 C# 中又称为 Monostate 模式）。
>
> ```python
> class Borg:
>     __shared_state = {}
>     def __init__(self):
>         self.__dict__ = self.__shared_state
>     # and whatever else you want in your class -- that's all!
> ```
>
> 通过 Borg 模式，可以创建任意数量的实例，但因为它们共享状态，从而保证了行为一致。Alex 的这个 Borg 模式仅适用于古典类（classic classess），Python2.2 以后的新式类（new-style classes）需要使用 `__getattr__` 和 `__setattr__` 方法来实现。

### 建议 51：用 mixin 模式让程序更加灵活

先来了解一下模版方法模式，模版方法模式就是在一个方法中定义一个算法的骨架，并将一些实现步骤延迟到子类中。模版方法可以使子类在不改变算法结构的情况下，重新定义算法中的某些步骤。

模版方法在 C++ 或其他语言中并无不妥，但在 Python 中有点画蛇添足。比如模版方法，需要先定义一个基类，而实现行为的某些步骤则必须在其子类中，在 Python 中并无必要。

```python
class People(object):
    def make_tea(self):
        teapot = self.get_teapot()
        teapot.put_in_tea()
        teapot.put_in_water()
        return teapot
```

`get_teapot()` 方法并不需要预先定义：

```python
class OfficePeople(People):
    def get_teapot(self):
        return SimpleTeaPot()
    
class HomePeople(People):
    def get_teapot(self):
        return KungfuTeapot()
```

虽然看起来像模板方法，但是基类并不需要预先声明抽象方法，甚至还带来吊事代码的便利。

如果子类没有实现 `get_teapot()` 方法，所以一调用 `make_tea()` 就会产生一个找不到方法的 `AttributeError`。

但是，这样导致方法只能实现一个，解决方法有两种：一种是继承子类，再重写 `get_teapot()`；另一个则是把 `get_teapot()` 方法提取出来，把它以多继承的方式做一次静态混入。

```python
class UseSimpleTeapot(object):
    def get_teapot(self):
        return SimpleTeaPot()
    
class UseKungfuTeapot(object):
    def get_teapot(self):
        return KungfuTeapot()
    
class OfficePeople(People, UseSimpleTeapot):
    pass
class HomePeople(People, UseKungfuTeapot):
    pass
class Boss(People, UseKungfuTeapot):
    pass
```

但是这样的代码仍然没有把 Python 的动态性表现出来，当新的需求出现时，需要更改类定义。

于是我们开始寄望于动态地生成不同的实例：

```python
def simple_tea_people():
    people = People()
    people.__bases__ += (UseSimpleTeapot, )
    return people
def coffee_people():
    people = People()
    people.__bases__ += (UseCoffeepot, )
    return people
def tea_and_coffee_people():
    people = People()
    people.__bases__ += (UseSimpleTeapot, UseCoffeepot, )
    return people
def boss():
    people = People()
    people.__bases__ += (KungfuTeapot, UseCoffeepot, )
    return people
```

这个代码能够运行的原理是，每个类都有一个 `__bases__` 属性，它是一个元组，用来存放所有的基类。与其他静态语言不同，Python 语言中的基类在运行中可以动态改变。所以当我们向其中添加新的基类时，这个类就拥有了新的方法，也就是所谓的混入（mixin）。这种动态性的好处在于代码获得了更丰富的扩展功能。

值得进一步探索的是，利用反射技术，甚至不需要修改代码：

```python
import mixins
def staff():
    people = People()
    bases = []
    for i in config.checked():
        bases.append(getattr(mixins, i))
    people.__bases__ += tuple(bases)
    return people
```

通过这个框架代码，OA 系统的开发人员只需要把常见的需求定义成 Mixin 预告放在 mixins 模块中，就可以在不修改代码的情况下通过管理界面满足几乎所有需求了。

### 建议 52：用发布订阅模式实现松耦合

发布订阅模式（publish/subscribe 或 pub/sub）是一种编程模式，消息的发送者（发布者）不会发送器消息给特定的接收者（订阅者），而是将发布的消息分为不同的类别直接发布，并不会关注订阅者是谁。而订阅者可以对一个或多个类别感兴趣，且只接收感兴趣的消息，并且不关注是哪个发布者发布的消息。这种发布者和订阅者的解耦可以允许更好地可扩放性和更为动态的网络拓扑。

发布订阅模式的优点是发布者与订阅者松散的耦合，双方不需要知道对方的存在。由于主题是被关注的，发布者和订阅者可以对系统拓扑毫无所知。无论对方是否存在，发送者和订阅者都可以继续正常操作。要实现这个模式，就需要有一个中间代理人，在实现中一般被称为 Broker，它维护着发布者和订阅者的关系：定于这把感兴趣的主题告诉它，而发布者的信息也通过它路由到各个订阅者处。

简单的实现如下：

```python
from collections import defaultdict
route_table = defaultdict(list)
def sub(self, topic, callback):
    if callback in route_table[topic]:
        return
    route_table[topic].append(callback)
def pub(self, topic, *a, **kw):
    for func in route_table[topic]:
        func(*a, **kw)
```

直接放在一个叫 `Broker.py` 的模块中（单件），省去了各种参数检测、优先处理的需求等，甚至没有取消订阅的函数，但它展现了发布订阅模式实现的最基础的结构。它的应用代码：

```python
import Broker
def greeting(name):
    print("Hello, {}".format(name))
Broker.sub("greet", greeting)
Broker.pub("greet", "LaiYonghao")
```

相对于这个简化版本，blinker 和 python-message 两个模块的实现要完备得多。blinker 已经被用在了多个广受欢迎的项目上，比如 flask 和 django；而 python-message 则支持更多丰富的特性。

安装 python-message：`pip install message`

验证：

```python
import message
def hello(name):
    print("Hello {}".format(name))
message.sub("greet", hello)
message.pub("greet", "lai")
```

假设现在有两个模块使用不同的形式进行日志输出，于是可以这么编写 `bar()` 函数：

```python
import message
LOG_MSG = ("log", "foo")
def bar():
    message.pub(LOG_MSG, "Haha, Calling bar().")
    do_sth()
```

在已有的项目中，只需要在项目开始处加上这样的代码，继续把日志放到标准输出：

```python
import message
import foo
def handle_foo_log_msg(txt):
    print(txt)
message.sub(foo.LOG_MSG, handle_foo_log_msg)
```

而在那个使用 logging 的新项目中，则这样修改：

```python
def handle_foo_log_msg(txt):
    import logging
    logging.debug(txt)
```

甚至在一些不关注底层库的日志项目中，直接无视就可以了。通过 message，可以轻松获得库与应用之间的解耦，因为库关注的是要有日志，而不关注日志输出到哪里；应用关注的是日志要统一放置，但不关系谁往日志文件中输出内容，这与发布订阅模式类似。

除了简单的 `sub()/pub()` 之外，python-message 还支持取消订阅（`unsub()`）和中止消息传递。

```python
import message
def hello(name):
    print("hello {}".format(name))
    ctx = message.Context()
    ctx.discontinued = True
    return ctx
def hi(name):
    print("u cann't c me.")
message.sub("greet", hello)
message.sub("greet", hi)
message.pub("greet", "lai")
```

python-message 利用回调函数的返回值来实现消息传递。这里消息在调用 `hello()` 后就中止传递了（Broker 使用 list 对象存储回调函数就是为了保证次序）

python-message 是同步调用回调函数的，也就是说谁先 sub 谁就先被调用。大部分情况下这样已经能够满足大部分需求，但有时需要后 sub 的函数先被调用，这时 `message.sub` 函数通过一个默认参数来支持，只需要在调用 `sub` 的时候加上 `front=True`，这个回调函数将被插到所有之前已经 sub 的回调函数之前：`sub("greet", hello, front=True)`。

订阅/发布模式是观察者模式的超集，它不关注消息是谁发布的，也不关心消息由谁处理。如果需要自己的类也能够方便地订阅/发布消息，也就是想退化为观察者模式，python-message 同样提供了支持：

```python
from message import observable
def greet(people):
    print("hello, {}".format(people.name))
@observable
class Foo(object):
    def __init__(self, name):
        print("Foo")
        self.name = name
        self.sub("greet", greet)
	def pub_greet(self):
        self.pub("greet", self)
foo = Foo("lai")
foo.pub_greet()
```

python-message 提供了类装饰函数 `observable()`，任何 class 只需要通过它装饰一下就拥有了 `sub/ubsub/pub/declare/retract` 等方法，它们的使用方法跟全局函数是类似的。

> #### 注意
>
> 因为 python-message 的消息订阅默认是全局性的，所以有可能产生名字冲突。在减少名字冲突方面，可以借鉴 `java/actionscript3` 的 package 起名策略，比如在应用中定义消息主题常量 `FOO='com.googlecode.python-message.FOO'`，这样多个库同时定义 FOO 常量也不容易冲突。除此之外，就是使用 uuid：
>
> ```python
> uuid = 'bd1321321312321312321'
> FOO = uuid + "FOO"
> ```

### 建议 53：用状态模式美化代码

状态模式，就是当一个对象的内在状态改变时允许改变其行为，但这个对象看起来像是改变了其类。状态模式主要用于控制一个对象状态的条件表达式过于复杂的情况，其可把状态的判断逻辑转移到表示不同状态的一系列类中，进而把复杂的判断逻辑简化。

由于 Python 语言的动态性，状态模式的 Python 实现与 C++ 等语言的版本比起来简单得多。

```python
def workday():
    print("work hard!")
def weekend():
    print("play harder!")
class People(object):
    pass
people = People()
while True:
    for i in xrange(1, 8)
        if i == 6:
            people.day = weekend
        if i == 1:
            people.day = workday
        people.day()
```

通过在不同的条件下将实例的方法（即行为）替换掉，就实现了状态模式。但仍然有缺陷：

* 查询对象的当前状态很麻烦
* 状态切换时需要对原状态做一些清扫工作，而对新的状态需要做一些初始化工作，因为每个状态需要做的事情不同，全部写在切换状态的代码中必然重复，所以需要一个机制来简化。

python-state 包通过几个辅助函数和修饰函数很好地解决了这个问题，并且定义了一个简明状态机框架：

安装：`pip install state`

然后改写：

```python
from state import curr, switch, stateful, State, behavior
@stateful
class People(object):
    class Workday(State):
        default = True
        @behavior
        def day(self):
            print("work hard!")
	class Weekend(State):
        @behavior
        def day(self):
            print("play harder!")
people = People()
while True:
    for i in xrange(1, 8):
        if i == 6:
            switch(people, People.Weekend)
		if i == 1:
            switch(people, People.Workday)
        people.day()
```

首先是 `@stateful` 这个修饰函数，其中最重要的是重载了被修饰类的 `__getattr__()` 方法从而使得 People 的实例能够调用当前状态类的方法。被 `@stateful` 修饰后的类的实例是带有状态的，能有使用 `curr()` 查询当前状态，也可以使用 `switch()` 进行状态切换。

可以看到类 Workday 继承自 State 类，这个 State 类也是来自于 state 包，从其派生的子类能够使用 `__begin__` 和 `__end__` 状态转换协议，通过重载这两个协议，子类能够自定义进入和离开当前状态时对宿主的初始化和清理工作。对于一个 `@stateful` 类而言，有一个默认的状态（即其实例初始化后的第一个状态），通过类定义的 default 属性标识，defalut 设置为 True 的类成为默认状态。`@behavior` 修饰函数用以修饰状态类的方法，其实它是内置函数 `staticmethod` 的别名。

之所以将状态类的方法实现为静态方法，这是因为 state 包的原则是状态类只有行为，没有状态（状态都保存在宿主上），这样可以更好地实现代码重用。然而既然 `day()` 方法是静态的，却有 self 参数。这其实使因为 self 并不是 Python 的关键字，在这里使用 self 有助于理解状态类的宿主是 People 的实例。

通过状态模式，可以像 decorator 一样去掉 `if...raise...` 上下文判断，而且真的是一个 `if...raise...` 都没有了。另外，需要多重判断的时候要给一个方法戴上多个装饰函数的情况也没有了，还通过把多个方法分派到不同的状态类，消灭掉巨类，保持类的短小，更容易维护和重用。而且还有一个好处：当调用当前状态不存在的行为时，出错信息抛出的是 AttributeError，从而避免把问题变为复杂的逻辑错误，让程序员更容易找到出错位置，进而修正问题。

```python
@stateful
class User(object):
    class NeedSignin(State):
        default = True
        @behavior
        def signin(self, usr, pwd):
            ...
            switch(self, Player.Signin)
	class Signin(State):
        @behavior
        def move(self, dst):
            ...
        @behavior
        def atk(self, other):
            ....
```

