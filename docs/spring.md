# 1.Spring IOC

## 1.1 什么是Spring IOC

IOC英文全称为Inversion of Control，中文意思是控制反转，同样也作为Denpendency Injection(DI,依赖注入)闻名。

Ioc具体意思如下：

即将创建对象的控制权交给Spring,应用只负责声明。举个例子：

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



# Spring Resources

# 验证、数据绑定和类型转换

# Spring表达式(SPEL)

# Spring Aop

# 空安全

