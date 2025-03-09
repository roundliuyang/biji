## dependencyManagement vs dependencies Tags

refer:https://www.baeldung.com/maven-dependencymanagement-vs-dependencies-tags

## 概述

在本教程中，我们将回顾两个重要的[Maven](https://www.baeldung.com/maven-guide)标签 — *dependencyManagement* 和 *dependency*。

**这些功能对于多模块项目特别有用。**

我们将回顾这两个标签的相同点和不同点，并研究开发人员在使用它们时可能犯的一些常见错误，这些错误可能会导致混淆。

## 使用方法

通常，我们使用 `dependencyManagement` 标签来避免在 `dependencies` 标签中重复定义 `version` 和 `scope` 标签。这样，所需的依赖可以在一个中央 POM 文件中统一声明。



### dependencyManagement

这个标签包含一个 `dependencies` 标签，而 `dependencies` 标签本身可能包含多个 `dependency` 标签。
 每个 `dependency` 都应至少包含三个主要标签：`groupId`、`artifactId` 和 `version`。
 让我们来看一个示例：

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

上面的代码只是声明了新的 `commons-lang3` 组件，但并没有真正将其添加到项目的依赖资源列表中。



### *dependencies*

此标签包含一个 `dependency` 标签列表。
 每个 `dependency` 至少应包含两个主要标签：`groupId` 和 `artifactId`。

让我们来看一个简单的示例：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

如果我们在 POM 文件中使用了 `dependencyManagement` 标签，那么 `version` 和 `scope` 标签可以被隐式继承。

```java
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies>
```



## 相似之处

这两个标签的作用都是声明某些第三方或子模块的依赖关系，它们是互补的。

实际上，我们通常会先定义 `dependencyManagement` 标签，然后再定义 `dependencies` 标签。这是为了在 POM 文件中声明依赖项。
 这种声明只是一个公告，并不会真正将依赖添加到项目中。

让我们来看一个添加 JUnit 库依赖的示例：

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

正如我们在上面的代码中看到的，`dependencyManagement` 标签本身包含了一个 `dependencies` 标签。

现在，让我们看看代码的另一部分，它将实际的依赖添加到项目中：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

因此，当前的标签与之前的标签非常相似，它们都用于定义依赖列表。 当然，它们之间存在一些细微的区别，我们很快会进行介绍。

在这两个代码片段中，`groupId` 和 `artifactId` 标签是相同的，并且它们之间存在重要的关联：它们都指向同一个 artifact。

正如我们所见，在后面的依赖标签中并没有 `version` 标签。 令人惊讶的是，这仍然是合法的语法，并且可以被解析和编译，而不会出现任何问题。 原因很容易猜到：它会使用 `dependencyManagement` 中声明的版本。



## 不同之处

### 结构差异

正如我们之前提到的，这两个标签之间的主要结构差异在于**继承逻辑**。
 我们在 `dependencyManagement` 标签中定义 `version`，然后在后续的 `dependencies` 标签中使用该版本时，就可以**省略显式指定 `version`**。



### 行为差异

`dependencyManagement` **只是一个声明**，它并不会真正添加依赖。在 `dependencyManagement` 中声明的依赖，必须**在后续的 `dependencies` 标签中使用**，否则不会生效。真正让依赖生效的**是 `dependencies` 标签**。 在上面的示例中，`dependencyManagement` **不会**将 `junit` 库添加到任何作用域，它只是为未来的 `dependencies` 标签提供一个**版本声明**。



## 现实世界的示例

几乎所有基于 Maven 的开源项目都使用这种机制。

让我们来看一个来自 Maven 项目的示例。我们可以看到 `hamcrest-core` 依赖项，它存在于 Maven 项目中。它首先在 `dependencyManagement` 标签中声明，然后通过主 `dependencies` 标签导入：

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



## 常见使用场景

这个功能的一个非常常见的使用场景是多模块项目。

假设我们有一个大型项目，由不同的模块组成。每个模块都有自己的依赖项，且每个开发人员可能使用不同版本的依赖项。这可能会导致不同的组件版本相互交织，进而引发难以解决的冲突。

解决这个问题的简单方法是，在根 POM 文件（通常称为“父级”）中使用 `dependencyManagement` 标签，然后在子模块的 POM 文件（即子模块）中使用依赖项，甚至在父模块本身（如果适用）中也可以使用。

如果我们只有一个模块，是否有必要使用这个功能呢？虽然在多模块环境中非常有用，但即便在单模块项目中，遵循这种最佳实践也可以作为一种经验法则。这不仅有助于提升项目的可读性，还使其为未来扩展为多模块项目做好准备。

## 常见错误

一个常见的错误是仅在 `dependencyManagement` 部分定义依赖，而没有将其包含在 `dependencies` 标签中。在这种情况下，根据所提到的作用域，我们可能会遇到编译或运行时错误。

让我们来看一个示例：

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

假设上述的 POM 代码片段。然后假设我们将在一个子模块的源文件中使用这个库：

```java
import org.apache.commons.lang3.StringUtils;

public class Main {

    public static void main(String[] args) {
        StringUtils.isBlank(" ");
    }
}
```

这段代码将无法编译，因为缺少该库。编译器会抱怨出现以下错误：

```properties
[ERROR] Failed to execute goal compile (default-compile) on project sample-module: Compilation failure
[ERROR] ~/sample-module/src/main/java/com/baeldung/Main.java:[3,32] package org.apache.commons.lang3 does not exist
```

为了避免这个错误，只需在子模块的 POM 文件中添加以下 `dependencies` 标签：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
</dependencies>
```

