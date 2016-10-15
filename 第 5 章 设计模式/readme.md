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

