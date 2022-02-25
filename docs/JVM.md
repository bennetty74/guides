# 

> 为什么需要了解JVM
> 
> - 了解我们的Java代码是怎么运行的
> 
> - 了解JVM是怎样自动管理内存放入
> 
> - 能够理解并定位JVM抛出的相关错误源头并修复

# 我们的Java代码是怎么运行的？

了解过编译原理的我们都知道，源代码到机器指令要经过编译阶段。

比如`C`语言编译过程：

```java
源代码文件    ===>       GCC编译   ======>  可执行文件(机器指令)
main.c      ===>   gcc main.c -o main   ===>  main
```

同理，Java的编译执行也要经过一样的阶段。不同的是，java编译后不会直接生成机器指令，而是生成**字节码**。

```java
源代码文件   ===>    javac编译     ===>  .class文件(字节码)
main.java  ===>  javac main.java  ===>  main.class
```

`.class文件(字节码)`本身不能够被执行，需要通过`JVM`加载后，由`JVM`解释执行。

此处`JVM`做了两件事情：

- 加载class文件到JVM内部（类加载）

- 解释执行字节码文件（解释执行）

了解`.class`文件(以下均称为**Class文件**)是如何被`JVM`执行之前，我们先来看看`JVM`的整体架构，方便理解后面提到的名词。

![JVM架构](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-39.png)

- **Class文件**： java文件被`javac`编译后产生的字节码文件。

- **ClassLoader**：负责将**Class文件**加载到**JVM**

- **JVM 运行时数据区**
  
  - **Method Area(方法区)**： 存储ClassLoader加载的类的全限定名、字节流和一些其他的元数据信息。
  
  - **Heap Area(堆)**: 存储创建的各种对象。
  
  - **Stack Area(栈)**: 用于存储`当前线程`方法执行过程中的局部变量、堆区域的对象引用地址、方法调用链。
  
  - **Native Stack Area**: 同上，不过是记录的本地方法
  
  - **Program Counter(PC) Register(程序计数器)**: 用于记录`当前线程`的执行指令地址。每一个线程都有一个。

- **Execution Engine(执行引擎)**： 用于将**Class文件**翻译成机器指令并执行。

## 读取class文件（类加载）

读取Class文件的过程就是`JVM`将磁盘上的`Class`文件加载到内存并初始化的过程。该加载过程大致分为 `加载` `链接` 和`初始化`

    加载 -------->  链接  -------->  初始化
                 /   |   \
                验证 准备 解析

### 加载

加载就是将`Class文件`通过`ClassLoader`加载到内存中(JVM内部)，并生成`java.lang.Class`对象的过程。

加载期间主要执行了两步操作：

- 第一步操作是将.class文件的字节流加载到方法区(元空间)

- 第二步则是根据字节流生成`java.lang.Class`对象

在JDK的默认实现中，`Class文件`加载存在委托行为(逐级委托给上级加载.class文件，如果上级无法加载，则有自己加载)，即常说的`双亲委派模型`。

    Bootstrap ClassLoader
            |
    Extension ClassLoader
            |
    Application ClassLoader
            |
    Customize ClassLoader

- **Bootstrap ClassLoader**:  在`JVM`内部由C++实现，负责加载JVM核心类，在`$JAVA_HOME/jre/lib`下
- **Extension ClassLoader**:  负责加载`$JAVA_HOME/jre/lib/ext`目录下的类.Classloader具体是`sun.misc.Launcher$ExtClassLoader`
- **Application ClassLoader**:  负责加载`JVM`参数`-cp`或者`--classpath`命令指定路径下的类。**Classloader**具体是`sun.misc.Launcher$AppClassLoader`

**为什么设计双亲委派模型？**

一定程度上是基于安全考虑我们可以考虑如下的场景：

> 我们自己应用内部创建了与`java.lang.String`全限定名相同的String类

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

### 链接

#### 验证

1.JVM考虑到安全和规范问题，所以加载的第一步是读取`.class`文件内容并判断是否是JVM可是被的.class文件。
对于JVM无法识别的文件，JVM会抛出错误`java.lang.VerifyError`并停止后续进程。

这一验证过程由组件`ByteCodeVerifier`完成。

> todo 自己亲自试验一下

#### 准备

准备阶段主要是对类静态变量分配内存，包含两个步骤：

- 为类静态变量分配内存
- 为默认值分配内存并设置默认值

如

```java
public static int a = 1;
```

在准备阶段会首先会为a分配内存，其次将int型的默认值0分配给a。

#### 解析

解析阶段主要是将类中的符号引用转变为直接引用的过程。

**什么是符号引用？** 

比如`java.lang.String` 就是符号引用，是字面上的引用。而直接引用就是该类型信息在JVM内部的内存地址。

**为什么需要解析成直接引用？**

对于符号引用，我们只知道它的全限定名，但是对于该类型的方法、属性我们无从知晓，所以需要转变为直接引用通过在JVM内存地址中拿到类的信息。

**怎样将符号引用转为直接引用呢？** 

答案是通过搜索方法区完成的。加载阶段我们知道，类的全限定名和字节流数据都存储在方法区，所以可以通过全限定名就找到类的信息了。

### 初始化

在初始化阶段，JVM会为静态变量赋值、执行静态代码块。本质是执行`<clinit>`方法的过程，被称为类的初始化。
如

```java
public static int a = 1;
```

在初始化阶段会将1赋值给静态变量a。

初始化阶段是在类和接口的继承关系维度上从上到下执行。

## 解释执行(执行引擎)

该阶段主要将读入到内存的Class内容(字节码)翻译成机器指令执行，具体执行过程由执行引擎负责。

执行引擎包含三个组件：**Interpreter(解释器)**、**Just-In-Time Compiler(JIT，即时编译器)**、**Garbage Collector(垃圾收集器)**。

### Interpreter(解释器)

负责按行将字节码(.class内容)解析成机器指令并执行。缺点就是每次执行的时候都需要进行这样的解析过程，效率较低。

### JIT(即时编译器)

**JIT**全称**Just-In-Time Compiler**(即时编译器)，主要用于弥补解释器的执行效率不足。

具体表现在热点方法上：在解释器执行过后，会留存对应的机器指令。下次执行时不再重新解释执行，而是直接执行留存的机器指令。

此外，留存的机器指令也会执行一系列的优化，从而提高执行效率。如方法内联、移除空循环、移除重复计算等等。

### Garbage Collector 垃圾收集器

Java语言不用像C/C++一样自己手动管理内存的申请和释放，而是由JVM去自动管理内存。
这一角色就是执行引擎中的垃圾收集器负责。垃圾收集器用于查找并清除不再引用的对象，从而释放内存以便有足够的内存空间供新对象生成。

#### 如何释放引用？

既然由JVM自动回收不再引用的对象，那么我们在编码中应该怎样告诉JVM该对象不再引用呢？

- **将引用设置为null**
  这样student对象就会在JVM进行垃圾回收时清除掉所占用内存。
  
  ```java
  Student student = new Student();
  student = null;
  ```

- **将引用指向给另外一个对象**
  
  ```java
  Student studentOne = new Student();
  Student studentTwo = new Student();
  studentOne = studentTwo;
  ```
  
  以上代码创建了两个Student对象，在studentOne指向了studentTwo后，第一个被创建的student对象则会在JVM进行垃圾回收时进行回收。

- **使用匿名对象**
  
  ```java
  register(new Student());
  ```

> todo 描述引用和堆内对象的相互引用关系。在processon上面。

#### 垃圾收集两阶段

垃圾收集分为两个阶段：**标记阶段**和**清除阶段**。

##### 标记阶段

在该阶段，垃圾收集器负责标记哪些对象是无用(不再引用的对象)
标记阶段是在GC Roots的概念上工作的。
GC Roots是标记阶段工作的起点对象集，根据起点对象的引用链，逐层往下遍历。能够遍历到的对象标记为Live。
当从所有的GC Roots往下遍历完成后，便完成了标记阶段。堆区的剩余对象则被标记为无用对象以便在清除阶段清除并释放内存。

![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-76.png)

##### 清除阶段

在清除阶段，垃圾收集器会将标记阶段被标记为死亡对象的对象进行清理，并释放内存空间。如下白色空格就是移除死亡对象后释放的内存空间。 

当然，一些垃圾收集器为了合理利用内存空间，在清除死亡对象后还会整理内存空间，将已用空间和空余内存空间分割开，以便于顺序访问和大对象的创建。
![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-85.png)

#### 垃圾收集器中的分代思想

分代主要针对Heap Area(堆)，分代思想将堆区分为年轻代和老年代。

- Heap
  
  - Young Generation
    - EdenSpce
    - Survive Space
      - From Space
      - To Space
  - Old Generation

- Non-Heap

可以通过JVM参数`-Xms` `-Xmx`设置初始堆大小和最大堆大小。

具体分布可见下图：
![](https://www.freecodecamp.org/news/content/images/size/w1600/2021/01/image-70.png)

**垃圾收集和分代关系**

**Minor GC**

所有对象的创建在Eden区域。当进行了一次垃圾回收后，Eden区域的死亡对象被清理，留存下来的对象被移动到Survivor区域中的一块。
这样的一次GC叫做Minor GC。
需要注意Minor GC同样会扫描Survivor区域，Survivor中留存下来的移动到另一块区域中。

可以使用JVM参数`-Xmn`指定年轻代的大小。

**Major GC**
Major GC指的是老年代中的死亡对象被回收的过程。

在年轻代发生Minor GC多次后仍然存活在Survivor区域的对象会进入老年代。
具体多少次Minor GC后进入老年代，可以通过JVM参数设置。

**Permanent Generation**
永久代指的是方法区，用于存储class信息、方法信息。
可以通过`-XX:PermGen` `-XX:MaxPermGen`指定永久代的初始大小和最大大小。

> 从Java 8开始，MetaSpace替换了永久代的概念。相比永久代，元空间可以选自动调节大小，从而避免内存溢出的错误。

说完垃圾回收的策略和分代思想，接下来我们可以看看每种垃圾收集器具体又是怎样实现的呢？

垃圾收集器分为以下几种类型：

- **Serial GC**： 是垃圾收集器最简单的实现，用于单线程的小应用。当该垃圾收集器执行时，会暂停整个应用用于垃圾收集。可以使用JVM参数`-XX:+UseSerialGC`使用Serial GC.
  ![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-68.png)

- **Parallel GC**: 是JVM垃圾收集器的默认实现，使用多线程去完成垃圾收集。尽管如此，还是会存在应用的暂停。`-XX:+UseParallelGC`指定该垃圾收集器。
  ![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-66.png)

- **CMS GC**： CMS相比其他垃圾收集器会占用更多的CPU以提高应用性能。`-XX:+UseConcMarkSweepGC`
  
  > 不过在Java 9已经被标记为过期的垃圾收集器并在Java 14被移除。
  > ![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-67.png)

- **G1(Garbage First) GC**: 旨在替代CMS。该垃圾收集器是为多线程且堆大小很大(超过4GB)的应用而设计的。
  它将堆均等划分为大小相等的区域，并使用多线程去扫描所有区域，扫描完成后，总是优先清除无用对象最多的区域。
  使用JVM参数`-XX:+UseG1GC`指定该垃圾收集器。
  ![](https://www.freecodecamp.org/news/content/images/size/w1000/2021/01/image-88.png)

- **ZGC**: 随JDK 11发布，为低延迟的应用(少于10ms暂停)或大堆应用而设计的。

#### 如何选择垃圾收集器

了解了各种垃圾收集器的设计思路，如何合理选择垃圾收集器也显得尤为重要。

- Serial：为单处理器系统应用设计的。

- Parallel： 追求性能，能够忍受1s及以上的应用暂停时间

- CMS/G1：追求响应时间，能够仍受较低的吞吐量

- ZGC：追求响应时间，或者使用超大堆。

## 附录

#### Java Native Interface(JNI)

JNI是Java提供访问硬件能力的一种渠道。具体表现为由Java'定义接口规范，由C/++这样的编程语言去实现。
如：

```java
public native String getHardwareInfo();
```

```cpp
public String getHardwareInfo(){
    return "info";
}
```

最后将cpp代码编译成.dll或.so文件，使用System.loadLibrary()加载。
最后就可以在java代码中调用native方法了。

## 引用