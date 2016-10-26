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

从表面上看它确实很像构造方法：当需要实例化一个对象的时候，使用 `a=Class(args...)` 便可以返回一个类的实例，其中 args 的参数与 `__init__()` 方法中申明的参数一样。

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

Guido 认为，基于 Python 目前的一些特性（如类中动态添加方法，在类风格的装饰器中没有 self 无法确认是返回一个静态方法还是类方法等）保留其原有设计是个更好的选择，更何况 Python 的哲学是：显示优于隐式（Explicit is better than implicit）。

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

当访问一个不存在的实例属性的时候就会抛出 `AttributeError` 异常。这个异常时由内部方法 `__getattribute__(self, name)` 抛出的，因为 `__getattribute__()` 会被无条件调用，也就是说只要涉及实例属性的访问就会调用该方法，它要么返回实际的值，要么抛出异常。Python 的[文档](http://docs.python.org/2/reference/datamodel.html#object.getattribute)中也提到了这一点。

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
            def __set__(obj, value, name=fset.__name__):
                fset = getattr(obj, name)
				return fset(value)
        	fset = update_meta(__set__, fset)
        
        if fdel is not None:
            def __delete__(obj, name=fdel.__name__):
                fdel = getattr(obj, name)
                return fdel()
            fdel = update_meta(__delete__, fdel)
        return property(fget, fset, fdel, doc)
    
class C(object):
    def get(self):
        return self._x
    def set(self, x):
        self._x = x
    def delete(self):
        del self._x
    x = UserProperty(get, set, delete)
```

上例中 UserProperty 继承自 property，其构造函数 `__new__(cls, fget=None, fset=None, fdel=None, doc=None)` 中重新定义了 `fget()`、`fset()` 以及 `fdel()` 方法以满足用户特定的需要，最后返回的对象实际还是 property 的实例，因此用户能够像使用 property 一样使用 UserProperty。

使用 property 并不能真正完全达到属性只读的目的，用户仍然可以绕过阻碍来修改变量。真正实现只读属性的可行实现：

```python
def ro_property(obj, name, value):
    setattr(obj.__class__, name, property(lambda obj: obj.__dict__["__" + name]))
    setattr(obj, "__" + name, value)
    
class ROClass(object):
    def __init__(self, name, available):
        ro_property(self, "name", name)
        self.available = available
        
a = ROClass("read only", True)
print(a.name)
a._Article__name = "modify"
print(a.__dict__)
print(ROClass.__dict__)
print(a.name)
```

### 建议 62：掌握 metaclass

什么是元类？

* 元类是关于类的类，是类的模版
* 元类是用来控制如何创建类的，正如类是创建对象的模版一样
* 元类的实例为类，正如类的实例为对象

当使用关键字 class 的时候，Python 解释器在执行的时候就会创建一个对象（这里的对象是指类而非类的实例）

既然类是对象，那么它就有其所属的类型，也一定还有什么东西能够控制它的生成。通过 type 查看会发现 UserClass 的类型为 type，而其对象 UserClass() 的类型为类 A。

同时我们知道 type 还可以这样使用：

`type(类名，父类的元组（针对继承的情况，可以为空)，包含属性的字典（名称和值））`

type 通过接受类的描述符作为参数返回一个对象，这个对象可以被继承，属性能够被访问，它实际是一个类，其创建由 type 控制，由 type 所创建的对象的 `__class__` 属性为 type。type 实际上是 Python 的一个内建元类，用来指导类的生成。当然，除了使用内建元类 type，用户也可以通过继承 type 来自定义元类。

利用元类实现强制类型检查：

```python
class TypeSetter(object):
    def __init__(self, fieldtype):
        self.fieldtype = fieldtype
    def is_valid(self, value):
        return isinstance(value, self.fieldtype)
class TypeCheckMeta(type):
    def __new__(cls, name, bases, dict):
        return super(TypeCheckMeta, cls).__new__(cls, name, bases, dict)
    def __init__(cls, name, bases, dict):
        cls._fields = {}
        for key, value in dict.items():
            if isinstance(value, TypeSetter):
                cls._fields[key] = value
  def sayHi(cls):
      print("Hi")
```

TypeSetter 用来设置属性的类型，TypeCheckMeta 为用户自定义的元类，覆盖了 type 元类中的 `__new__()` 方法和 `__init__()` 方法，虽然也可以直接使用 `TypeCheckMeta(name, bases, dict)` 这种方式来创建类，但更为常见的是在需要被生成的类中设置 `__metaclass__` 属性，两种用法是等价的：

```python
class TypeCheck(object):
    __metaclass__ = TypeCheckMeta
    def __setattr__(self, key, value):
        if key in self._fields:
            if not self._fields[key].is_valid(value):
                raise TypeError("Invalid type for field")
            super(TypeCheck, self).__setattr__(key, value)
            
class MetaTest(TypeCheck):
    name = TypeSetter(str)
    num = TypeSetter(int)
    
mt = MetaTest()
mt.name = "apple"
mt.num = "test"
```

当类中设置了 `__metaclass__` 属性的时候，所有继承自该类的子类都将使用所设置的元类来指导类的生成。

实际上，在新式类中当一个类未设置 `__metaclass__` 属性的时候，它将使用默认的 `type` 元类来生成类。而当该属性被设置时查找规则如下：

* 如果存在 `dict["__metaclass__"]`，则使用对应的值来构建类；否则使用其父类 `dict["__metaclass__"]` 中所指定的元类来构建类，当父类中也不存在指定的 `metaclass` 的情形下使用默认元类 type。
* 对于古典类，条件 1不满足的情况下，如果存在全局变量 `__metaclass__`，则使用该变量所对应的元类来构建类；否则使用 `type.ClassType`。

需要额外提醒的是，元类中所定义的方法为其所创建的类的类方法，并不属于该类的对象。比如上例中的 `mt.sayHi()` 会抛出异常，正确调用方法为：`MetaTest.sayHi()`。

什么情况下会用到元类？有句话是这么说的：当你面临一个问题还在纠结要不要使用元类的时候，往往会有其他的更为简单的解决方案。

几个使用元类的场景：

* 利用元类来实现单例模式：

  ```python
  class Singleton(type):
      def __init__(cls, name, bases, dic):
          super(Singleton, cls).__init__(name, bases, dic)
          cls.instance = None
      def __call__(cls, *args, **kwargs):
          if cls.instance is None:
              cls.instance = super(Singleton, cls).__call__(*args, **kwargs)
          else:
              print("warning: only allowed to create one instance, minstance already exists!")
          return cls.instance

  class MySingleton(object):
      __metaclass__ = Singleton
  ```

* 第二个例子来源于 Python 的标准库 string.Template.string，它提供简单的字符串替换功能。`Template("$name $age").substitute({"name":"admin"}, age=26)`

  该标准库的源代码中就用到了元类，`Template` 的元类为 `_TemplateMetaclass`。`_TemplateMetaclass` 的 `__init__()` 方法通过查找属性（pattern、delimiter 和 idpattern）并将其构建为一个编译好的正则表达式存放在 pattern 属性中。用户如果需要自定义分隔符（delimiter）可以通过继承 Template 并覆盖它的类属性 delimiter 来实现。

  另外在 Django ORM、AOP 编程中也有大量使用元类的情形。


谈谈关于元类需要注意的几点：

* 区别类方法与元方法（定义在元类中的方法）。元方法可以从元类或者类中调用，而不能从类的实例中调用；但类方法可以从类中调用，也可以从类的实例中调用
* 多继承需要严格限制，否则会产生冲突。因为 Python 解释器并不知道多继承的类是否兼容，因此会发出冲突警告。解决冲突的办法是重新定义一个派生的元类，并在要集成的类中将其 `__metaclass__` 属性设置为该派生类。

### 建议 63：熟悉 Python 对象协议

 因为 Python 是一门动态语言，Duck Typing 的概念遍布其中，所以其中的 Concept 并不以类型的约束为载体，而另外使用称为协议的概念。在 Python 中就是我需要调用你某个方法，你正好就有这个方法。比如在字符串格式化中，如果有占位符 %s，那么按照字符串转换的协议，Python 会自动地调用相应对象的 `__str__()` 方法。

除了 `__str__()` 外，还有其他的方法，比如 `__repr__()`、`__init__()`、`__long__()`、`__float__()`、`__nonzero__()` 等，统称类型转换协议。除了类型转换协议之外，还要许多其他协议。

* 用以比较大小的协议，这个协议依赖于 `__cmp__()`，与 C 语言库函数 cmp 类似，当两者相等时，返回 0，当 `self < other` 时返回负值，反之返回正值。因为这种复杂性，所以 Python 又有 `__eq__()`、`__ne__()`、`__lt__()`、`__gt__()` 等方法来实现相等、不等、小于和大于的判定。这也就是 Python 对 `==`、`!=`、`<` 和 `>` 等操作符的进行重载的支撑机制。
* 数值类型相关的协议，这一类的函数比较多。基本上，只要实现了那么几个方法，基本上就能够模拟数值类型了。不过还需要提到一个 Python 中特有的概念：反运算。类似 `__radd__()` 的方法，所有的数值运算符和位运算符都是支持的，规则也是一律在前面加上前缀 r 即可。


* 容器类型协议。容器的协议是非常浅显的，既然为容器，那么必然要有协议查询内含多少对象，在 Python 中，就是要支持内置函数 `len()`，通过 `__len__()` 来完成。而 `__getitem__()`、`__setitem__()`、`__delitem__()` 则对应读、写和删除，也很好理解。`__iter__()` 实现了迭代器协议，而 `__reversed__()` 则提供对内置函数 `reversed()` 的支持。容器类型中最有特色的是对成员关系的判断符 in 和 not in 的支持，这个方法叫 `__contains__()`，只要支持这个函数就能够使用 in 和 not in 运算符了。

* 可调用对象协议。所谓可调用对象，即类似函数对象，能够让类实例表现得像函数一样，这样就可以让每一个函数调用都有所不同。

  ```python
  class Functor(object):
      def __init__(self, context):
          self._context = context
      def __call__(self):
          print("do something with {}".format(self._context))
  lai_functor = Functor("lai")
  yong_functor = Functor("yong")
  lai_functor()
  yong_functor()
  ```

* 与可调用对象差不多的，还有一个可哈希对象，它是通过 `__hash__()` 方法来支持 `hash()` 这个内置函数的，这在创建自己的类型时非常有用，因为只有支持可哈希协议的类型才能作为 dict 的键类型（不过只要继承自 object 的新式类就默认支持了）

* 描述符协议和属性交互协议（`__getattr__()`、`__setattr__()`、`__delattr__()`），还有上下文管理器协议，也就是对 with 语句的支持，这个协议通过 `__enter__()` 和 `__exit__()` 两个方法来实现对资源的清理，确保资源无论在什么情况下都会正常清理。

协议不像 C++、Java 等语言中的接口，它更像是声明，没有语言上的约束力。

### 建议 64：使用操作符重载实现中缀语法

模拟 C++ 的流输出，是一种对特性的滥用，不应提倡。

管道的处理非常清晰，因为它是中缀语法，而我们常用的 Python 是前缀语法，比如类似的 Python 代码应该是 `sort(ls(), reverse=True)`。

管道符号在 Python 中，也是或符号，由 Julien Palard 开发了一个 pipe 库，这个 pipe 库的核心代码只有几行，就是重载了 `__ror__()` 方法：

```python
class Pipe:
    def __init__(self, function):
        self.function = function
    def __ror__(self, other):
        return self.function(other)
    def __call__(self, *args, **kwargs):
        return Pipe(lambda x: self.function(x, *args, **kwargs))
```

这个 Pipe 类可以当成函数的 decorator 来使用：

```python
@Pipe
def where(iterable, predicate):
    return (x for x in iterable if (predicate(x)))
```

`pipe` 库内置了一堆这样的处理函数，比如 `sum`、`select`、`where` 等函数尽在其中：

```python
fib() | take_while(lambda x: x < 1000000) \
      | where(lambda x: x % 2) \
      | select(lambda x: x * x) \
      | sum()
```

这段代码就是找出小于 1000000 的斐波那契数，并计算其中的偶数的平方之和。

此外，pipe 是惰性求值的，所以我们完全可以弄一个无穷生成器而不用担心内存被用完。

除了处理数值很方便，用它来处理文本也一样简单。比如读取文件，统计文件中每个单词出现的次数，然后按照次数从高到低对单词排序：

```python
from __future__ import print_function
from re import split
from pipe import *
with open("test_descriptor.py") as f:
    print(f.read()
          | Pipe(lambda x: split("/W+", x))
          | Pipe(lambda x:(i for i in x if i.strip()))
          | groupby(lambda x:x)
          | select(lambda x:(x[0], (x[1] | count)))
          | sort(key=lambda x: x[1], reverse=True)
         )
```

### 建议 65：熟悉 Python 的迭代器协议

Python 的迭代器集成在语言之中，不像 C++ 中那样需要专门去理解这一个概念。

但是，并非所有的时候都能够隐藏细节。首先介绍一下 `iter()` 函数，`iter()` 可以输入两个实参，第二个可选参数可以忽略。`iter()` 函数返回一个迭代器对象，接受的参数是一个实现了 `__iter__()` 方法的容器或迭代器（精确来说，还支持仅有 `__getitem__()` 方法的容器）。对于容器而言，`__iter__()` 方法返回一个迭代器对象，而对迭代器而言，它的 `__iter__()` 方法返回其自身。

迭代器协议，所谓协议，是一种松散的约定，并没有相应的接口定义，所以把协议简单归纳如下：

* 实现 `__iter__()` 方法，返回一个迭代器
* 实现 `next()` 方法，返回当前的元素，并指向下一个元素的位置，如果当前位置已无元素，则抛出 `StopIteration` 异常。

其实 for 语句就是对获取容器的迭代器、调用迭代器的 `next()` 方法以及对 `StopIteration` 进行处理等流程进行封装的语法糖（类似的语法糖还有 `in/not in` 语句）。

迭代器最大的好处是定义了统一的访问容器（或集合）的统一接口，所以程序员可以随时定义自己的迭代器，只要实现了迭代器协议即可。除此之外，迭代器还有惰性求值的特性，它仅可以在迭代至当前元素时才计算（或读取）该元素的值，在此之前可以不存在，在此之后也可以销毁，也就是说不需要在遍历之前实现准备好整个迭代过程中的所有元素，所以非常适合遍历无穷个元素的集合或或巨大的事物（斐波那契数列、文件）。

迭代器在一些应用场景更省 CPU 计算资源，所以在编写代码中应当多多使用迭代器协议，避免劣化代码。从 Python2.3 版本开始，`itertools` 成为了标准库的一员已经充分印证这个观点。

`itertools` 的目标是提供一系列计算快速、内存高效的函数，这些函数可以单独使用，也可以进行组合，这个模块受到了 Haskell 等函数式编程语言的启发，所以大量使用 `itertools` 模块中的函数的代码，看起来有点像函数式编程语言。比如 `sum(imap(operator.mul, vector1, vector2))` 能够用来运行两个向量的对应元素乘积之和。

`itertools` 最为人所熟知的版本，应该算是 zip、map、filter、slice 的替代，`izip（izip_longest）`、`imap（startmap）`、`ifilter（ifilterfalse）`、`islice`，它们与原来的那几个内置函数有一样的功能，只是返回的是迭代器（在 Python3 中，新的函数彻底替换掉了旧函数）

除了对标准函数的替代，`itertools` 还提供了以下几个有用的函数：`chain()` 用以同时连续地迭代多个序列；`compress()`、`dropwhile()` 和 `takewhile()` 能用遴选序列元素；`tee()` 就像同名的 UNIX 应用程序，对序列作 n 次迭代；而 `groupby` 的效果类似 SQL 中相同拼写的关键字所带的效果。

```python
[k for k, g in groupby("AAAABBBCCDAABB")] --> A B C D A B
[list(g) for k, g in groupby("AAAABBBCCD")] --> AAAA BBB CC D
```

除了这些针对有限元素的迭代帮助函数之外，还有 `count()`、`cycle()`、`repeat()` 等函数产生无穷序列，这 3 个函数就分别可以产生算术递增数列、无限重复实参的序列和重复产生同一个值的序列。

还有几个组合函数：

* `product()`：计算 m 个序列的 n 次笛卡尔积
* `permutations()`：产生全排列
* `combinations()`：产生无重复元素的组合
* `combinations_with_replacement()`：产生有重复元素的组合

```python
>>> list(product("ABCD", repeat=2))
[('A', 'A'), ('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'A'), ('B', 'B'), ('B', 'C'), ('B', 'D'), ('C', 'A'), ('C', 'B'), ('C', 'C'), ('C', 'D'), ('D', 'A'), ('D', 'B'), ('D', 'C'), ('D', 'D')]
>>> list(permutations("ABCD", 2))
[('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'A'), ('B', 'C'), ('B', 'D'), ('C', 'A'), ('C', 'B'), ('C', 'D'), ('D', 'A'), ('D', 'B'), ('D', 'C')]
>>> list(combinations("ABCD", 2))
[('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'C'), ('B', 'D'), ('C', 'D')]
>>> list(combinations_with_replacement("ABCD", 2))
[('A', 'A'), ('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'B'), ('B', 'C'), ('B', 'D'), ('C', 'C'), ('C', 'D'), ('D', 'D')]
>>> for i in product("ABC", "123", repeat=2):
	print("".join(i))

# product() 可以接受多个序列
A1A1
A1A2
A1A3
A1B1
....
```

### 建议 66：熟悉 Python 的生成器

生成器，就是按一定的算法生成一个序列。迭代器虽然在某些场景表现得像生成器，但它绝非生成器；反而是生成器实现了迭代器协议的，可以在一定程度上看作迭代器。

如果一个函数，使用了 yield 语句，那么它就是一个生成器函数。当调用生成器函数时，它返回一个迭代器，不过这个迭代器是以生成器对象的形式出现的，这个对象带有 `__iter__()` 和 `next()` 方法。

每一个生成器函数调用之后，它的函数并不执行，而是到第一次调用 `next()` 的时候才开始执行。

当第一次调用 `next()` 方法时，生成器函数开始执行，执行到 yield 表达式为止。

直率地说，`send()` 方法很绕，其实 `send()` 是全功能版本的 `next()`，或者说 `next()` 是 `send()` 的快捷方式，相当于 `send(None)`。yield 表达式有一个返回值，`send()` 方法的作用就是控制这个返回值，使得 `yield` 表达式的返回值是它的实参。

除了能 yield 表达式的“返回值”之外，也可以让它抛出异常，这就是 `throw()` 方法的能力。对于常规业务逻辑的代码来说，对特定的异常有很好的处理（比如将异常信息写入日志后优雅的返回），从而实现从外部影响生成器内部的控制流。

当调用 `close()` 方法时，yield 表达式就抛出 `GeneratorExit` 异常，生成器对象会自行处理这个异常。当调用 `close()` 方法，再次调用 `next()`、`send()` 会使生成器对象抛出 `StopIteration` 异常。换言之，这个生成器对象已经不再可用。当生成器对象被 GC 回收时，会自动调用 `close()`。

生成器还有两个很棒的用处，其中之一就是实现 with 语句的上下文管理协议，利用的是调用生成器函数时函数体并不执行，当第一次调用 `next()` 方法时才开始执行，并执行到 yield 表达式后中止，直到下一次调用 `next()` 方法这个特性；其二是实现协程，利用的是 `send()`、`throw()`、`close()` 等特性。

上下文管理器协议，其实就是要求类实现 `__enter__()` 和 `__exit__()` 方法，但是生成器对象并没有这两个方法，所以 `contextlib` 提供了 `contextmanager` 函数来适配这两种协议：

```python
from contextlib import contextmanager
@contextmanager
def tag(name):
    print("<{}>".format(name))
    yield
    print("</{}>".format(name))
>>> with tag("h1"):
    print("foo")
<h1>
foo
</h1>
```

通过 `contextmanager` 对 `next()`、`throw()`、`close()` 的封装，yield 大大简化了上下文管理器的编程复杂度，对提高代码可维护性有着极大的意义。除此之外，`yield` 和 `contextmanager` 也可以用以“池”模式中对资源的管理和回收。

### 建议 67：基于生成器的协程及 greenlet

协程，又称微线程和纤程等，据说源于 Simula 和 Modula-2 语言，现代编程语言基本上都支持这个特性，比如 Lua 和 ruby 都有类似的概念。协程往往实现在语言的运行时库或虚拟机中，操作系统对其存在一无所知，所以又被称为用户空间线程或绿色线程。又因为大部分协程的实现是协作式而非抢占式的，需要用户自己去调度，所以通常无法利用多核，但用来执行协作式多任务非常合适。用协程来做的东西，用线程或进程通常也是一样可以做的，但往往多了许多加锁和通信的操作。

基于生产着消费者模型，比较抢占式多线程编程实现和协程编程实现。线程实现至少有两点硬伤：

* 对队列的操作需要有显式/隐式（使用线程安全的队列）的加锁操作。
* 消费者线程还要通过 sleep 把 CPU 资源适时地“谦让”给生产者线程使用，其中的适时只能静态地使用经验值。

而使用协程可以比较好地解决：

```python
# 队列容器
q = new queue
# 生产者协程
loop
    while q is not full
    	create some new items
        add the items to q
    yield to consume
# 消费者协程
loop
	while q is not empty
    	remove some items from q
        use the items
	yield to produce
```

但是这样做，损失了利用多核 CPU 的能力。

具体的生成器函数代码：

```python
def consumer():
    while True:
        line = yield
        print(line.upper())
def producter():
    with open("/var/log/apache2/error_log", "r") as f:
        for i, line in enumerate(f):
            yield line
            print("processed line {}".format(i))
c = consumer()
c.next()
for line in producter():
    c.send(line)
```

协程，每输出一行大写的文字后都有一行来自主程序的处理信息，不会像抢占式的多线程程序那样“乱序”。Python2.X 版本的生成器无法实现所有的协程特性，是因为缺乏对协程之间复杂关系的支持。比如一个 yield 协程依赖另一个 yield 协程，且需要由最外层往最内层进行传值的时候，就没有解决办法。

这个问题直到 Python3.3 增加了 `yield from` 表达式以后才得以解决，通过 `yield from`，外层的生成器在接收到 `send()` 或 `throw()` 调用时，能够把实参直接传入内层生成器。

因为 Python2.x 版本对协程的支持有限，而协程又是非常有用的特性，所以很多 Pythonista 就开始寻求语言之外的解决方案，并编写了一系列的程序库，其中最受欢迎的是 greenlet。

greenlet 是一个 C 语言编写的程序库，它与 yield 关键字没有密切的关系。greenlet 这个库里最为关键的一个类型就是 PyGreenlet 对象，它是一个 C 结构体，每一个 PyGreenlet 都可以看到一个调用栈，从它的入口函数开始，所有的代码都在这个调用栈上运行。它能够随时记录代码运行现场，并随时中止，以及恢复。它跟 yield 所能够做到的相似，但更好的是它提供从一个 PyGreenlet 切换到另一个 PyGreenlet 的机制。

协程虽然不能充分利用多核，但它跟异步 I/O 结合起来以后编写 I/O 密集型应用非常容易，能够在同步的代码表面下实现异步的执行，其中的代表当属将 greenlet 与 libevent/libev 结合起来的 gevent 程序库，它是 Python 网络编程库。最后，以 gevent 并发查询 DNS 的例子为例，使用它进行并发查询 n 个域名，能够获得几乎 n 倍的性能提升：

```python
import gevent
from gevent import socket
urls = ["www.google.com", "www.example.com", "www.python.org"]
jobs = [gevent.spawn(socket.gethostbyname, url) for url in urls]
gevent.joinall(jobs, timeout=2)
print([job.value for job in jobs])
```

### 建议 68：理解 GIL 的局限性

 多线程 Python 程序运行的速度比只有一个线程的时候还要慢，除了程序本身的并行性之外，很大程度上与 GIL 有关。由于 GIL 的存在，多线程编程在 Python 中并不理想。GIL 被称为全局解释器锁（Global Interpreter Lock），是 Python 虚拟机上用作互斥线程的一种机制，它的作用是保证任何情况下虚拟机中只会有一个线程被运行，而其他线程都处于等待 GIL 锁被释放的状态。不管是在单核系统还是多核系统中，始终只有一个获得了 GIL 锁的线程在运行，每次遇到  I/O 操作便会进行 GIL 锁的释放。

但如果是纯计算的程序，没有 I/O 操作，解释器则会根据 sys.setcheckinterval 的设置来自动进行线程间的切换，默认情况下每隔 100 个时钟（这里的时钟指的是 Python 的内部时钟，对应于解释器执行的指令）就会释放 GIL 锁从而轮换到其他线程的执行。

在单核 CPU 中，GIL 对多线程的执行并没有太大影响，因为单核上的多线程本质上就是顺序执行的。但对于多核 CPU，多线程并不能真正发挥优势带来效率上明显的提升，甚至在频繁 I/O 操作的情况下由于存在需要多次释放和申请 GIL 的情形，效率反而会下降。

鉴于 Python 中对象的管理与引用计数器，在 Python 解释器中引入了 GIL，以保证对虚拟机内部共享资源访问的互斥性。GIL 的引入确实使得多线程不能再多核系统中发挥优势，但它也带来了一些好处：大大简化了 Python 线程中共享资源的管理，在单核 CPU 上，由于其本质是顺序执行的，一般情况下多线程能够获得较好的性能。此外，对于扩展的 C 程序的外部调用，即使其不是线程安全的，但由于 GIL 的存在，线程会阻塞直到外部调用函数返回，线程安全不再是一个问题。

针对 Python1.5，Greg Stein 发布了一个补丁，该补丁中 GIL 被完全移除，使用高粒度的锁来代替，然而多核多线程速度的提升并没有随着核数的增加而线性增长，反而给单线程程序的执行速度带来了一定的代价，速度大约降低了 40%。在 Python3.2 中重新实现了 GIL，其实现机制主要集中在两个方面：一方面是使用固定的时间而不是固定数量的操作指令来进行线程的强制切换；另一个方面是在线程释放 GIL 后，开始等待，直到某个其他线程获取 GIL 后，再开始尝试去获取 GIL，这样虽然可以避免此前获得 GIL 的线程，不会立即再次获取 GIL，但仍然无法保证优先级高的线程优先获取 GIL。这种方式只能解决部分问题，并未改变 GIL 的本质。

Python 提供了其他方式可以绕过 GIL 的局限，比如使用多进程 multiprocess 模块或者采用 C 语言扩展的方式，以及通过 ctypes 和 C 动态库来充分利用物理内核的计算能力。

### 建议 69：对象的管理与垃圾回收

通常来说 Python 并不需要用户自己来管理内存，它与 Perl、Ruby 等很多动态语言一样具备垃圾回收功能，可以自动管理内存的分配与回收。

Python 中内存管理的方式：Python 使用引用计数器（Reference counting）的方法来管理内存中的对象，即针对每一个对象维护一个引用计数值来表示该对象当前有多少个引用。当其他对象引用该对象时，其引用计数会增加 1，而删除一个队当前对象的引用，其引用计数会减 1。只有当引用计数的值为 0 时的时候该对象才会被垃圾收集器回收，因为它表示这个对象不再被其他对象引用，是个不可达对象。引用计数算法最明显的缺点是无法解决循环引用的问题，即两个对象相互引用。

循环引用常常会在列表、元组、字典、实例以及函数使用时出现。对于由循环引用而导致的内存泄漏的情况，可以使用 Python 自带的一个 gc 模块，它可以用来跟踪对象的“入引用（incoming reference）“和”出引用（outgoing reference）”，并找出复杂数据结构之间的循环引用，同时回收内存垃圾。有两种方式可以触发垃圾回收：一种是通过显式地调用 `gc.collect()` 进行垃圾回收；还有一种是在创建新的对象为其分配内存的时候，检查 threshold 阈值，当对象的数量超过 threshold 的时候便自动进行垃圾回收。默认情况下阈值设为（700，10，10），并且 gc 的自动回收功能是开启的，这些可以通过 `gc.isenabled()` 查看。

```python
import gc
print(gc.isenabled())
print(gc.get_threshold())
```

一个解决循环引用内存回收的示例：

```python
def main():
    collected = gc.collect()
    print("Garbage collector before running: collected {} objects.".format(collected))
    print("Creating reference cycles...")
    A = Leak()
    B = Leak()
    A.b = B
    B.a = A
    A = None
    B = None
    collected = gc.collect()
	print(gc.garbage)
    print("Garbage collector after running: collected {} objects".format(collected))
    
if __name__ == "__main__":
    ret = main()
    sys.exit(ret)
```

`gc.garbage` 返回的是由于循环引用而产生的不可达的垃圾对象的列表，输出为空表示内存中此时不存在垃圾对象。`gc.collect()` 显示所有收集和销毁的对象的数目，此处为 4（2 个对象 A、B，以及其实例属性 dict）。

如果在类 Leak 中添加析构方法 `__del__()`，会发现 `gc.garbage` 的输出不再为空，而是对象 A、B 的内存地址，也就是说这两个对象在内存中仍然以“垃圾”的形式存在。

实际上当存在循环引用并且当这个环中存在多个析构方法时，垃圾回收器不能确定对象析构的顺序，所以为了安全起见仍然保持这些对象不被销毁。而当环被打破时，gc 在回收对象的时候便会再次自动调用 `__del__()` 方法。

gc 模块同时支持 DEBUG 模式，当设置 DEBUG 模式之后，对于循环引用造成的内存泄漏，gc 并不释放内存，而是输出更为详细的诊断信息为发现内存泄漏提供便利，从而方便程序员进行修复。更多 gc 模块可以参考[文档](http://docs.python.org/2/library/gc.html)