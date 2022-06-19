# Spring Boot 安全框架 Spring Security 入门

## 1.概述

基本上，在所有开发的系统中，都必须做认证(authentication)和授权(authorization)，以保证系统的安全性。😈 考虑到很多胖友对认证和授权有点分不清楚，艿艿这里引用一个网上有趣的例子：

> FROM [《认证 (authentication) 和授权 (authorization) 的区别》](https://www.cnblogs.com/joooy/archive/2010/08/08/1795257.html)
>
> - authentication [ɔ,θɛntɪ'keʃən] 认证
> - authorization [,ɔθərɪ'zeʃən] 授权
>
> 以打飞机举例子：
>
> - 【认证】你要登机，你需要出示你的 passport 和 ticket，passport 是为了证明你张三确实是你张三，这就是 authentication。
> - 【授权】而机票是为了证明你张三确实买了票可以上飞机，这就是 authorization。
>
> 以论坛举例子：
>
> > - 【认证】你要登录论坛，输入用户名张三，密码 1234，密码正确，证明你张三确实是张三，这就是 authentication。
> > - 【授权】再一 check 用户张三是个版主，所以有权限加精删别人帖，这就是 authorization 。

所以简单来说：认证解决“你是谁”的问题，授权解决“你能做什么”的问题。另外，在推荐阅读下[《认证、授权、鉴权和权限控制》](http://www.iocoder.cn/Fight/user_login_auth_terms/?self) 文章，更加**详细明确**。

在 Java 生态中，目前有 [Spring Security](https://spring.io/projects/spring-security) 和 [Apache Shiro](https://shiro.apache.org/) 两个安全框架，可以完成认证和授权的功能。本文，我们先来学习下 Spring Security 。其官方对自己介绍如下：

>FROM [《Spring Security 官网》](https://spring.io/projects/spring-security)
>
>Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
>Spring Security 是一个功能强大且高度可定制的身份验证和访问控制框架。它是用于保护基于 Spring 的应用程序。
>
>Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements
>Spring Security 是一个框架，侧重于为 Java 应用程序提供身份验证和授权。与所有 Spring 项目一样，Spring 安全性的真正强大之处，在于它很容易扩展以满足定制需求。



## 2. 快速入门

>示例代码对应仓库：[lab-01-springsecurity-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-01-spring-security/lab-01-springsecurity-demo) 。

在本小节中，我们来快速入门下 Spring Security ，实现访问 API 接口时，需要首先进行登录，才能进行访问。





## 3. 进阶使用

>示例代码对应仓库：[lab-01-springsecurity-demo-role](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-01-spring-security/lab-01-springsecurity-demo-role) 。

在[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Spring-Security/?self#)中，我们很**快速**的完成了 Spring Security 的入门。本小节，我们将会自定义 Spring Security 的配置，实现**权限控制**。

考虑到不污染上述的示例，我们新建一个 [lab-01-springsecurity-demo-role](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-01-spring-security/lab-01-springsecurity-demo-role) 项目。





## 4. 整合 Spring Session

参见[《芋道 Spring Boot 分布式 Session 入门》](http://www.iocoder.cn/Spring-Boot/Distributed-Session/?self)文章的[「5. 整合 Spring Security」](https://www.iocoder.cn/Spring-Boot/Spring-Security/?self#)小节。





## 5. 整合 OAuth2

参见[《芋道 Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/?self)文章，详细到爆炸。



## 6. 整合 JWT

参见[《前后端分离 SpringBoot + SpringSecurity + JWT + RBAC 实现用户无状态请求验证》](http://www.iocoder.cn/Fight/Separate-SpringBoot-SpringSecurity-JWT-RBAC-from-front-and-rear-to-achieve-user-stateless-request-authentication/?self)文章，写的很不错。





## 7. 项目实战

在开源项目翻了一圈，找到一个相对合适项目 [RuoYi-Vue](https://gitee.com/y_project/RuoYi-Vue) 。主要以下几点原因：