## DDD Aggregates and @DomainEvents



### **1. Overview**



In this tutorial, we'll explain how to use *@DomainEvents* annotation and *AbstractAggregateRoot* class to conveniently publish and handle domain events produced by aggregate – one of the key tactical design patterns in Domain-driven design.

**Aggregates accept business commands, which usually results in producing an event related to the business domain – the Domain Event**.

If you'd like to learn more about DDD and aggregates, it's best to start with Eric Evans' [original book](https://www.amazon.com/exec/obidos/ASIN/0321125215/domainlanguag-20). There's also a great [series about effective aggregate design](http://dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_1.pdf) written by Vaughn Vernon. Definitely worth reading.

It can be cumbersome to manually work with domain events. Thankfully, **Spring Framework allows us to easily publish and handle domain events when working with aggregate roots** using data repositories.



### **2. Maven Dependencies**



Spring Data introduced *@DomainEvents* in Ingalls release train. It's available for any kind of repository.

Code samples provided for this article use Spring Data JPA. **The simplest way to integrate Spring domain events with our project is to use the Spring Boot Data JPA Starter:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```



### **3. Publish Events Manually** 



First, let's try to publish domain events manually. We'll explain the *@DomainEvents* usage in the next section.

For the needs of this article, we'll use an empty marker class for domain events – the *DomainEvent*.

We're going to use standard *ApplicationEventPublisher* interface.

**There're two good places where we can publish events: service layer or directly inside the aggregate**.



#### 3.1. Service Layer

**We can simply publish events after calling the repository save method inside a service method**.

If a service method is part of a transaction and we handle the events inside the listener annotated with *@TransactionalEventListener*, then events will be handled only after the transaction commits successfully.

Therefore, there's no risk of having “fake” events handled when the transaction is rolled back and the aggregate isn't updated:

```java
@Service
public class DomainService {
 
    // ...
    @Transactional
    public void serviceDomainOperation(long entityId) {
        repository.findById(entityId)
            .ifPresent(entity -> {
                entity.domainOperation();
                repository.save(entity);
                eventPublisher.publishEvent(new DomainEvent());
            });
    }
}
```

Here's a test that proves events are indeed published by service*DomainOperation*:

```java
@DisplayName("given existing aggregate,"
    + " when do domain operation on service,"
    + " then domain event is published")
@Test
void serviceEventsTest() {
    Aggregate existingDomainEntity = new Aggregate(1, eventPublisher);
    repository.save(existingDomainEntity);

    // when
    domainService.serviceDomainOperation(existingDomainEntity.getId());

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```



#### 3.2. Aggregate

**We can also publish events directly from within the aggregate**.

This way we manage the creation of domain events inside the class which feels more natural for this:

```java
@Entity
class Aggregate {
    // ...
    void domainOperation() {
        // some business logic
        if (eventPublisher != null) {
            eventPublisher.publishEvent(new DomainEvent());
        }
    }
}
```

Unfortunately, this might not work as expected because of how Spring Data initializes entities from repositories.

Here's the corresponding test that shows the real behavior:

```java
@DisplayName("given existing aggregate,"
    + " when do domain operation directly on aggregate,"
    + " then domain event is NOT published")
@Test
void aggregateEventsTest() {
    Aggregate existingDomainEntity = new Aggregate(0, eventPublisher);
    repository.save(existingDomainEntity);

    // when
    repository.findById(existingDomainEntity.getId())
      .get()
      .domainOperation();

    // then
    verifyNoInteractions(eventHandler);
}
```

As we can see, the event isn't published at all. Having dependencies inside the aggregate might not be a great idea. In this example, *ApplicationEventPublisher* is not initialized automatically by Spring Data.

The aggregate is constructed by invoking the default constructor. To make it behave as we would expect, we'd need to manually recreate entities (e.g. using custom factories or aspect programming).

Also, we should avoid publishing events immediately after the aggregate method finishes. At least, unless we are 100% sure this method is part of a transaction. Otherwise, we might have “spurious” events published when change is not yet persisted. This might lead to inconsistencies in the system.

If we want to avoid this, we must remember to always call aggregate methods inside a transaction. Unfortunately, this way we couple our design heavily to the persistence technology. We need to remember that we don't always work with transactional systems.

**Therefore, it's generally a better idea to let our aggregate simply manage a collection of domain events and return them when it's about to get persisted**.

In the next section, we'll explain how we can make domain events publishing more manageable by using @DomainEvents and *@AfterDomainEvents* annotations.



### 4. Publish Events Using *@DomainEvents*

**Since Spring Data Ingalls release train we can use the @DomainEvents annotation to automatically publish domain events**.

A method annotated with *@DomainEvents* is automatically invoked by Spring Data whenever an entity is saved using the right repository.

Then, events returned by this method are published using the *ApplicationEventPublisher* interface:

```java
@Entity
public class Aggregate2 {
 
    @Transient
    private final Collection<DomainEvent> domainEvents;
    // ...
    public void domainOperation() {
        // some domain operation
        domainEvents.add(new DomainEvent());
    }

    @DomainEvents
    public Collection<DomainEvent> events() {
        return domainEvents;
    }
}
```

Here's the example explaining this behavior:

```java
@DisplayName("given aggregate with @DomainEvents,"
    + " when do domain operation and save,"
    + " then event is published")
@Test
void domainEvents() {
 
    // given
    Aggregate2 aggregate = new Aggregate2();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

After domain events are published, the method annotated with *@AfterDomainEventsPublication* is called.

The purpose of this method is usually to clear the list of all events, so they aren't published again in the future:

```java
@AfterDomainEventPublication
public void clearEvents() {
    domainEvents.clear();
}
```

Let's add this method to the *Aggregate2* class and see how it works:

```java
@DisplayName("given aggregate with @AfterDomainEventPublication,"
    + " when do domain operation and save twice,"
    + " then an event is published only for the first time")
@Test
void afterDomainEvents() {
 
    // given
    Aggregate2 aggregate = new Aggregate2();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

We clearly see that event is published only for the first time. **If we removed the @AfterDomainEventPublication annotation from the clearEvents method, then the same event would be published for the second time**.

However, it's up to the implementor what would actually happen. Spring only guarantees to call this method – nothing more.



### 5. Use AbstractAggregateRoot Template

**It's possible to further simplify publishing of domain events thanks to the AbstractAggregateRoot template class**. All we have to do is to call *register* method when we want to add the new domain event to the collection of events:

```java
@Entity
public class Aggregate3 extends AbstractAggregateRoot<Aggregate3> {
    // ...
    public void domainOperation() {
        // some domain operation
        registerEvent(new DomainEvent());
    }
}
```

This is a counterpart to the example shown in the previous section.

Just to make sure everything works as expected – here are the tests:

```java
@DisplayName("given aggregate extending AbstractAggregateRoot,"
    + " when do domain operation and save twice,"
    + " then an event is published only for the first time")
@Test
void afterDomainEvents() {
 
    // given
    Aggregate3 aggregate = new Aggregate3();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}

@DisplayName("given aggregate extending AbstractAggregateRoot,"
    + " when do domain operation and save,"
    + " then an event is published")
@Test
void domainEvents() {
    // given
    Aggregate3 aggregate = new Aggregate3();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

As we can see, we can produce a lot less code and achieve exactly the same effect.



### **6. Implementation Caveats** 

While it might look like a great idea to use the *@DomainEvents* feature at first, there are some pitfalls we need to be aware of.



#### 6.1. Unpublished Events

When working with JPA we don't necessarily call save method when we want to persist the changes.

If our code is part of a transaction (e.g. annotated with *@Transactional*) and makes changes to the existing entity, then we usually simply let the transaction commit without explicitly calling the *save* method on a repository. So, even if our aggregate produced new domain events they will never get published.

**We need also remember that @DomainEvents feature works only when using Spring Data repositories**. This might be an important design factor.



#### 6.2. Lost Events

**If an exception occurs during events publication, the listeners will simply never get notified**.

Even if we could somehow guarantee notification of event listeners, currently there's no backpressure to let publishers know something went wrong. If event listener gets interrupted by an exception, the event will remain unconsumed and it will never be published again.

This design flaw is known to the Spring dev team. One of the lead developers even suggested a possible [solution to this problem](https://github.com/olivergierke/spring-domain-events).



#### 6.3. Local Context

Domain events are published using a simple *ApplicationEventPublisher* interface.

**By default, when using ApplicationEventPublisher, events are published and consumed in the same thread**. Everything happens in the same container.

Usually, we want to send events through some kind of message broker, so the other distributed clients/systems get notified. In such case, we'd need to manually forward events to the message broker.

It's also possible to use [Spring Integration](https://spring.io/projects/spring-integration) or third-party solutions, such as [Apache Camel](https://camel.apache.org/spring-event.html).



### **7. Conclusion**

In this article, we've learned how to manage aggregate domain events using *@DomainEvents* annotation.

**This approach can greatly simplify events infrastructure so we can focus only on the domain logic**. We just need to be aware that there's no silver bullet and the way Spring handles domain events is not an exception.

The full source code of all the examples is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations).





