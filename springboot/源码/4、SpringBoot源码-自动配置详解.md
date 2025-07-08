# 一 Import注解详解

通过上面的分析，我们知道了配置类解析及加载的完整过程，从ConfigurationClassPostProcessor重载的BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry()方法开始，到ConfigurationClassParser解析，再到注册加载所有的配置类结束的完整流程。

而上述涉及到自动配置的流程，是在 [3、SpringBoot源码-自动配置原理](http://halo.shangsw.com/archives/articles202109291632883616454html) 2.5.1.4 解析处理@Import注解 开始的，大致代码如下：

```java
if (candidate.isAssignable(ImportSelector.class)) {//Import选择器
   Class<?> candidateClass = candidate.loadClass();
   ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
         this.environment, this.resourceLoader, this.registry);//反射实例化Import选择器
   Predicate<String> selectorFilter = selector.getExclusionFilter();//排除的过滤器
   if (selectorFilter != null) {
      exclusionFilter = exclusionFilter.or(selectorFilter);
   }

   if (selector instanceof DeferredImportSelector) {//处理DeferredImportSelector(如自动配置等)
      this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);//处理
   } else {//普通的ImportSelector, 递归处理
      String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
      Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames, exclusionFilter);
      //递归处理
      processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
   }
} else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {//ImportBeanDefinitionRegistrar注册器
   Class<?> candidateClass = candidate.loadClass();

   ImportBeanDefinitionRegistrar registrar =
         ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
               this.environment, this.resourceLoader, this.registry);
   configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
} else {//不是ImportSelector也不是ImportBeanDefinitionRegistrar(普通的配置类)
   this.importStack.registerImport(
         currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
   processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);//递归调用
}
```

可以看出，对于@Import的处理其实是有三种方式：

- @Import(ImportSelector实现类.class)
- @Import(ImportBeanDefinitionRegistrar实现类.class)
- 普通的类

我们先看一下@Import具体的使用



## 1.1 @Import注解使用

我们看一下@Import具体的配置使用，然后进行具体的源码分析。

### 1.1.1 实现ImportSelector

**(1) 定义配置类**

首先我们定义一个配置类ImportSelectorConfig：

```java
public class ImportSelectorConfig {
    private String userName = "张三";

    public void printName() {
        System.out.println("Hello, " + userName);
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }
}
```

这个类不需要任何注解，而是使用ImportSelector对其进行加载。

**(2) 定义ImportSelector选择器**

定义CustomImportSelector选择器，实现ImportSelector，并重写selectImports()方法：

```java
public class CustomImportSelector implements ImportSelector {
    @NonNull
    @Override
    public String[] selectImports( @NonNull AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.bianjf.config.imports.ImportSelectorConfig"};
    }

    /**
     * 不做任何的过滤处理
     * @return 过滤器
     */
    @Override
    public Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

在selectImports()方法中返回了上面的ImportSelectConfig.class.getName的类

**(3) 使用@Import引入**

最后使用@Import注解因如CustomImportSelector：

```java
@Configuration
@Import({CustomImportSelector.class})
public class ImportConfig {
}
```

最后修改启动类测试：

```java
@SpringBootApplication(scanBasePackages = {"com.bianjf"})
public class Application {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Application.class, args);
        ImportSelectorConfig importSelectorConfig = applicationContext.getBean(ImportSelectorConfig.class);
        importSelectorConfig.printName();
    }
}
```

最后控制台打印如下：

```java
Hello, 张三
```



### 1.1.2 实现ImportBeanDefinitionRegistrar

这种方式比上述的ImportSelector要简单点，只需要注册一个Bean就可以了。

**(1) 定义配置类**

首先定义一个配置类ImportBeanDefinitionRegistrarConfig：

```java
public class ImportBeanDefinitionRegistrarConfig {
    private String userName = "李四";

    public void sayHello() {
        System.out.println("Hello, " + userName);
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }
}
```

**(2) 定义ImportBeanDefinitionRegistrar**

创建CustomImportBeanDefinitionRegistrar，实现ImportBeanDefinitionRegistrar接口，并重写registerBeanDefinitions方法：

```java
public class CustomImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        RootBeanDefinition beanDefinition = new RootBeanDefinition(ImportBeanDefinitionRegistrarConfig.class);
        registry.registerBeanDefinition("importBeanDefinitionRegistrarConfig", beanDefinition);
    }
}
```

这里创建了RootBeanDefinition，即Spring中的Bean定义，然后使用注册器注册该Bean，这样Spring在刷新的时候就可以对其进行加载了

**(3) 使用@Import引入**

最后引入CustomImportBeanDefinitionRegistrar：

```java
@Configuration
@Import({CustomImportSelector.class, CustomImportBeanDefinitionRegistrar.class})
public class ImportConfig {
}
```

测试修改启动类如下：

```java
@SpringBootApplication(scanBasePackages = {"com.bianjf"})
public class Application {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Application.class, args);
        ImportBeanDefinitionRegistrarConfig importBeanDefinitionRegistrarConfig = applicationContext.getBean(ImportBeanDefinitionRegistrarConfig.class);
        importBeanDefinitionRegistrarConfig.sayHello();
    }
}
```

控制台打印如下：

```java
Hello, 李四
```



### 1.1.3 普通的配置类

这里只定义普通的实体类BeanEntity：

```java
public class BeanEntity {
    private String userName = "王五";

    private String fruit = "西瓜";

    public void eat() {
        System.out.println(userName + "吃" + fruit);
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getFruit() {
        return fruit;
    }

    public void setFruit(String fruit) {
        this.fruit = fruit;
    }
}
```

然后使用@Import引入：

```java
@Configuration
@Import({CustomImportSelector.class, CustomImportBeanDefinitionRegistrar.class, BeanEntity.class})
public class ImportConfig {
}
```

修改启动类测试：

```java
@SpringBootApplication(scanBasePackages = {"com.bianjf"})
public class Application {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Application.class, args);
        BeanEntity beanEntity = applicationContext.getBean(BeanEntity.class);
        beanEntity.eat();
    }
}
```

控制台打印如下：

```java
王五吃西瓜
```

通过上述的使用分析，我们看具体的源码实现。



# 二 ImportSelector详解

首先我们看ImportSelector部分，在该部分也有自动配置AutoConfigurationImportSelector的处理，大致代码如下：

```java
if (candidate.isAssignable(ImportSelector.class)) {//Import选择器
   Class<?> candidateClass = candidate.loadClass();
   ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
         this.environment, this.resourceLoader, this.registry);//反射实例化Import选择器
   Predicate<String> selectorFilter = selector.getExclusionFilter();//排除的过滤器
   if (selectorFilter != null) {
      exclusionFilter = exclusionFilter.or(selectorFilter);
   }

   if (selector instanceof DeferredImportSelector) {//处理DeferredImportSelector(如自动配置等)
      this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);//处理
   } else {//普通的ImportSelector, 递归处理
      String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
      Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames, exclusionFilter);
      //递归处理
      processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
   }
}
```

首先判断是不是DeferredImportSelector，如果是的话调用DeferredImportSelectorHandler#handle执行，如果是一个普通的ImportSelector的话，则调用其**selectImports()** 方法，然后递归调用本方法。

而我们从[3、SpringBoot源码-自动配置原理](http://halo.shangsw.com/archives/articles202109291632883616454html) 中整体的流程来看，对于DeferredImportSelector的处理总共可以分成三步：

- 就是这里的DeferredSelectorHandler#handle进行处理
- 在其上层调用地方ConfigurationClassParser#parse(Set configCandidates)的地方最后进行DeferredImportSelectorHandler#process()进行处理
- 加载BeanDefinitions

所以我们从这三个地方入手。



## 2.1 DeferredImportSelectorHandler#handle处理

其中**AutoConfigurationImportSelector**就是实现DeferredImportSelector接口的，即SpringBoot的自动配置也是在此进行加载的。进入**DeferredImportSelectorHandler#handle**方法：

```java
/**
* 处理特殊的DeferredImportSelector。将DeferredImportSelector封装成DeferredImportSelectorHolder,
* 然后进一步进行处理
* @param configClass 配置类
* @param importSelector 选择器
*/
public void handle(ConfigurationClass configClass, DeferredImportSelector importSelector) {
   DeferredImportSelectorHolder holder = new DeferredImportSelectorHolder(configClass, importSelector);
   if (this.deferredImportSelectors == null) {//如果全局变量为空(调用了process()方法后可能会存在递归的情况, 此时会变null)
      DeferredImportSelectorGroupingHandler handler = new DeferredImportSelectorGroupingHandler();
      handler.register(holder);
      handler.processGroupImports();
   } else {
      this.deferredImportSelectors.add(holder);
   }
}
```

这一步的逻辑很简单，首先是将DeferredImportSelector封装成DeferredImportSelectorHolder，然后判断DeferredImportSelectors是否为空，不为空则进行add操作，我们看一下this.deferredImportSelectors：

```java
@Nullable
private List<DeferredImportSelectorHolder> deferredImportSelectors = new ArrayList<>();
```

可以看到，正常流程下这里是不会为空的，但是可能会存在递归调用，所以这里是可能为空的。我们就以正常的流程推断，先假设这里不为空，然后程序将DeferredImportSelectorHolder添加到了全局变量**deferredImportSelectors**集合中了。

## 2.2 DeferredImportSelectorHandler#process处理

经过上述的handle处理后，DeferredImportSelectorHandler中的**deferredImportSelectors**集合中已经有值了，此时程序返回到**ConfigurationClassParser#parse(Set configCandidates)** 的地方：

```java
/**
* 解析配置类(@Configuration、@Component)
* @param configCandidates 有相关注解的BeanDefinitionHolder
*/
public void parse(Set<BeanDefinitionHolder> configCandidates) {
   for (BeanDefinitionHolder holder : configCandidates) {
      BeanDefinition bd = holder.getBeanDefinition();//BeanDefinition
      try {
         if (bd instanceof AnnotatedBeanDefinition) {//程序是通过注解定义的
            parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());//解析
         } else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {//程序是通过配置文件定义的
            parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());//解析
         } else {//其他的情况(比如通过className配置的)
            parse(bd.getBeanClassName(), holder.getBeanName());//解析
         }
      } catch (BeanDefinitionStoreException ex) {
         throw ex;
      } catch (Throwable ex) {
         throw new BeanDefinitionStoreException(
               "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
      }
   }
   this.deferredImportSelectorHandler.process();//最后再处理
}
```

这个方法的最后一步开始调用DeferredImportSelectorHandler#process方法：

```java
/**
* 解析完成之后处理
*/
public void process() {
   List<DeferredImportSelectorHolder> deferredImports = this.deferredImportSelectors;
   this.deferredImportSelectors = null;
   try {
      if (deferredImports != null) {
         DeferredImportSelectorGroupingHandler handler = new DeferredImportSelectorGroupingHandler();
         deferredImports.sort(DEFERRED_IMPORT_COMPARATOR);
         deferredImports.forEach(handler::register);//注册Handler
         handler.processGroupImports();//处理
      }
   } finally {
      this.deferredImportSelectors = new ArrayList<>();
   }
}
```

程序走到这里就开始进行处理了，经过上述的**handle**处理之后，**this.deferredImportSelectors**是不为空的，即**deferredImports**是不为空的，然后立马将this.deferredImportSelectors置为null，这里是为了在下面的逻辑处理过程中递归调用，进入**handle**方法处理。先不追究handle方法中的处理。

- 首先创建一个DeferredImportSelectorGroupingHandler
- 然后对deferredImports进行排序
- 然后循环调用DeferredImportSelectorGroupingHandler#register方法进行注册
- 最后调用DeferredImportSelectorGroupingHandler#processGroupImports进行最后的处理

## 2.3 DeferredImportSelectorGroupingHandler#register注册

DeferredImportSelectorGroupingHandler#register代码如下：

```java
/**
* 对选择器进行注册操作
* @param deferredImport 选择器的持有者
*/
public void register(DeferredImportSelectorHolder deferredImport) {
   Class<? extends Group> group = deferredImport.getImportSelector().getImportGroup();
   DeferredImportSelectorGrouping grouping = this.groupings.computeIfAbsent(
         (group != null ? group : deferredImport),
         key -> new DeferredImportSelectorGrouping(createGroup(group)));
   grouping.add(deferredImport);
   this.configurationClasses.put(deferredImport.getConfigurationClass().getMetadata(),
         deferredImport.getConfigurationClass());
}
```

首先获取一个Import组的Class，然后将组作为key，DeferredImportSelectorGrouping作为value放入到groupings的Map中，最后将ConfigurationClass的metadata作为key，ConfigurationClass作为value放入到configurationClasses的Map中。

而对于**AutoConfigurationImportSelector#getImportGroup()** 方法看一眼其返回的是什么：

```java
/**
* 2、这个类(DeferredImportSelector实现类)第二步执行的方法: 获取导入组
* 这里返回的是AutoConfigurationGroup
* @return AutoConfigurationGroup类
*/
@Override
public Class<? extends Group> getImportGroup() {
   return AutoConfigurationGroup.class;
}
```

这里返回的是一个**AutoConfigurationGroup.class**

## 2.4 DeferredImportSelectorGroupingHandler#processGroupImports()处理

通过上一步的注册，我们将必要值存到了DeferredImportSelectorGroupingHandler中了，即groupings和configurationClasses中。接下来就是进行处理了：**processGroupImports()** 代码如下：

```java
/**
* 处理Group Import
*/
public void processGroupImports() {
   for (DeferredImportSelectorGrouping grouping : this.groupings.values()) {
      Predicate<String> exclusionFilter = grouping.getCandidateFilter();//过滤器
      grouping.getImports().forEach(entry -> {//获取Group
         ConfigurationClass configurationClass = this.configurationClasses.get(entry.getMetadata());
         try {
            processImports(configurationClass, asSourceClass(configurationClass, exclusionFilter),
                  Collections.singleton(asSourceClass(entry.getImportClassName(), exclusionFilter)),
                  exclusionFilter, false);
         } catch (BeanDefinitionStoreException ex) {
            throw ex;
         } catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                  "Failed to process import candidates for configuration class [" +
                        configurationClass.getMetadata().getClassName() + "]", ex);
         }
      });
   }
}
```

这里调用了DeferredImportSelectorGrouping的getImports()方法，然后进行的递归调用。我们进入getImports()方法：

```java
/**
* 获取imports 定义的组
* @return Group.Entry迭代器
*/
public Iterable<Group.Entry> getImports() {
   for (DeferredImportSelectorHolder deferredImport : this.deferredImports) {
      this.group.process(deferredImport.getConfigurationClass().getMetadata(),
            deferredImport.getImportSelector());
   }
   return this.group.selectImports();
}
```

这里方法逻辑很简单，先是调用group的process，然后调用group的selectImports()方法，而主要的处理逻辑是在process中进行的

### 2.4.1 自动导入选择器组处理

依据上文，我们知道在SpringBoot中，group其实是**AutoConfigurationGroup** ，我们进入**process** 方法：

```java
/**
* 3、父类(ImportSelector实现类)第三步执行的方法: 执行
* @param annotationMetadata 注解元数据
* @param deferredImportSelector DeferredImportSelector
*/
@Override
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
   //断言是不是AutoConfigurationImportSelector类 Start
   Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector,
         () -> String.format("Only %s implementations are supported, got %s",
               AutoConfigurationImportSelector.class.getSimpleName(),
               deferredImportSelector.getClass().getName()));
   //断言是不是AutoConfigurationImportSelector类 End

   AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
         .getAutoConfigurationEntry(annotationMetadata);//获取AutoConfigurationEntry
   this.autoConfigurationEntries.add(autoConfigurationEntry);//存放数据
   for (String importClassName : autoConfigurationEntry.getConfigurations()) {
      this.entries.putIfAbsent(importClassName, annotationMetadata);
   }
}
```

这里调用了DeferredImportSelector#getAutoConfigurationEntry()方法获取AutoConfigurationEntry，进入**AutoConfigurationIImportSelector#getAutoConfigurationEntry** 方法：

```java
/**
* 返回一个AutoConfigurationEntry结构的数据。
* @param annotationMetadata 注解元数据
* @return 需要导入的自动配置类
*/
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
   //校验是否开启了自动配置功能, 即@EnableAutoConfiguration注解 Start
   if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
   }
   //校验是否开启了自动配置功能, 即@EnableAutoConfiguration注解 End

   AnnotationAttributes attributes = getAttributes(annotationMetadata);//获取EnableAutoConfiguration注解的属性
   //加载META-INF/spring.factories中的EnableAutoConfiguration值
   List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   configurations = removeDuplicates(configurations);//去重
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);//获取需要排除的配置(即@EnableAutoConfiguration的两个属性的值)
   checkExcludedClasses(configurations, exclusions);//校验
   configurations.removeAll(exclusions);//去除掉
   configurations = getConfigurationClassFilter().filter(configurations);//再过滤掉需要跳过的
   fireAutoConfigurationImportEvents(configurations, exclusions);//执行相关的监听器
   return new AutoConfigurationEntry(configurations, exclusions);//返回
}
```

在这个函数中，主要有以下几点：

- getCandidateConfigurations()：获取所有的自动配置类，该自动配置类是SpringBoot为我们默认提供的自动配置，如AopAutoConfiguration、RabbitAutoConfiguration等等
- 通知相关监听器

**getCandidateConfigurations()** 获取自动配置：

```java
/**
* 获取所有的META-INF/spring.factories中key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的值
* @param metadata 注解元数据
* @param attributes 注解属性
* @return EnableAutoConfiguration的值
*/
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
         getBeanClassLoader());
   Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
         + "are using a custom packaging, make sure that file is correct.");
   return configurations;
}
```

**getSpringFactoriesLoaderFactoryClass()** 代码如下：

```java
/**
* 返回EnableAutoConfiguration的class
* @return EnableAutoConfiguration的class
*/
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
   return EnableAutoConfiguration.class;
}
```

在[3、SpringBoot源码-自动配置原理](http://halo.shangsw.com/archives/articles202109291632883616454html) 中的 1.1.2 中我们已经介绍过**SpringFactoriesLoader#loadFactoryNames**了，其实就是加载**META-INF/spring.factories**中指定的key的值，所以这里是加载META-INF/spring.factories中EnableAutoConfiguration的配置：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\ org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\ org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\ org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\ org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\ org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\ org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\ org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\ org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\ org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\ org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\ org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\ org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\ org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\ org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\ org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\ org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\ org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\ org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\ org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\ org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\ org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\ org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\ org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\ org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\ org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\ org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\ org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\ org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\ org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\ org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\ org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\ org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\ org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\ org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\ org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\ org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\ org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\ org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\ org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\ org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\ org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\ org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\ org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\ org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\ org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\ org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\ org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\ org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\ org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\ org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\ org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\ org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\ org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\ org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\ org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\ org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\ org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\ org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\ org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\ org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\ org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\ org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\ org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\ org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\ org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\ org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\ org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\ org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\ org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\ org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\ org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\ org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\ org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\ org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\ org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\ org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\ org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\ org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\ org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\ org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\ org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\ org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\ org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\ org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\ org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\ org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\ org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\ org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\ org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\ org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\ org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\ org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\ org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\ org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

SpringBoot默认为我们提供了这些自动配置，当然，如果我们想**制作自己的spring-boot-starter的话，就是从这里开始，在项目的resource目录下创建META-INF/spring.factories文件，然后配置EnableAutoConfiguration为我们自己的指定的自动配置类**，这样其实就是一个spring-boot-starter了。

对于每一个Starter我这里就不做过多的介绍了，每一个Starter都有一些功能，有兴趣的可以自行研究。

加载上SpringBoot默认提供的自动配置以后，经过一些过滤和去重等操作，最后就是通知相关监听器了：**fireAutoConfigurationImportEvents()**：

```java
/**
* 执行AutoConfigurationImportListener中的onAutoConfigurationImportEvent方法。这个监听器主要监听的是
* AutoConfigurationImportEvent事件
* @param configurations 自动配置类的class.getName
* @param exclusions 过滤的class.getName
*/
private void fireAutoConfigurationImportEvents(List<String> configurations, Set<String> exclusions) {
   List<AutoConfigurationImportListener> listeners = getAutoConfigurationImportListeners();
   if (!listeners.isEmpty()) {
      AutoConfigurationImportEvent event = new AutoConfigurationImportEvent(this, configurations, exclusions);
      for (AutoConfigurationImportListener listener : listeners) {
         invokeAwareMethods(listener);
         listener.onAutoConfigurationImportEvent(event);
      }
   }
}
```

getAutoConfigurationImportListeners()代码如下：

```java
/**
* 获取META-INF/spring.factories中key为AutoConfigurationImportListener的配置, 然后反射加载
* @return AutoConfigurationImportListener实例集合
*/
protected List<AutoConfigurationImportListener> getAutoConfigurationImportListeners() {
   return SpringFactoriesLoader.loadFactories(AutoConfigurationImportListener.class, this.beanClassLoader);
}
```

所以我们可以知道，这里是加载**META-INF/spring.factories中的AutoConfigurationImportListener**的配置：

```java
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\ 
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener
```

如果我们需要对自动配置进行事件监听的话，主要监听的是存在哪些自动配置和排除掉了哪些自动配置类，我们可以**实现AutoConfigurationImportListener接口，然后重写onAutoConfigurationImportEvent()方法，该方法其实监听的就是AutoConfigurationImportEvent事件。然后将该监听器在META-INF/spring.factories中配置，key为org.springframework.boot.autoconfigure.AutoConfigurationImportListener即可**

### 2.4.2 获取Import

通过上述的加载，我们加载出了**META-INF/spring.factories中key为EnableAutoConfiguration**的所有值，然后存储到了autoConfigurationEntries中，接下来就是**AutoConfigurationGroup#selectImports()** 获取了：

```java
/**
* 获取自动配置的Entry
* @return Entry迭代器
*/
@Override
public Iterable<Entry> selectImports() {
   //不存在直接返回空 Start
   if (this.autoConfigurationEntries.isEmpty()) {
      return Collections.emptyList();
   }
   //不存在直接返回空 End

   Set<String> allExclusions = this.autoConfigurationEntries.stream()
       .map(AutoConfigurationEntry::getExclusions).flatMap(Collection::stream).collect(Collectors.toSet());//需要过滤的
   Set<String> processedConfigurations = this.autoConfigurationEntries.stream()
         .map(AutoConfigurationEntry::getConfigurations).flatMap(Collection::stream)
         .collect(Collectors.toCollection(LinkedHashSet::new));//全部的自动配置类
   processedConfigurations.removeAll(allExclusions);//过滤

   return sortAutoConfigurations(processedConfigurations, getAutoConfigurationMetadata()).stream()
         .map((importClassName) -> new Entry(this.entries.get(importClassName), importClassName))
         .collect(Collectors.toList());//排序并返回
}
```

这里的逻辑很简单，就是获取的过程了，最后经过排序并返回。

## 2.5 递归处理

DeferredImportSelectorGrouping#getImports获取到所有的配置之后，我们再回到**DeferredImportSelectorGroupingHandler#processGroupImports()** 方法：

```java
/**
* 处理Group Import
*/
public void processGroupImports() {
   for (DeferredImportSelectorGrouping grouping : this.groupings.values()) {
      Predicate<String> exclusionFilter = grouping.getCandidateFilter();//过滤器
      grouping.getImports().forEach(entry -> {//获取Group
         ConfigurationClass configurationClass = this.configurationClasses.get(entry.getMetadata());
         try {
            processImports(configurationClass, asSourceClass(configurationClass, exclusionFilter),
                  Collections.singleton(asSourceClass(entry.getImportClassName(), exclusionFilter)),
                  exclusionFilter, false);
         } catch (BeanDefinitionStoreException ex) {
            throw ex;
         } catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                  "Failed to process import candidates for configuration class [" +
                        configurationClass.getMetadata().getClassName() + "]", ex);
         }
      });
   }
}
```

发现这里又调用了processImports()方法，而该方法我们在[3、SpringBoot源码-自动配置原理](http://halo.shangsw.com/archives/articles202109291632883616454html)已经完整介绍过其流程了，所以最终会走到普通的配置并进行加载。所以这里就加载了所有自动配置了。

# 三 ImportBeanDefinitionRegistrar详解

我们通过上述的使用示例，知道@Import有三种的解析，其中一种就是**ImportBeanDefinitionRegistrar**方式，代码是在**ConfigurationClassParser#processImports**中：

```java
else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {//ImportBeanDefinitionRegistrar注册器
    Class<?> candidateClass = candidate.loadClass();
    ImportBeanDefinitionRegistrar registrar =
            ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
                    this.environment, this.resourceLoader, this.registry);
    configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
}
```

如果是ImportBeanDefinitionRegistrar的话，将其加入到configClass中：

```java
public void addImportBeanDefinitionRegistrar(ImportBeanDefinitionRegistrar registrar, AnnotationMetadata importingClassMetadata) {
   this.importBeanDefinitionRegistrars.put(registrar, importingClassMetadata);
}
```

添加到importBeanDefinitionRegistrars的Map中后，我们看其是怎么加载的，**ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsFromRegistrars**代码如下：

```java
/**
* 加载从@Import(ImportBeanDefinitionRegistrar)形式装载的Bean
* @param registrars ImportBeanDefinitionRegistrar集合
*/
private void loadBeanDefinitionsFromRegistrars(Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> registrars) {
   registrars.forEach((registrar, metadata) ->
         registrar.registerBeanDefinitions(metadata, this.registry, this.importBeanNameGenerator));
}
default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
   registerBeanDefinitions(importingClassMetadata, registry);
}
default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
}
```

最后的registerBeanDefinitions(importingClassMetadata, registry)是一个空的方法，目的是留给子类覆盖，即子类重写该方法即可注册指定的配置。

到这里@Import的完整原理已经介绍完毕了，其中自动配置**DeferredImportSelector（AutoConfigurationImportSelector）** 只是@Import中的一种加载形式。

我们这里总结一下SpringBoot中@Import的完整流程：

- 首先创建**AnnotationConfigServletWebApplicationContext**，在创建的过程中添加了一个**ConfigurationClassPostProcessor**，ConfigurationClassPostProcessor是一个BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor，所以在ApplicationContext刷新的过程中会执行postProcessBeanDefinitionRegistry方法。在postProcessBeanDefinitionRegistry方法中，有调用**ConfigurationClassParser#parse()** 解析的流程
- 解析流程中，解析了 **@Component（@Repository、@Service、@Controller、@Configuration等）、@PropertySource、@ComponentScan（@ComponentScans）、@Import、@ImportResource、@Bean** 的内容。其中自动配置是在@Import中解析的。@Import是我们重点介绍的，对于@Import解析分成了三种：
  - **ImportSelector**的情况：
    - 对于普通的ImportSelector，是调用其selectImports获取配置类名，然后递归调用加载。
    - 而对于DeferredImportSelector，是一种特殊的处理方式，其中自动配置AutoConfigurationImportSelector就是一种DeferredImportSelector、
  - 是**ImportBeanDefinitionRegistrar**，其实是调用registry的注册
  - 普通的配置类，递归调用processConfigurationClass方法

解析完成之后，就是注册加载的过程

而对于AutoConfigurationImportSelector（DeferredImportSelector）自动配置，首先获取**META-INF/spring.factories中key为：org.springframework.boot.autoconfigure.EnableAutoConfiguration**的所有值，然后递归加载。在获取META-INF/spring.factories中，其中发布了一个事件AutoConfigurationImportEvent，该事件的监听者需要配置在META-INF/spring.factories中key为：org.springframework.boot.autoconfigure.AutoConfigurationImportListener中。