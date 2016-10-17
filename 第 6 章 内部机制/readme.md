# 第 6 章 内部机制

### 建议 54：理解 built-in objects

Python 中一切皆对象：字符是对象，列表是对象，内建类型（`built-in type`）也是对象；用户定义的类型是对象，object 是对象，type 也是对象。自 Python2.2 以后，为了弥补内建类型和古典类（classic classes）之间的鸿沟引入了新式类（new-style classes）。在新式类中，object 是所有内建类型的基类，用户所定义的类可以继承自 object 也可以继承自内建类型。

> #### 鸿沟
>
> 在 2.2 版本之前，类和类型并不统一，如 a 是古典类 ClassA 的一个实例，那么 `a.__class__` 返回 `class__main__ClassA`，type(a) 返回 `<type 'instance'>`。当引入新类后，比如 ClassB 是个新类，b 是 ClassB 的实例，`b.__class__` 和 `type(b)` 都是返回 `class__main__.ClassB`。

一些结论：

* 在 Python 中一切皆对象，type 也是对象


* `object` 和古典类没有基类，`type` 的基类为 `object`
* 新式类中 `type()` 的值和 `__class__` 的值是一样的，但古典类中实例的 `type` 为 `instance`，其 `type()` 的值和 `__class__` 的值不一样
* 继承自内建类型的用户类的实例也是 `object` 的实例，`object` 是 `type` 的实例，`type` 实际是个元类（metaclass）
* `object` 和内建类型以及所有基于 `type` 构建的用户类都是 `type` 的实例
* 在古典类中，所有用户定义的类的类型都为 `instance`

古典类和新式类的一个区别是：新式类继承自 `object` 类或者内建类型。我们不能简单地从定义的形式上来判断一个类是新式类还是古典类（`__metaclass__` 属性会影响到），应当通过元类的类型来确定类的类型：古典类的元类为 `types.ClassType`，新式类的元类为 `type` 类。

新式类相对于古典类来说有很多优势：能够基于内建类型构建新的用户类型，支持 property 和描述符特性等。

作为新式类的祖先，Object 类中还定义了一些特殊方法，如：`__new__()、__init__()、__delattr__()、__getattribute__()、__setattr__()、__hash__()、__repr__()、__str__()` 等。object 的子类可以对这些方法进行覆盖以满足自身的特殊需求。

### 建议 55：`__init__()` 不是构造方法

从表面上看它确实很想构造方法：当需要实例化一个对象的时候，使用 `a=Class(args...)` 便可以返回一个类的实例，其中 args 的参数与 `__init__()` 方法中申明的参数一样。

`__init__()` 并不是真正意义上的构造方法，`__init__()` 方法所做的工作是在类的对象创建好之后进行变量的初始化。`__new__()` 方法才会真正创建实例，是类的构造方法。这两个方法都是 object 类中默认的方法，继承自 object 的新式类，如果不覆盖这两个方法将会默认调用 object 中对应的方法。

来看看 `__new__()` 方法和 `__init__()` 方法的定义：

* `object.__new__(cls[, args...])`：其中 cls 代表类，args 为参数列表
* `object.__init__(self[, args...])`：其中 self 代表实例对象，args 为参数列表

这两个方法之间有些不同点，总结如下：

* 根据 Python [文档](htts://docs.python.org/2/reference/datamodel.html#object.__new__)可知，`__new__()` 方法是静态方法，而 `__init__()` 为实例方法
* `__new__()` 方法一般需要返回类的对象，当返回类的对象时将会自动调用 `__init__()` 方法进行初始化，如果没有对象返回，则 `__init__()` 方法不会被调用。`__init__()` 方法不需要显示返回，默认为 None，否则会在运行时抛出 TypeError
* 当需要控制实例创建的时候可使用 `__new__()` 方法，而控制实例初始化的时候使用 `__init__()` 方法
* 一般情况下不需要覆盖 `__new__()` 方法，但当子类继承自不可变类型，如 `str`、`int`、`unicode` 或者 `tuple` 的时候，往往需要覆盖该方法
* 当需要覆盖 `__new__()` 和 `__init__()` 方法的时候这两个方法的参数必须保持一致，如果不一致将导致异常。

一般情况下覆盖 `__init__()` 能满足大部分需求，特殊情况下需要覆盖 `__new__()` 方法：

* 当类继承（如 str、int、unicode、tuple 或者 forzenset 等）不可变类型且默认的 `__new__()` 方法不能满足需求的时候。比如需要一个不可修改的集合，该集合能够将任何以空格隔开的字符串变为集合中的元素：

  ```python
  class UserSet(frozenset):
      def __new__(cls, *args):
          if args and isinstance(args[0], basestring):
              args = (args[0].split(), ) + args[1:]
          return super(UserSet, cls).__new__(cls, *args)
  ```

* 用来实现工厂模式或者单例模式或者进行元类编程（元类编程中常常需要使用 `__new__()` 来控制对象创建）的时候。

  以简单工厂为例子，它由一个工厂类根据传入的参量决定创建出哪一种产品类的实例，属于类的创建型模式：

  ```python
  class Shape(object):
      def __init__(object):
          pass
      def draw(self):
          pass
      
  class Triangle(Shape):
      def __init__(self):
          print("I am a triangle")
      def draw(self):
          print("I am drawing triangle")
          
  class Rectangle(Shape):
      def __init__(self):
          print("I am a rectangle")
      def draw(self):
          print("I am drawing triangle")
          
  class Trapezoid(Shape):
      def __init__(self):
          print("I am a trapezoid")
      def draw(self):
          print("I am drawing triangle")
          
  class Diamond(Shape):
      def __init__(self):
          print("I am a diamond")
      def draw(self):
          print("I am drawing triangle")
          
  class ShapeFactory(object):
      shapes = {"triangle": Triangle, "rectangle": Rectangle, "trapezoid": Trapezoid, "diamond": Diamond}
      def __new__(class, name):
          if name in ShapeFactory.shapes.keys():
              print("creating a new shape {}".format(name))
              return ShapeFactory.shapes[name]()
          else:
              print("craeting a new shape {}".format(name))
              return Shape()
  ```

  在 ShapeFactory 类中重新覆盖了 `__new__()` 方法，外界通过调用该方法来创建其所需的对象类型，但如果所请求的类是系统所不支持的，则返回 Shape 对象。在引入了工厂类之后，只需要使用如下形式就可以创建不同的图形对象：

  `ShapeFactory("rectangle").draw()`

* 作为用来初始化的 `__init__()` 方法在多继承的情况下，子类的 `__init__()` 方法如果不显式调用父类的 `__init__()` 方法，则父类的 `__init__()` 方法不会被调用。

要显式调用父类的 `__init__()` 方法：`super(子类, self).__init__()`。对于多继承的情况，我们可以通过迭代子类的 `__bases__` 属性中的内容来逐一调用父类的初始化方法。

### 建议 56：理解名字查找机制

在 Python 中，所有所谓的变量，其实都是名字，这些名字指向一个或多个 Python 对象。

所有的这些名字，都存在于一个表里（又称为命名空间），一般情况下，我们称之为局部变量（locals），可以通过 `locals()` 函数调用看到。

在一个 `globals()` 的表里可以看到全局变量，注意如果是在 Python shell 中执行  `locals()`，也可以看到全局的变量。如果在一个函数里面定义这些变量，情况就会有所不同。

Python 中所有的变量名都是在赋值的时候生成的，而对任何变量名的创建、查找或者改变都会在命名空间（namespace）中进行。变量名所在的命名空间直接决定了其能访问到的范围，即变量的作用域。Python 中的作用域自 Python2.2 之后分为局部作用域（local）、全局作用域（Global）、嵌套作用域（enclosing functionas locals）以及内置作用域（Build-in）这 4 种。

* 局部作用域：一般来说函数的每次调用都会创建一个新的本地作用域，拥有新的命名空间。因此函数内的变量名可以与函数外的其他变量名相同，由于其命名空间不通过，并不会产生冲突。默认情况下函数内部任意的赋值操作（包括 = 语句、import 语句、def 语句、参数传递等）所定义的变量名，如果没用 global 语句，则申明都为局部变量，即仅在该函数内可见
* 全局作用域：定义在 Python 模块文件中的变量名拥有全局作用域，这里的全局仅限单个文件，即在一个文件的顶层的变量名仅在这个文件内可见，并非所有的文件，其他文件中想使用这些变量名必须先导入文件对应的模块。当在函数之外给一个变量赋值时是在其全局作用域的情况下进行的。
* 嵌套作用域：一般在多重函数嵌套的情况下才会注意到。需要注意的是 global 语句仅针对全局变量，在嵌套作用域的情况下，如果想在嵌套的函数内修改外层函数中定义的变量，即使使用 global 进行申明也不能达到目的，其结果最终是在嵌套的函数所在的命名空间中创建了一个新的变量。
* 内置作用域：它是通过一个标准库中名为 `__builtin__` 的模块来实现的

当访问一个变量的时候，其查找顺序遵循变量解析机制 LEGB 法则，即依次搜索 4 个作用域：局部作用域、嵌套作用域、全局作用域以及内置作用域，并在第一个找到的地方停止搜寻，如果没有搜到，则会抛出异常。具体来说 Python 的名字查找机制如下：

* 在最内层的范围内查找，一般而言，就是函数内部，即在 `locals()` 里面查找
* 在模块内查找，即在 `globals()` 里面查找
* 在外层查找，即在内置模块中查找，也就是在 `__builtin__` 中查找

在 CPython 的实现中，只要出现了赋值语句（或者称为名字绑定），那么这个名字就被当做局部变量来对待。需要改变全局变量时，使用 global 关键字。

在 Python 闭包中，有这样的问题：

```python
>>> def foo():
	a = 1
	def bar():
		b = a * 2
		a = b + 1
		print(a)
	return bar

>>> foo()()
Traceback (most recent call last):
  File "<pyshell#9>", line 1, in <module>
    foo()()
  File "<pyshell#8>", line 4, in bar
    b = a * 2
UnboundLocalError: local variable 'a' referenced before assignment
```

在闭包 `bar()` 中，在编译代码为字节码时，因为存在 `a = b + 1` 这条语句，所以 a 被当做了局部变量看待，而执行时 `b = a * 2` 先执行，此时局部变量 a 尚不存在，所以就产生了一个异常。在 Python2.X 中可以使用 global 关键字解决部分问题，先把 a 创建为一个模块全局变量，然后在所有读写（包括只是访问）该变量的作用域中都要先使用 global 声明其为全局变量。

```python
>>> a = 1
>>> def foo(x):
        global a
        a = a * x
        def bar():
            global a
            b = a * 2
            a = b + 1
            print(a)
        return bar
>>> foo(1)()
3
```

编程语言并不提倡全局变量，而且这种写法有时候还影响业务逻辑。此外，还有把 a 作为容器的一个元素来对待的方案，但也都相当复杂。真正的解决方案是 Python3 引入的 `nonlocal` 关键字：

```python
>>> def foo(x):
	a = x
	def bar():
		nonlocal a
		b = a * 2
		a = b + 1
		print(a)
	return bar

>>> foo(1)
<function foo.<locals>.bar at 0x105de2bf8>
```

### 建议 57：为什么需要 self 参数

在类中当定义实例方法的时候需要将第一个参数显式声明为 self，而调用的时候并不需要传入该参数。

self 表示的就是实例对象本身，即类的对象在内存中的地址。self 是对对象本身的引用。我们在调用实例方法的时候也可以直接传入实例对象。其实 self 本身并不是 Python 的关键字（cls 也不是），可以将 self 替换成任何你喜欢的名称，如 this、obj 等，实际效果和 self 是一样的（但并不推荐，因为 self 更符合约定俗成的原则）

在方法声明的时候需要定义 self 作为第一个参数，而调用方法的时候却不用传入这个参数。虽然这并不影响语言本身的使用，而且也很容易遵循这个规则，但既然如此，为什么必须在定义方法的时候声明 self 参数？原因如下：

* Python 在当初设计的时候借鉴了其他语言的一些特征，如 `Moudla-3` 中方法会显式地在参数列表中传入 self。Python 起源于 20 世纪 80 年代末，那个时候的很多语言都有 self，如 `Smalltalk`、`Modula-3` 等。Python 在最开始设计的时候受到了其他语言的影响，因此借鉴了其中的一些理念。

* Python 语言本身的动态性决定了使用 self 能够带来一定便利。

  Python 属于一级对象语言（first class object)，如果 m 是类 A 的一个方法，有好几种方式都可以引用该方法：

  ```python
  >>> class A:
          def m(self, value):
              pass
  >>> A.__dict__["m"]
  >>> A.m.__func__
  ```

  实例方法是作用于对象的，最简单的方式就是将对象本身传递到该方法中去，self 的存在保证了 `A.__dict__['m'](a, 2)` 的使用和 `a.(2)` 一致。同时当子类覆盖了父类中的方法但仍然想调用该父类的方法的时候，可以方便地使用 `baseclass.methodname(self, <argument list>)` 或 `super(childclass, self).methodname(<argument list>)` 来实现。

* 在存在同名的局部变量以及实例变量的情况下使用 self 使得实例变量更容易被区分

Guido 认为，基于 Python 目前的一些特性（如类中动态添加方法，在类风格的装饰器中没有 self 无法确认是返回一个静态方法还是累方法等）保留其原有设计是个更好的选择，更何况 Python 的哲学是：显示优于隐式（Explicit is better than implicit）。

### 建议 58：理解 MRO 与多继承

Python 也支持多继承，语法：

`class DerivedClassName(Base1, Base2, Base3)`

古典类和新式类之间所采用的 MRO（Method Resolution Order，方法解析顺序）实现方式存在差异。

在古典类中，MRO 搜索采用简单的自左向右的深度优先方法，即按照多继承申明的顺序形成继承树结构，自顶向下采用深度优先的搜索顺序，当找到所需要的属性或者方法的时候就停止搜索。

而新式类采用的而是 C3 MRO 搜索方法，该算法描述如下：

> 假定，C1C2...CN 表示类 C1 到 CN 的序列，其中序列头部元素（head）=C1，序列尾部（tail）定义 = C2...CN；
>
> C 继承的基类自左向右分别表示为 B1，B2...BN
>
> L[C] 表示 C 的线性继承关系，其中 L[object] = object。
>
> 算法具体过程如下：
>
> L[C(B1...BN)] = C + merge(L[B1] ... L[BN], B1 ... BN)
>
> 其中 merge 方法的计算规则如下：在 L[B1]...L[BN]，B1...BN 中，取 L[B1] 的 head，如果该元素不在 L[B2]...L[BN]，B1...BN 的尾部序列中，则添加该元素到 C 的线性继承序列中，同时将该元素从所有列表中删除（该头元素也叫 good head），否则取 L[B2] 的 head。继续相同的判断，直到整个列表为空或者没有办法找到任何符合要求的头元素（此时，将引发一个异常）。

关于 MRO 的搜索顺序也可以在新式类中通过查看 `__mro__` 属性得到证实。

实际上 MRO 虽然叫方法解析顺序，但它不仅是针对方法搜索，对于类中的数据属性也适用。

菱形继承是我们在多继承设计的时候需要尽量避免的一个问题。

### 建议 59：理解描述符机制

除了在不同的局部变量、全局变量中查找名字，还有一个相似的场景，那就是查找对象的属性。在 Python 中，一切皆是对象，所以类也是对象，类的实例也是对象。

每一个类都有一个 `__dict__` 属性，其中包含的是它的所有属性，又称为类属性。

除了与类相关的类属性之外，每一个实例也有相应的属性表（`__dict__`），称为实例属性。当我们通过实例访问一个属性时，它首先会尝试在实例属性中查找，如果找不到，则会到类属性中查找。

实例可以访问类属性，但与读操作有所不同，如果通过实例增加一个属性，只能改变此实例的属性，对类属性而言，并没有变化。

能不能给类增加一个属性？答案是，能，也不能。说能是因为每一个 class 也是一个对象，动态地增减对象的属性与方法正是 Python 这种动态语言的特性，自然是支持的。

说不能，是因为在 Python 中，内置类型和用户定义的类型是有分别的，内置类型并不能够随意地为它增加属性或方法。

当我们通过 "." 操作符访问一个属性时，如果访问的实例属性，与直接通过 `__dict__` 属性获取相应的元素是一样的；而如果访问的是类属性，则并不相同；"." 操作符封装了对两种不同属性进行查找的细节。

访问类属性时，通过 `__dict__` 访问和使用 "." 操作符访问是一样的，但如果是方法，却又不是如此了。

当通过 "." 操作符访问时，Python 的名字查找并不是先在实例属性中查找，然后再在类属性中查找那么简单，实际上，根据通过实例访问属性和根据类访问属性的不同，有以下两种情况：

* 一种是通过实例访问，比如代码 `obj.x`，如果 x 是一个描述符，那么 `__getattribute__()` 会返回 `type(obj).__dict__['x'].__get__(obj, type(obj))` 结果，即：`type(obj)` 获取 obj 的类型；`type(obj).__dict__['x']` 返回的是一个描述符，这里有一个试探和判断的过程；最后调用这个描述符的 `__get__()` 方法。
* 另一个是通过类访问的情况，比如代码 `cls.x`，则会被 `__getattribute__()` 转换为 `cls.__dict__['x'].__get__(None, cls)`。

描述符协议是一个 Duck Typing 的协议，而每一个函数都有 `__get__` 方法，也就是说其他每一个函数都是描述符。

描述符机制有什么作用？其实它的作用编写一般程序的话还真用不上，但对于编写程序库的读者来说就有用了，比如已绑定方法和未绑定方法。

由于对描述符的 `__get__()` 的调用参数不同，当以 `obj.x` 的形式访问时，调用参数是 `__get__(obj, type(obj))`；而以 `cls.x` 的形式访问时，调用参数是 `__get__(None, type(obj))`，这可以通过未绑定方法的 `im_self` 属性为 None 得到印证。

除此之外，所有对属性、方法进行修饰的方案往往都用到了描述符，比如 `classmethod`、`staticmethod` 和 `property` 等。以下是 property 的参考实现：

```python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc
	def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError, "unreadable attribute"
        return self.fget(obj)
	def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError, "can't set attribute"
        self.fset(obj, value)
	def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError, "can't delete attribute"
        self.fdel(obj)
```

### 建议 60：区别 `__getattr__()` 和 `__getattribute__()` 方法

`__getattr__()` 和 `__getattribute__()` 都可以用作实例属性的获取和拦截（仅对实例属性（instance varibale）有效，非类属性），`__getattr__()` 适用于未定义的属性，即该属性在实例中以及对应的类的基类以及祖先类中都不存在，而 `__getattribute__()` 对于所有属性的访问都会调用该方法。

需要注意的是 `__getattribute__()` 仅用于新式类。

当访问一个不存在的实例属性的饿时候就会抛出 `AttributeError` 异常。这个异常时由内部方法 `__getattribute__(self, name)` 抛出的，因为 `__getattribute__()` 会被无条件调用，也就是说只要涉及实例属性的访问就会调用该方法，它要么返回实际的值，要么抛出异常。Python 的[文档](http://docs.python.org/2/reference/datamodel.html#object.getattribute)中也提到了这一点。

实际上 `__getattr__()` 方法仅如下情况才被调用：属性不在实例的 `__dict__` 中；属性不在其基类以及祖先类的 `__dict__` 中；触发 `AttributeError` 异常时（注意，不仅仅是 `__getattribute__()` 方法的 `AttributeError` 异常，property 中定义的 `get()` 方法抛出异常的时候也会调用该方法）。

当这两个方法同时被定义的时候，要么在 `__getattribute__()` 中显式调用，要么触发 `AttributeError` 异常，否则 `__getattr__()` 永远不会被调用。`__getattribute__()` 及 `__getattr__()` 方法都是 Object 类中定义的默认方法，当用户需要覆盖这些方法时有以下几点注意事项：

* 避免无穷递归。比如覆盖 `__getattribute__()` 时使用了 `self.__dict__[attr]`，正确的做法是使用 `super(obj, self).__getattribute__(attr)` 或者 `object.__getattribute(self, attr)`。无穷递归是覆盖 `__getattr__()` 和 `__getattribute__()` 方法的时候需要特别小心
* 访问未定义的属性。如果在 `__getattr__()` 方法中不抛出 `AttributeError` 异常或者显式返回一个值，则会返回 None，此时可能会影响到程序的实际运行预期。

另外关于 `__getattr__()` 和 `__getattribute__()` 有以下两点提醒：

* 覆盖了 `__getattribute__()` 方法之后，任何属性的访问都会调用用户定义的 `__getattribute__()` 方法，性能上会有所损耗，比使用默认的方法要慢。
* 覆盖的 `__getattr__()` 方法如果能够动态处理事先未定义的属性，可以更好地实现数据隐藏。因为 `dir()` 通常只显示正常的属性和方法，因此不会将该属性列为可用属性。
* property 也能控制属性的访问，如果一个类中同时定义了 `property`、`__getattribute__()` 和 `__getattr__()` 来对属性进行访问控制，则最先搜索的是 `__getattribute__()` 方法，然而由于 property 对象并不存在 dict 中，因此并不能返回该方法，此时会搜索 property 中定义的 `get()` 方法。当用 `property` 中的 `set()` 方法进行修改并再次访问 `property` 的 `get()` 方法时会抛出异常，这种情况下会触发对 `__getattr__()` 方法的调用。对类变量的访问不会涉及 `__getattribute__()` 和 `__getattr__()` 方法。

### 建议 61：使用更为安全的 property

property 是用来实现属性可管理性的 `built-in` 数据类型（注意：很多地方将 property 称为函数，然而它实际上是一种实现了 `__get__()`、`__set__()` 方法的类，用户也可以根据自己的需要定义个性化的 property），其实质是一种特殊的数据描述符（数据描述符：如果一个对象同时定义了 `__get__()` 和 `__set__()` 方法，则称为数据描述符，如果仅定义了 `__get__()` 方法，则称为非数据描述符）。它和普通描述符的区别在于：普通描述符提供的是一种较为低级的控制属性访问的机制，而 property 是它的高级应用，它以标准库的形式提供描述符的实现，其签名形式为：

`property(fget=None, fset=None, fdel=None, doc=None) -> property attribute`

Property 常见的使用形式有以下几种：

* 第一种形式如下：

  ```python
  class Some_Class(object):
      def __init__(self):
          self._somevalue = 0
      def get_value(self):
          return self._somevalue
      def set_value(self, value):
          self._somevalue = value
      def del_attr(self):
          del self._somevalue
  	x = property(get_value, set_value, del_attr, "I am the ''x' property.")
  ```

* 第二种形式如下：

  ```python
  class Some_Class(object):
      _x = None
      def __init__(self):
          self._x = None
      @property
      def x(self):
          return self._x
      @x.setter
      def x(self, value):
          self._x = value
      @x.deleter
      def x(self):
          del self._x
  ```

property 的优势可以简单地概括为以下几点：

* 代码更简洁，可读性更强。

* 更好的管理属性的访问。property 将对属性的访问直接转换为对对应的 get、set 等相关函数的调用，属性能够更好地被控制和管理，常见的应用场景如设置校验、检查赋值的范围以及对某个属性进行二次计算之后再返回给用户或者计算某个依赖于其他属性的属性。
  创建一个 property 实际上就是将其属性的访问与特定的函数关联起来，相对于标准属性的访问，property 的作用相当于一个分发器，对某个属性的访问并不直接操作具体的对象，而对标准属性的访问没有中间这一层，直接访问存储属性的对象。

* 代码可维护性更好。property 对属性进行再包装，以类似于接口的形式呈现给用户，以统一的语法来访问属性，当具体实现需要改变的时候，访问的方式仍然可以保持一致

* 控制属性访问权限，提高数据安全性。如果用户想设置某个属性为只读：

  ```python
  class PropertyTest(object):
      def __init__(self):
          self.__var1 = 20
      @property
      def x(self):
          return self.__var1
  ```

  值得注意的是，使用 property 并不能真正完全达到属性只读的目的，正如以双下划线命令的变量并不是真正的私有变量一样，这些方法只是在直接修改属性这条道路上增加了一些障碍。

property 本质并不是函数，而是特殊类，既然是类的话，那么就可以被继承，因此用户便可以根据自己的需要定义 property：

```python
def update_meta(self, other):
    self.__name__ = other.__name__
    self.__doc__ = other.__doc__
    self.__dict__.update(other.__dict__)
    return self

class UserProperty(property):
    def __new__(cls, fget=None, fset=None, fdel=None, doc=None):
        if fget is not None:
            def __get__(obj, objtype=None, name=fget.__name__):
                fegt = getattr(obj, name)
                return fget()
        	fget = update_meta(__get__, fget)
        
        if fset is not None:
            
```

