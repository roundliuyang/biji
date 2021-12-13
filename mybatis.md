## mybatis

### mybatis驼峰式命名

mybatis驼峰式命名规则自动转换：

- 使用前提：数据库表设计按照规范“字段名中各单词使用下划线"_"划分”；
- 使用好处：省去mapper.xml文件中繁琐编写表字段列表与表实体类属性的映射关系，即resultMap。

示例：

```xml
  <resultMap id ="UserInfoMap" type="com.example.mybaitsxml.dao.entity.User">
        <result column="name_" property="name"/>
        <result column="sex" property="sex"/>
        <result column="age" property="age"/>
        <result column="class_no" property="classNo"/>
    </resultMap>
```



pringBoot整合mybatis，开启mybatis驼峰式命名规则自动转换，通常根据配置文件不同分为两种方式。



#### 方式一 :直接application.yml文件中配置开启

```
#mybatis配置
mybatis:
  typeAliasesPackage: com.example.mybaitsxml.dao.entity
  mapperLocations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```



#### 方式二:mybatis-config.xml文件中配置开启，application.yml文件指定配置文件。　

application.yml文件：

```yml
#mybatis配置
mybatis:
  typeAliasesPackage: com.example.mybaitsxml.dao.entity
  mapperLocations: classpath:mapper/*.xml
  configLocation: classpath:/mybatis-config.xml
```

mybatis-config.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--开启驼峰命名规则自动转换-->
    <settings>
    <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>
</configuration>
```



### MyBatis 中Mapper的返回类型

#### insert、update 、delete 语句的返回值类型

对数据库执行修改操作时，数据库会返回受影响的行数。

在MyBatis（使用版本3.4.6，早期版本不支持）中insert、update、delete语句的返回值可以是**Integer、Long和Boolean**。在定义Mapper接口时直接指定需要的类型即可，无需在对应的<insert><update><delete>标签中显示声明。

对应的代码在 **org.apache.ibatis.binding.MapperMethod** 类中，如下：

- 对于insert、update、delete语句，MyBatis都会使用 **rowCountResult** 方法对返回值进行转换；
- **rowCountResult** 方法会根据Mapper声明中方法的返回值类型来对参数进行转换；
- 对于返回类型为Boolean的情况，如果返回的值大于0，则返回True，否则返回False

```java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
      }
      case UPDATE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.update(command.getName(), param));
        break;
      }
      case DELETE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.delete(command.getName(), param));
        break;
      }
      case SELECT:
        if (method.returnsVoid() && method.hasResultHandler()) {
          executeWithResultHandler(sqlSession, args);
          result = null;
        } else if (method.returnsMany()) {
          result = executeForMany(sqlSession, args);
        } else if (method.returnsMap()) {
          result = executeForMap(sqlSession, args);
        } else if (method.returnsCursor()) {
          result = executeForCursor(sqlSession, args);
        } else {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = sqlSession.selectOne(command.getName(), param);
          if (method.returnsOptional()
              && (result == null || !method.getReturnType().equals(result.getClass()))) {
            result = Optional.ofNullable(result);
          }
        }
        break;
      case FLUSH:
        result = sqlSession.flushStatements();
        break;
      default:
        throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName()
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
......  
```

#### select 语句的返回类型s

对于select语句，必须在Mapper映射文件中显示声明返回值类型，否则会抛出异常，指出“A query was run and no Result Maps were found for the Mapped Statement”。

- **返回Integer或Long类型**
  - 与修改语句不同，select语句必须指定返回值类型，且mapper映射文件中的返回值类型需要与mapper接口中声明的返回值类型一致，都是int或long；
  - mapper接口中的返回值类型可以是Integer、Long,也可以是int、long不能返回Boolean类型

- **不能返回Boolean类型**

  - select语句不能返回Boolean类型

- **返回自定义类型的Bean**

  - select语句返回的column值与Mapper方法返回值的属性的映射有两种方式（具体设置见下文）：
    - 通过名称实现自动映射
    - 通过resultMap标签指定映射方式

- **返回List、Set、Array类型**

  - 直接指定select方法的返回值为List<T>、Set<T>、即可，注意此时mapper映射文件中select标签的 resultType类型一定要是自定义的Bean类型，**而不是集合类型**，可以理解为resultType指定的类型对应的是数据库中的一行数据

- **返回Map类型**

  - 　返回Map类型有两种含义：
    - resultType的类型为map，mapper接口的返回值也为Map，此时返回结果将Bean的属性作为key，值作为Value返回
    - resultType的类型为非map类型（自定义类型、数值类型、String等），mapper接口的返回值为Map，且指定了mapKey,此时的返回结果以指定的mapKey作为Key,以resultType类型对象作为value

  

  #### 通过column别名实现自定义类型的自动映射

  只要保证数据库查询返回的column名称和Bean的属性名一致，Mybatis便能够实现自动映射。如：

  ```xml
      <select id="selectActorById"  resultType="canger.study.chapter04.bean.Actor">
          select actor_id as id, first_name as firstName ,last_name as lastName
          from actor
          where actor_id=#{id}
      </select>
  ```

  

  ```java
  public class Actor {
      Long id;
      String firstName;
      String lastName;
  }
  ```

   需要特别说明的有3个地方：

  1. 返回值Bean无需为属性设置getter/setter方法，Mybatis依然能够为其赋值；
  2. 如果column名称和Bean的属性名只有部分相同，那么只有名称相同的属性会被赋值，Bean依然会被创建；
  3. 如果column名称与所有Bean的属性名都不相同，则select语句会返回null值，即使数据库中存在符合查询条件的记录；

#### 设置mapUnderscoreToCamelCase属性实现自动映射

```xml
 <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
```

- 如果数据库采用了严格的under score命名规则，则通过设置mapUnderscoreToCamelCase属性并可实现column到实体类property的自动映射；
- 实际项目中，可将命名不规范的column通过resultMap进行显示声明，符合命名规范的进行自动映射

#### 通过resultMap标签指定自定义类型的映射方式

需要在mapper映射文件中定义 resultMap标签

```xml
<resultMap id="actorResultMap" type="canger.study.chapter04.bean.Actor">
     <id column="id" property="id" jdbcType="BIGINT" javaType="long" 			               									typeHandler="org.apache.ibatis.type.LongTypeHandler"/>
     <result column="firstName" property="firstName" jdbcType="VARCHAR" javaType="string" 
                typeHandler="org.apache.ibatis.type.StringTypeHandler"/>
     <result column="lastName" property="lastName" jdbcType="VARCHAR" javaType="string" 
                typeHandler="org.apache.ibatis.type.StringTypeHandler"/>
</resultMap>
```

```xml
   <select id="selectActorById"  resultMap="actorResultMap">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id=#{id}
   </select>
```

需要注意的几点

- id或result标签中的column属性一定要使用SQL select 语句中最终返回到数据库客户端的列名，如对于select actor_id as id，在resultMap中需要使用 id而不是 actor_id；
- 无需将查询返回的所有column都进行显示声明，没有在resultMap中声明映射关系的column将进行自动映射
- typeHandler属性：如果使用MyBatis提供的typeHandler,则一定要是用全类名，MyBatis默认没有为typeHandler其注册别名；
- jdbcType 、 javaType和typeHandler的对照关系，以及jdbcType和数据库类型的对照关系参见 ***Mybatis中jdbcType和javaType、typeHandler的对照关系(https://www.cnblogs.com/canger/p/9979606.html)***

MyBatis注册的默认TypeHandler参见类 org.apache.ibatis.type.TypeHandlerRegistry，具体如下

```java
public TypeHandlerRegistry() {
    register(Boolean.class, new BooleanTypeHandler());
    register(boolean.class, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());

    register(Byte.class, new ByteTypeHandler());
    register(byte.class, new ByteTypeHandler());
    register(JdbcType.TINYINT, new ByteTypeHandler());

    register(Short.class, new ShortTypeHandler());
    register(short.class, new ShortTypeHandler());
    register(JdbcType.SMALLINT, new ShortTypeHandler());

    register(Integer.class, new IntegerTypeHandler());
    register(int.class, new IntegerTypeHandler());
    register(JdbcType.INTEGER, new IntegerTypeHandler());

    register(Long.class, new LongTypeHandler());
    register(long.class, new LongTypeHandler());

    register(Float.class, new FloatTypeHandler());
    register(float.class, new FloatTypeHandler());
    register(JdbcType.FLOAT, new FloatTypeHandler());

    register(Double.class, new DoubleTypeHandler());
    register(double.class, new DoubleTypeHandler());
    register(JdbcType.DOUBLE, new DoubleTypeHandler());
  ......
}
```



### MyBatis 的mapper参数传递

简单参数传递是指：

- 传递单个基本类型参数，数字类型、String
- 传递多个基本类型参数

#### parameterType 属性可以省略；

#### 传递单个基本类型参数

SQL语句中参数的引用名称并不需要和接口中的参数名称相同，如selectActorById元素的where语句改为  where actor_id=#{abc} 也能够得到正确的结果；

```java
　　Actor selectActorById(Long id);
```

```sql
    <select id="selectActorById"  resultType="canger.study.chapter04.bean.Actor">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id=#{abc}
    </select>
```

```java
Actor actor = mapper.selectActorById(1L)
```



#### 传递多个基本类型参数

#####  使用map进行参数传递

```java
Boolean insertActorByMap(Map map);
```

```xml
   <insert id="insertActorByMap" parameterType="map">
        insert into actor(first_name,last_name) values (#{firstName},#{lastName})
    </insert>
```

```java
Map<String,String> map = new HashMap<String, String>(){
    {
        put("firstName","James");
        put("lastName","Harden");
    }
};
Boolean result = mapper.insertActorByMap(map);
System.out.println(result);
```



##### 通过param1、param2进行多参数引用（此时接口方法中的参数可以使用任意名称，SQL语句中使用 param1、param2进行引用）

```
　Boolean insertActorByString(String var1,String var2);
```

```xml
<insert id="insertActorByString">
        insert into actor(first_name,last_name) values (#{param1},#{param2})
</insert>
```

```java
   Boolean result = mapper.insertActorByString("James", "Harden");
   System.out.println(result);
```



##### 通过命名参数进行引用，通过使用@Parma注解，可以在SQL语句中使用命名参数引用，注意命名参数区分大小写

```java
Boolean insertActorByParamString(@Param("firstName")String var1, @Param("lastName")String var2);
```

```xml
 <insert id="insertActorByParamString">
        insert into actor(first_name,last_name) values (#{firstName},#{lastName})
 </insert>
```

```java
 	Boolean result = mapper.insertActorByParamString("James", "Harden");
  System.out.println(result)
```



##### 命名参数和非命名参数混合使用，此时非命名参数只能采用 param1，param2...的方式使用，命名参数即能采用@Param指定名字也能使用 param1，param2...的方式进行引用

```java
Boolean insertActorByMixedString(String var1, @Param("lastName")String var2);
```

```xml
 <!--使用命名参数和非命名参数传递-->
    <insert id="insertActorByMixedString">
        insert into actor(first_name,last_name) values (#{param1},#{lastName})
    </insert>
或者

    <!--使用命名参数和非命名参数传递-->
    <insert id="insertActorByMixedString">
        insert into actor(first_name,last_name) values (#{param1},#{param2})
    </insert>
```



#### 自定义类型参数传递

传递单个自定义类型参数，定义用于传递参数的Bean，如 id=“insertActor”语句中的 canger.study.chapter04.bean.Actor，可以直接在SQL语句中使用Bean的属性名；
需要注意的是，如果此时采用命名参数（如@Param("actor")）传递单个自定义类型参数，则不能直接引用属性名，而需要采用级联引用（actor.firstName 或 param1.firstName）

```java
Boolean insertActor(Actor actor);
```

```xml
   <!--参数Bean-->
    <insert id="insertActor" parameterType="canger.study.chapter04.bean.Actor">
        insert into actor(first_name,last_name) values (#{firstName},#{lastName})
    </insert>
```

```java
Actor actor = new Actor("James","Harden");
Boolean  result = mapper.insertActor(actor);
```

#### 传递自定义参数和基本参数

这种情况和传递多个基本参数没有什么打的区别，唯一需要注意的是需要使用级联引用，才能使用自定义参数属性的值



#### 集合类型参数传递

##### 传递单个非命名集合参数

此时的引用规则如下：

- Set 类型参数的引用名为 collection
- List 类型参数的引用名为 collection 或 list
- Array 类型参数的引用名为 array

```java
    List<Actor> selectActorByIdList( List<Long> list);
    List<Actor> selectActorByIdSet( Set<Long> set);
    List<Actor> selectActorByIdArray( Long[] array);
```

```xml
  <select id="selectActorByIdList"  resultType="canger.study.chapter04.bean.Actor">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id  in
        <foreach collection="list" item="actorId" index="index"
                 open="(" close=")" separator=",">
            #{actorId}
        </foreach>
    </select>

    <select id="selectActorByIdSet"  resultType="canger.study.chapter04.bean.Actor">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id  in
        <foreach collection="collection" item="actorId" index="index"
                 open="(" close=")" separator=",">
            #{actorId}
        </foreach>
    </select>


    <select id="selectActorByIdArray"  resultType="canger.study.chapter04.bean.Actor">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id  in
        <foreach collection="array" item="actorId" index="index"
                 open="(" close=")" separator=",">
            #{actorId}
        </foreach>
    </select>

```



```java
 List<Actor> actors = mapper.selectActorByIdList(asList(1L, 2L, 3L));
 System.out.println(actors);
 actors =mapper.selectActorByIdSet(new HashSet<Long>(){
      {
           add(4L);
           add(6L);
           add(5L);
       }
 });
 mapper.selectActorByIdArray(new Long[]{7L,8L,9L});
```

##### 单个集合命名参数传递

此时默认名称 collection、list、array均失效，只能使用指定的名称或param1进行引用

引用集合中的单个元素，通过索引操作符 []进行引用

```java
List<Actor> selectActorByIdList( @Param("actorList") List<Long> list);
```

```xml
    <select id="selectActorByIdList"  resultType="canger.study.chapter04.bean.Actor">
        select actor_id as id, first_name as firstName ,last_name as lastName
        from actor
        where actor_id  = #{actorList[0]}
    </select>
```

##### 集合参数和其他参数一起使用

无特殊之处

例如：

```java
int deleteByExampleAndList(@Param("idList") List<Integer> idList,@Param("mediaCopyrightClassId") Integer mediaCopyrightClassId);
```

```xml
 <delete id="deleteByExampleAndList" parameterType="java.lang.Integer">
    delete  from `cms_media_hotel_copyright_class`
    where mediaCopyrightClassId = #{mediaCopyrightClassId}
    and hotelId in
    <foreach collection="idList" open="(" separator="," close=")" item="hotelId">
      #{hotelId}
    </foreach>
  </delete>
```

如果不使用@Param 则需要mapper接口参数的顺序和相应的xml文件的参数顺序相一致。



### MyBatis官方代码生成工具

>在我们使用MyBatis的过程中，如果所有实体类和单表CRUD代码都需要手写，那将会是一件相当麻烦的事情。MyBatis官方代码生成器MyBatis Generator可以帮助我们解决这个问题，在我的开源项目mall中也是使用的这个代码生成器，用习惯了也挺不错的。本文将介绍MyBatis Generator的使用方法及使用技巧，希望对大家有所帮助！

#### 简介

MyBatis Generator（简称MBG）是MyBatis官方提供的代码生成工具。可以通过数据库表直接生成实体类、单表CRUD代码、mapper.xml文件，从而解放我们的双手！



#### 开始使用

>首先我们通过一个入门示例将MBG用起来，该示例会包含基础的CRUD操作。

#### 集成MBG

- 在`pom.xml`中添加如下依赖，主要添加了MyBatis、PageHelper、Druid、MBG和MySQL驱动等依赖；

```xml
<dependencies>
    <!--SpringBoot整合MyBatis-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.3</version>
    </dependency>
    <!--MyBatis分页插件-->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.3.0</version>
    </dependency>
    <!--集成druid连接池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <!-- MyBatis 生成器 -->
    <dependency>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-core</artifactId>
        <version>1.4.0</version>
    </dependency>
    <!--Mysql数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>
</dependencies>
```

- 在`application.yml`中对数据源和MyBatis的`mapper.xml`文件路径进行配置，这里做个约定，MBG生成的放在`resources/com/**/mapper`目录下，自定义的放在`resources/dao`目录下；

  ```yml
  # 数据源配置
  spring:
    datasource:
      url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
      username: root
      password: root
  
  # MyBatis mapper.xml路径配置
  mybatis:
    mapper-locations:
      - classpath:dao/*.xml
      - classpath*:com/**/mapper/*.xml
  ```

- 添加Java配置，用于扫码Mapper接口路径，这里还有个约定，MBG生成的放在`mapper`包下，自定义的放在`dao`包下。

```java
/**
 * MyBatis配置类
 * Created by macro on 2019/4/8.
 */
@Configuration
@MapperScan({"com.macro.mall.tiny.mbg.mapper","com.macro.mall.tiny.dao"})
public class MyBatisConfig {
}
```

#### 使用代码生成器

- 在使用MBG生成代码前，我们还需要对其进行一些配置，首先在`generator.properties`文件中配置好数据库连接信息；

```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
jdbc.userId=root
jdbc.password=root
```

- 然后在`generatorConfig.xml`文件中对MBG进行配置，配置属性说明直接参考注释即可；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="generator.properties"/>
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <!--生成mapper.xml时覆盖原文件-->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
        <!-- 为模型生成序列化方法-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!--可以自定义生成model的代码注释-->
        <commentGenerator type="com.macro.mall.tiny.mbg.CommentGenerator">
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接-->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        <!--指定生成model的路径-->
        <javaModelGenerator targetPackage="com.macro.mall.tiny.mbg.model" targetProject="mall-tiny-generator\src\main\java"/>
        <!--指定生成mapper.xml的路径-->
        <sqlMapGenerator targetPackage="com.macro.mall.tiny.mbg.mapper" targetProject="mall-tiny-generator\src\main\resources"/>
        <!--指定生成mapper接口的的路径-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.macro.mall.tiny.mbg.mapper"
                             targetProject="mall-tiny-generator\src\main\java"/>
        <!--生成全部表tableName设为%-->
        <table tableName="ums_admin">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
        <table tableName="ums_role">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
        <table tableName="ums_admin_role_relation">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
        <table tableName="ums_resource">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
        <table tableName="ums_resource_category">
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

- 这里值得一提的是`targetRuntime`这个属性，设置不同的属性生成的代码和生成代码的使用方式会有所不同，常用的有`MyBatis3`和`MyBatis3DynamicSql`两种，这里使用的是`MyBatis3`；
- 如果你想自定义MBG生成的代码的话，可以自己写一个CommentGenerator来继承DefaultCommentGenerator，这里我自定义了实体类代码的生成，添加了Swagger注解的支持；

```java
package com.macro.mall.tiny.mbg;

import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.dom.java.CompilationUnit;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.FullyQualifiedJavaType;
import org.mybatis.generator.internal.DefaultCommentGenerator;
import org.mybatis.generator.internal.util.StringUtility;

import java.util.Properties;

/**
 * 自定义注释生成器
 * Created by macro on 2018/4/26.
 */
public class CommentGenerator extends DefaultCommentGenerator {
    private boolean addRemarkComments = false;
    private static final String EXAMPLE_SUFFIX="Example";
    private static final String MAPPER_SUFFIX="Mapper";
    private static final String API_MODEL_PROPERTY_FULL_CLASS_NAME="io.swagger.annotations.ApiModelProperty";

    /**
     * 设置用户配置的参数
     */
    @Override
    public void addConfigurationProperties(Properties properties) {
        super.addConfigurationProperties(properties);
        this.addRemarkComments = StringUtility.isTrue(properties.getProperty("addRemarkComments"));
    }

    /**
     * 给字段添加注释
     */
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                                IntrospectedColumn introspectedColumn) {
        String remarks = introspectedColumn.getRemarks();
        //根据参数和备注信息判断是否添加备注信息
        if(addRemarkComments&&StringUtility.stringHasValue(remarks)){
            //数据库中特殊字符需要转义
            if(remarks.contains("\"")){
                remarks = remarks.replace("\"","'");
            }
            //给model的字段添加swagger注解
            field.addJavaDocLine("@ApiModelProperty(value = \""+remarks+"\")");
        }
    }

    @Override
    public void addJavaFileComment(CompilationUnit compilationUnit) {
        super.addJavaFileComment(compilationUnit);
        //只在model中添加swagger注解类的导入
        String fullyQualifiedName = compilationUnit.getType().getFullyQualifiedName();
        if(!fullyQualifiedName.contains(MAPPER_SUFFIX)&&!fullyQualifiedName.contains(EXAMPLE_SUFFIX)){
            compilationUnit.addImportedType(new FullyQualifiedJavaType(API_MODEL_PROPERTY_FULL_CLASS_NAME));
        }
    }
}
```

- 最后我们写个Generator类用于生成代码，直接运行main方法即可生成所有代码；

```java
/**
 * 用于生产MBG的代码
 * Created by macro on 2018/4/26.
 */
public class Generator {
    public static void main(String[] args) throws Exception {
        //MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        //当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        //读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}
```

- 一切准备就绪，执行main方法，生成代码结构信息如下。

  ![img](mybatis.assets/3185693477-7c2b2682ccc23524_fix732.png)

#### 实现基本的CRUD操作

- 查看下MBG生成的Mapper接口，发现已经包含了基本的CRUD方法，具体SQL实现也已经在mapper.xml中生成了，单表CRUD直接调用对应方法即可；

```java
public interface UmsAdminMapper {
    long countByExample(UmsAdminExample example);

    int deleteByExample(UmsAdminExample example);

    int deleteByPrimaryKey(Long id);

    int insert(UmsAdmin record);

    int insertSelective(UmsAdmin record);

    List<UmsAdmin> selectByExample(UmsAdminExample example);

    UmsAdmin selectByPrimaryKey(Long id);

    int updateByExampleSelective(@Param("record") UmsAdmin record, @Param("example") UmsAdminExample example);

    int updateByExample(@Param("record") UmsAdmin record, @Param("example") UmsAdminExample example);

    int updateByPrimaryKeySelective(UmsAdmin record);

    int updateByPrimaryKey(UmsAdmin record);
}
```

- 生成代码中有一些Example类，比如UmsAdminExample，我们可以把它理解为一个条件构建器，用于构建SQL语句中的各种条件；

![img](mybatis.assets/1709080061-55460948d90a7b17_fix732.png)

- 利用好MBG生成的代码即可完成单表的CRUD操作了，比如下面最常见的操作。

```java
/**
 * 后台用户管理Service实现类
 * Created by macro on 2020/12/8.
 */
@Service
public class UmsAdminServiceImpl implements UmsAdminService {
    @Autowired
    private UmsAdminMapper adminMapper;
    @Autowired
    private UmsAdminDao adminDao;

    @Override
    public void create(UmsAdmin entity) {
        adminMapper.insert(entity);
    }

    @Override
    public void update(UmsAdmin entity) {
        adminMapper.updateByPrimaryKeySelective(entity);
    }

    @Override
    public void delete(Long id) {
        adminMapper.deleteByPrimaryKey(id);
    }

    @Override
    public UmsAdmin select(Long id) {
        return adminMapper.selectByPrimaryKey(id);
    }

    @Override
    public List<UmsAdmin> listAll(Integer pageNum, Integer pageSize) {
        PageHelper.startPage(pageNum, pageSize);
        return adminMapper.selectByExample(new UmsAdminExample());
    }
}
```

#### 进阶使用

>想要用好MBG，上面的基础操作是不够的，还需要一些进阶的使用技巧。

##### 条件查询

>使用Example类构建查询条件可以很方便地实现条件查询。

- 这里以按用户名和状态查询后台用户并按创建时间降序排列为例，SQL实现如下；

```sql
SELECT
    id,
    username,
    PASSWORD,
    icon,
    email,
    nick_name,
    note,
    create_time,
    login_time,
STATUS 
FROM
    ums_admin 
WHERE
    ( username = 'macro' AND STATUS IN ( 0, 1 ) ) 
ORDER BY
    create_time DESC;
```

- 对应的Java代码实现如下。

```java
/**
 * 后台用户管理Service实现类
 * Created by macro on 2020/12/8.
 */
@Service
public class UmsAdminServiceImpl implements UmsAdminService {
    
    @Override
    public List<UmsAdmin> list(Integer pageNum, Integer pageSize, String username, List<Integer> statusList) {
        PageHelper.startPage(pageNum, pageSize);
        UmsAdminExample umsAdminExample = new UmsAdminExample();
        UmsAdminExample.Criteria criteria = umsAdminExample.createCriteria();
        if(StrUtil.isNotEmpty(username)){
            criteria.andUsernameEqualTo(username);
        }
        criteria.andStatusIn(statusList);
        umsAdminExample.setOrderByClause("create_time desc");
        return adminMapper.selectByExample(umsAdminExample);
    }
}
```

##### 子查询

>使用MBG生成的代码并不能实现子查询，需要自己手写SQL实现。

- 这里以按角色ID查询后台用户为例，首先定义一个UmsAdminDao接口，这里约定下Dao里面存放的方法都是自定义SQL实现的方法，首先在Dao接口中添加`subList`方法；

```java
/**
 * Created by macro on 2020/12/9.
 */
public interface UmsAdminDao {

    List<UmsAdmin> subList(@Param("roleId") Long roleId);
}
```

- 然后创建一个UmsAdminDao.xml文件，对应UmsAdminDao接口的SQL实现，写好对应的SQL实现，注意使用的resultMap MBG已经帮我们生成好了，无需自己手写。

```xml
<select id="subList" resultMap="com.macro.mall.tiny.mbg.mapper.UmsAdminMapper.BaseResultMap">
    SELECT *
    FROM ums_admin
    WHERE id IN (SELECT admin_id FROM ums_admin_role_relation WHERE role_id = #{roleId})
</select>
```



##### 	Group和Join查询

> Group和Join查询也不能使用MBG生成的代码实现。

- 这里以按角色统计后台用户数量为例，首先在Dao接口中添加`groupList`方法；

```java
public interface UmsAdminDao {

    List<RoleStatDto> groupList();
}
```

- 然后在mapper.xml中添加对应的SQL实现，我们可以通过给查询出来的列起别名，直接把列映射到resultType所定义的对象中去。

```xml
<select id="groupList" resultType="com.macro.mall.tiny.domain.RoleStatDto">
    SELECT ur.id        AS roleId,
           ur.NAME      AS roleName,
           count(ua.id) AS count
    FROM ums_role ur
             LEFT JOIN ums_admin_role_relation uarr ON ur.id = uarr.role_id
             LEFT JOIN ums_admin ua ON uarr.admin_id = ua.id
    GROUP BY ur.id;
</select>
```

##### 条件删除

> 条件删除很简单，直接使用Example类构造删除条件即可。

- 这里以按用户名删除后台用户为例，SQL实现如下；

```sql
DELETE 
FROM
    ums_admin 
WHERE
    username = 'andy';
```

- 对应Java中的实现为。

```java
/**
 * 后台用户管理Service实现类
 * Created by macro on 2020/12/8.
 */
@Service
public class UmsAdminServiceImpl implements UmsAdminService {
    @Override
    public void deleteByUsername(String username) {
        UmsAdminExample example = new UmsAdminExample();
        example.createCriteria().andUsernameEqualTo(username);
        adminMapper.deleteByExample(example);
    }
}
```

##### 条件修改

>条件删除很简单，直接使用Example类构造修改条件，然后传入修改参数即可。

```sql
UPDATE ums_admin 
SET STATUS = 1 
WHERE
    id IN ( 1, 2 );
```

- 对应Java中的实现为。

```java
/**
 * 后台用户管理Service实现类
 * Created by macro on 2020/12/8.
 */
@Service
public class UmsAdminServiceImpl implements UmsAdminService {
    @Override
    public void updateByIds(List<Long> ids, Integer status) {
        UmsAdmin record = new UmsAdmin();
        record.setStatus(status);
        UmsAdminExample example = new UmsAdminExample();
        example.createCriteria().andIdIn(ids);
        adminMapper.updateByExampleSelective(record,example);
    }
}
```

##### 一对多查询

>一对多查询无法直接使用MBG生成的代码实现，需要手写SQL实现，并使用resultMap来进行结果集映射。

- 这里以按ID查询后台用户信息（包含对应角色列表）为例，先在Dao接口中添加`selectWithRoleList`方法；

```java
/**
 * Created by macro on 2020/12/9.
 */
public interface UmsAdminDao {

    AdminRoleDto selectWithRoleList(@Param("id") Long id);
}
```

- 然后在mapper.xml中添加对应的SQL实现，这里有个小技巧，可以给角色表查询出来的列取个别名，添加一个`role_`前缀；

```sql
<select id="selectWithRoleList" resultMap="AdminRoleResult">
    SELECT ua.*,
           ur.id          AS role_id,
           ur.NAME        AS role_name,
           ur.description AS role_description,
           ur.create_time AS role_create_time,
           ur.STATUS      AS role_status,
           ur.sort        AS role_sort
    FROM ums_admin ua
             LEFT JOIN ums_admin_role_relation uarr ON ua.id = uarr.admin_id
             LEFT JOIN ums_role ur ON uarr.role_id = ur.id
    WHERE ua.id = #{id}
</select>
```

- 然后定义一个叫做`AdminRoleResult`的ResultMap，通过`collection`标签直接将以`role_`开头的列映射到UmsRole对象中去即可。

```xml
<resultMap id="AdminRoleResult" type="com.macro.mall.tiny.domain.AdminRoleDto"
           extends="com.macro.mall.tiny.mbg.mapper.UmsAdminMapper.BaseResultMap">
    <collection property="roleList" resultMap="com.macro.mall.tiny.mbg.mapper.UmsRoleMapper.BaseResultMap"
                columnPrefix="role_">
    </collection>
</resultMap>
```

##### 一对一查询

>一对一查询无法直接使用MBG生成的代码实现，需要手写SQL实现，并使用resultMap来进行结果集映射。

- 这里以按ID查询资源信息（包括分类信息）为例，先在Dao接口中添加`selectResourceWithCate`方法；

```java
/**
 * Created by macro on 2020/12/9.
 */
public interface UmsAdminDao {

    ResourceWithCateDto selectResourceWithCate(@Param("id")Long id);
}
```

- 然后在mapper.xml中添加对应的SQL实现，可以给分类表查询出来的列取个别名，添加一个`cate_`前缀；

```xml
<select id="selectResourceWithCate" resultMap="ResourceWithCateResult">
    SELECT ur.*,
           urc.id          AS cate_id,
           urc.`name`      AS cate_name,
           urc.create_time AS cate_create_time,
           urc.sort        AS cate_sort
    FROM ums_resource ur
             LEFT JOIN ums_resource_category urc ON ur.category_id = urc.id
    WHERE ur.id = #{id}
</select>
```

- 然后定义一个叫做`ResourceWithCateResult`的ResultMap，通过`association`标签直接将以`cate_`开头的列映射到UmsResourceCategory对象中去即可。

```sql
<resultMap id="ResourceWithCateResult" type="com.macro.mall.tiny.domain.ResourceWithCateDto"
           extends="com.macro.mall.tiny.mbg.mapper.UmsResourceMapper.BaseResultMap">
    <association property="resourceCategory"
                 resultMap="com.macro.mall.tiny.mbg.mapper.UmsResourceCategoryMapper.BaseResultMap"
                 columnPrefix="cate_">
    </association>
</resultMap>
```

#### 总结

总的来说MyBatis官方代码生成器MBG还是很强大的，可以生成一些常用的单表CRUD方法，减少了我们的工作量。但是对于子查询、多表查询和一些复杂查询支持有点偏弱，依然需要在mapper.xml中手写SQL实现。