## @Configuration

**Spring @Configuration 注解**有助于**基于 Spring 注解的配置**。**@Configuration**注解表示一个类声明了一个或多个`@Bean`方法，并且可以由[Spring 容器](https://howtodoinjava.com/spring-core/different-spring-ioc-containers/)处理以在运行时为这些 bean 生成 bean 定义和服务请求。

从 spring 2 开始，我们将 bean 配置写入 xml 文件。但是 Spring 3 提供了将 bean 定义移出 xml 文件的自由。我们可以在 Java 文件本身中给出 bean 定义。这称为**Spring Java Config**功能（使用[`@Configuration`](https://docs.spring.io/spring-framework/docs/3.1.x/javadoc-api/org/springframework/context/annotation/Configuration.html)注解）。

### 一、Spring@Configuration注解用法

`@Configuration`在任何类的顶部使用注解来声明该类提供一个或多个**@Bean**方法，并且可以由 Spring 容器处理以在运行时为这些 bean 生成 bean 定义和服务请求。

```java
应用程序配置文件
@Configuration
public class AppConfig {
 
    @Bean(name="demoService")
    public DemoClass service() 
    {
        
    }
}
```

#### 带有@Configuration注解的Spring配置类

```java
@Configuration
public class ApplicationConfiguration {
 
    @Bean(name="demoService")
    public DemoManager helloWorld() 
    {
        return new DemoManagerImpl();
    }
}
```

#### 演示

```java
public class VerifySpringCoreFeature
{
    public static void main(String[] args)
    {
        ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfiguration.class);
 
        DemoManager  obj = (DemoManager) context.getBean("demoService");
 
        System.out.println( obj.getServiceName() );
    }
}
```



