## 《Effective Java》第三版

### 一、创建和销毁对象（Creating and Destroying Objects）

#### 1. 考虑使用静态工厂方法代替构造方法

#### 2. 遇到多个构造器参数是考虑使用构建起

#### 3.  用私有构造器或这枚举类强化Singleton

#### 4. 通过私有构造器强化不可实例化的能力

#### 5. 避免创建不必要的对象

#### 6. 消除过期的对象引用

#### 7. 避免使用finalize方法

### 二、对所有对象都通用的方法（Method common to All Objects）

#### 8. 覆盖equals方法时的规约

#### 9. 覆盖equals时总要覆盖hashCode方法

#### 10. 始终要覆盖toString方法

#### 11. 谨慎地覆盖clone

#### 12. 考虑使用Comparable 接口

### 三、类和接口（Classes and Interfaces）

#### 15. 使用类和成员的可访问性最小化

* Infomation hiding(信息隐藏)、encapsulation（封装），是软件设计的原则之一。

* 对于成员（fileds、methods、nested classes、nested interfaces）有四个访问级别，下面按照访问可见性递增的方式列出：

  * `private` 只有方法声明所在的顶级类可以访问。
  * `default( package-private)` 声明该成员所在的包内不的任何类可以访问，它也被称为“缺省的访问级别”
  * `proteced` 声明该成员的子类可以访问这个成员，声明该成员所在的包内不的任何类可以访问。
  * `public` 在任何地方都可以访问该成员。

* 私有成员和包级私有成员都是一个类实现中的一个部分，一般不会影响它的导出的API，但是如果这个类实现了 `Serializable` 接口，这些fileds 就有可能被泄露。

* 如果方法覆盖了超类中的一个方法，子类中的访问级别就不允许列于超类中的访问级别[JLS,8.4,8.3]。 、

* 共有类的成员变量应该尽可能的设置为 `private` ,一般而言，具有公有的成员变量的类不是线程安全的。同样的建议也是用与静态成员变量，

  ⚠️值得注意

  - 常量暴露一般通过声明为 `public static final` 的成员变量，这些变量一般用大写字母表示中间用下划线隔开。这些变量应该包含基本类型的值或者只想不可变对象的引用。如果它所指向的对象包含可变对象的引用，那么它便具有非 `final`域的所有缺点。即便变量本身是不能被修改的，但是它所引用的对象却是可以改变的，这可能会导致灾难性的后果。

  - 一个非零长度的数据一般是可变，因此会有潜在的风险。解决的方案是提供一个公有的访问方法，它返回私有方法的一个副本。

  ```java
  //Potential security hole
  public static final Ting[] VALUES={...}
  
  //solution
  public static Thing[] values(){
  	return VALUES.clone
  }
  ```

* 总之，你应该尽可能的有理由地降低可访问性。你仔细地设计了一个最小的公有API之后，应该防止把任何散乱的类、接口、或者成员变量变成API的一部分。除了公有静态final变量的特殊情况之外，公有类不应该包含public fileds 。并且要确保公有静态final变量所引用的对象是不可变的。

#### 16. 在公有类中使用访问方法，而非公有域

#### 17. 使可变形最小化

#### 18. 复合（composition）优于继承（inheritance）

#### 19. 要么为继承而设计并准备文档，要么禁止继承

#### 20. 接口优于抽象类

#### 21. 使用接口为后代定义方法

#### 22. 接口仅定义类型

#### 23. 类层次优于标签类

#### 24. 优先考虑静态成员类

#### 25. 禁止一个源文件中定义多个顶级类

### 四、泛型（Generic）

### 五、枚举和注解（Enums and Annotations）

### 六、Lambdas and Streams

### 七、方法（Methods）

#### 49. 检验参数的有效性

#### 50. 必要时使用保护性拷贝

#### 51. 谨慎地设计方法签名

#### 52. 慎用重载

#### 53. 慎用可变参数

#### 54. 返回零长度的数组或者集合，而非null

#### 55. 谨慎地返回Optionals

#### 56. 为所有导出的API

### 八、通用程序设计（General Programing）

#### 57. 将局部变量的作用域最小化

#### 58. For-each循环由于传统的for循环

* for-each 在简洁性和预防bug反面较传统的for循环有巨大优势。
* 有三种常用情况无法使用for-each循环
  * **过滤** -- 如果要遍历集合，并删除选中的元素，就需要使用显示的迭代器，以便可以调用它的remove 方法。
  * **转换** -- 如果需要遍历列表或者数组，并取代它部分或者全部的元素值，就需要列表列表迭代器或者数组索引，以便设定元素的值。
  * **平行迭代** -- 如果需要并行地遍历多个集合，就需要显示地控制迭代器或者索引变量，以便所有迭代器或者索引变量可以得到同步前移。

#### 59. 理解和使用类库

#### 60. 如果需要精确结果，请避免使用float和double

#### 61. 基本类型由于装箱基本类型

#### 62. 当其他类型更适合时就尽量避免使用string

#### 63. 注意字符串连接的性能问题

#### 64. 使用接口指向对象

#### 65. 接口优于反射

#### 66. 慎用native method 

#### 67. 谨慎进行优化

#### 68. 遵守普遍接受的命名规范

### 九、异常（Exceptions）

> 充分发挥异常的优势，可以提高程序的可读性、可靠性和可维护性。但是使用不当，会带来负面的影响，本章的给出一些知道异常的指导原则。

#### 69. 仅针对异常情况使用异常

#### 70. 对可恢复的情况使用受检异常对于编程错误使用运行时异常

>  java 提供了三种可抛出结构（throwable）：
>
>  * 受检的异常（checked exception）
>  * 运行时异常（run-time exception）
>  * 错误（error）

#### 71. 避免不必要的checked exception

#### 72. 优先使用标准异常

| Exception                         | Occasion for use                                             |
| --------------------------------- | ------------------------------------------------------------ |
| `IllegalArgumentException`        | Non-null parameter value is inappropriate                    |
| `IllegalStateException`           | Object state is inappropriate for method invocation          |
| `NullPointerException`            | Parameter value is null where prohibited                     |
| `IndexOutOfBoundsException`       | Index parameter value is out of range                        |
| `ConcurrentModificationException` | Concurrent modification of an object has been detected where it is prohibited |
| `UnsupportedOperationException`   | Object does not support method                               |

#### 73. 抛出与抽象对应的异常

#### 74. 对每个方法抛出的所有异常编写文档

#### 75. 在详细信息里面包含失败的信息

#### 76. 努力使失败保持原子性

#### 77. 不要忽视异常

### 十、并发（Concurrency）

#### 78. 同步获取共享可变数据

> ```
> 当多个线程共享可变数据的时候，每个读或者写数据的线程都必须执行同步。
> 如果没有同步，就无法保证一个线程所做的修改可以被另一个线程获知。未能同步共享变量数据会造成程序的
> 活性失败（liveness failure）和安全性失效（safety failure）。这样的失败是最难调试的。它们可能是
> 间歇性的，且与时间相关，程序的行文在不同的VM上可能根本不同。
> 如果只需要线程之间的交互通信，而不需要互斥，volatile 修饰符就是一种可以接受的同步形势，但要正确使用它还需要一些技巧。
> ```

#### 79. 避免过多的同步

> 过度的同步可能会导致效率低下、死锁，甚至不确定的行为。

#### 80. executor和 task 优于threads

#### 81. 并发工具优于wait 和 notify

#### 82. 文档话线程安全

#### 83. 慎用延迟初始化

#### 84. 不要依赖线程调度器

### 十一、序列化（Serialization）

#### 85. 谨慎地实现Serializable接口

#### 86. 考虑使用自定义的序列化形式

#### 87. 保护性地编写readObject方法

#### 88. 对于实例控制，枚举类型优于readResolve

#### 89. 考虑序列化代理代替序列化实现