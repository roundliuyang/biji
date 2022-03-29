## dependencyManagement vs dependencies Tags

refer:https://www.baeldung.com/maven-dependencymanagement-vs-dependencies-tags

## 1. Overview

In this tutorial, we will review two important [Maven](https://www.baeldung.com/maven-guide) tags — *dependencyManagement* and *dependencies*.

**These features are especially useful for multi-module projects.**

We'll review the similarities and differences of the two tags, and we'll also look at some common mistakes that developers make when using them that can cause confusion.



## 2. Usage

In general, we use the *dependencyManagement* tag to avoid repeating the *version* and *scope* tags when we define our dependencies in the *dependencies* tag. In this way, the required dependency is declared in a central POM file.



### 2.1. *dependencyManagement*

This tag consists of a *dependencies* tag which itself might contain multiple *dependency* tags. Each *dependency* is supposed to have at least three main tags: *groupId*, *artifactId,* and *version*. Let's see an example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

The above code just declares the new artifact *commons-lang3*, but it doesn't really add it to the project dependency resource list.



### 2.2. *dependencies*

This tag contains a list of *dependency* tags. Each *dependency* is supposed to have at least two main tags, which are *groupId* and *artifactId.*

Let's see a quick example:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

**The version and scope tags can be inherited implicitly if we have used the dependencyManagement tag before in the POM file**:

```java
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies>
```



## 3. Similarities

Both of these tags aim to declare some third-party or sub-module dependency. They complement each other.

In fact, we usually define the *dependencyManagement* tag once, preceding the *dependencies* tag. This is used in order to declare the dependencies in the POM file. **This declaration is just an announcement, and it doesn't really add the dependency to the project.**

Let's see a sample for adding the JUnit library dependency:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

As we can see in the above code, there is a *dependencyManagement* tag that itself contains another *dependencies* tag.

Now, let's see the other side of the code, which adds the actual dependency to the project:

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

So, the current tag is very similar to the previous one. Both of them would define a list of dependencies. Of course, there are small differences which we will cover soon.

The same *groupId* and *artifactId* tags are repeated in both code snippets, and there is a meaningful correlation between them: Both of them refer to the same artifact.

As we can see, there is not any *version* tag present in our later *dependency* tag. Surprisingly, it's valid syntax, and it can be parsed and compiled without any problem. The reason can be guessed easily: It will use the version declared by *dependencyManagement*.



## 4. Differences

### **4.1. Structural Difference**

**As we covered earlier, the main structural difference between these two tags is the logic of inheritance.** We define the version in the *dependencyManagement* tag, and then we can use the mentioned version without specifying it in the next *dependencies* tag.



### **4.2. Behavioral Difference**

**dependencyManagement is just a declaration, and it does not really add a dependency.** The declared *dependencies* in this section must be later used by the *dependencies* tag. It is just the *dependencies* tag that causes real dependency to happen. In the above sample, the *dependencyManagement* tag will not add the *junit* library into any scope. It is just a declaration for the future *dependencies* tag.



## 5. Real-World Example

Nearly all Maven-based open-source projects use this mechanism.

Let's see an example from [the Maven project](https://github.com/apache/maven/blob/master/pom.xml) itself. We see the *hamcrest-core* dependency, which exists in the Maven project. It's declared first in the *dependencyManagement* tag, and then it is imported by the main *dependencies* tag:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>2.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest-core</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



## 6. Common Use Cases

A very common use case for this feature is a multi-module project.

Imagine we have a big project which consists of different modules. Each module has its own dependencies, and each developer might use a different version for the used dependencies. Then it could lead to a mesh of different artifact versions, which can also cause difficult and hard-to-resolve conflicts.

**The easy solution for this problem is definitely using the dependencyManagement tag in the root POM file (usually called the “parent”) and then using the dependencies in the child's POM files (sub-modules) and even the parent module itself (if applicable).**

If we have a single module, does it make sense to use this feature or not? Although this is very useful in multi-module environments, it can also be a rule of thumb to obey it as a best practice even in a single-module project. This helps the project readability and also makes it ready to extend to a multi-module project.

## 7. Common Mistakes

One common mistake is defining a dependency just in the *dependencyManagement* section and not including it in the *dependencies* tag. In this case, we will encounter compile or runtime errors, depending on the mentioned *scope*.

Let's see an example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
        ...
    </dependencies>
</dependencyManagement>
```

Imagine the above POM code snippet. Then suppose we're going to use this library in a sub-module source file:

```java
import org.apache.commons.lang3.StringUtils;

public class Main {

    public static void main(String[] args) {
        StringUtils.isBlank(" ");
    }
}
```

This code will not compile because of the missing library. The compiler complains about an error:

```properties
[ERROR] Failed to execute goal compile (default-compile) on project sample-module: Compilation failure
[ERROR] ~/sample-module/src/main/java/com/baeldung/Main.java:[3,32] package org.apache.commons.lang3 does not exist
```

To avoid this error, it's enough to add the below *dependencies* tag to the sub-module POM file:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies>
```

