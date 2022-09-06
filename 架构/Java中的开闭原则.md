# Java中的开闭原则

## 1、概述

在本教程中，我们将讨论开放/封闭原则 (OCP) 作为 面向对象编程[的 SOLID 原则之一。](https://www.baeldung.com/solid-principles)

总的来说，我们将详细介绍这个原则是什么以及在设计我们的软件时如何实现它。

## 2、开闭原则

**顾名思义，这个原则表明软件实体应该对扩展开放，但对修改关闭。**因此，当业务需求发生变化时，实体可以扩展，但不能修改。

对于下图，**我们将重点关注接口如何成为遵循 OCP 的一种方式。**

### 2.1 不合规

**假设我们正在构建一个计算器应用程序，该应用程序可能具有多种操作，例如加法和减法。**

首先，我们将定义一个顶级接口*——CalculatorOperation*：

```java
public interface CalculatorOperation {}
```

**让我们定义一个Addition类，它将添加两个数字并实现计算器操作：**

```java
public class Addition implements CalculatorOperation {
    private double left;
    private double right;
    private double result = 0.0;

    public Addition(double left, double right) {
        this.left = left;
        this.right = right;
    }

    // getters and setters

}
```

到目前为止，我们只有一个类*Addition，*所以我们需要定义另一个名为*Subtraction*的类：

```java
public class Subtraction implements CalculatorOperation {
    private double left;
    private double right;
    private double result = 0.0;

    public Subtraction(double left, double right) {
        this.left = left;
        this.right = right;
    }

    // getters and setters
}
```

**现在让我们定义我们的主类，它将执行我们的计算器操作：** 

```java
public class Calculator {

    public void calculate(CalculatorOperation operation) {
        if (operation == null) {
            throw new InvalidParameterException("Can not perform operation");
        }

        if (operation instanceof Addition) {
            Addition addition = (Addition) operation;
            addition.setResult(addition.getLeft() + addition.getRight());
        } else if (operation instanceof Subtraction) {
            Subtraction subtraction = (Subtraction) operation;
            subtraction.setResult(subtraction.getLeft() - subtraction.getRight());
        }
    }
}
```

**尽管这看起来不错，但这并不是 OCP 的一个很好的例子。**当增加乘法或除法功能的新需求出现时，我们除了更改*Calculator*类的*计算*方法外别无他法。

**因此，我们可以说此代码不符合 OCP。**



### 2.2 符合 OCP 标准

正如我们所见，我们的计算器应用程序还不符合 OCP。*计算*方法中的代码将随着每个传入的新操作支持请求而改变。所以，我们需要把这段代码提取出来，放到一个抽象层。

一种解决方案是将每个操作委托给它们各自的类：

```java
public interface CalculatorOperation {
    void perform();
}
```

**因此，Addition类可以实现两个数字相加的逻辑：**

```java
public class Addition implements CalculatorOperation {
    private double left;
    private double right;
    private double result;

    // constructor, getters and setters

    @Override
    public void perform() {
        result = left + right;
    }
}
```

同样，更新后的*减法*类将具有类似的逻辑。*和加**减法*类似，作为一个新的变更请求，我们可以实现*除法*逻辑：

```java
public class Division implements CalculatorOperation {
    private double left;
    private double right;
    private double result;

    // constructor, getters and setters
    @Override
    public void perform() {
        if (right != 0) {
            result = left / right;
        }
    }
}
```

**最后，我们的Calculator类不需要实现新逻辑，因为我们引入了新的运算符：**

```java
public class Calculator {

    public void calculate(CalculatorOperation operation) {
        if (operation == null) {
            throw new InvalidParameterException("Cannot perform operation");
        }
        operation.perform();
    }
}
```

这样，该类对修改*关闭*，但对扩展*开放。*

## 3、结论

在本教程中，我们了解了 OCP 的定义，然后详细说明了该定义。然后，我们看到了一个简单的计算器应用程序示例，该应用程序的设计存在缺陷。最后，我们通过使其符合 OCP 来使设计变得更好。







