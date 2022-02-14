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

### 读取class文件（类加载）

读取class文件的过程就是jvm将磁盘上的`.class`文件加载到内存（JVM内部）的过程。
该加载过程大致分为 `加载` `链接` `初始化`

    加载 -------->  链接  -------->  初始化
                 /   |   \
                验证 准备 解析

#### 加载

加载就是将.class文件通过ClassLoader加载到内存中(JVM内部)，并生成`java.lang.Class`对象的过程.

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


- Bootstrap ClassLoader: 在JVM内部由C++实现，负责加载JVM核心类
- Extension ClassLoader: 负责加载`JAVA_HOME/ext`目录下的类
- Application ClassLoader: 负责加载CLASSAPTH路径下的类

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
对于JVM无法识别的文件，JVM会抛出错误并停止后续进程。
> todo 自己亲自试验一下

##### 准备

准备阶段主要是对类静态变量进行初始化的值的过程中

### 解释执行

该阶段主要将读入到内存的class文件内容翻译成机器指令执行。
