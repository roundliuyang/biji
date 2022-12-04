# Open/Closed Principle in Java



## 1. Overview

In this tutorial, we'll discuss the Open/Closed Principle (OCP) as one of [the SOLID principles](https://www.baeldung.com/solid-principles) of object-oriented programming.

Overall, we'll go into detail on what this principle is and how to implement it when designing our software.



## 2. Open/Closed Principle



**As the name suggests, this principle states that software entities should be open for extension, but closed for modification.** As a result, when the business requirements change then the entity can be extended, but not modified.

For the illustration below, **we'll focus on how interfaces are one way to follow OCP.**



### 2.1. Non-Compliant



**Let's consider we're building a calculator app that might have several operations, such as addition and subtraction.**

First of all, we'll define a top-level interface – *CalculatorOperation*:

```java
public interface CalculatorOperation {}
```

**Let's define an Addition class, which would add two numbers and implement the CalculatorOperation:**

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

As of now, we only have one class *Addition,* so we need to define another class named *Subtraction*:

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

**Let's now define our main class, which will perform our calculator operations:** 

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

**Although this may seem fine, it's not a good example of the OCP.** When a new requirement of adding multiplication or divide functionality comes in, we've no way besides changing the *calculate* method of the *Calculator* class.

**Hence, we can say this code is not OCP compliant.**



### 2.2. OCP Compliant

As we've seen our calculator app is not yet OCP compliant. The code in the *calculate* method will change with every incoming new operation support request. So, we need to extract this code and put it in an abstraction layer.

One solution is to delegate each operation into their respective class:

```java

public interface CalculatorOperation {
    void perform();
}
```

**As a result, the Addition class could implement the logic of adding two numbers:**

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

Likewise, an updated *Subtraction* class would have similar logic. And similarly to *Addition* and *Subtraction*, as a new change request, we could implement the *division* logic:

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

**And finally, our Calculator class doesn't need to implement new logic as we introduce new operators:**

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

That way the class is *closed* for modification but *open* for an extension.



## 3. Conclusion

In this tutorial, we've learned what is OCP by definition, then elaborated on that definition. We then saw an example of a simple calculator application that was flawed in its design. Lastly, we made the design good by making it adhere to the OCP.

As always, the code is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/patterns-modules/solid).

















