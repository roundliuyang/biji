# Spring Application Context Events



## **1. Introduction**

In this tutorial, we'll learn about the event support mechanism provided by the Spring framework. We'll explore the various built-in events provided by the framework and then see how to consume an event.

To learn about creating and publishing custom events, have a look at [our previous tutorial here.](https://www.baeldung.com/spring-events)

Spring has an eventing mechanism which is built around the *ApplicationContext.* It can be used to **exchange information between different beans.** We can make use of application events by listening for events and executing custom code.

For example, a scenario here would be to execute custom logic on the complete startup of the *ApplicationContext*.



## **2. Standard Context Events**

In fact, there're a variety of built-in events in Spring, that **lets a developer hook into the lifecycle of an application and the context** and do some custom operation.

Even though we rarely use these events manually in an application, the framework uses it intensively within itself. Let's start by exploring various built-in events in Spring.



### **2.1. ContextRefreshedEvent**

On either **initializing or refreshing the** ***ApplicationContext**,* Spring raises the *ContextRefreshedEvent*. Typically a refresh can get triggered multiple times as long as the context has not been closed.

Notice that, we can also have the event triggered manually by calling the *refresh()* method on the *ConfigurableApplicationContext* interface.



### **2.2. ContextStartedEvent**

**By calling the start() method** on the *ConfigurableApplicationContext,* we trigger this event and start the *ApplicationContext*. As a matter of fact, the method is typically used to restart beans after an explicit stop. We can also use the method to deal components with no configuration for autostart.

**Here, it's important to note that the call to start() is always explicit as opposed to** ***refresh*****().**



### **2.3. ContextStoppedEvent**

A *ContextStoppedEvent* is published **when the ApplicationContext is stopped**, by invoking the *stop()* method on the *ConfigurableApplicationContext.* As discussed earlier, we can restart a stopped event by using *start()* method.



### **2.4. ContextClosedEvent**

This event is published **when the ApplicationContext is closed**, using the *close()* method in *ConfigurableApplicationContext*.
In reality, after closing a context, we cannot restart it.

A context reaches its end of life on closing it and hence we cannot restart it like in a *ContextStoppedEvent.*





## **3. @EventListener**

Next, let us explore how to consume the published events. Starting from version 4.2, Spring supports an annotation-driven event listener – *@EventListener.*

In particular, we can make use of this annotation to **automatically register an ApplicationListener based on the signature of the method** :

```java
@EventListener
public void handleContextRefreshEvent(ContextStartedEvent ctxStartEvt) {
    System.out.println("Context Start Event received.");
}
```

Significantly, **@EventListener is a core annotation and hence doesn't need any extra configuration**. In fact, the existing *<context:annotation-driven/>* element provides full support to it.

A method annotated with *@EventListener* can return a non-void type. If the value returned is non-null, the eventing mechanism will publish a new event for it.



### **3.1. Listening to Multiple Events**

Now, there might arise situations where we will need our listener to consume multiple events.

For such a scenario, we can make use of [classes](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/event/EventListener.html#classes) attribute:

```java
@EventListener(classes = { ContextStartedEvent.class, ContextStoppedEvent.class })
public void handleMultipleEvents() {
    System.out.println("Multi-event listener invoked");
}
```

## **4. Application Event Listener**

If we're using earlier versions of Spring (<4.2), we'll have to introduce a [custom *ApplicationEventListener* and override the method *onApplicationEvent* ](https://www.baeldung.com/spring-events)to listen to an event.



## **5. Conclusion**

In this article, we've explored the various built-in events in Spring. In addition, we've seen various ways to listen to the published events.

As always, the code snippets used in the article can be found [over on Github](https://github.com/eugenp/tutorials/tree/master/spring-core).









