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

