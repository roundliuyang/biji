# Spring Events



## **1. Overview**



In this tutorial, we'll be discussing **how to use events in Spring.**

Events are one of the more overlooked functionalities in the framework but also one of the more useful. And like many other things in Spring, event publishing is one of the capabilities provided by *ApplicationContext*.

There are a few simple guidelines to follow:

- The event class should extend *ApplicationEvent* if we're using versions before Spring Framework 4.2. [As of the 4.2 version](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisher.html#publishEvent-java.lang.Object-), the event classes no longer need to extend the *ApplicationEvent* class.
- The publisher should inject an *ApplicationEventPublisher* object.
- The listener should implement the *ApplicationListener* interface.



## **2. A Custom Event**



Spring allows us to create and publish custom events that by default **are synchronous.** This has a few advantages, such as the listener being able to participate in the publisher’s transaction context.



### **2.1. A Simple Application Event**

Let’s create **a simple event class** — just a placeholder to store the event data.

In this case, the event class holds a String message:

```java
public class CustomSpringEvent extends ApplicationEvent {
    private String message;

    public CustomSpringEvent(Object source, String message) {
        super(source);
        this.message = message;
    }
    public String getMessage() {
        return message;
    }
}
```





### **2.2. A Publisher**

Now let’s create **a publisher of that event.** The publisher constructs the event object and publishes it to anyone who's listening.

To publish the event, the publisher can simply inject the *ApplicationEventPublisher* and use the *publishEvent()* API:

```java
@Component
public class CustomSpringEventPublisher {
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void publishCustomEvent(final String message) {
        System.out.println("Publishing custom event. ");
        CustomSpringEvent customSpringEvent = new CustomSpringEvent(this, message);
        applicationEventPublisher.publishEvent(customSpringEvent);
    }
}
```

Alternatively, the publisher class can implement the *ApplicationEventPublisherAware* interface, and this will also inject the event publisher on the application startup. Usually, it's simpler to just inject the publisher with *@Autowire*.

As of Spring Framework 4.2, the *ApplicationEventPublisher* interface provides a new overload for the *publishEvent(Object event)* method that accepts any object as the event. **Therefore, Spring events no longer need to extend the ApplicationEvent class.**



### **2.3. A Listener**

Finally, let's create the listener.

The only requirement for the listener is to be a bean and implement *ApplicationListener* interface:

```java
@Component
public class CustomSpringEventListener implements ApplicationListener<CustomSpringEvent> {
    @Override
    public void onApplicationEvent(CustomSpringEvent event) {
        System.out.println("Received spring custom event - " + event.getMessage());
    }
}
```

Notice how our custom listener is parametrized with the generic type of custom event, which makes the *onApplicationEvent()* method type-safe. This also avoids having to check if the object is an instance of a specific event class and casting it.

And, as already discussed (by default **Spring events are synchronous**), the *doStuffAndPublishAnEvent()* method blocks until all listeners finish processing the event.



## **3. Creating Asynchronous Events**



In some cases, publishing events synchronously isn't really what we're looking for — **we may need async handling of our events.**

We can turn that on in the configuration by creating an *ApplicationEventMulticaster* bean with an executor.

For our purposes here, *SimpleAsyncTaskExecutor* works well:

```java
@Configuration
public class AsynchronousSpringEventsConfig {
    @Bean(name = "applicationEventMulticaster")
    public ApplicationEventMulticaster simpleApplicationEventMulticaster() {
        SimpleApplicationEventMulticaster eventMulticaster =
          new SimpleApplicationEventMulticaster();
        
        eventMulticaster.setTaskExecutor(new SimpleAsyncTaskExecutor());
        return eventMulticaster;
    }
}
```

The event, the publisher and the listener implementations remain the same as before, but now **the listener will asynchronously deal with the event in a separate thread.**



## **4. Existing Framework Events**

Spring itself publishes a variety of events out of the box. For example, the *ApplicationContext* will fire various framework events: *ContextRefreshedEvent*, *ContextStartedEvent*, *RequestHandledEvent* etc.

These events provide application developers an option to hook into the life cycle of the application and the context and add in their own custom logic where needed.

Here's a quick example of a listener listening for context refreshes:

```java
public class ContextRefreshedListener 
  implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent cse) {
        System.out.println("Handling context re-freshed event. ");
    }
}
```

To learn more about existing framework events, have a look at [our next tutorial here](https://www.baeldung.com/spring-context-events).



## **5. Annotation-Driven Event Listener**

Starting with Spring 4.2, an event listener is not required to be a bean implementing the *ApplicationListener* interface — it can be registered on any *public* method of a managed bean via the *@EventListener* annotation:

```java
@Component
public class AnnotationDrivenEventListener {
    @EventListener
    public void handleContextStart(ContextStartedEvent cse) {
        System.out.println("Handling context started event.");
    }
}
```

As before, the method signature declares the event type it consumes.

By default, the listener is invoked synchronously. However, we can easily make it asynchronous by adding an *@Async* annotation. We just need to remember to [enable *Async* support](https://www.baeldung.com/spring-async#enable-async-support) in the application.





## **6. Generics Support**

It is also possible to dispatch events with generics information in the event type.

### **6.1. A Generic Application Event**

**Let's create a generic event type.**

In our example, the event class holds any content and a *success* status indicator:

```java
public class GenericSpringEvent<T> {
    private T what;
    protected boolean success;

    public GenericSpringEvent(T what, boolean success) {
        this.what = what;
        this.success = success;
    }
    // ... standard getters
}
```

Notice the difference between *GenericSpringEvent* and *CustomSpringEvent*. We now have the flexibility to publish any arbitrary event and it's not required to extend from *ApplicationEvent* anymore.



### **6.2. A Listener**

Now let’s create **a listener of that event.**

We could define the listener by implementing the *ApplicationListener* interface like before:

```java
@Component
public class GenericSpringEventListener 
  implements ApplicationListener<GenericSpringEvent<String>> {
    @Override
    public void onApplicationEvent(@NonNull GenericSpringEvent<String> event) {
        System.out.println("Received spring generic event - " + event.getWhat());
    }
}
```

But this definition unfortunately requires us to inherit *GenericSpringEvent* from the *ApplicationEvent* class. So for this tutorial, let's make use of an annotation-driven event listener discussed [previously](https://www.baeldung.com/spring-events#annotation-driven).

It is also possible to **make the event listener conditional** by defining a boolean SpEL expression on the *@EventListener* annotation.

In this case, the event handler will only be invoked for a successful *GenericSpringEvent* of *String*:

```java
@Component
public class AnnotationDrivenEventListener {
    @EventListener(condition = "#event.success")
    public void handleSuccessful(GenericSpringEvent<String> event) {
        System.out.println("Handling generic event (conditional).");
    }
}
```

The [Spring Expression Language (SpEL)](https://www.baeldung.com/spring-expression-language) is a powerful expression language that's covered in detail in another tutorial.



### **6.3. A Publisher**

The event publisher is similar to the one described [above](https://www.baeldung.com/spring-events#publisher). But due to type erasure, we need to publish an event that resolves the generics parameter we would filter on, for example, *class GenericStringSpringEvent extends GenericSpringEvent<String>*.

Also, there's **an alternative way of publishing events.** If we return a non-null value from a method annotated with *@EventListener* as the result, Spring Framework will send that result as a new event for us. Moreover, we can publish multiple new events by returning them in a collection as the result of event processing.





## **7. Transaction-Bound Events**

This section is about using the *@TransactionalEventListener* annotation. To learn more about transaction management, check out [Transactions With Spring and JPA](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring).

Since Spring 4.2, the framework provides a new *@TransactionalEventListener* annotation, which is an extension of *@EventListener*, that allows binding the listener of an event to a phase of the transaction.



Binding is possible to the following transaction phases:

- *AFTER_COMMIT* (default) is used to fire the event if the transaction has **completed successfully.**
- *AFTER_ROLLBACK* – if the transaction has **rolled back**
- *AFTER_COMPLETION* – if the transaction has **completed** (an alias for *AFTER_COMMIT* and *AFTER_ROLLBACK*)
- *BEFORE_COMMIT* is used to fire the event right **before** transaction **commit.**

Here's a quick example of a transactional event listener:

```java
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
public void handleCustom(CustomSpringEvent event) {
    System.out.println("Handling event inside a transaction BEFORE COMMIT.");
}
```

This listener will be invoked only if there's a transaction in which the event producer is running and it's about to be committed.

And if no transaction is running, the event isn’t sent at all unless we override this by setting *fallbackExecution* attribute to *true*.



## **8. Conclusion**

In this quick article, we went over the basics of **dealing with events in Spring**, including creating a simple custom event, publishing it and then handling it in a listener.

We also had a brief look at how to enable the asynchronous processing of events in the configuration.

Then we learned about improvements introduced in Spring 4.2, such as annotation-driven listeners, better generics support and events binding to transaction phases.

As always, the code presented in this article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-core-2). This is a Maven-based project, so it should be easy to import and run as it is.