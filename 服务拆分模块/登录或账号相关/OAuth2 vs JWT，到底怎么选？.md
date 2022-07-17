# OAuth2 vs JWT，到底怎么选？



##  **背景**

本文会详细描述两种通用的保证API安全性的方法：OAuth2和JSON Web Token (JWT)

**假设:**

- 你已经或者正在实现API；
- 你正在考虑选择一个合适的方法保证**API的安全性**；



## JWT和OAuth2比较？

要比较JWT和OAuth2？首先要明白一点就是，这两个根本没有可比性，是两个完全不同的东西。

- JWT是一种认证协议

  JWT提供了一种用于发布接入令牌（Access Token),并对发布的签名接入令牌进行验证的方法。令牌（Token）本身包含了一系列声明，应用程序可以根据这些声明限制用户对资源的访问。

- OAuth2是一种授权框架

  OAuth2是一种授权框架，提供了一套详细的授权机制（指导）。用户或应用可以通过公开的或私有的设置，授权第三方应用访问特定资源。既然JWT和OAuth2没有可比性，为什么还要把这两个放在一起说呢？实际中确实会有很多人拿JWT和OAuth2作比较。标题里把这两个放在一起，确实有误导的意思。很多情况下，在讨论OAuth2的实现时，会把JSON Web Token作为一种认证机制使用。这也是为什么他们会经常一起出现。

先来搞清楚JWT和OAuth2究竟是干什么的～





## **JSON Web Token (JWT)**

JWT在标准中是这么定义的：

> JSON Web Token (JWT) is a compact URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature (JWS). -RFC7519 https://tools.ietf.org/html/rfc7519

JWT是一种安全标准。基本思路就是用户提供用户名和密码给认证服务器，服务器验证用户提交信息信息的合法性；如果验证成功，会产生并返回一个Token（令牌），用户可以使用这个token访问服务器上受保护的资源。

一个token的例子:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

**一个token包含三部分：**

```
header.claims.signature
```

为了安全的在url中使用，所有部分都 base64 URL-safe进行编码处理。

**Header头部分**头部分简单声明了类型(JWT)以及产生签名所使用的算法。

```
{  "alg" : "AES256",  "typ" : "JWT"}
```



**Claims声明**

声明部分是整个token的核心，表示要发送的用户详细信息。有些情况下，我们很可能要在一个服务器上实现认证，然后访问另一台服务器上的资源；或者，通过单独的接口来生成token，token被保存在应用程序客户端（比如浏览器）使用。

一个简单的声明（claim）的例子：

```
{  "sub": "1234567890",  "name": "John Doe",  "admin": true}
```

**Signature签名**

签名的目的是为了保证上边两部分信息不被篡改。如果尝试使用Base64对解码后的token进行修改，签名信息就会失效。一般使用一个私钥（private key）通过特定算法对Header和Claims进行混淆产生签名信息，所以只有原始的token才能与签名信息匹配。   

这里有一个重要的实现细节。只有获取了私钥的应用程序（比如服务器端应用）才能完全认证token包含声明信息的合法性。所以，永远不要把私钥信息放在客户端（比如浏览器）。





## JWT 实战

暂略





## OAuth2是什么？

相反，OAuth2不是一个标准协议，而是一个安全的授权框架。它详细描述了系统中不同角色、用户、服务前端应用（比如API），以及客户端（比如网站或移动App）之间怎么实现相互认证。

>The OAuth 2.0 authorization framework enables a third-party application to obtain limited access to an HTTP service, either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf. -RFC6749 https://tools.ietf.org/html/rfc6749

这里简单说一下涉及到的基本概念。

**Roles角色**

应用程序或者用户都可以是下边的任何一种角色：

- 资源拥有者
- 资源服务器
- 客户端应用
- 认证服务器

**Client Types客户端类型**

这里的客户端主要指API的使用者。它可以是的类型：

- 私有的
- 公开的

**Client Profile客户端描述**

OAuth2框架也指定了集中客户端描述，用来表示应用程序的类型：

- Web应用
- 用户代理
- 原声应用

**Authorization Grants认证授权**

认证授权代表资源拥有者授权给客户端应用程序的一组权限，可以是下边几种形式：

- 授权码
- 隐式授权
- 资源拥有者密码证书
- 客户端证书
- Endpoints终端

**OAuth2框架需要下边几种终端：**

- 认证终端
- Token终端
- 重定向终端

从上边这些应该可以看出，OAuth2定义了一组相当复杂的规范。

#### 使用HTTPS保护用户密码

在进一步讨论OAuth2和JWT的实现之前，有必要说一下，两种方案都需要SSL安全保护，也就是对要传输的数据进行加密编码。   

安全地传输用户提供的私密信息，在任何一个安全的系统里都是必要的。否则任何人都可以通过侵入私人wifi，在用户登录的时候窃取用户的用户名和密码等信息。

**一些重要的实施考虑**

在做选择之前，参考一下下边提到的几点。

**时间投入**

OAuth2是一个安全框架，描述了在各种不同场景下，多个应用之间的授权问题。有海量的资料需要学习，要完全理解需要花费大量时间。甚至对于一些有经验的开发工程师来说，也会需要大概一个月的时间来深入理解OAuth2。这是个很大的时间投入。相反，JWT是一个相对轻量级的概念。可能花一天时间深入学习一下标准规范，就可以很容易地开始具体实施。

**出现错误的风险**

OAuth2不像JWT一样是一个严格的标准协议，因此在实施过程中更容易出错。尽管有很多现有的库，但是每个库的成熟度也不尽相同，同样很容易引入各种错误。在常用的库中也很容易发现一些安全漏洞。

当然，如果有相当成熟、强大的开发团队来持续OAuth2实施和维护，可以一定程度上避免这些风险。

**社交登录的好处**

在很多情况下,使用用户在大型社交网站的已有账户来认证会方便。

如果期望你的用户可以直接使用Facebook或者Gmail之类的账户,使用现有的库会方便得多。

##  结论

做结论前，我们先来列举一下  JWT和OAuth2的主要使用场景。

### JWT使用场景

#### 无状态的分布式API

JWT的主要优势在于使用无状态、可扩展的方式处理应用中的用户会话。服务端可以通过内嵌的声明信息，很容易地获取用户的会话信息，而不需要去访问用户或会话的数据库。在一个分布式的面向服务的框架中，这一点非常有用。   

但是，如果系统中需要使用黑名单实现长期有效的token刷新机制，这种无状态的优势就不明显了。

**优势**

- 快速开发
- 不需要cookie
- JSON在移动端的广泛应用
- 不依赖于社交登录
- 相对简单的概念理解

**限制**

- Token有长度限制
- Token不能撤销
- 需要token有失效时间限制(exp)
- OAuth2使用场景



### **在作者看来两种比较有必要使用OAuth2的场景：**

#### 外包认证服务器

上边已经讨论过，如果不介意API的使用依赖于外部的第三方认证提供者，你可以简单地把认证工作留给认证服务商去做。也就是常见的，去认证服务商（比如facebook）那里注册你的应用，然后设置需要访问的用户信息，比如电子邮箱、姓名等。当用户访问站点的注册页面时，会看到连接到第三方提供商的入口。用户点击以后被重定向到对应的认证服务商网站，获得用户的授权后就可以访问到需要的信息，然后重定向回来。

**优势**

- 快速开发
- 实施代码量小
- 维护工作减少

#### 大型企业解决方案

如果设计的API要被不同的App使用，并且每个App使用的方式也不一样，使用OAuth2是个不错的选择。

考虑到工作量，可能需要单独的团队，针对各种应用开发完善、灵活的安全策略。当然需要的工作量也比较大！这一点，OAuth2的作者也指出过：

>To be clear, OAuth 2.0 at the hand of a developer with deep understanding of web security will likely result is a secure implementation. However, at the hands of most developers – as has been the experience from the past two years – 2.0 is likely to produce insecure implementations. hueniverse - OAuth 2.0 and the Road to Hell

**优势**

- 灵活的实现方式
- 可以和JWT同时使用
- 可针对不同应用扩展

**参考：**

- http://jwt.io - JWT官方网站，也可以查看到使用不同语言实现的库的状态。
- http://oauth.net/2/ OAuth2官方网站, 也也可以查看到使用不同语言实现的库的状态。
- OAuth 2 tutorials - Useful overview of how OAuth 2 works
- Oauth2 Spec issues Eran Hammer’s (推进OAuth标准的作者) views on what went wrong with the OAuth 2 spec process. Whatever your own opinion, good to get some framing by someone who understand’s key aspects of what make a security standard successful.
- Thoery and implemnetation: with Laravel and Angular Really informative guide to JWT in theory and in practice for Laravel and Angular.

