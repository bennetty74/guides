# Spring Boot

相比Spring来说，Spring Boot主要增加了如下核心特性：

## 1. SpringApplication

`SpringApplication`相较于Spring提供了更为方便的启动Spring项目的方式。

如下代码就是aSpringBoot的例子，主要有以下两点：

- @SpringBootApplication注解

- SpringApplication.run()

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

### 1.1 启动失败

Spring Boot提供了很多的`FailureAnalyzer`实现去处理启动失败问题。

如果仍然处理失败，我们仍然可以在控制台查看错误报告。

比如web应用常见的端口占用问题。

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that is listening on port 8080 or configure this application to listen on another port.
```

如果想查看更详细的日志，可以启用DEBUG级别的日志。

如启动时设置界别

```bash
java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

如配置文件配置日志级别

```properties
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

### 1.2 延迟初始化

延迟初始化指的是SpringBoot应用启动时是否同时将Spring Bean初始化完成。

延迟初始化可以加快应用的启动速度，但是也伴随着相应的问题：比如bean之间的依赖注入问题可能在运行时才发现，导致应用异常。

Spring Boot综合考虑，延迟初始化默认是关闭的，可以通过一下方法开启。

- application.properties

```properties
spring.main.lazy-initialization=true
```

- SpringApplication.setLazyInitialization()

- SpringApplicationBuilder.lazyInitialization()

如果只想取消部分Spring Bean延迟初始化，可以在对应的bean上加上注解

`@Lazy(false)` 

### 1.3 自定义Banner

Spring Boot提供了自定义banner的途径，

我们可以通过定义banner路径的方式自定义banner，默认为banner.txt

- application.properties

```properties
spring.banner.location
```

当然，也可以设置为图片的格式，使用如下属性标注路径即可.支持banner.jpg,banner.png,banner.gif

```properties
spring.banner.image.location
```

- SpringApplication.setBanner()设置banner，需要实现Banner接口并实现printBanner()方法

### 1.4 自定义SpringApplication

如果静态的`SpringApplication.run()`方法无法满足我们的需求，可以创建SpringApplication实例，动态设置相关属性。比如关闭`Banner` 

```java
import org.springframework.boot.Banner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {

        // 传入Spring Bean。@SpringBootApplication是众多注解的合集，
        //其中包含了@Configuration注解
        SpringApplication application = 
                    new SpringApplication(MyApplication.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }

}
```

或者使用Builder模式构建

```java
new SpringApplicationBuilder()
        .sources(MyApplication.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

### 1.5 应用的可用性

### 1.6 应用的事件和监听器

在Spring Boot应用的启动过程中，会有很多的事件发生，而监听器则负责监听对应的事件和处理。

可以通过`SpringApplication.addListeners()` 或 `SpringApplicationBuilder.listeners()`注册监听器

如果需要在应用没启动的时候自动注册监听器，可以添加文件`META-INF/spring.factories`，内容如下：

```
org.springframework.context.ApplicationListener
            = com.example.project.MyListener
```

以下是应用事件的分发顺序：

- ApplicationStartingEvent，应用刚启动的事件，除了注册监听器和初始化器之外，未做其他处理

- ApplicationEnvironmentPreparedEvent：上下文已知`Environment`但是未创建触发的事件

- ApplicationContextInitializedEvent

- ApplicationPreparedEvent
  
  - WebServerInitializedEvent
  
  - ContextRefreshedEvent

- ApplicationStartedEvent

- AvailabilityChangeEvent

- ApplicationReadyEvent

- AvailabilityChangeEvent

- ApplicationFailedEvent

### 1.7 Web 环境

SpringApplication在启动时会创建对应的ApplicationContext. 对应的算法如下：

- 如果使用了SpringMVC，则`AnnotationConfigServletWebServerApplicationContext`作为SpringBoot应用的上下文；

- 如果没使用Spring MVC，而是使用SpringWebFlux，则`AnnotationConfigReactiveWebServerApplicationContext`作为应用上下文

- 否则使用`AnnotationConfigApplicationContext`作为应用上下文

当然我们可以通过`setWebApplicationType(WebApplicationType)`方法指定Web应用类型。

或者完全设置上下文类型`setApplicationContextClass`

### 1.8 访问启动参数

如果需要应用的启动参数`SpringApplication.run(args)`，可以在自己定义的Bean中传入`ApplicationArguments`对象

```java
@Component
public class MyBean {

    public MyBean(ApplicationArguments args) {

    }

}
```

### 1.9 使用ApplicationRunner或CommandLineRunner

`ApplicationRunner` 或 `CommandLineRunner` 提供了接口在SpringBoot应用启动完成后的回调功能。

```java
@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Started end....");
    }

}
```

### 1.10 SpringBoot应用启动跟踪

`BufferingApplicationStartup`  就是用于追踪Spring应用的启动。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.metrics.buffering.BufferingApplicationStartup;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MyApplication.class);
        application.setApplicationStartup(new BufferingApplicationStartup(2048));
        application.run(args);
    }

}
```

## 额外配置

## Profile

profile用于应用配置的区分，可以通过配置文件的方式设置，也可以通过编程的方式设置。

设置生产环境的配置

```java
import org.springframework.context.annotation.Configuratio
import org.springframework.context.annotation.Profile;

@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // 


@Configuration(proxyBeanMethods = false)
@Profile("dev")
public class DevConfiguration {

    //...

}
```

然后通过配置文件选择使用的环境配置,如下使用dev、hsqldb的环境配置

```properties
spring.profiles.active=dev,hsqldb
```

## 日志

SpringBoot使用Commons Logging作为内部的日志记录接口，具体实现包括LOG4J、LOGBACK.

在使用starter的情况下，Logback是默认的日志输出实现。

### 日志格式

日志输出格式如下,包含时间、日志级别(INFO), 进程ID，线程名，Logger名和log内容。

```
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

### 日志输出方式

#### 控制台输出

可以在启动时控制SpringBoot应用的日志界别

```shell
$ java -jar myapp.jar --debug
```

其中`--debug`指定输出日志级别为`DEBUG`

也可以在`pplication.properties`件中设置`debug=true`

带颜色的日志输出

如果终端支持ANSI颜色支持，我们可以在配置日志格式的时候配置日志颜色

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

#### 文件输出

在`application.properties`文件中配置输出的文件名即可.

```properties
logging.file.name=my_log_file.txt
```

或者指定路径,默认为文件名未spring.log

```properties
logging.file.path = /path/to/log/
```

### 日志级别

日志级别从详细到简单分为：`TRACE` `DEBUG` `INFO` `WARN` `ERROR` `FATAL`

可以根据如下方式指定日志级别

`logging.level.<logger-name>=<level>`

比如如下配置`application.properties`

```properties
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

#### 日志组

比如如下配置`application.properties`

```properties
logging.group.tomcat=org.apache.catalina,org.apache.coyote,org.apache.tomcat

logging.level.tomcat=trace
```

### 自定义日志配置

## 国际化

## JSON

JSON是序列化的有效工具之一。主要用来将Java对象转为JSON字符串，或者反之。

Spring boot集成了如下三种JSON库

- Gson

- Jackson（默认使用）

- JSON-B

Jackson使用ObjectMapper作为Java对象序列化和反序列化的工具。

关于如何自定义序列化，可参考如下来链接

> todo 待补充

## 任务执行和调度

## 测试

## 自动配置
