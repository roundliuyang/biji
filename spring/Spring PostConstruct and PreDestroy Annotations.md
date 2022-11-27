# Spring PostConstruct and PreDestroy Annotations



## 1. Introduction

Spring allows us to attach custom actions to [bean creation and destruction](https://www.baeldung.com/running-setup-logic-on-startup-in-spring). We can, for example, do it by implementing the *InitializingBean* and *DisposableBean* interfaces.

In this quick tutorial, we'll look at a second possibility, the *@PostConstruct* and *@PreDestroy* annotations.



## 2. *@PostConstruct*

**Spring calls the methods annotated with @PostConstruct only once, just after the initialization of bean properties**. Keep in mind that these methods will run even if there's nothing to initialize.

The method annotated with *@PostConstruct* can have any access level, but it can't be static.

One possible use of *@PostConstruct* is populating a database. For instance, during development, we might want to create some default users:

```java
@Component
public class DbInit {

    @Autowired
    private UserRepository userRepository;

    @PostConstruct
    private void postConstruct() {
        User admin = new User("admin", "admin password");
        User normalUser = new User("user", "user password");
        userRepository.save(admin, normalUser);
    }
}
```

The above example will first initialize *UserRepository* and then run the *@PostConstruct* method.



## 3. *@PreDestroy*

A method annotated with *@PreDestroy* runs only once, just before Spring removes our bean from the application context.

Same as with *@PostConstruct*, the methods annotated with *@PreDestroy* can have any access level, but can't be static.

```java
@Component
public class UserRepository {

    private DbConnection dbConnection;
    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
```

The purpose of this method should be to release resources or perform other cleanup tasks, such as closing a database connection, before the bean gets destroyed.



## 4. Java 9+

Note that both the *@PostConstruct* and *@PreDestroy* annotations are part of Java EE. Since [Java EE was deprecated in Java 9](https://www.baeldung.com/java-enterprise-evolution), and removed in Java 11, we have to add an additional dependency to use these annotations:

```java
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```



## 5. Conclusion

In this brief article, we learned how to use the *@PostConstruct* and *@PreDestroy* annotations.

As always, all the source code is available on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-core).















