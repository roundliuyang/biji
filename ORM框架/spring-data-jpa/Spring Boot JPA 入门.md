# Spring Boot JPA 入门



## 1.概述

相信不是胖友之前有了解过 JPA,Hibernat,那么JPA、Hebernate、Spring Data  JPA这三者是什么关系呢？



JPA，全称 Java Persistence API,是JAVA 定义的 Java ORM 以及实体操作API 的白标准。正如最早学习JDBC 规范，Java 自身并未提供相关的实现，而是MySQL 提供MySQL [mysql-connector-java](https://mvnrepository.com/artifact/mysql/mysql-connector-java) 驱动，Oracle 提供 [oracle-jdbc](https://mvnrepository.com/artifact/oracle/oracle-jdbc) 驱动。而实现 JPA 规范的有：

- Hibernate ORM
- [Oracle TopLink](https://www.oracle.com/middleware/technologies/top-link.html)
- [Apache OpenJPA](http://openjpa.apache.org/)



Spirng Data JPA ,是 Spring Data提供的一套简化的JPA 开发的框架。

- 内置 CRUD、分页、排序等功能的操作。
- 根据约定好的[方法名](https://docs.spring.io/spring-data/jpa/docs/2.2.0.RELEASE/reference/html/#jpa.query-methods.query-creation)规则，自动生成对应的查询操作。
- 使用 `@Query` 注解，自定义 SQL 。



所以，绝大多数情况下，我们无须编写代码，直接调用JPA 的API,也因此，在我们使用的Sprng Data JPA 的项目中，如果想要替换底层使用的JPA 实现框架，在未使用相关JAP 实现框架的 特殊特性的情况下，可以透明替换。

> 关于这一点，我们在 [《芋道 Spring Boot Redis 入门》](http://www.iocoder.cn/Spring-Boot/Redis/?self) 中，已经看到 [Spring Data Redis](https://spring.io/projects/spring-data-redis) 也是已经看到这样的好处。

总的来说，就是如下一张图：

FROM [《spring data jpa hibernate jpa 三者之间的关系》](https://www.cnblogs.com/xiaoheike/p/5150553.html)

![img](Spring Boot JPA 入门.assets/e55f8f7084a6d6e9e908de509c7f5a54)



当然，绝大多数情况下，我们使用的 JPA 实现框架是 [Hibernate ORM](https://hibernate.org/orm/) 。所以整个调用过程是：

>应用程序 => Repository => Spring Data JPA => Hibernate



## 2. 快速入门

>示例代码对应仓库：[lab-13-jpa](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-13-spring-data-jpa/lab-13-jpa) 。

本小节，我们会使用 [`spring-boot-starter-data-jpa`](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa) 自动化配置 Spring Data JPA 。同时，演示 Spring Data JPA 的 CRUD 的操作。





## 3. 分页操作

示例代码对应仓库：[lab-13-jpa](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-13-spring-data-jpa/lab-13-jpa) 。

Spring Data 提供 [`org.springframework.data.repository.PagingAndSortingRepository`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/PagingAndSortingRepository.java) 接口，继承 CrudRepository 接口，在 CRUD 操作的基础上，额外提供分页和排序的操作。代码如下：

```java
// PagingAndSortingRepository.java

public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	Iterable<T> findAll(Sort sort); // 排序操作

	Page<T> findAll(Pageable pageable); // 分页操作

}
```



## 4. 基于方法名查询

示例代码对应仓库：[lab-13-jpa](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-13-spring-data-jpa/lab-13-jpa) 。

在 Spring Data 中，支持根据方法名作生成对应的查询（`WHERE`）条件，进一步进化我们使用 JPA ，具体是方法名以 `findBy`、`existsBy`、`countBy`、`deleteBy` 开头，后面跟具体的条件。具体的规则，在 [《Spring Data JPA —— Query Creation》](https://docs.spring.io/spring-data/jpa/docs/2.2.0.RELEASE/reference/html/#jpa.query-methods.query-creation) 文档中，已经详细提供。如下：



| 关键字                 | 方法示例                                                     | JPQL snippet                                                 |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection ages)`                               | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection ages)`                            | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

- 注意，如果我们有排序需求，可以使用 `OrderBy` 关键字。



## 5.基于注解查询

虽然 Spring Data JPA 提供了非常强大的功能，可以满足绝大多数业务场景下的 CRUD 操作，但是可能部分情况下，我们可以使用在方法上添加 [`org.springframework.data.jpa.repository.@Query`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html) 注解，实现自定义的 SQL 操作。

如果是更新或删除的SQL操作，需要额外在方法上添加[`org.springframework.data.jpa.repository.@Modifying`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Modifying.html) 注解。

## 666. 彩蛋

😈 本文仅仅是 Spring Data JPA 的简单入门，还有部分内容，胖友可以自己在去学习下：

- [《Using JPA Named Queries》](https://docs.spring.io/spring-data/jpa/docs/2.2.0.RELEASE/reference/html/#jpa.query-methods.named-queries) ，可以使用 XML 自定义 SQL 操作。

- [《Spring Data JPA 实现逻辑删除》](https://my.oschina.net/weechang93/blog/1576594) ，绝大多数业务场景下，我们不会使用 `DELETE` 物理删除，而是通过标志位进行逻辑删除。

- 多表查询

  - 方式一：[《JPA 多表查询的解决办法》](https://blog.wuwii.com/jpa-query-muti.html)
  - 方式二：[《JPA 多表关联查询》](https://blog.csdn.net/moshowgame/article/details/80058270)

- [《Spring Data JPA 使用 Example 快速实现动态查询》](https://blog.csdn.net/long476964/article/details/79677526)

  > 艿艿，如果在这种情况下，Repository 需要继承 JpaRepository 接口。

如果胖友想找一个完整的，使用 JPA 的项目，可以看看 [Apollo](https://github.com/ctripcorp/apollo) 。它是携程开源的配置中心，目前最好用的配置中心，基本没有之一，嘿嘿。























​                    	