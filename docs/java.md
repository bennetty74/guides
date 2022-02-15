# Java基础



# JVM

> 为什么需要了解JVM
> 
> - 了解我们的代码是怎么运行的？
> 
> - 了解JVM如何进行内存回收的？
> 
> - 定位线上问题

## 我们的Java代码是怎么运行的？

了解过编译原理的我们都知道，源代码到机器码要经过编译和执行阶段，比如C语言要经过如下阶段：

`源代码文件` ===> `GCC编译` ===> `可执行文件(机器指令)`

`main.c`  ===> `gcc main.c -o main` ===> `main` 

同理，Java的编译执行也要经过一样的阶段。不同的是，java编译后不会直接生成机器指令，而是生成字节码。

-------------------------------------------------

                                                                           |                        JVM内部

---------------------

源代码文件 ===> 编译 ===> 字节码文件       |    字节码文件 ===> 机器指令

----------------------------------------------------------------------------------------------------------------------------

`main.java`   ===> `javac main.java` ===> `main.class` ===> `java main`

-----------------------------

即javac将java文件编译成class文件，`java main`则告诉jvm将main.class文件作为输入读入jvm，由jvm将class文件翻译成机器指令并执行。

此处jvm做了两件事情：

- 读取class文件到JVM内部（类加载）

- 将class文件内容翻译成机器指令并执行（解释执行）

了解.class文件是如何被JVM执行之前，我们先来看看JVM的整体架构，方便理解后面提到的名词。

- .class文件： .java文件被javac编译后产生的字节码文件。
- ClassLoader：负责将.class文件加载到JVM

JVM 运行时数据区：

- Method Area(方法区)： 存储ClassLoader加载的类的全限定名、字节流和一些其他的元数据信息。
- Heap Area(堆): 存储创建的各种对象。
- Stack Area(栈): 用于存储`当前线程`方法执行过程中的局部变量、堆区域的对象引用地址、方法调用链。
- Native Stack Area: 同上，不过是存储的本地方法
- PC Register(程序计数器): 用于记录`当前线程`的执行指令地址。每一个线程都有一个。

Execution Engine

用于将.class文件翻译成机器指令并执行。


![JVM架构](https://media.geeksforgeeks.org/wp-content/uploads/jvm-3.jpg)

### 读取class文件（类加载）

读取class文件的过程就是jvm将磁盘上的`.class`文件加载到内存（JVM内部）的过程。
该加载过程大致分为 `加载` `链接` `初始化`

    加载 -------->  链接  -------->  初始化
                 /   |   \
                验证 准备 解析

#### 加载

加载就是将.class文件通过ClassLoader加载到内存中(JVM内部)，并生成`java.lang.Class`对象的过程.

期间主要执行了两步操作：第一步操作是将.class文件的字节流加载到方法区(元空间), 第二步则是将生成`java.lang.Class`.

.class加载的过程是通过`ClassLoader`进行的。在JDK的默认实现中，`.class`文件加载存在委托行为(逐级委托给上级加载.class文件，如果上级无法加载，则有自己加载)，即常说的`双亲委派模型`。

            Bootstrap ClassLoader
                     ^
                     |
            Extension ClassLoader
                     ^
                     |
            Application ClassLoader
                     ^
                     |
            Customize ClassLoader


- Bootstrap ClassLoader: 在JVM内部由C++实现，负责加载JVM核心类(JAVA_HOME/jre/lib)
- Extension ClassLoader: 负责加载`JAVA_HOME/jre/lib/ext`目录下的类,classloader具体是`sun.misc.Launcher$ExtClassLoader`
- Application ClassLoader: 负责加载CLASSAPTH(java.class.path)路径下的类,classloader具体是`sun.misc.Launcher$AppClassLoader`

至于JDK为什么这样子设计，一定程度上是基于安全考虑我们可以考虑如下的场景：

> 我们自己应用内部创建了`java.lang.String`类，这显然和JDK内部的String实现冲突。

在这样的场景下，由于双亲委派模型的出现，从而避免了从外部加载全限定名一样的同名类，保护JVM安全。


既然如此，`双亲委派模型就无法打破吗？`

显然不是的，我们可以在自定义的`ClassLoader`实现`loadClass()`方法的过程中打破这个委派过程。
即在实现loadClass方法的过程中不委托给上级ClassLoader即可。

那么，在什么情况下会需要打破双亲委派模型呢？答案就在`Tomcat`

**为什么要打破双亲委派模型？**

tomcat在部署项目时，我们会将项目的war包放在webapps目录下。
假设app1和app2有两个全限定名相同的类，

因为Tomcat可以加载多个web应用，所以如果不破坏双亲委派模型，就会导致app2在加载该类的时候加载了app1中的限定名相同的类。

以下是tomcat类加载机制的结构图

        Bootstrap
            |
         System
            |
         Common
        /     \
    Webapp1   Webapp2 ...

> 具体打破双亲委派模型的地方在于`WebappX ClassLoader`,该ClassLoader首先在webappX目录下查找，没找到才会委托给上级ClassLoader.

#### 链接

##### 验证
1.JVM考虑到安全和规范问题，所以加载的第一步是读取`.class`文件内容并判断是否是JVM可是被的.class文件。
对于JVM无法识别的文件，JVM会抛出错误`java.lang.VerifyError`并停止后续进程。

这一验证过程由组件`ByteCodeVerifier`完成。

> todo 自己亲自试验一下

##### 准备

准备阶段主要是对类变量分配内存，包含两个步骤：

- 为类变量分配内存
- 为默认值分配内存

##### 解析

解析阶段主要是将类中的符号引用转变为直接引用的过程。
什么是符号引用？ 比如`java.lang.String` 就是符号引用，是字面上的引用。而直接引用就是该类型信息在JVM内部的内存地址。
为什么需要解析成直接引用？
对于符号引用，我们只知道它的全限定名，但是对于该类型的方法属性我们无从知晓，所以需要转变为直接引用通过在JVM内存地址中拿到类的信息。

那又是通过怎样的方式将符号引用转为直接引用的呢？
答案是通过搜索方法区完成的。加载阶段我们知道，类的全限定名和字节流数据都存储在方法区，所以可以通过全限定名就找到类的信息了。

#### 初始化

在初始化阶段，JVM会为静态变量赋值、执行静态代码块。
初始化阶段是在类和接口的继承关系维度上从上到下执行。

### 解释执行(执行引擎)

该阶段主要将读入到内存的class内容(字节码)翻译成机器指令执行。

包含三个组件：
- Interpreter(解释器)：负责按行将字节码(.class内容)解析成机器指令并执行。缺点就是每次执行的时候都需要进行这样的解析过程，效率较低。
- ust-In-Time Compiler(JIT，即时编译器)：主要用于弥补解释器的执行效率不足。具体表现在热点方法上：在解释器执行过后，会留存对应的机器指令。下次执行时不再重新解释执行，而是直接执行留存的机器指令。此外，留存的机器指令也会执行一系列的优化，从而提高执行效率。如方法内联、移除空循环、移除重复计算等等。
- GC Collector(垃圾收集器): 负责移除不再引用的对象，释放占用的内存。垃圾回收主要发生在Heap Area（堆区）。

