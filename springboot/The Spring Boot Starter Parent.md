# The Spring Boot Starter Parent

## 介绍

在本教程中，我们将学习关于 `spring-boot-starter-parent` 的内容。我们将讨论如何利用它来更好地进行依赖管理、为插件提供默认配置，并快速构建我们的 Spring Boot 应用程序。

我们还将了解如何覆盖 `starter-parent` 提供的现有依赖项和属性的版本。



##  Spring Boot Starter Parent

`spring-boot-starter-parent` 项目是一个特殊的启动器项目，提供了我们应用程序的默认配置和一个完整的依赖树，用于快速构建 Spring Boot 项目。它还为 Maven 插件提供了默认配置，例如 `maven-failsafe-plugin`、`maven-jar-plugin`、`maven-surefire-plugin` 和 `maven-war-plugin`。

除此之外，它还继承了 `spring-boot-dependencies` 的依赖管理，而 `spring-boot-dependencies` 是 `spring-boot-starter-parent` 的父级。

我们可以通过在项目的 `pom.xml` 中将其作为父级来开始使用它：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.5</version>
</parent>
```

我们始终可以从 Maven Central 获取 `spring-boot-starter-parent` 的最新版本。



## 管理依赖项

一旦我们在项目中声明了 `starter parent`，就可以直接在 `dependencies` 标签中声明依赖项，而不需要手动指定版本。Maven 会根据 `parent` 标签中定义的 `starter parent` 版本来下载相应的 JAR 文件。

例如，如果我们正在构建一个 Web 项目，我们可以直接添加 `spring-boot-starter-web`，而无需指定版本：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```



## `dependencyManagement` 标签

如果我们想要使用 `starter parent` 提供的依赖项但采用不同的版本，可以在 `dependencyManagement` 部分**显式**声明该依赖及其版本：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>3.1.5</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```



## Properties

要更改 `starter parent` 中定义的任何属性值，我们可以在 `properties` 部分重新声明它。

`spring-boot-starter-parent` 通过其父级 `spring-boot-dependencies` 使用属性来配置所有依赖项的版本、Java 版本和 Maven 插件版本。因此，我们只需更改相应的属性值，就能轻松控制这些配置。

如果我们想修改从 `starter parent` 获取的某个依赖项的版本，可以在 `dependency` 标签中添加该依赖，并直接配置其属性：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>  <!-- Spring Boot 版本 -->
        <relativePath/> <!-- 指定默认的 Spring Boot 依赖管理 -->
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo-project</artifactId>
    <version>1.0.0</version>

    <properties>
        <junit.version>4.11</junit.version>  <!-- 修改 JUnit 版本 -->
    </properties>

    <dependencies>
        <!-- 直接引入 JUnit 依赖，Spring Boot 依赖管理会自动匹配 properties 中的版本 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
</project>
```



## Other Property Overrides

我们还可以使用 `properties` 来配置其他内容，例如 **管理插件版本**，或者一些**基础配置**，比如 **Java 版本** 和 **源码编码**。
 只需要重新声明该属性并赋予新的值即可。



### **示例：修改 Java 版本**

如果我们想要更改默认的 Java 版本，可以在 `pom.xml` 的 `properties` 中声明 `java.version`，例如：

```
<properties>
    <java.version>17</java.version>  <!-- 修改 Java 版本为 17 -->
</properties>
```

### **完整示例**

```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo-project</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>  <!-- 设定 Java 版本 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  <!-- 设定源码编码 -->
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```



### **说明**

1. **修改 Java 版本**：通过 `java.version` 设定 Java 版本，Spring Boot 会自动应用这个配置。
2. **设置源码编码**：使用 `project.build.sourceEncoding=UTF-8` 以确保源码文件的正确编码。
3. **编译器目标版本**：`maven.compiler.source` 和 `maven.compiler.target` 继承 `java.version`，确保 Maven 也使用相同的 Java 版本。
4. **自动管理依赖版本**：Spring Boot 的 `starter-parent` 会自动管理 `spring-boot-starter-web` 的版本，无需手动指定。

这使得项目更加可维护，同时确保所有团队成员使用相同的 Java 版本和编码规范。



## 不使用 Spring Boot Starter Parent 的 Spring Boot 项目

有时候，我们可能有**自定义的 Maven 父项目**，或者**希望手动声明所有 Maven 配置**。

在这种情况下，我们可以选择**不使用 `spring-boot-starter-parent`**，但仍然可以通过**引入 `spring-boot-dependencies` 作为 `import` 依赖**来**复用**其**依赖管理机制**。

下面是一个简单示例，展示如何**使用其他父项目**，而不是 `spring-boot-starter-parent`：

```xml
<parent>
    <groupId>com.baeldung</groupId>
    <artifactId>spring-boot-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</parent>
```

在这里，我们使用了 **`parent-modules`** 作为父依赖，而不是 `spring-boot-starter-parent`。

在这种情况下，我们仍然可以通过**将 `spring-boot-dependencies` 以 `import` 作用域** 和 **`pom` 类型** 添加到 `pom.xml` 中，从而享受相同的依赖管理机制。

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

此外，我们只需在 `dependencies` 中声明任何依赖，就像我们之前的示例一样。对于这些依赖项，**不需要指定版本号**。



## 总结

在本文中，我们概述了 `spring-boot-starter-parent` 以及将其作为父项目添加到任何子项目中的好处。

接下来，我们学习了如何管理依赖项。我们可以通过 `dependencyManagement` 或通过 `properties` 来覆盖依赖项。