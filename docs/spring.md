# 1.Spring IOC

## 1.1 什么是Spring IOC

IOC英文全称为Inversion of Control，中文意思是控制反转，同样也作为Denpendency Injection(DI,依赖注入)闻名。

Ioc具体意思如下：

即将创建对象的控制权交给Spring,应用只负责声明。对象的创建则是使用`反射`技术实现的。

举个例子：

原方式
```java
public class Student{

   // 声明
    private Book book;

    public Student(){
        // 创建
        book = new Book();
    }

    public void readBook(){
        book.showText();
    }

}

public class Book{

    public void showText(){
        System.out.println("text);
    }
}
```

IOC后

```java
@Component
public class Student{

    // 声明
    @Autowire
    private Book book;

    public void readBook(){
        book.showText();
    }

}

@Component
public class Book{

    public void showText(){
        System.out.println("text);
    }
}
```

很明显，Student依赖的Book对象我们不再负责创建，创建(依赖注入)工作移交给Spring负责。
这个过程就叫做控制反转。

负责存储Spring创建对象的容器叫做Ioc容器(`BeanFactory`和`ApplicationContext`)，创建的对象在Spring中被声明为`BeanDefinition`.

## 1.2 Ioc容器
在Spring中，核心的IOC容器有以下两种，分别是`BeanFactory`和`ApplicationContext`。
IOC容器负责`bean`的初始化、配置和装配

### 1.2.1 BeanFactory

`BeanFactory`作为存储`BeanDefinition`的基础容器，主要存储了如下对象的信息。
!> todo，补充描述信息


### 1.2.2 ApplicationContext

`ApplicationContext`是`BeanFactory`的子接口，并原基础上增加了如下特性：

- 更方便集成AOP特性
- 国际化处理
- 事件发布机制
- 创建了`WebApplicationContext`用于web应用

`ApplicationContext`有很多实现类，负责根据不同的方式装载并初始化`Bean`

- `ClassPathXmlApplicationContext` 负责从CLASSPATH路径下的xml文件装载bean
- `FileSystemXmlApplicationContext` 负责从文件系统xml文件中装载bean
- `AnnotationConfigApplicationContext` 负责根据Java注解装载bean。上诉例子就是通过Java注解的方式声明
- ...

#### XML方式
以下是声明bean的xml文件结构，其中id用于唯一标识bean，class指定bean的类型。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 引入其他xml文件定义的bean -->
    <import resource="services.xml"/>

    <bean id="student" class="Student">  
        <!-- 声明student依赖的bean book -->
        <property name="book" ref="book"/>
    </bean>

    <bean id="book" class="Book">  
    </bean>


    <!-- 定义其他bean -->

</beans>
```

#### Java Annotation方式

如`Student`例子所示。


### 1.2.3 使用IOC容器

```java
// 初始化IOC容器
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
// 获取bean
Student student = context.getBean("student", Student.class);
// 使用bean
student.readBook();
```


## 1.3 BeanDefinition

以上bean是`BeanDefinition`的简称。在Spring IOC容器中，bean以`BeanDefinition`的方式存储.
BeanDefinition包含如下bean的信息：

- 类的全限定名
- bean的配置: scope、bean生命周期的各种回调等。
- bean的依赖：如Student依赖于Book
- ...

### 如何定义为Spring的Bean

有如下方式：

- 基于XML配置

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
```

- 基于Java Annotation

如在类上面使用注解`@Component`、`@Configuration`、`@Service`等。

此外还可以在声明为Spring Bean的类方法上使用`@Bean`，表示该方法的返回值会作为Spring Bean，bean的名称则是方法名。



### 1.3.1 Bean的初始化

Bean的初始化指的是创建bean对象的过程。初始化bean有如下几种方式：

- 使用构造器
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>
```
按照如上声明的bean，Spring IOC容器在创建bean对象时是通过反射调用bean的构造方法创建的对象。

- 使用静态工厂方法
```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

- 使用实例工厂方法

```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
</bean>
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

## 1.4 依赖注入

依赖注入英文全称为`Dependency Injection`，指的是将类属性所需要的bean通过反射的方式创建并复制给类属性的过程中。

如下Student类的book成员的初始化的过程就是依赖注入的过程中.
```java
@Component
public class Student{

    // 声明
    @Autowire
    private Book book;

    public void readBook(){
        book.showText();
    }

}

@Component
public class Book{

    public void showText(){
        System.out.println("text);
    }
}
```

### 1.4.1 依赖注入的方式

依赖注入的方式有很多种，分别是基于构造器注入、基于setter方式注入。

#### 基于构造器注入

```java
public class SimpleMovieLister {

    
    private final MovieFinder movieFinder;

    
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
  
    // ...
}
```
!> todo 如何确定使用那个构造器？

#### 基于Setter注入

```java
public class SimpleMovieLister {


    private MovieFinder movieFinder;


    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
**如何选择注入方式**
在Spring官网文档说道，必须的依赖建议通过构造器注入，可选的通过setter注入。

### 依赖注入的解析过程


#### 循环依赖

如果使用构造器注入的方式，可能会存在循环依赖的场景。

什么是循环依赖？

A依赖B，B依赖A。或A依赖B、B依赖C、C依赖A。等等。
在这种情况下，A的构造需要等待B构造，B构造又需要等待A构造，从而导致A、B无法完成初始化。
例子: 
```java
public class A{

    private B b;

    public A(B b){
        this.b = b;
    }
}
public class B{

    private A a;

    public B(A a){
        this.a = a;
    }
}

```

如何解决循环依赖?

答案是避免全部使用构造器注入或编码时注意循环依赖的产生。

Spring是如何解决循环依赖的？

### Depends On

`depends-on`用于说明该bean依赖于另一个bean。

什么意思呢？

如下例子就是说`beanOne`初始化之前需要完成`manager` bean初始化。

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

### Bean 懒加载

默认情况下，`ApplicationContext`所管理的bean是singlton scope。
在初始化的时候已经完成了singlton scope的bean的初始化。

懒加载的意思是在ApplicationContext启动的时候不会立即初始化。

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
```

不过在懒加载的bean作为非懒加载的bean的以来时，懒加载的bean也会立即被初始化。



### Autowiring

!> todo 自动装配，byName、byType


### Bean的作用域

- singleton 默认的作用域，在IOC容器中只存在一份bean
- prototype 存在多份bean
- request 一次http请求就会创建一次
- session 一次http session就会创建一次
- application
- websocket



### Bean生命周期内的回调

英文名叫`Lifecycle Callbacks`. 指的是Bean从创建到销毁期间的各种回调事件。

如Spring提供`InitializingBean` 和 `DisposableBean`两个接口。

我们在声明为Spring Bean的时候实现如上两个接口，那么在该Bean在创建完成后会调用`afterPropertiesSet()`，
在Bean被销毁前调用`destroy()`。

不过Spring更推荐注解的方式声明回调方法。

上述两个接口等同于 `@PostConstruct` 和 `@PreDestroy`标注的方法。

Spring是如何找到并调用我们在Bean中声明的回调方法的。

答案是Spring的内部组件`BeanPostProcessor`. 如果Spring提供的生命周期回掉函数不满足应用场景，我们可以实现`BeanPostProcessor`从而自定义生命周期回调.



此外，Spring还提供了`Lifecycle`接口,用以监听Bean的启动和关闭事件。

```JAVA
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```


### ApplicationContextAware

ApplicationContextAware

如果应用的Bean实现了ApplicationContextAware，那么Spring IOC容器在创建的时候会将ApplicationContext引用传递给该Bean。

```Java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}

### 其他Aware接口

- BeanNameAware
- BeanFactoryAware
- ...
```





### 基于注解的IOC依赖注入

基于注解的IOC依赖注入优先于XML，所以注解注入的有可能被XML的替换掉。

AutowiredAnnotationBeanPostProcessor负责处理基于注解的Bean生命周期处理。

#### @Autowired

@Autowired注解用来告诉Spring当前bean依赖于标注体上的Bean.

该注解可以标注在构造器、setter、属性上。

需要注意的是，被注入的bean是根据类型注入的，什么意思呢？

举个例子

Student依赖Book。Book有两种实现Book1和Book2.

此时启动会报错。

```java
@Component
public class Student{

    @Autowired
    private Book book;


}

@Component
public class Book1 extends Book{

}
@Component
public class Book2 extends Book{

}
```

如果我们指定其中一个bean名称为`book`，那么就能正常运行了。
```java
@Component("book")
public class Book2 extends Book{

}
```


另一种方案，使用@Primary注解标注在多个可用bean的情况下优先选择该bean
```java
@Primary
@Component
public class Book2 extends Book{

}
```

也可以使用@Qulifier注解注入指定名称的bean,此时注入到Student的是Book1
```java
@Component
public class Student{

    @Autowired
    @Qulifier("main")
    private Book book;


}

@Component("main")
public class Book1 extends Book{

}

@Primary
@Component
public class Book2 extends Book{

}
```


总结：@Autowired标注的依赖是先根据类型匹配的，如有多个再根据@Qulifier匹配，
匹配不到再根据@Primary匹配，最后根据类名首字母小写后的bean name与@Autowired标注的属性名或方法名匹配。


当然，Spring提供以下方式找到类型相同的bean
```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

#### @Resource

@Resource注解是JSR 250提供的注解，相比@Autowired的 根据类型查找。
@Resource是根据name查找的，当@Resource的name属性为空，则表示名称为@Resource标注的字段名。

```java
@Component
public class Student{

    @Resource(name = "myBook")
    private Book book;


}
```

该注解的处理是组件`CommonAnnotationBeanPostProcessor`负责的。

#### @Value

@Value注解是Spring用来加载外部属性的。

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}

@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

application.properties
```
catalog.name=MovieCatalog
```

如果需要配置外部属性

可以注入自定义Bean
```java
@Configuration
public class AppConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

> Spring Boot 默认注入了默认的`PropertySourcesPlaceholderConfigurer`，用来从application.properties或者application.yml中获取属性。

Spring内置了转换器用来将外部文件的属性转换为@Value标注的属性。

如果需要自定义转换器，可以参考如下

```java

```

#### @PostConstruct 和 @PreDestroy

用来标注Bean生命周期的相关周期的回调方法。

### 组件扫描

IOC容器启动时要知道加载哪些Java类作为Spring的Bean，这个查找过程就叫做组件扫描。

前面说过，将Class定义为Spring Bean有两种方式，基于xml和基于Java注解。

其中基于xml的前面已经描述过，此处不再赘述了。

下面讲讲基于Java注解的方式。

#### @Component及其相同注解

被@Component注解标注的类会被作为Spring Bean放入Spring IOC容器中管理。

```java
@Component("main")
public class Book1 extends Book{

}
```

此外，Spring为了区分用途，还做了语义化的注解，作用和@Component注解相同。

分别是@Service @Repository @Controller 

### 自动检测

@ComponentScan注解用来告诉Spring扫描该package及其子package下的类，对于标注为Spring的类会被放在IOC容器中。

@ComponentScan需要标注在@Configuration注解的类上

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

@ComponentScan可以设置过滤器用来过滤扫描类
```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```

### IOC容器配置

#### @Configuration 和 @Bean

除开@Component的方式，也可以在@Configuration的类中声明bean

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```


# Spring Resources

# 验证、数据绑定和类型转换

# Spring表达式(SPEL)

# Spring Aop

AOP是Aspect-oriented Programming的简称，中文意思是面向切面编程。

## 基本概念

- Aspect: 切面，被@Aspect注解标注的类。

- Join Point：切入点，即被切入的方法

- Advice：通知，用来在目标方法上做增强。
    - Before advice： 前置通知
    - After returning advice： 后置正常返回通知(目标方法不抛出异常)
    - After throwing advice：后置异常通知(目标方法抛出异常)
    - After finally advice: 不管是否抛出异常，方法执行完就会执行该通知。
    - Around advice: 环绕通知，能够在目标方法执行前后执行的通知。

- Pointcut：切点，用来匹配哪些方法(切入点)要被切入

- Target object：目标对象，即被切面增强的对象。

- AOP Proxy：AOP代理对象，代理对象包含目标对象，用来增强目标对象，从而在目标对象运行前后执行相应的通知。

- Weaving：织入，在编译期间将目标对象和切面链接起来，生成AOP代理对象的过程。


## AOP Proxies

AOP针对接口是同的JDK的动态代理；针对类使用CGLIB。
如果目标类实现了多个接口，Spring AOP使用CGLIB。

如果使用JDK动态代理，只有声明在接口的public方法会被拦截

如果使用CGLIB，public和protected和default的方法会被拦截。
需要注意final方法无法被AOP，原因在于无法override。

如何开启@Aspect支持,使用@EnableAspectJAutoProxy注解
```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

## AOP例子

@Aspect注解表明该类是个切面
切面实现了`Ordered`接口，表明在多个切面切入同一个方法的时候，该切面的执行顺序
@Around注解表明切面有个环绕通知
Pointcut是`com.xyz.myapp.CommonPointcuts.businessService()`，表明要被代理的方法
Join point则是com.xyz.myapp.CommonPointcuts类对象的businessService()方法
doConcurrentOperation(ProceedingJoinPoint pjp)方法是增强逻辑。增加逻辑是用来多次尝试获取锁，并将未达到最大尝试次数的异常拦截下来，避免中途抛出异常给其他调用者。


```java

@Aspect
public class ConcurrentOperationExecutor implements Ordered {

    private static final int DEFAULT_MAX_RETRIES = 2;

    private int maxRetries = DEFAULT_MAX_RETRIES;
    private int order = 1;

    public void setMaxRetries(int maxRetries) {
        this.maxRetries = maxRetries;
    }

    public int getOrder() {
        return this.order;
    }

    public void setOrder(int order) {
        this.order = order;
    }

    @Around("com.xyz.myapp.CommonPointcuts.businessService()")
    public Object doConcurrentOperation(ProceedingJoinPoint pjp) throws Throwable {
        int numAttempts = 0;
        PessimisticLockingFailureException lockFailureException;
        do {
            numAttempts++;
            try {
                return pjp.proceed();
            }
            catch(PessimisticLockingFailureException ex) {
                lockFailureException = ex;
            }
        } while(numAttempts <= this.maxRetries);
        throw lockFailureException;
    }
}
```

## AOP 原理

Calling code叫做客户端，他需要调用被代理(pojo)的foo()方法。

它首先调用的不是pojo.foo()，而是调用代理对象(proxy)的foo()方法。 

代理对象(proxy)此时就可以根据前置后置还是环绕等顺序调用我们的通知方法，最后调用被代理对象的foo()方法，并将结果返回给客户端。


![](https://docs.spring.io/spring-framework/docs/current/reference/html/images/aop-proxy-call.png)

# 空安全

