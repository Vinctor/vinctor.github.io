title: AS3.0中的Java8新特性
date: 2017-10-26
tags:
  - android
  - android studio
  - java8
  - lambda
categories:
  - android
	

---

今天google官方放出了期待已久的AS3.0正式版，加入了很多新特性，支持kotlin，java8，编译速度更快等等，想必很多kotlin小迷弟，小迷妹们已经欢呼雀跃。然鹅，菜鸡如我，不会使用kotlin。只能研究一下作为android开发者在AS3.0中可以使用到的java8新特性。


官方对于AS中J8新特性的页面如下[Use Java 8 Language Features](https://developer.android.com/studio/write/java8-support.html#supported_features)（请自备红杏）

附上[gradle-4.1-all.zip](http://pan.baidu.com/s/1slqgIpn)地址

# 配置Java8

## 首先需要抛弃之前的第三方插件

Android Studio为使用某些Java 8语言功能和使用它们的第三方库提供内置支持。，默认工具链通过desugar在javac编译器的输出上执行字节码转换来实现新的语言特性。

![默认工具链](http://upload-images.jianshu.io/upload_images/1583231-1f0845d48bef169a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)


在3.0之前，相信很多小伙伴为了尝试java8的新特性，会使用第三方的插件来使用lambda，如：Jack, Retrolambda, 或 DexGuard，AS3.0将对其不再提供支持，如果as检测到你继续使用这些插件，as将继续使用第三方插件，而不使用默认的工具链。

#### 删除Jack支持

将`jackOptions`从`build.gradle`删除

      android {
          ...
          defaultConfig {
          ...
        // 删除此代码块
            jackOptions {
                enabled true
                  ...
            }
    }

#### 删除Retrolambda

请从项目级`build.gradle`文件中删除`Retrolambda`依赖关系

    buildscript {
      ...
       dependencies {
          // 删除此依赖
          classpath 'me.tatarka:gradle-retrolambda:<version_number>'
       }
    }

并删除每个moudle下的`build.gradle`文件中`Retrolambda`引入

    //删除
    apply plugin: 'me.tatarka.retrolambda'
    //删除(如果有的话)
    retrolambda {
        jvmArgs '-Xmx2048m'
    }

## 设置Java8

在`build.gradle`文件下`android`节点添加如下

    android {
      ...
      //将项目的源和目标兼容性值设置为Java 8
      compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
      }
    }

# AS中的Java8新特性

放一张官方的资料图：


![image.png](http://upload-images.jianshu.io/upload_images/1583231-da9752543a2ceef6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

可以看到新版AS中指支持部分Java8的新特性，且有一部分还有`minSdkVersion`限制。接下来，我们对其比较常用的进行讲解：

### Lambda

相信大家已经lambda已经非常熟悉，这里我们不讲语法，讲点别的需要注意的地方。

#### 与匿名内部类的区别
lambda大家看作是一种匿名内部类，但实际却与匿名内部类是有区别的，
* `lambda`编译完成之后，使用的是J7新增的指令`invokedynamic`来表示，具体的实现不体现在字节码文件中，由虚拟机进行翻译`lambda`的具体执行策略，以便于后续JDK版本优化此语法特性的策略，同时可以很大程度上减少字节码的文件的大小和数量，分配方式类似于`java.lang.invoke`中`MethodHandle`中的动态调用方法（具体请百度）。目前`lambda`的具体执行方式还是与匿名内部类差不多的策略，据说oracle在以后的版本更新中会优化分配测罗，选用`MethodHandle`。
* 而匿名内部类方式生成新类，新建一个标有`$xx.class`的文件。

#### 分类

>`lambda`表达式的类型是一种特殊的接口，这种接口只有一个抽象方法，成为函数接口。

在J8中，lambda表达式倾向于函数式编程，根据其输入与返回类型不同，分为几种类型的函数接口，下面是系统提供的一组函数式接口类型，并以Rxjava的操作符举例：


| 接口        | 参数           | 返回类型 |说明 |
| :-----------------: |:---------:|:-------:|:-------:|
| Predicate<T>        | T         | boolean|  判断型函数，filter
| Consumer<T>       | T         |    void     |消费型函数，subcribe
| Function<T,R>       | T         |   R        |转换型函数，map
| Supplier<T>           | None   |   T        |生产型函数，create
| UnaryOperator<T>| T         |   T        |Function的特殊情况
| BinaryOperator<T>| (T,T)   |   T        |（二元）多元运算，zip

我们可以同过自定义接口进行使用，如android的View.OnclickListener极为消费性函数。

>注意，在自定义函数接口时，站在用途的角度来考虑，为了区分普通接口与函数接口的区别，建议自定的函数接口都应该加上`@FunctionalInterface`注解。该直接会强制编译器检查一个接口是否符合函数接口的标准。如果该注释添加给一个枚举类型，类或者接口包含不止一个抽象方法，编译器就会报错。这对我们规范化代码有很大的帮助。

#### 重载解析

，在运行过程中，会根据，java中可以重载方法，造成多个方法会有相同的方法名，但签名却不一样。这在推断类型是会带来问题，因为系统可能会推断出多种方法类型。当lambda表达式作为参数时，当出现重载情况时，这时，

>java会挑出最具体的类型。

看一个例子：
`
static interface IntergerBifunciton extends BinaryOperator<Integer>{
    }
    public  void overloadedMethoe(BinaryOperator<Integer> l){
    }
    public  void overloadedMethoe(IntergerBifunciton l){
    }
`
接口`IntergerBifunciton`是`BinaryOperator`的子接口，在调用`overloadedMethoe((x,y)->x+y)`的时候，虚拟机会选用`最具体的类型`。

再看一个无法推断哪个更具体类型的例子：

`
    static interface Intergerfunciton{
        public boolean test(int x);
    }
    public  void overloadedMethoe(Predicate<Integer> l){
    }
    public  void overloadedMethoe(Intergerfunciton l){
    }
`
我们使用` overloadedMethoe((x)->true);`调用的时候，会出现错误：

![模糊调用](http://upload-images.jianshu.io/upload_images/1583231-693d5210a1ec844d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

这是因为，编译器无法推断出哪种类型更具体 更适合方法的参数，故报错。我们在日常开发中应该避免出现这种情况。

### Default and static interface methods（默认方法和静态接口方法）

* 默认方法
java8相比之前最大的改动无疑是对集合类的改动和添加`Stream`API（minSdkVersion为24可用），用于在集合类中更方便的进行函数是编程，大家可以把`Stream`API看成类似`Rxjava`的类库，用以方便的对数据进行变换。（同样的，`Stream`也有很多类似`Rxjava`操作符的方法，比如filter，map，toList，flatmap等等，使用方法大同小异）。

java为`Collection`接口添加了`stream`方法（见下图），但是问题出现了，这意味着java1-java7中，实现了`Collection`接口的全部类都需要实现`stream`方法，否则，那些类将无法在java8里面通过编译。java为了保持向下兼容性，java8引入了`默认方法`，用关键字`default`表示。形象化说法就是：`Collection`告诉他的子类，如果你没有实现`stream`方法，就用我的吧。

![stream方法](http://upload-images.jianshu.io/upload_images/1583231-d902382b44d577f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

我们可以看到，在j8中，很多地方都使用了默认方法，如`Iterable`接口中的`forEach`和`spliterator`。

子接口中或者实现类中实现覆盖默认方法，需要`Override`注解。

看一下默认方法在复杂的继承关系中的一种特殊情况：
![image.png](http://upload-images.jianshu.io/upload_images/1583231-4f4c03a3bb66988f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图，`OverrideChild`既实现了`Parent`接口，又继承了实现了`Parent`接口的类，那么调用时，应该调用哪个方法呢？此例子打印出来的是`from ParentImpl`,原因在于，`与接口中定义的默认方法相比，类中重写的方法更具体`。

调用优先级为：类重写方法>子接口重写方法>父接口方法

接口允许多继承，因此，有了默认方法，子类就可以继承多个接口中的默认实现。这在某种形式上实现了多重继承的功能。（注意：如果在继承的多个接口中，有多个签名相同的默认方法，编译器将报错）

* 接口的静态方法

在以往的jdk版本中，接口是不允许存在静态方法的，j8发布后，为了使`Stream.of`静态方法，加入了静态方法，使得同一接口的子类可以共享这一静态方法。

### try-with-resources

AS3.0在所有android api层面对`try-with-resources`进行支持。

 try-with-resources声明一个或多个资源的 try 语句。一个资源作为一个对象，必须在程序结束之后随之关闭。 try-with-resources语句确保在语句的最后每个资源都被关闭 。

>任何实现了 java.lang.AutoCloseable的对象, 包括所有实现了 java.io.Closeable 的对象, 都可以用作一个资源。

先来一段代码：

![image.png](http://upload-images.jianshu.io/upload_images/1583231-6552822edb23760f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

可以看到， `try-with-resources` 语句声明的资源是一个 BufferedReader。声明语句在紧跟在 try 关键字的括号里面。在j8之前，如果我要是有一个资源，在使用之后，需要显示的效用其`close`方法将其关闭，但j8新的语法糖出现后，我们只需要将其声明在try中，在资源使用完毕之后，将自动关闭，不需要手动调用。上面的语句相当于：

![image.png](http://upload-images.jianshu.io/upload_images/1583231-f5c495d2af3802a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

我们可以在try后声明多个资源，见下图：


![image.png](http://upload-images.jianshu.io/upload_images/1583231-9211c4b19b662ea1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们也可以在`try-with-resources` 语句后添加自己需要捕获的异常或添加`finally`语句块，如下图：

![image.png](http://upload-images.jianshu.io/upload_images/1583231-78bac7ca5a828728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意，在try-with-resources 语句中, 任意的 catch 或者 finally 块都是在声明的资源被关闭以后才运行。

如果try-with-resources语句中资源的Close方法和try代码块中都抛出了异常，Close 方法抛出的异常被抑制，try代码块中的异常会被抛出。
Java7之后，可以使用Throwable.getSuppressed方法获得被抑制的异常。
