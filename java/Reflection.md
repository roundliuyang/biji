



**Java 反射**提供了检查和修改应用程序运行时行为的能力。Java中的反射是Java核心的前沿课题之一。使用 java 反射，我们可以在运行时检查类、[接口](https://www.journaldev.com/1601/interface-in-java)、[枚举](https://www.journaldev.com/716/java-enum)，获取它们的结构、方法和字段信息，即使类在编译时不可访问。我们还可以使用反射来实例化一个对象，调用它的方法，更改字段值。

## Java 反射

Java 中的反射是一个非常强大的概念，在普通编程中用处不大，但它是大多数 Java、J2EE 框架的支柱。一些使用java反射的框架是：

1. **JUnit** – 使用反射解析@Test 注解来获取测试方法，然后调用它。
2. **Spring** – 依赖注入，在[Spring Dependency Injection](https://www.journaldev.com/2410/spring-dependency-injection)阅读更多
3. **Tomcat** Web 容器通过解析它们的 web.xml 文件和请求 URI 将请求转发到正确的模块。
4. **Eclipse**自动完成方法名称
5. **Struts**
6. **Hibernate**

这个列表是无穷无尽的，它们都使用 java 反射，因为所有这些框架都不知道和访问用户定义的类、接口、它们的方法等。



由于以下缺点，我们不应该在正常编程中使用反射，因为我们已经可以访问类和接口。

- **性能差**——由于java反射动态解析类型，它涉及扫描类路径以查找要加载的类等处理，导致性能下降。
- **安全限制**– 反射需要运行时权限，在安全管理器下运行的系统可能无法使用这些权限。由于安全管理器，这可能会导致您的应用程序在运行时失败。
- **安全问题**——使用反射我们可以访问我们不应该访问的部分代码，例如我们可以访问类的私有字段并更改它的值。这可能是一个严重的安全威胁，并导致您的应用程序行为异常。
- **高维护**– 反射代码难以理解和调试，并且在编译时也无法发现代码的任何问题，因为类可能不可用，从而使其灵活性降低且难以维护。

## 类的 Java 反射

在 java 中，每个对象要么是原始类型，要么是引用。所有的类、枚举、数组都是引用类型并继承自`java.lang.Object`. 原始类型是——boolean、byte、short、int、long、char、float 和 double。



java .lang.Class是所有反射操作的入口点，java.lang.Class是所有反射操作的入口点。对于每一种类型的对象，JVM都会实例化一个不可变的java.lang.Class实例，它提供了检查对象的运行时属性和创建新对象、调用其方法和获取/设置对象字段的方法。

在本节中，我们将研究 Class 的重要方法，为方便起见，我将创建一些具有继承层次结构的类和接口。

```java
package com.journaldev.reflection;

public interface BaseInterface {
	
	public int interfaceInt=0;
	
	void method1();
	
	int method2(String str);
}
```

```java
package com.journaldev.reflection;

public class BaseClass {

	public int baseInt;
	
	private static void method3(){
		System.out.println("Method3");
	}
	
	public int method4(){
		System.out.println("Method4");
		return 0;
	}
	
	public static int method5(){
		System.out.println("Method5");
		return 0;
	}
	
	void method6(){
		System.out.println("Method6");
	}
	
	// inner public class
	public class BaseClassInnerClass{}
		
	//member public enum
	public enum BaseClassMemberEnum{}
}
```

```java
package com.journaldev.reflection;

@Deprecated
public class ConcreteClass extends BaseClass implements BaseInterface {

	public int publicInt;
	private String privateString="private string";
	protected boolean protectedBoolean;
	Object defaultObject;
	
	public ConcreteClass(int i){
		this.publicInt=i;
	}

	@Override
	public void method1() {
		System.out.println("Method1 impl.");
	}

	@Override
	public int method2(String str) {
		System.out.println("Method2 impl.");
		return 0;
	}
	
	@Override
	public int method4(){
		System.out.println("Method4 overriden.");
		return 0;
	}
	
	public int method5(int i){
		System.out.println("Method4 overriden.");
		return 0;
	}
	
	// inner classes
	public class ConcreteClassPublicClass{}
	private class ConcreteClassPrivateClass{}
	protected class ConcreteClassProtectedClass{}
	class ConcreteClassDefaultClass{}
	
	//member enum
	enum ConcreteClassDefaultEnum{}
	public enum ConcreteClassPublicEnum{}
	
	//member interface
	public interface ConcreteClassPublicInterface{}

}
```

让我们看看一些重要的类反射方法。

### 获取类对象

我们可以使用三种方法获得对象的Class，即通过静态变量Class、使用对象的getClass()方法和java.lang.Class.forName(String fullyClassifiedClassName)。对于原始类型和数组，我们可以使用静态变量类。b包装类提供另一个静态变量TYPE来获取类。

```java
// Get Class using reflection
Class<?> concreteClass = ConcreteClass.class;
concreteClass = new ConcreteClass(5).getClass();
try {
	// below method is used most of the times in frameworks like JUnit
	//Spring dependency injection, Tomcat web container
	//Eclipse auto completion of method names, hibernate, Struts2 etc.
	//because ConcreteClass is not available at compile time
	concreteClass = Class.forName("com.journaldev.reflection.ConcreteClass");
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}

System.out.println(concreteClass.getCanonicalName()); // prints com.journaldev.reflection.ConcreteClass

//for primitive types, wrapper classes and arrays
Class<?> booleanClass = boolean.class;
System.out.println(booleanClass.getCanonicalName()); // prints boolean

Class<?> cDouble = Double.TYPE;
System.out.println(cDouble.getCanonicalName()); // prints double

// Class<?> cDoubleArray = Class.forName("[D");
// System.out.println(cDoubleArray.getCanonicalName()); //prints double[]

Class<?> twoDStringArray = String[][].class;
System.out.println(twoDStringArray.getCanonicalName()); // prints java.lang.String[][]
```

getCanonicalName()返回底层类的标准名称。注意，java.lang.Class使用了泛型，它帮助框架确保检索到的类是框架基类的子类。查看Java泛型教程，了解泛型和它的通配符。



### 获取Super Class

在一个Class对象上的getSuperclass()方法返回该类的超类。如果这个Class代表Object类，一个接口，一个原始类型，或者是void，那么将返回null。如果这个对象代表一个数组类，那么将返回代表Object类的Class对象。

```java
Class<?> superClass = Class.forName("com.journaldev.reflection.ConcreteClass").getSuperclass();
System.out.println(superClass); // prints "class com.journaldev.reflection.BaseClass"
System.out.println(Object.class.getSuperclass()); // prints "null"
System.out.println(String[][].class.getSuperclass());// prints "class java.lang.Object"
```



类代表对象的getClasses()方法返回一个数组，该数组包含代表所有公有类、接口和枚举的类对象，它们是该类对象所代表的类的成员。这包括从超类继承的公有类和接口成员以及由该类声明的公有类和接口成员。如果这个Class对象没有公共成员类或接口，或者这个Class对象代表一个原始类型、一个数组类或void，这个方法返回一个长度为0的数组



### 获取Declared Classes

getDeclaredClasses()方法返回一个Class对象数组，该数组反映了作为该Class对象所代表的类的成员所声明的所有类和接口。返回的数组不包括在继承的类和接口中声明的类。(注意：则是获取所声明的所有类和接口，不是属性)

```java

//getting all of the classes, interfaces, and enums that are explicitly declared in ConcreteClass
Class<?>[] explicitClasses = Class.forName("com.journaldev.reflection.ConcreteClass").getDeclaredClasses();
//prints [class com.journaldev.reflection.ConcreteClass$ConcreteClassDefaultClass, 
//class com.journaldev.reflection.ConcreteClass$ConcreteClassDefaultEnum, 
//class com.journaldev.reflection.ConcreteClass$ConcreteClassPrivateClass, 
//class com.journaldev.reflection.ConcreteClass$ConcreteClassProtectedClass, 
//class com.journaldev.reflection.ConcreteClass$ConcreteClassPublicClass, 
//class com.journaldev.reflection.ConcreteClass$ConcreteClassPublicEnum, 
//interface com.journaldev.reflection.ConcreteClass$ConcreteClassPublicInterface]
System.out.println(Arrays.toString(explicitClasses));
```



### 获取 Declaring Class

getDeclaringClass()方法返回代表它所声明的类的Class对象。

```java
Class<?> innerClass = Class.forName("com.journaldev.reflection.ConcreteClass$ConcreteClassDefaultClass");
//prints com.journaldev.reflection.ConcreteClass
System.out.println(innerClass.getDeclaringClass().getCanonicalName());
System.out.println(innerClass.getEnclosingClass().getCanonicalName());
```



### 获取包名

getPackage() 方法返回此类的包。此类的类加载器用于查找包。我们可以调用 Package 的 getName() 方法来获取包的名称。

```java

//prints "com.journaldev.reflection"
System.out.println(Class.forName("com.journaldev.reflection.BaseInterface").getPackage().getName());
```



### 获取类修饰符

getModifiers() 方法返回类修饰符的 int 表示，我们可以使用 java.lang.reflect.Modifier.toString() 方法以源代码中使用的字符串格式获取它。

```java
System.out.println(Modifier.toString(concreteClass.getModifiers())); //prints "public"
//prints "public abstract interface"
System.out.println(Modifier.toString(Class.forName("com.journaldev.reflection.BaseInterface").getModifiers())); 
```



### 获取类型参数

`getTypeParameters()`如果有任何与类关联的类型参数，则返回 TypeVariable 数组。类型参数的返回顺序与声明的顺序相同。

```java
//Get Type parameters (generics)
TypeVariable<?>[] typeParameters = Class.forName("java.util.HashMap").getTypeParameters();
for(TypeVariable<?> t : typeParameters)
System.out.print(t.getName()+",");
```



### 获取 Implemented Interfaces 

`getGenericInterfaces()` method returns the array of interfaces implemented by the class with generic type information. We can also use `getInterfaces()` to get the class representation of all the implemented interfaces.

```java
Type[] interfaces = Class.forName("java.util.HashMap").getGenericInterfaces();
//prints "[java.util.Map<K, V>, interface java.lang.Cloneable, interface java.io.Serializable]"
System.out.println(Arrays.toString(interfaces));
//prints "[interface java.util.Map, interface java.lang.Cloneable, interface java.io.Serializable]"
System.out.println(Arrays.toString(Class.forName("java.util.HashMap").getInterfaces()));
```



















