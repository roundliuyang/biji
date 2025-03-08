# 具有自定义父级的 Spring Boot 依赖管理



##  **概述**

Spring Boot 提供了父 POM，以便更轻松地创建 Spring Boot 应用程序。

然而，如果我们已经有一个父项目继承自它，使用父 POM 可能并不总是可取的。

在本教程中，我们将了解如何在不使用父启动器的情况下，仍然使用 Spring Boot。



## 没有父 POM 的 Spring Boot

父 POM 文件负责依赖和插件的管理。因此，继承它为应用程序提供了宝贵的支持，因此在创建 Spring Boot 应用程序时，通常是首选的做法。关于如何基于父启动器构建应用程序的更多细节，可以参考我们之前的文章。

然而，在实践中，我们可能会受到设计规则或其他偏好的限制，不能使用父项目。

幸运的是，Spring Boot 提供了一种替代方案，允许我们不继承父启动器，但仍然可以享受它的一些优势。

如果我们不使用父 POM，我们仍然可以通过添加 `spring-boot-dependencies` 依赖项并将其作用域设置为 `import` 来受益于依赖管理。

```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.1.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接下来，我们可以开始简单地添加 Spring 依赖并利用 Spring Boot 的特性：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

另一方面，如果不使用父 POM，我们将无法享受插件管理的便利。这意味着我们需要显式添加 `spring-boot-maven-plugin`：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```



## 覆盖依赖版本

如果我们想使用与 Boot 管理的版本不同的某个依赖版本，我们需要在 `dependencyManagement` 部分声明它，并且要在 `spring-boot-dependencies` 之前声明：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>3.1.5</version>
        </dependency>
    </dependencies>
    // ...
</dependencyManagement>
```

相比之下，仅在 `dependencyManagement` 标签之外声明依赖的版本将不再生效。