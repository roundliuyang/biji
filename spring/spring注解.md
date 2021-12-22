## @Configuration

**Spring @Configuration 注解**有助于**基于 Spring 注解的配置**。**@Configuration**注解表示一个类声明了一个或多个`@Bean`方法，并且可以由[Spring 容器](https://howtodoinjava.com/spring-core/different-spring-ioc-containers/)处理以在运行时为这些 bean 生成 bean 定义和服务请求。

从 spring 2 开始，我们将 bean 配置写入 xml 文件。但是 Spring 3 提供了将 bean 定义移出 xml 文件的自由。我们可以在 Java 文件本身中给出 bean 定义。这称为**Spring Java Config**功能（使用[`@Configuration`](https://docs.spring.io/spring-framework/docs/3.1.x/javadoc-api/org/springframework/context/annotation/Configuration.html)注解）。

### Spring@Configuration注解用法

`@Configuration`在任何类的顶部使用注解来声明该类提供一个或多个**@Bean**方法，并且可以由 Spring 容器处理以在运行时为这些 bean 生成 bean 定义和服务请求。

```java
应用程序配置文件
@Configuration
public class AppConfig {
 
    @Bean(name="demoService")
    public DemoClass service() 
    {
        
    }
}
```

#### 带有@Configuration注解的Spring配置类

```java
@Configuration
public class ApplicationConfiguration {
 
    @Bean(name="demoService")
    public DemoManager helloWorld() 
    {
        return new DemoManagerImpl();
    }
}
```

#### 演示

```java
public class VerifySpringCoreFeature
{
    public static void main(String[] args)
    {
        ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfiguration.class);
 
        DemoManager  obj = (DemoManager) context.getBean("demoService");
 
        System.out.println( obj.getServiceName() );
    }
}
```



## @Autowired, @Resource and @Inject

### 概述

三个注解中有两个属于 Java 扩展包：javax.annotation.Resource 和 javax.inject.Inject。 @Autowired 注解属于 org.springframework.beans.factory.annotation 包。

这些注释中的每一个都可以通过字段注入或 setter 注入来解决依赖关系。我们将使用一个简化但实际的示例来演示三个注解之间的区别，但是根据每个注释所采用的执行路径来演示三个注释之间的区别的实际示例。

这些例子将集中在如何在集成测试中使用三个注入注解。测试所需的依赖性可以是一个任意的文件或一个任意的类。

### @Resource

@Resource 注释是 JSR-250 注释集合的一部分，并与 Jakarta EE 打包在一起。此注解具有以下执行路径，按优先级列出：

按名称匹配

按类型匹配

按 Qualifier匹配

这些执行路径适用于 setter 和字段注入

#### Field 注入

我们可以通过给实例变量加上@Resource注解，通过字段注入来解决依赖关系。

##### 按名称匹配

我们将使用以下集成测试来演示按名称匹配字段注入：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class FieldResourceInjectionIntegrationTest {

    @Resource(name="namedFile")
    private File defaultFile;

    @Test
    public void givenResourceAnnotation_WhenOnField_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

让我们看一下代码。在FieldResourceInjectionTest集成测试的第7行，我们通过将Bean名称作为属性值传递给@Resource注解来解决依赖关系。

```java
@Resource(name="namedFile")
private File defaultFile;
```

此配置将使用按名称匹配的执行路径解析依赖项。我们必须在 ApplicationContextTestResourceNameType 应用程序上下文中定义 bean namedFile。

请注意，Bean的id和相应的引用属性值必须匹配。

```java
@Configuration
public class ApplicationContextTestResourceNameType {

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

如果我们未能在应用上下文中定义Bean，将导致抛出org.springframework.beans.factory.NoSuchBeanDefinitionException。我们可以通过改变ApplicationContextTestResourceNameType应用上下文中传入@Bean注解的属性值，或改变FieldResourceInjectionTest集成测试中传入@Resource注解的属性值，来证明这点。

##### 按类型匹配

为了演示按类型匹配的执行路径，我们只需删除 FieldResourceInjectionTest 集成测试第 7 行的属性值：

```java
@Resource
private File defaultFile;
```

然后我们再次运行测试。

测试仍然会通过，因为如果 @Resource 注释没有接收 bean 名称作为属性值，Spring Framework 将继续下一级优先级，按类型匹配，以尝试解决依赖关系。

>@Resource装配规则
>
>- @Resource设置name属性，则按name在IoC容器中将bean注入
>- @Resource未设置name属性
>  - 2.1 以属性名作为bean name在IoC容器中匹配bean,如有匹配则注入
>  - 2.2 按属性名未匹配，则按类型进行匹配，同@Autowired,需加入@Primary解决类型冲突

##### 按 Qualifier 匹配

为了演示 match-by-qualifier (按限定符匹配)执行路径，将修改集成测试场景，以便在 ApplicationContextTestResourceQualifier 应用程序上下文中定义两个 bean

```java
@Configuration
public class ApplicationContextTestResourceQualifier {

    @Bean(name="defaultFile")
    public File defaultFile() {
        File defaultFile = new File("defaultFile.txt");
        return defaultFile;
    }

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

我们将使用 QualifierResourceInjectionTest 集成测试来演示按限定符匹配的依赖项解析。在这种情况下，需要将特定的 bean 依赖项注入到每个引用变量中：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class QualifierResourceInjectionIntegrationTest {

    @Resource
    private File dependency1;
	
    @Resource
    private File dependency2;

    @Test
    public void givenResourceAnnotation_WhenField_ThenDependency1Valid(){
        assertNotNull(dependency1);
        assertEquals("defaultFile.txt", dependency1.getName());
    }

    @Test
    public void givenResourceQualifier_WhenField_ThenDependency2Valid(){
        assertNotNull(dependency2);
        assertEquals("namedFile.txt", dependency2.getName());
    }
}
```

当我们运行集成测试时，将抛出org.springframework.beans.factory.NoUniqueBeanDefinitionException。这将发生，因为应用程序上下文将找到两个File类型的Bean定义，并且不知道哪个Bean应该解决这个依赖关系。

为了解决这个问题，我们需要参考QualifierResourceInjectionTest集成测试的第7至第10行。

```java
@Resource
private File dependency1;

@Resource
private File dependency2;
```

我们必须添加以下代码行：

```java
@Qualifier("defaultFile")

@Qualifier("namedFile")
```

使代码块如下所示

```java
@Resource
@Qualifier("defaultFile")
private File dependency1;

@Resource
@Qualifier("namedFile")
private File dependency2;
```

当我们再次运行集成测试时，它应该通过。我们的测试表明，即使我们在一个应用上下文中定义了多个Bean，我们也可以使用@Qualifier注解来清除任何混淆，允许我们将特定的依赖注入到一个类中。

####  Setter 注入

在字段上注入依赖关系时的执行路径也适用于基于setter的注入。

##### 按名称匹配

唯一的区别是 MethodResourceInjectionTest 集成测试有一个 setter 方法：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodResourceInjectionIntegrationTest {

    private File defaultFile;

    @Resource(name="namedFile")
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

我们通过注解引用变量对应的setter方法，通过setter注入来解决依赖关系。然后我们将 bean 依赖项的名称作为属性值传递给 @Resource 注解。

```java
private File defaultFile;

@Resource(name="namedFile")
protected void setDefaultFile(File defaultFile) {
    this.defaultFile = defaultFile;
}
```

我们将在此示例中重用 namedFile bean 依赖关系。 bean 名称和相应的属性值必须匹配。

当我们运行集成测试时，它将通过。

为了让我们验证逐名匹配的执行路径解决了依赖关系，我们需要将传递给@Resource注解的属性值改为我们选择的值，然后再次运行测试。这一次，测试将以NoSuchBeanDefinitionException失败。

##### 按类型匹配

为了演示基于 setter 的、按类型匹配的执行，我们将使用 MethodByTypeResourceTest 集成测试：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodByTypeResourceIntegrationTest {

    private File defaultFile;

    @Resource
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenValidDependency(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

当我们运行这个测试时，它将通过。

为了让我们验证match-by-type执行路径解决了File的依赖性，我们需要将defaultFile变量的类类型改为另一种类类型，如String。然后我们可以再次执行MethodByTypeResourceTest集成测试，这次将抛出NoSuchBeanDefinitionException。

这个异常验证了match-by-type确实被用来解决File的依赖性。NoSuchBeanDefinitionException证实了引用变量名不需要与Bean名相匹配。相反，依赖关系的解决取决于 bean 的类类型是否与引用变量的类类型匹配。

##### 按 Qualifier 匹配

我们将使用 MethodByQualifierResourceTest 集成测试来演示 match-by-qualifier 执行路径：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class MethodByQualifierResourceIntegrationTest {

    private File arbDependency;
    private File anotherArbDependency;

    @Test
    public void givenResourceQualifier_WhenSetter_ThenValidDependencies(){
      assertNotNull(arbDependency);
        assertEquals("namedFile.txt", arbDependency.getName());
        assertNotNull(anotherArbDependency);
        assertEquals("defaultFile.txt", anotherArbDependency.getName());
    }

    @Resource
    @Qualifier("namedFile")
    public void setArbDependency(File arbDependency) {
        this.arbDependency = arbDependency;
    }

    @Resource
    @Qualifier("defaultFile")
    public void setAnotherArbDependency(File anotherArbDependency) {
        this.anotherArbDependency = anotherArbDependency;
    }
}
```

们的测试表明，即使我们在一个应用上下文中定义了多个特定类型的Bean实现，我们也可以使用@Qualifier注解和@Resource注解来解决依赖关系。

与基于字段的依赖注入类似，如果我们在一个应用上下文中定义了多个Bean，我们必须使用@Qualifier注解来指定使用哪个Bean来解决依赖关系，否则将抛出NoUniqueBeanDefinitionException。

### @Inject

@Inject 注解属于 JSR-330 注解集合。此注解具有以下执行路径，按优先级列出：

> 按类型匹配
>
> 按 Qualifier 匹配
>
> 按名称匹配

这些执行路径适用于 setter 和字段注入。为了让我们访问 @Inject 注释，我们必须将 javax.inject 库声明为 Gradle 或 Maven 依赖项。

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

#### Field 注入

##### 按类型匹配

我们将修改集成测试的例子，使用另一种类型的依赖，即ArbitraryDependency类。ArbitraryDependency类的依赖仅仅是作为一个简单的依赖，没有进一步的意义。

```java
@Component
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

这是有关的FieldInjectTest集成测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectType.class)
public class FieldInjectIntegrationTest {

    @Inject
    private ArbitraryDependency fieldInjectDependency;

    @Test
    public void givenInjectAnnotation_WhenOnField_ThenValidDependency(){
        assertNotNull(fieldInjectDependency);
        assertEquals("Arbitrary Dependency",
          fieldInjectDependency.toString());
    }
}
```

与首先按名称解析依赖项的@Resource 注解不同，@Inject 注解的默认行为是按类型解析依赖项。

这意味着即使类的引用变量名与 bean 名不同，只要 bean 在应用程序上下文中被定义，该依赖关系仍将被解决。请注意下面的测试中的引用变量名是怎样的。

```java
@Inject
private ArbitraryDependency fieldInjectDependency;
```

与应用程序上下文中配置的 bean 名称不同：

```java
Bean
public ArbitraryDependency injectDependency() {
    ArbitraryDependency injectDependency = new ArbitraryDependency();
    return injectDependency;
}
```

当我们执行测试时，我们能够解决这个依赖关系。

##### 按 Qualifier 匹配

  如果某个类的类型有多个实现，而某个类需要一个特定的Bean，怎么办？让我们修改一下集成测试的例子，使其需要另一个依赖关系。

在此示例中，我们将在按类型匹配示例中使用的 ArbitraryDependency 类进行子类化，以创建 AnotherArbitraryDependency 类：

```java
public class AnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

每个测试案例的目的是确保我们将每个依赖关系正确地注入到每个参考变量中。

```java
@Inject
private ArbitraryDependency defaultDependency;

@Inject
private ArbitraryDependency namedDependency;
```

我们可以使用 FieldQualifierInjectTest 集成测试来演示按限定符的匹配

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectQualifier.class)
public class FieldQualifierInjectIntegrationTest {

    @Inject
    private ArbitraryDependency defaultDependency;

    @Inject
    private ArbitraryDependency namedDependency;

    @Test
    public void givenInjectQualifier_WhenOnField_ThenDefaultFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Arbitrary Dependency",
          defaultDependency.toString());
    }

    @Test
    public void givenInjectQualifier_WhenOnField_ThenNamedFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Another Arbitrary Dependency",
          namedDependency.toString());
    }
}
```

如果我们在应用程序上下文中具有特定类的多个实现，并且 FieldQualifierInjectTest 集成测试尝试以下面列出的方式注入依赖项，则会抛出 NoUniqueBeanDefinitionException 。

```java
@Inject 
private ArbitraryDependency defaultDependency;

@Inject 
private ArbitraryDependency namedDependency;
```

抛出这个异常是Spring框架指出某个类有多种实现，它对使用哪一个感到困惑。为了澄清这个困惑，我们可以去看FieldQualifierInjectTest集成测试的第7和第10行。

```java
@Inject
private ArbitraryDependency defaultDependency;

@Inject
private ArbitraryDependency namedDependency;
```

我们可以将所需的bean名称传递给@Qualifier注解，我们将其与@Inject注解一起使用。这就是现在的代码块的样子。

```java
@Inject
@Qualifier("defaultFile")
private ArbitraryDependency defaultDependency;

@Inject
@Qualifier("namedFile")
private ArbitraryDependency namedDependency;
```

@Qualifier 注解期望在接收 bean 名称时进行严格匹配。我们必须确保Bean名称被正确地传递给Qualifier，否则将抛出NoUniqueBeanDefinitionException。如果我们再次运行该测试，它应该会通过。

##### 按名称匹配

用来演示按名称匹配的FieldByNameInjectTest集成测试与按类型匹配的执行路径相似。唯一的区别是现在我们需要一个特定的bean，而不是一个特定的类型。在这个例子中，我们再次对ArbitraryDependency类进行子类化，产生YetAnotherArbitraryDependency类。

```java
public class YetAnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Yet Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

为了演示按名称匹配的执行路径，我们将使用以下集成测试：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectName.class)
public class FieldByNameInjectIntegrationTest {

    @Inject
    @Named("yetAnotherFieldInjectDependency")
    private ArbitraryDependency yetAnotherFieldInjectDependency;

    @Test
    public void givenInjectQualifier_WhenSetOnField_ThenDependencyValid(){
        assertNotNull(yetAnotherFieldInjectDependency);
        assertEquals("Yet Another Arbitrary Dependency",
          yetAnotherFieldInjectDependency.toString());
    }
}
```

我们列出应用程序上下文：

```java
@Configuration
public class ApplicationContextTestInjectName {

    @Bean
    public ArbitraryDependency yetAnotherFieldInjectDependency() {
        ArbitraryDependency yetAnotherFieldInjectDependency =
          new YetAnotherArbitraryDependency();
        return yetAnotherFieldInjectDependency;
    }
}
```

如果我们运行集成测试，它将通过。

为了验证我们是否通过按名称匹配的执行路径注入了依赖项，我们需要将传递给 @Named 注释的值 YetAnotherFieldInjectDependency 更改为我们选择的另一个名称。当我们再次运行测试时，将抛出 NoSuchBeanDefinitionException。

####  Setter 注入

@Inject 注释的基于 Setter 的注入类似于用于基于 @Resource setter 的注入的方法。我们不是注释引用变量，而是注释相应的 setter 方法。基于字段的依赖注入所遵循的执行路径也适用于基于 setter 的注入。

###  @Autowired

@Autowired注解的行为与@Inject注解相似。唯一的区别是，@Autowired注解是Spring框架的一部分。这个注解的执行路径与@Inject注解相同，按优先级排列。

按类型匹配

按 Qualifier匹配

按名称匹配
这些执行路径适用于 setter 和字段注入。

#### Field 注入

##### 按类型匹配

用于演示@Autowired match-by-type 执行路径的集成测试示例将类似于用于演示@Inject match-by-type 执行路径的测试。我们使用以下 FieldAutowiredTest 集成测试来演示使用 @Autowired 注解的按类型匹配：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredType.class)
public class FieldAutowiredIntegrationTest {

    @Autowired
    private ArbitraryDependency fieldDependency;

    @Test
    public void givenAutowired_WhenSetOnField_ThenDependencyResolved() {
        assertNotNull(fieldDependency);
        assertEquals("Arbitrary Dependency", fieldDependency.toString());
    }
}
```

我们列出了此集成测试的应用程序上下文：

```java
@Configuration
public class ApplicationContextTestAutowiredType {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }
}
```

我们使用此集成测试来证明按类型匹配优先于其他执行路径。注意 FieldAutowiredTest 集成测试第 8 行的引用变量名称：

```java
@Autowired
private ArbitraryDependency fieldDependency;
```

这与应用程序上下文中的 bean 名称不同

```java
@Bean
public ArbitraryDependency autowiredFieldDependency() {
    ArbitraryDependency autowiredFieldDependency =
      new ArbitraryDependency();
    return autowiredFieldDependency;
}
```

当我们运行测试时，它应该通过。

为了确认确实使用 match-by-type 执行路径解决了依赖关系，我们需要更改 fieldDependency 引用变量的类型并再次运行集成测试。这一次，FieldAutowiredTest 集成测试将失败，并抛出 NoSuchBeanDefinitionException。这验证了我们使用 match-by-type 来解决依赖关系。

##### 按 Qualifier 匹配

如果我们面临在应用程序上下文中定义多个 bean 实现的情况，该怎么办：

```java
@Configuration
public class ApplicationContextTestAutowiredQualifier {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }

    @Bean
    public ArbitraryDependency anotherAutowiredFieldDependency() {
        ArbitraryDependency anotherAutowiredFieldDependency =
          new AnotherArbitraryDependency();
        return anotherAutowiredFieldDependency;
    }
}
```

如果我们执行以下 FieldQualifierAutowiredTest 集成测试，将抛出 NoUniqueBeanDefinitionException：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredQualifier.class)
public class FieldQualifierAutowiredIntegrationTest {

    @Autowired
    private ArbitraryDependency fieldDependency1;

    @Autowired
    private ArbitraryDependency fieldDependency2;

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep1Valid(){
        assertNotNull(fieldDependency1);
        assertEquals("Arbitrary Dependency", fieldDependency1.toString());
    }

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep2Valid(){
        assertNotNull(fieldDependency2);
        assertEquals("Another Arbitrary Dependency",
          fieldDependency2.toString());
    }
}
```

这个异常是由于应用上下文中定义的两个Bean引起的歧义。Spring框架不知道哪个Bean依赖应该被自动连接到哪个引用变量。我们可以通过在FieldQualifierAutowiredTest集成测试的第7和第10行添加@Qualifier注解来解决这个问题。

```java
@Autowired
private FieldDependency fieldDependency1;

@Autowired
private FieldDependency fieldDependency2;
```

使代码块如下所示：

```java
@Autowired
@Qualifier("autowiredFieldDependency")
private FieldDependency fieldDependency1;

@Autowired
@Qualifier("anotherAutowiredFieldDependency")
private FieldDependency fieldDependency2;
```

当我们再次运行测试时，它会通过。

##### 按名称匹配

我们将使用相同的集成测试场景来使用@autowired注释来演示按名称匹配的执行路径去注入字段依赖项。当按名称自动依赖项时，必须与应用程序上下文，ApplicationContextTestaUtowiredName一起使用@ComponentsCan注释：

```java
@Configuration
@ComponentScan(basePackages={"com.baeldung.dependency"})
    public class ApplicationContextTestAutowiredName {
}
```

我们使用@ComponentScan注解来搜索包中已经被@Component注解的Java类。例如，在应用上下文中，com.baeldung.dependency包将被扫描，以寻找已被@Component注解的类。在这种情况下，Spring框架必须检测到ArbitraryDependency类，它有@Component注解。	

```java
@Component(value="autowiredFieldDependency")
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

传递到@Component注解中的属性值autowiredFieldDependency告诉Spring框架，ArbitraryDependency类是一个名为autowiredFieldDependency的组件。为了让@Autowired注解按名称解析依赖关系，组件名称必须与FieldAutowiredNameTest集成测试中定义的字段名称一致；请参考第8行

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredName.class)
public class FieldAutowiredNameIntegrationTest {

    @Autowired
    private ArbitraryDependency autowiredFieldDependency;

    @Test
    public void givenAutowiredAnnotation_WhenOnField_ThenDepValid(){
        assertNotNull(autowiredFieldDependency);
        assertEquals("Arbitrary Dependency",
          autowiredFieldDependency.toString());
	}
}
```

当我们运行FieldAutowiredNameTest集成测试时，它将通过。

但是我们怎么知道@Autowired注解真的调用了按名字匹配的执行路径呢？我们可以把引用变量autowiredFieldDependency的名字改为我们选择的另一个名字，然后再次运行测试。

这一次，测试将失败并抛出*NoUniqueBeanDefinitionException*。类似的检查是将*@Component*属性值*autowiredFieldDependency*更改为我们选择的另一个值并再次运行测试。一个*NoUniqueBeanDefinitionException*也将被抛出。

这个异常证明如果我们使用不正确的 bean 名称，将找不到有效的 bean。这就是我们如何知道调用了按名称匹配的执行路径。

#### setter 注入

@Autowired注解的基于设置器的注入与@Resource基于设置器的注入所展示的方法类似。我们没有用@Inject注解来注解引用变量，而是注解了相应的设置器。基于字段的依赖注入所遵循的执行路径也适用于基于设置器的注入。

### 应用这些注解

这就提出了应该使用哪个注解以及在什么情况下使用的问题。这些问题的答案取决于相关应用程序面临的设计场景，以及开发人员希望如何基于每个注解的默认执行路径利用多态性。

#### Application-Wide Use of Singletons Through Polymorphism

如果设计是这样的：应用程序的行为是基于接口或抽象类的实现，并且这些行为在整个应用程序中使用，那么我们可以使用@Inject或@Autowired注释。

这种方法的好处是，当我们升级应用程序，或者为了修复一个错误而应用一个补丁时，类可以被替换，而对整个应用程序的行为产生最小的负面影响。在这种情况下，主要的默认执行路径是按类型匹配。

#### 通过多态进行细粒度的应用行为配置

如果设计是应用程序具有复杂的行为，每个行为都基于不同的接口/抽象类，并且这些实现中的每一个的用法因应用程序而异，那么我们可以使用*@Resource*注解。在这种情况下，主要的默认执行路径是按名称匹配。

#### 依赖注入应该由 Jakarta EE 平台单独处理

如果 Jakarta EE 平台（而不是 Spring）注入所有依赖项的设计要求，那么选择是在*@Resource*注释和*@Inject*注释之间。我们应该根据需要哪个默认执行路径来缩小两个注释之间的最终决定范围。

#### 依赖注入应该由 Spring 框架单独处理

如果要求 Spring 框架处理所有依赖项，则唯一的选择是*@Autowired*注释

#### 讨论总结

| Scenario                                                     | @Resource | @Inject | @Autowired |
| :----------------------------------------------------------- | --------- | ------- | ---------- |
| Application-wide use of singletons through polymorphism      | ✗         | ✔       | ✔          |
| Fine-grained application behavior configuration through polymorphism | ✔         | ✗       | ✗          |
| Dependency injection should be handled solely by the Jakarta EE platform | ✔         | ✔       | ✗          |
| Dependency injection should be handled solely by the Spring Framework | ✗         | ✗       | ✔          |



## CommandLineRunner，@Order

在我们实际工作中，总会遇到这样需求，在项目启动的时候需要做一些初始化的操作，比如初始化线程池，提前加载好加密证书等。今天就给大家介绍一个 Spring Boot 神器，专门帮助大家解决项目启动初始化资源操作。

这个神器就是 `CommandLineRunner`，`CommandLineRunner` 接口的 `Component` 会在所有 `Spring Beans `都初始化之后，`SpringApplication.run() `之前执行，非常适合在应用程序启动之初进行一些数据初始化的工作。

接下来我们就运用案例测试它如何使用，在测试之前在启动类加两行打印提示，方便我们识别 `CommandLineRunner` 的执行时机。

```java
@SpringBootApplication
public class CommandLineRunnerApplication {
	public static void main(String[] args) {
		System.out.println("The service to start.");
		SpringApplication.run(CommandLineRunnerApplication.class, args);
		System.out.println("The service has started.");
	}
}
```



接下来我们直接创建一个类继承 `CommandLineRunner` ，并实现它的 `run()` 方法。

```java
@Component
public class Runner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The Runner start to initialize ...");
    }
}
```

我们在 `run()` 方法中打印了一些参数来看出它的执行时机。完成之后启动项目进行测试：

```properties
...
The service to start.

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.0.RELEASE)
...
2018-04-21 22:21:34.706  INFO 27016 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-21 22:21:34.710  INFO 27016 --- [           main] com.neo.CommandLineRunnerApplication     : Started CommandLineRunnerApplication in 3.796 seconds (JVM running for 5.128)
The Runner start to initialize ...
The service has started.
```

根据控制台的打印信息我们可以看出 `CommandLineRunner` 中的方法会在 Spring Boot 容器加载之后执行，执行完成后项目启动完成。

如果我们在启动容器的时候需要初始化很多资源，并且初始化资源相互之间有序，那如何保证不同的 `CommandLineRunner` 的执行顺序呢？Spring Boot 也给出了解决方案。那就是使用 `@Order` 注解。

我们创建两个 `CommandLineRunner` 的实现类来进行测试：

第一个实现类：

```java
@Component
@Order(1)
public class OrderRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner1 start to initialize ...");
    }
}
```

第二个实现类：

```java
@Component
@Order(2)
public class OrderRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner2 start to initialize ...");
    }
}
```



添加完成之后重新启动，观察执行顺序：

```java
...
The service to start.

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.0.RELEASE)
...
2018-04-21 22:21:34.706  INFO 27016 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-21 22:21:34.710  INFO 27016 --- [           main] com.neo.CommandLineRunnerApplication     : Started CommandLineRunnerApplication in 3.796 seconds (JVM running for 5.128)
The OrderRunner1 start to initialize ...
The OrderRunner2 start to initialize ...
The Runner start to initialize ...
The service has started.
```

通过控制台的输出我们发现，添加 `@Order` 注解的实现类最先执行，并且`@Order()`里面的值越小启动越早。

在实践中，使用`ApplicationRunner`也可以达到相同的目的，两着差别不大。看来使用 Spring Boot 解决初始化资源的问题非常简单。