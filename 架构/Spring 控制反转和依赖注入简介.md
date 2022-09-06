# Spring 控制反转和依赖注入简介

## 1、**概述**

在本教程中，我们将介绍 IoC（控制反转）和 DI（依赖注入）的概念，并了解它们在 Spring 框架中是如何实现的。

## **2、什么是控制反转？**

控制反转是软件工程中的一项原则，它将对对象或程序部分的控制转移到容器或框架中。我们最常在面向对象编程的上下文中使用它。

与我们的自定义代码调用库的传统编程相比，IoC 使框架能够控制程序的流程并调用我们的自定义代码。为了实现这一点，框架使用内置附加行为的抽象。**如果我们想添加自己的行为，我们需要扩展框架的类或插入我们自己的类。**

这种架构的优点是：

- 将任务的执行与其实现分离
- 更容易在不同的实现之间切换
- 程序的更大模块化
- 通过隔离组件或模拟其依赖关系并允许组件通过合约进行通信，从而更轻松地测试程序

我们可以通过各种机制来实现控制反转，例如：策略设计模式、服务定位器模式、工厂模式和依赖注入（DI）。

接下来我们将研究 DI。



## **3、什么是依赖注入？**

依赖注入是我们可以用来实现 IoC 的一种模式，其中被反转的控制是设置对象的依赖关系。

将对象与其他对象连接起来，或将对象“注入”到其他对象中，是由汇编程序完成的，而不是由对象本身完成的。

以下是我们如何在传统编程中创建对象依赖项：

```java
public class Store {
    private Item item;
 
    public Store() {
        item = new ItemImpl1();    
    }
}
```

在上面的示例中，我们需要在*Store*类本身中实例化*Item*接口的实现。

通过使用 DI，我们可以重写示例，而无需指定我们想要的*Item的实现：*

```java
public class Store {
    private Item item;
    public Store(Item item) {
        this.item = item;
    }
}
```

在接下来的部分中，我们将了解如何通过元数据提供*Item的实现。*

IoC 和 DI 都是简单的概念，但它们对我们构建系统的方式有着深远的影响，因此它们非常值得充分理解。



## **4、Spring IoC 容器**

IoC 容器是实现 IoC 的框架的共同特征。

在 Spring 框架中，接口 *ApplicationContext*代表 IoC 容器。Spring 容器负责实例化、配置和组装称为*bean*的对象，以及管理它们的生命周期。

Spring 框架提供了*ApplicationContext*接口的几种实现：*ClassPathXmlApplicationContext*和*FileSystemXmlApplicationContext*用于独立应用程序，以及*WebApplicationContext*用于 Web 应用程序。

为了组装 bean，容器使用配置元数据，它可以是 XML 配置或注解的形式。

这是手动实例化容器的一种方法：

```java
ApplicationContext context
  = new ClassPathXmlApplicationContext("applicationContext.xml");
```

为了设置上面例子中的项目属性，我们可以使用元数据。然后容器会读取这个元数据，并在运行时使用它来组装Bean。

Spring中的依赖注入可以通过构造函数、设置器或字段来完成。

## **5、基于构造函数的依赖注入**

在基于构造函数的依赖注入的情况下，容器将调用一个构造函数，其参数分别代表我们想要设置的依赖。

Spring主要通过类型来解析每个参数，然后是属性名称和索引，以便进行歧义处理。让我们看看使用注解配置Bean及其依赖关系的情况。

```java
@Configuration
public class AppConfig {

    @Bean
    public Item item1() {
        return new ItemImpl1();
    }

    @Bean
    public Store store() {
        return new Store(item1());
    }
}
```

*@Configuration*注解表明该类是 bean 定义的来源。我们还可以将它添加到多个配置类中。

我们在方法上使用*@Bean*注解来定义一个 bean。如果我们不指定自定义名称，那么 bean 名称将默认为方法名称。

对于具有默认*单例*范围的 bean，Spring 首先检查 bean 的缓存实例是否已经存在，如果不存在则只创建一个新实例。如果我们使用*原型*作用域，容器会为每个方法调用返回一个新的 bean 实例。

创建 bean 配置的另一种方法是通过 XML 配置：

```java
<bean id="item1" class="org.baeldung.store.ItemImpl1" /> 
<bean id="store" class="org.baeldung.store.Store"> 
    <constructor-arg type="ItemImpl1" index="0" name="item" ref="item1" /> 
</bean>
```

## **6、基于 Setter 的依赖注入**

对于基于 setter 的 DI，容器会在调用无参数构造函数或无参数静态工厂方法实例化 bean 后调用我们类的 setter 方法。让我们使用注解创建这个配置：

```java
@Bean
public Store store() {
    Store store = new Store();
    store.setItem(item1());
    return store;
}
```

我们还可以将 XML 用于相同的 bean 配置：

```java
<bean id="store" class="org.baeldung.store.Store">
    <property name="item" ref="item1" />
</bean>
```

我们可以为同一个 bean 组合基于构造函数和基于 setter 的注入类型。Spring 文档建议对强制依赖项使用基于构造函数的注入，对可选依赖项使用基于 setter 的注入。



## **7、 基于字段的**依赖注入

略

## **8、自动装配依赖**

略

## **9、 惰性初始化 Bean**

默认情况下，容器在初始化期间创建和配置所有单例 bean。为了避免这种情况，我们可以在 bean 配置中使用值为*true的**惰性初始化属性：*

```xml
<bean id="item1" class="org.baeldung.store.ItemImpl1" lazy-init="true" />
```

因此，*item1* bean 只会在第一次被请求时被初始化，而不是在启动时被初始化。这样做的好处是更快的初始化时间，但代价是在请求 bean 之前我们不会发现任何配置错误，这可能是应用程序已经运行后的几个小时甚至几天。

