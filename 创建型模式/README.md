# 单例

> https://www.yrunz.com/archives/%E4%BD%BF%E7%94%A8Go%E5%AE%9E%E7%8E%B0GoF%E7%9A%8423%E7%A7%8D%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%EF%BC%88%E4%B8%80%EF%BC%89/

## 简述

![](https://picb.zhimg.com/80/v2-d60bddfa23f2cd603be9fa3a296d22f2_1440w.jpg)

`单例模式算是23中设计模式里最简单的一个了，它主要用于保证一个类仅有一个实例，并提供一个访问它的全局访问点。`

在程序设计中，有一些对象通常我们只需要一个共享的实例，比如线程池、全局缓存、对象池等，这种场景下就适合使用单例模式。

但是，并非所有全局唯一的场景都适合使用单例模式。比如，考虑需要统计一个API调用的情况，有两个指标，成功调用次数和失败调用次数。这两个指标都是全局唯一的，所以有人可能会将其建模成两个单例SuccessApiMetric和FailApiMetric。按照这个思路，随着指标数量的增多，你会发现代码里类的定义会越来越多，也越来越臃肿。这也是单例模式最常见的误用场景，更好的方法是将两个指标设计成一个对象ApiMetric下的两个实例ApiMetic success和ApiMetic fail。

如何判断一个对象是否应该被建模成单例？

通常，被建模成单例的对象都有“中心点”的含义，比如线程池就是管理所有线程的中心。所以，在判断一个对象是否适合单例模式时，先思考下，这个对象是一个中心点吗？

Go实现
在对某个对象实现单例模式时，有两个点必须要注意：（1）限制调用者直接实例化该对象；（2）为该对象的单例提供一个全局唯一的访问方法。

对于C++/Java而言，只需把类的构造函数设计成私有的，并提供一个static方法去访问该类点唯一实例即可。但对于Go语言来说，即没有构造函数的概念，也没有static方法，所以需要另寻出路。

我们可以利用Go语言package的访问规则来实现，将单例结构体设计成首字母小写，就能限定其访问范围只在当前package下，模拟了C++/Java中的私有构造函数；再在当前package下实现一个首字母大写的访问函数，就相当于static方法的作用了。

在实际开发中，我们经常会遇到需要频繁创建和销毁的对象。频繁的创建和销毁一则消耗CPU，二则内存的利用率也不高，通常我们都会使用对象池技术来进行优化。考虑我们需要实现一个消息对象池，因为是全局的中心点，管理所有的Message实例，所以将其实现成单例，实现代码如下：


# 建造者模式

## 简述

![](https://pic3.zhimg.com/80/v2-0bceb00df2829f0f94e10dcd4d0acc6c_1440w.jpg)

`在程序设计中，我们会经常遇到一些复杂的对象，其中有很多成员属性，甚至嵌套着多个复杂的对象。这种情况下，创建这个复杂对象就会变得很繁琐。对于C++/Java而言，最常见的表现就是构造函数有着长长的参数列表：`

MyObject obj = new MyObject(param1, param2, param3, param4, param5, param6, ...)
而对于Go语言来说，最常见的表现就是多层的嵌套实例化：

obj := &MyObject{
  Field1: &Field1 {
    Param1: &Param1 {
      Val: 0,
    },
    Param2: &Param2 {
      Val: 1,
    },
    ...
  },
  Field2: &Field2 {
    Param3: &Param3 {
      Val: 2,
    },
    ...
  },
  ...
}
上述的对象创建方法有两个明显的缺点：（1）对对象使用者不友好，使用者在创建对象时需要知道的细节太多；（2）代码可读性很差。

针对这种对象成员较多，创建对象逻辑较为繁琐的场景，就适合使用建造者模式来进行优化。

建造者模式的作用有如下几个：

1、封装复杂对象的创建过程，使对象使用者不感知复杂的创建逻辑。

2、可以一步步按照顺序对成员进行赋值，或者创建嵌套对象，并最终完成目标对象的创建。

3、对多个对象复用同样的对象创建逻辑。

其中，第1和第2点比较常用，下面对建造者模式的实现也主要是针对这两点进行示例。

# 工厂方法模式

## 简述

![](https://pic1.zhimg.com/80/v2-847d44791ce4640c584bf411a473ed5a_1440w.jpg)

工厂方法模式跟上一节讨论的建造者模式类似，都是将对象创建的逻辑封装起来，为使用者提供一个简单易用的对象创建接口。两者在应用场景上稍有区别，建造者模式更常用于需要传递多个参数来进行实例化的场景。

使用工厂方法来创建对象主要有两个好处：

1、代码可读性更好。相比于使用C++/Java中的构造函数，或者Go中的{}来创建对象，工厂方法因为可以通过函数名来表达代码含义，从而具备更好的可读性。比如，使用工厂方法productA := CreateProductA()创建一个ProductA对象，比直接使用productA := ProductA{}的可读性要好。

2、与使用者代码解耦。很多情况下，对象的创建往往是一个容易变化的点，通过工厂方法来封装对象的创建过程，可以在创建逻辑变更时，避免霰弹式修改。

工厂方法模式也有两种实现方式：（1）提供一个工厂对象，通过调用工厂对象的工厂方法来创建产品对象；（2）将工厂方法集成到产品对象中（C++/Java中对象的static方法，Go中同一package下的函数）


# 抽象工厂模式（Abstract Factory Pattern）

## 简述

![](https://pic3.zhimg.com/80/v2-4783c1df7c6d9953376e2c863f90d38d_1440w.jpg)

在工厂方法模式中，我们通过一个工厂对象来创建一个产品族，具体创建哪个产品，则通过swtich-case的方式去判断。这也意味着该产品组上，每新增一类产品对象，都必须修改原来工厂对象的代码；而且随着产品的不断增多，工厂对象的职责也越来越重，违反了单一职责原则。

抽象工厂模式通过给工厂类新增一个抽象层解决了该问题，如上图所示，FactoryA和FactoryB都实现·抽象工厂接口，分别用于创建ProductA和ProductB。如果后续新增了ProductC，只需新增一个FactoryC即可，无需修改原有的代码；因为每个工厂只负责创建一个产品，因此也遵循了单一职责原则。

## Go实现

![](https://pic3.zhimg.com/80/v2-e7e42e6229876fb79b1bfc3168a69c96_1440w.jpg)

考虑需要如下一个插件架构风格的消息处理系统，pipeline是消息处理的管道，其中包含了input、filter和output三个插件。我们需要实现根据配置来创建pipeline ，加载插件过程的实现非常适合使用工厂模式，其中input、filter和output三类插件的创建使用抽象工厂模式，而pipeline的创建则使用工厂方法模式。


各类插件和pipeline的接口定义如下：

# 原型模式（Prototype Pattern）

## 简述

![](https://picb.zhimg.com/80/v2-fd7c37e987284366cd37889e79d02336_1440w.jpg)

原型模式主要解决对象复制的问题，它的核心就是clone()方法，返回Prototype对象的复制品。在程序设计过程中，往往会遇到有一些场景需要大量相同的对象，如果不使用原型模式，那么我们可能会这样进行对象的创建：新创建一个相同对象的实例，然后遍历原始对象的所有成员变量， 并将成员变量值复制到新对象中。这种方法的缺点很明显，那就是使用者必须知道对象的实现细节，导致代码之间的耦合。另外，对象很有可能存在除了对象本身以外不可见的变量，这种情况下该方法就行不通了。

对于这种情况，更好的方法就是使用原型模式，将复制逻辑委托给对象本身，这样，上述两个问题也都迎刃而解了。

# 总结

本文主要介绍了GoF的23种设计模式中的5种创建型模式，创建型模式的目的都是提供一个简单的接口，让对象的创建过程与使用者解耦。其中，单例模式主要用于保证一个类仅有一个实例，并提供一个访问它的全局访问点；建造者模式主要解决需要创建对象时需要传入多个参数，或者对初始化顺序有要求的场景；工厂方法模式通过提供一个工厂对象或者工厂方法，为使用者隐藏了对象创建的细节；抽象工厂模式是对工厂方法模式的优化，通过为工厂对象新增一个抽象层，让工厂对象遵循单一职责原则，也避免了霰弹式修改；原型模式则让对象复制更加简单。

创建型模式是处理对象创建的一类设计模式，主要思想是向对象的使用者隐藏对象创建的具体细节，从而达到解耦的目的。

下一篇文章，将介绍23种设计模式中的7种结构型模式（Structural Pattern），及其Go语言的实现。