# Spring Component Scanning



## **1. Overview**

In this tutorial, we'll cover component scanning in Spring. When working with Spring, we can annotate our classes in order to make them into Spring beans. Furthermore, **we can tell Spring where to search for these annotated classes,** as not all of them must become beans in this particular run.

Of course, there are some defaults for component scanning, but we can also customize the packages for search.

First, let's look at the default settings.



## **2. @ComponentScan Without Arguments**



### **2.1. Using @ComponentScan in a Spring Application**



With Spring, **we use the @ComponentScan annotation along with the @Configuration annotation to specify the packages that we want to be scanned**. *@ComponentScan* without arguments tells Spring to scan the current package and all of its sub-packages.

Let's say we have the following *@Configuration* in *com.baeldung.componentscan.springapp* package:

```java
@Configuration
@ComponentScan
public class SpringComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = 
          new AnnotationConfigApplicationContext(SpringComponentScanApp.class);

        for (String beanName : applicationContext.getBeanDefinitionNames()) {
            System.out.println(beanName);
        }
    }
}
```

In addition, we have the *Cat* and *Dog* components in *com.baeldung.componentscan.springapp.animals* package:

```java
package com.baeldung.componentscan.springapp.animals;
// ...
@Component
public class Cat {}
```

```java
package com.baeldung.componentscan.springapp.animals;
// ...
@Component
public class Dog {}
```

Finally, we have the *Rose* component in *com.baeldung.componentscan.springapp.flowers* package:

```java
package com.baeldung.componentscan.springapp.flowers;
// ...
@Component
public class Rose {}
```

The output of the *main()* method will contain all the beans of *com.baeldung.componentscan.springapp* package and its sub-packages:

```
springComponentScanApp
cat
dog
rose
exampleBean
```

Note that the main application class is also a bean, as it's annotated with *@Configuration,* which is a *@Component*.

We should also note that the main application class and the configuration class are not necessarily the same. If they are different, it doesn't matter where we put the main application class. **Only the location of the configuration class matters, as component scanning starts from its package by default**.

Finally, note that in our example, *@ComponentScan* is equivalent to:

```
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp")
```

The *basePackages* argument is a package or an array of packages for scanning.



### **2.2. Using @ComponentScan in a Spring Boot Application**

The trick with Spring Boot is that many things happen implicitly. We use the *@SpringBootApplication* annotation, but it's a combination of three annotations:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

Let's create a similar structure in *com.baeldung.componentscan.springbootapp* package. This time the main application will be:

```java
package com.baeldung.componentscan.springbootapp;
// ...
@SpringBootApplication
public class SpringBootComponentScanApp {
    private static ApplicationContext applicationContext;

    @Bean
    public ExampleBean exampleBean() {
        return new ExampleBean();
    }

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(SpringBootComponentScanApp.class, args);
        checkBeansPresence(
          "cat", "dog", "rose", "exampleBean", "springBootComponentScanApp");

    }

    private static void checkBeansPresence(String... beans) {
        for (String beanName : beans) {
            System.out.println("Is " + beanName + " in ApplicationContext: " + 
              applicationContext.containsBean(beanName));
        }
    }
}
```

All other packages and classes remain the same, we'll just copy them to the nearby *com.baeldung.componentscan.springbootapp* package.

Spring Boot scans packages similarly to our previous example. Let's check the output:

```java
Is cat in ApplicationContext: true
Is dog in ApplicationContext: true
Is rose in ApplicationContext: true
Is exampleBean in ApplicationContext: true
Is springBootComponentScanApp in ApplicationContext: true
```

The reason we're just checking the beans for existence in our second example (as opposed to printing out all the beans), is that the output would be too large.

This is because of the implicit *@EnableAutoConfiguration* annotation, which makes Spring Boot create many beans automatically, relying on the dependencies in *pom.xml* file.



## **3. @ComponentScan With Arguments**

Now let's customize the paths for scanning. For example, let's say we want to exclude the *Rose* bean.

### 3.1. *@ComponentScan* for Specific Packages

We can do this a few different ways. First, we can change the base package:

```java
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals")
@Configuration
public class SpringComponentScanApp {
   // ...
}
```

Now the output will be:

```
springComponentScanApp
cat
dog
exampleBean
```

Let's see what's behind this:

- *springComponentScanApp* is created as it's a configuration passed as an argument to the *AnnotationConfigApplicationContext*
- *exampleBean* is a bean configured inside the configuration
- *cat* and *dog* are in the specified *com.baeldung.componentscan.springapp.animals* package

All of the above-listed customizations are applicable in Spring Boot too. We can use *@ComponentScan* together with *@SpringBootApplication* and the result will be the same:

```
@SpringBootApplication
@ComponentScan(basePackages = "com.baeldung.componentscan.springbootapp.animals")
```



### 3.2. *@ComponentScan* with Multiple Packages

Spring provides a convenient way to specify multiple package names. To do so, we need to use a string array.

Each string of the array denotes a package name:

```java
@ComponentScan(basePackages = {"com.baeldung.componentscan.springapp.animals", "com.baeldung.componentscan.springapp.flowers"})
```

Alternatively, since spring 4.1.1, **we can use a comma, a semicolon, or a space to separate the packages list**:

```java
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals;com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals,com.baeldung.componentscan.springapp.flowers")
@ComponentScan(basePackages = "com.baeldung.componentscan.springapp.animals com.baeldung.componentscan.springapp.flowers")
```



### 3.3. *@ComponentScan* with Exclusions

Another way is to use a filter, specifying the pattern for the classes to exclude:

```java
@ComponentScan(excludeFilters = 
  @ComponentScan.Filter(type=FilterType.REGEX,
    pattern="com\\.baeldung\\.componentscan\\.springapp\\.flowers\\..*"))
```

We can also choose a different filter type, as **the annotation supports several flexible options for filtering the scanned classes**:

```java
@ComponentScan(excludeFilters = 
  @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = Rose.class))
```

## **4. The Default Package**

We should avoid putting the *@Configuration* class [in the default package](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-structuring-your-code.html) (i.e. by not specifying the package at all). If we do, Spring scans all the classes in all jars in a classpath, which causes errors and the application probably doesn't start.

























