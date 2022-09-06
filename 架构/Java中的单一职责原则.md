# Java中的单一职责原则

## 1、概述

在本教程中，我们将讨论单一职责原则，它是面向对象编程[的 SOLID 原则之一。](https://www.baeldung.com/solid-principles)

总的来说，我们将深入探讨这个原则是什么以及在设计我们的软件时如何实现它。此外，我们将解释这一原则何时会产生误导。

SRP = 单一职责原则

## 2、单一职责原则

顾名思义，这个原则表明**每个类都应该有** **一个职责，一个目的**。这意味着一个类只会做一项工作，这导致我们得出结论，它应该**只有一个改变的理由**。

我们不希望对象知道太多并且具有不相关的行为。这些类更难维护。例如，如果我们有一个类由于不同的原因发生了很多变化，那么这个类应该被分解成更多的类，每个类处理一个关注点。当然，**如果发生错误，将更容易找到。**

让我们考虑一个包含以某种方式更改文本的代码的类。这个类的唯一工作应该是**处理文本**。

```java
public class TextManipulator {
    private String text;

    public TextManipulator(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public void appendText(String newText) {
        text = text.concat(newText);
    }
    
    public String findWordAndReplace(String word, String replacementWord) {
        if (text.contains(word)) {
            text = text.replace(word, replacementWord);
        }
        return text;
    }
    
    public String findWordAndDelete(String word) {
        if (text.contains(word)) {
            text = text.replace(word, "");
        }
        return text;
    }

    public void printText() {
        System.out.println(textManipulator.getText());
    }
}
```

尽管这看起来不错，但这并不是 SRP 的一个很好的例子。这里我们有**两个** **职责**：**操作和打印文本**。

在此类中使用打印出文本的方法违反了单一职责原则。为此，我们应该创建另一个类，它只处理打印文本：

```java
public class TextPrinter {
    TextManipulator textManipulator;

    public TextPrinter(TextManipulator textManipulator) {
        this.textManipulator = textManipulator;
    }

    public void printText() {
        System.out.println(textManipulator.getText());
    }

    public void printOutEachWordOfText() {
        System.out.println(Arrays.toString(textManipulator.getText().split(" ")));
    }

    public void printRangeOfCharacters(int startingIndex, int endIndex) {
        System.out.println(textManipulator.getText().substring(startingIndex, endIndex));
    }
}
```

现在，在这个类中，我们可以根据需要为打印文本的多种变体创建方法，因为这是它的工作。



## 3、这个原则怎么会有误导性呢？

**在我们的软件中实现 SRP 的诀窍是知道**每个类的责任。

但是，**每个开发人员对类目的都有自己的看法**，这使事情变得棘手。由于我们没有关于如何实施这一原则的严格说明，因此我们对责任的解释留有余地。

这意味着有时只有我们，作为我们应用程序的设计者，才能决定某些东西是否在类的范围内。

在按照 SRP 原则编写类时，我们要考虑问题域、业务需求和应用架构。这是非常主观的，这使得执行这个原则比看起来更难。它不会像我们在本教程中的示例那么简单。

这将我们引向下一点。



## 4、凝聚

遵循 SRP 原则，我们的类将遵循一种功能。他们的方法和数据将关注一个明确的目的。这意味着**高内聚性**和**鲁棒性，它们共同减少了错误**。

在基于 SRP 原则设计软件时，凝聚是必不可少的，因为它可以帮助我们找到类的单一职责。这个概念还可以帮助我们找到具有多个职责的类。

让我们回到我们的*TextManipulator*类方法：

```java
...

public void appendText(String newText) {
    text = text.concat(newText);
}

public String findWordAndReplace(String word, String replacementWord) {
    if (text.contains(word)) {
        text = text.replace(word, replacementWord);
    }
    return text;
}

public String findWordAndDelete(String word) {
    if (text.contains(word)) {
        text = text.replace(word, "");
    }
    return text;
}

...
```

在这里，我们清楚地表示了这个类的作用：文本操作。

但是，如果我们不考虑凝聚力，也没有明确定义这个类的职责是什么，我们可以说编写和更新文本是两个不同的独立工作。由这个想法引导，我们可以得出结论，这些应该是两个独立的类：*WriteText*和*UpdateText*。

实际上，我们会得到**两个紧耦合和松散内聚的类**，它们几乎总是应该一起使用。这三种方法可能**执行不同的操作，但它们本质上只有一个目的**：文本操作。关键是不要想太多。

LCOM 是有助于在方法中实现高内聚的工具之一。本质上，**LCOM 测量类组件之间的连接以及它们之间的关系。**

Martin Hitz和Behzad Montazeri介绍了LCOM4，Sonarqube曾一度对其进行计量，但后来被废弃。



## 5、结论

尽管该原理的名称是不言自明的，但我们可以看到错误地实施是多么容易。确保在开发项目时区分每个类的职责，并特别注意凝聚力。



