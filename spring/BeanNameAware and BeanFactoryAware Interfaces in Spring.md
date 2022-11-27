# BeanNameAware and BeanFactoryAware Interfaces in Spring



## **1. Overview**



In this quick tutorial, **we're going to focus on the BeanNameAware and BeanFactoryAware interfaces, in the Spring Framework**.

We'll describe each interface separately with the pros and cons of their usage.

## **2. Aware Interface**

Both *BeanNameAware* and *BeanFactoryAware* belong to the *org.springframework.beans.factory.Aware* root marker interface. This uses setter injection to get an object during the application context startup.

**The Aware interface is a mix of callback, listener, and observer design patterns**. It indicates that the bean is eligible to be notified by the Spring container through the callback methods.





## **3. BeanNameAware**

**BeanNameAware makes the object aware of the bean name defined in the container**.

Let's have a look at an example:

```java
public class MyBeanName implements BeanNameAware {

    @Override
    public void setBeanName(String beanName) {
        System.out.println(beanName);
    }
}
```

The *beanName* property represents the bean id registered in the Spring container. In our implementation, we're simply displaying the bean name.

Next, let's register a bean of this type in a Spring configuration class:

```java
@Configuration
public class Config {

    @Bean(name = "myCustomBeanName")
    public MyBeanName getMyBeanName() {
        return new MyBeanName();
    }
}
```

Here we've explicitly assigned a name to our *MyBeanName* class with the *@Bean(name* = *“myCustomBeanName”)* line.

Now we can start the application context and get the bean from it:

```java
AnnotationConfigApplicationContext context 
  = new AnnotationConfigApplicationContext(Config.class);

MyBeanName myBeanName = context.getBean(MyBeanName.class);
```

As we expect, the *setBeanName* method prints out *“myCustomBeanName”*.

If we remove the *name = “…”* code from the *@Bean* annotation the container, in this case, assigns *getMyBeanName()*  method name into the bean. So the output will be *“getMyBeanName”*.



## **4. BeanFactoryAware**

**BeanFactoryAware is used to inject the BeanFactory object**. This way we get access to the *BeanFactory* which created the object.

Here's an example of a *MyBeanFactory* class:

```java
public class MyBeanFactory implements BeanFactoryAware {

    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void getMyBeanName() {
        MyBeanName myBeanName = beanFactory.getBean(MyBeanName.class);
        System.out.println(beanFactory.isSingleton("myCustomBeanName"));
    }
}
```

With the help of the *setBeanFactory()* method, we assign the *BeanFactory* reference from the IoC container to the *beanFactory property*.

After that, we can use it directly like in the *getMyBeanName()* function.

Let's initialize the *MyBeanFactory* and call the *getMyBeanName()* method:

```java
MyBeanFactory myBeanFactory = context.getBean(MyBeanFactory.class);
myBeanFactory.getMyBeanName();
```

As we have already instantiated the *MyBeanName* class in the previous example, Spring will invoke the existing instance here.

The *beanFactory.isSingleton(“myCustomBeanName”)* line verifies that.

## **5. When to Use?**

The typical use case for *BeanNameAware* could be acquiring the bean name for logging or wiring purposes. For the *BeanFactoryAware* it could be the ability to use a spring bean from legacy code.

In most cases, we should avoid using any of the *Aware* interfaces, unless we need them. **Implementing these interfaces will couple the code to the Spring framework.**



## **6. Conclusion**

In this write-up, we learned about the *BeanNameAware* and *BeanFactoryAware* interfaces and how to use them in practice.

As usual, the complete code for this article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-core).





