# 封装一个自己的 starter



# 0 写在前面

SpringBoot 中有一个非常重要的机制——starter，它是遵循“约定大于配置”理念的一个重要表现。能够将功能集成进 starter 中，无需繁杂的配置（可以认为特指一大堆 xml 文件）即可在 maven 项目中引入并使用。**日常开发中，经常会遇到一些独立于业务之外的通用模块（例如日志处理、redis 工具等），秉承着不重复造轮子的理念，我们可以考虑将类似的模块封装成一个 starter，按需引入，利用 SpringBoot 的自动装配将 Bean 注册进 IOC 容器中。**

# 1 前置知识

封装一个 starter 并不适合零基础的小白。在了解如何封装一个 starter 之前，至少需要了解的前置知识如下：

1. SpringBoot 的基础使用（如创建项目、编写配置文件以及配置类等）
2. 如果能了解SpringBoot 的自动装配原理更好，不了解也没关系，不影响本 demo 的学习
3. 了解Redis 的基础使用（其实这个也不强求，只是因为本 demo 选择做的是一个 redis 的工具类一样的 starter，所以最好还是有一点基础）
4. maven 的基础知识，至少要了解本地仓库跟远程仓库这些概念。
5. 了解自定义注解
6. 已安装 redis

# 2 开始编写 starter

1. 创建一个maven 项目

   ![image-20210810152751530](封装一个自己的 starter.assets/52839293d3d74dc794572d28a7a81670_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

2. 引入 springboot 父依赖

   ```xml
   xml
   复制代码<parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.5.3</version>
           <relativePath/>
   </parent>
   ```

3. 引入 redis 相关依赖

   ```xml
   xml
   复制代码<properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <spring.boot.version>2.5.3</spring.boot.version>
   </properties>
   
   <dependencies>
           <!-- 添加 redis 依赖 -->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-redis</artifactId>
               <version>${spring.boot.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.data</groupId>
               <artifactId>spring-data-redis</artifactId>
               <version>${spring.boot.version}</version>
           </dependency>
   </dependencies>
   ```

4. 创建基础包结构

   - ![image-20210810153807752](封装一个自己的 starter.assets/042f28daface4551a5404b3da0c1ef13_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

5. 在 config 包下创建 RedisTemplateConfig 类

   ```kotlin
   kotlin
   复制代码package com.study.redis.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.data.redis.connection.RedisConnectionFactory;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
   import org.springframework.data.redis.serializer.StringRedisSerializer;
   
   /**
    * @author markcwg
    * @date 2021/8/10 3:48 下午
    */
   @Configuration
   public class RedisTemplateConfig {
   
       @Bean
       public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
           RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
           redisTemplate.setConnectionFactory(redisConnectionFactory);
           //定义key序列化方式
           redisTemplate.setKeySerializer(new StringRedisSerializer());
           //定义value序列化方式
           redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
           return redisTemplate;
       }
   }
   ```

6. 在 helper 包下创建 RedisHelper 类

   ```typescript
   typescript
   复制代码package com.study.redis.helper;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.stereotype.Component;
   
   import java.util.Set;
   
   /**
    * @author markcwg
    * @date 2021/8/10 3:53 下午
    */
   @Component
   public class RedisHelper {
       @Autowired
       private RedisTemplate<String, String> redisTemplate;
   
       /**
        * 是否存在key
        *
        * @param key 键
        * @return 是否存在
        */
       public Boolean hasKey(String key) {
           return redisTemplate.hasKey(key);
       }
   
       /**
        * 查找匹配的key
        *
        * @param pattern 匹配表达式
        * @return 所有匹配的数据
        */
       public Set<String> keys(String pattern) {
           return redisTemplate.keys(pattern);
       }
   
       /**
        * 设置指定 key 的值
        * @param key 键
        * @param value 值
        */
       public void set(String key, String value) {
           redisTemplate.opsForValue().set(key, value);
       }
   
       /**
        * 获取指定 key 的值
        * @param key 键
        * @return 值
        */
       public String get(String key) {
           return redisTemplate.opsForValue().get(key);
       }
   
   }
   
   ```

7. 在 config 包下创建 CustomRedisAutoConfiguration

   ```kotlin
   kotlin
   复制代码package com.study.redis.config;
   
   import com.study.redis.helper.RedisHelper;
   import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;
   
   /**
    * @author markcwg
    * @date 2021/7/30 4:10 下午
    */
   @ConditionalOnWebApplication
   @Configuration(proxyBeanMethods = false)
   @ComponentScan(basePackages = "com.study.redis.config")
   public class CustomRedisAutoConfiguration {
       @Bean
       public RedisHelper redisHelper() {
           return new RedisHelper();
       }
   }
   ```

8. 在 annotations 包下创建 LoadRedis 注解

   ```java
   java
   复制代码package com.study.redis.annotations;
   
   import com.study.redis.config.CustomRedisAutoConfiguration;
   import org.springframework.context.annotation.Import;
   
   import java.lang.annotation.*;
   
   /**
    * @author markcwg
    * @date 2021/8/10 4:00 下午
    */
   @Target(ElementType.TYPE)
   @Documented
   @Retention(RetentionPolicy.RUNTIME)
   @Import(CustomRedisAutoConfiguration.class)
   public @interface LoadRedis {
   }
   ```

9. 使用 mvn install 将 starter 放入自己的 maven 仓库

   ![image-20210810160609905](封装一个自己的 starter.assets/f97773ebd8c347dda077d5ebe69af970_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

10. 判断 maven 本地仓库是否已存在成品(此步可忽略)

    ![image-20210810160805940](封装一个自己的 starter.assets/5e4648806e0444338d7181c1a89a237c_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

# 3 原理解释

> 此处简单解释一下此 starter 的原理：通过 LoadRedis 注解引入 CustomRedisAutoConfiguration 自动配置类，在这个类中将 RedisHelper 注册进 IOC 容器，而 RedisHelper 里封装了常用的 redis 方法，可自行扩展

# 4 使用自定义的 starter

1. 创建一个新的springboot 项目

   ![image-20210810161013780](封装一个自己的 starter.assets/e38714d2a1294268b27de9f2d27c386d_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

2. 引入自定义的 starter

   ```xml
   xml
   复制代码<dependency>
               <groupId>com.study</groupId>
               <artifactId>starter-demo</artifactId>
               <version>0.1</version>
   </dependency>
   ```

3. 编写配置文件(此步要求已安装 redis)

   ```yaml
   yaml
   复制代码server:
     port: 8082
   
   spring:
     redis:
       host: 127.0.0.1
       port: 6379
   ```

4. 在启动类上加上 LoadRedis 注解

   ```less
   less
   复制代码@SpringBootApplication
   @LoadRedis
   public class StarterTestDemoApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(StarterTestDemoApplication.class, args);
       }
   
   }
   ```

5. 创建一个 RedisController 类用作测试

   > 由于只用作测试，此处直接在 Controller 里直接写业务逻辑

   ```kotlin
   kotlin
   复制代码package com.example.startertestdemo.com.study;
   
   import com.study.redis.helper.RedisHelper;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   /**
    * @author markcwg
    * @date 2021/8/10 4:19 下午
    */
   @RestController
   @RequestMapping("redis")
   public class RedisController {
   
       private RedisHelper redisHelper;
       @Autowired
       public RedisController(RedisHelper redisHelper) {
           this.redisHelper = redisHelper;
       }
   
       @PostMapping
       public String save(String key, String value) {
           redisHelper.set(key, value);
           return "success";
       }
   
       @GetMapping
       public String get(String key) {
           return redisHelper.get(key);
       }
   }
   
   ```

6. 使用 postman 测试向 redis 中发送数据

   ![image-20210810162758258](封装一个自己的 starter.assets/f8e2e4d1cdbc49499f15cbf356a99e1e_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

7. 使用 postman 测试从 redis 中获取数据

   ![image-20210810162842918](封装一个自己的 starter.assets/965e099ef1354a93922d9a5d724e1c2d_tplv-k3u1fbpfcp-zoom-in-crop-mark_1512_0_0_0.awebp)

