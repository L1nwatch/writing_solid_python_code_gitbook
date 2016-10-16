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

* 局部作用域：