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

#### select 语句的返回类型

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

