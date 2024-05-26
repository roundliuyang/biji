# Kong 极简入门（微服务网关）

## 1. 概述

>如下 Kong 简介的内容，基于 [Kong Github](https://github.com/Kong/kong)、[Kong 官方文档](https://docs.konghq.com/)、[《API 网关之 Kong 简介》](http://www.iocoder.cn/Fight/Introduction-to-API-gateway-Kong)梳理~
>
>另外也推荐下 Kong 的“竞品” [APISIX](http://www.iocoder.cn/APISIX/install/?self)，一款**国产**的微服务 API 网关，目前已经进入 Apache 中孵化。想要学习的胖友，可以点击[传送门](http://www.iocoder.cn/APISIX/install/?self)学习！

Kong 是由 Mashape 公司开源的**云原生**、**高性能**、**可扩展**的微服务 API 网关。它基于 [OpenResty](https://github.com/openresty/) 实现，使用 [Cassandra](http://cassandra.apache.org/) 或 [PostgreSQL](https://www.postgresql.org/) 存储数据。

![Kong Logo](Kong 极简入门（微服务网关）.assets/00.png)

> - OpenResty：通过 Lua 扩展 Nginx 实现的可伸缩的 Web 平台。
> - Cassandra：Key/Value 存储系统。
> - PostgreSQL：数据库服务器。

![Kong 整体](Kong 极简入门（微服务网关）.assets/01.png)

- Kong 提供了 **RESTful Admin API**，可用于管理员进行 Kong 的配置。



### 1.1 特性

Kong 网关具有以下的特性：

- **可扩展性**：通过简单地添加更多的服务器，可以轻松地进行横向扩展，这意味着您的平台可以在一个较低负载的情况下处理任何请求。

- **模块化**::可以通过添加新的[插件](https://docs.konghq.com/hub/)进行扩展，这些插件可以通过 RESTful Admin API 轻松配置。

  > 在 https://docs.konghq.com/hub/ 地址下，可以看到 Kong 内置了丰富的插件。

- **在任何基础架构上运行**：Kong 网关可以在任何地方都能运行。可以在云或内部网络环境中部署 Kong，包括单个或多个数据中心设置，以及 public，private 或 invite-only APIs。

  > 在 https://konghq.com/install/ 地址中，可以看到 Kong 支持在各种环境下的安装。



### 1.2 架构

Kong 的整体架构如下所示：

![Kong 整体架构](Kong 极简入门（微服务网关）.assets/02.png)

> 友情提示：自上往下看哈~注意加粗部分的字！

- Kong **RESTful 管理 API**，提供了 API、API消费者、插件、upstreams、证书等管理。
- Kong **插件**拦截请求/响应，类似于 Java Servlet 中的过滤器，实现请求/响应的 AOP 处理。
- Kong **数据存储**用于存储 Kong 集群节点信息、API、消费者、插件等信息，目前提供了 Cassandra 和 PostgreSQL 支持。如果需要高可用，建议使用 Cassandra。

- Kong **集群**中的节点通过 [gossip 协议](https://en.wikipedia.org/wiki/Gossip)自动发现其它节点。当通过一个 Kong 节点的管理 API 进行一些变更时，也会通知其他节点。每个 Kong 节点的配置信息是会**缓存**的，例如插件，那么当在某一个 Kong 节点修改了插件配置时，需要通知其他节点配置的变更。
- Kong 核心基于 **OpenResty**，使用 **Lua** 进行请求/响应的处理。



### 1.3 插件

Kong 内置了丰富的[插件](https://docs.konghq.com/hub/)，提供了强大的功能以及集成能力，同时我们又可以基于**插件机制**自行进行拓展。

![插件体系](Kong 极简入门（微服务网关）.assets/03.png)

- **Authentication 身份认证**插件：Kong 提供了 Basic Authentication、Key authentication、OAuth2.0 authentication、HMAC authentication、JWT、LDAP authentication 等等实现。
- **Security 安全控制**插件：ACL（访问控制）、CORS（跨域资源共享）、动态SSL、IP 限制、爬虫检测等等实现。
- **Traffic Control 流量控制**插件：请求限流（基于请求计数限流）、上游响应限流（根据 upstream 响应计数限流）、请求大小限制等等实现。限流支持本地、Redis 和集群三种限流模式。
- **Analytics & Monitoring 分析监控**插件：对接 Datadog、[Prometheus](http://www.iocoder.cn/Prometheus/install/?self)、[Zipkin](http://www.iocoder.cn/Zipkin/install/?self) 等等监控系统的实现。
- **Transformations 协议转换**插件：请求转换（在转发到 upstream 之前修改请求）、响应转换（在 upstream 响应返回给客户端之前修改响应）。
- **Logging 日志应用**插件：支持 TCP、UDP、HTTP、File、Syslog、StatsD、Loggly 等等方式传输日志。
- **Serverless 插件**：提供对 AWS Lambda、Azure Functions、Apache OpenWhisk、Kong 自带 Serverless Functions 等等的 Serverless 解决方案的支持。
- **Deployment 插件**



### 1.4 请求流程

为了更好地使用 Kong 网关，我们需要理解它处理 API 接口的典型请求工作流程：

![请求流程](Kong 极简入门（微服务网关）.assets/04.png)

**Kong 是每个 API 请求的入口点（Endpoint）**：

- 当 Kong 运行时，每个对 API 的请求将先被 Kong 命中，然后这个请求将会被代理转发到最终的 API 接口。
- 在请求（Requests）和响应（Responses）之间，Kong 将会执行已经事先安装和配置好的插件，授权 API 访问操作。



## 2. 快速安装



### docker 方式安装



#### 构建Kong的容器网络

```
docker network create kong-net
```



#### 搭建数据库环境

如果你使用的是PostgreSQL，想挂载卷持久化数据到宿主机。通过 `-v` 命令是不好用的。这里推荐你使用 `docker volume create` 命令来创建一个挂载。

```
docker volume create kong-volume
```

然后上面的PostgreSQL就可以通过`- v kong-volume:/var/lib/postgresql/data` 进行挂载了。

```
docker run -d --name kong-database \
    --network=kong-net \
    -p 5432:5432 \
    -v kong-volume:/var/lib/postgresql/data \
    -e "POSTGRES_USER=kong" \
    -e "POSTGRES_DB=kong" \
    -e POSTGRES_PASSWORD=123456 \
    postgres:9.6
```



#### 初始化或者迁移数据库

我们使用`docker run --rm`来初始化数据库，该命令执行后会退出容器而保留内部的数据卷（volume）。

```
docker run --rm \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_PG_USER=kong" \
    -e "KONG_PG_PASSWORD=123456" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:1.4.1 kong migrations bootstrap
```



#### 启动Kong容器

```
docker run -d --name kong \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_PG_USER=kong" \
    -e "KONG_PG_PASSWORD=123456" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001,0.0.0.0:8444 ssl" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong:1.4.1
```



#### 验证

```
curl -i http://localhost:8001/
```



### Konga 安装



```
docker volume create konga-postgresql
```

```
docker run -d --name konga-database  \
--network=kong-net  \
-p 5433:5432 \
-v  konga-postgresql:/var/lib/postgresql/data  \
-e "POSTGRES_USER=konga"  \
-e "POSTGRES_DB=konga" \
-e "POSTGRES_PASSWORD=konga" \
postgres:9.6
```



#### 初始化Konga数据库

```
docker run --rm  --network=kong-net  \
pantsel/konga:latest
-a postgres
-u postgres://konga:konga@konga-database:5432/konga
```



#### 启动Konga

```
docker run -d -p 1337:1337 \
    --network kong-net \
    -e "DB_ADAPTER=postgres" \
    -e "DB_URI=postgres://konga:konga@konga-database:5432/konga" \
    -e "NODE_ENV=production" \
    -e "DB_PASSWORD=konga" \
    --name konga \
    pantsel/konga
```



## 3.动态负载均衡

本小节，我们来实现对后端服务提供的 API 接口进行**负载均衡**。考虑到方便，艿艿在 https://github.com/YunaiV/SpringBoot-Labs 仓库提供了一个 Spring Boot 项目 [`lab-56-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-56/lab-56-demo01)，如下图所示：

![Spring Boot 项目](Kong 极简入门（微服务网关）.assets/11.png)

最终示例的效果，如下图所示：

![项目效果](Kong 极简入门（微服务网关）.assets/12.png)

最终我们使用 Kong 实现负载均衡的效果等同于如下 Nginx 的配置：

```shell
upstream demo-upstream {
	server localhost:18081 weight=100;
	server localhost:18082 weight=50;
}

server {
	listen	80;
	location /demo-api/ {
		proxy_pass http://demo-upstream;
	}
}
```

下面，我们来通过 Kong **Admin API** 进行上述的负载均衡的配置。



### 3.1 创建 upstream 和 target

① 调用 Kong Admin API `/upstreams`，创建名字为 `demo-upstream` 的 **upstream**：

```
$ curl -X POST http://127.0.0.1:8001/upstreams --data "name=demo-upstream"
```

![Kong upstream](Kong 极简入门（微服务网关）.assets/50.png)

② 调用 Kong Admin API `/upstreams/{upstream}/targets`，创建 Spring Boot 项目对应的 2 个 **target**：

> 注意，`{upstream}` 路径参数，为 upstream 的名字。

```shell
# 端口 18080 对应的 target
$ curl -X POST http://127.0.0.1:8001/upstreams/demo-upstream/targets --data "target=10.8.8.34:18080" --data "weight=100"
# 端口 28080 对应的 target
$ curl -X POST http://127.0.0.1:8001/upstreams/demo-upstream/targets --data "target=10.8.8.34:28080" --data "weight=50"
```

>友情提示：艿艿这里 Spring Boot 项目启动在 IP 为 10.8.8.34 的服务器上。

![Kong target](Kong 极简入门（微服务网关）.assets/51.png)

③ 如上的配置，效果等同于如下 Nginx 的配置：

```shell
upstream demo-upstream {
    server localhost:18081 weight=100;
    server localhost:18082 weight=50;
}
```



### 3.2 创建 service 和 route

① 调用 Kong Admin API `/services`，创建名字为 `demo-service` 的 **service**：

```shell
$ curl -X POST http://127.0.0.1:8001/services --data "name=demo-service" --data "host=demo-upstream"
```

- `host` 参数，用于设置对应的 **upstream** 的名字。

  ![Kong service](Kong 极简入门（微服务网关）.assets/52.png)

② 调用 Kong Admin API `services/${service}/routes`，创建一个请求路径为 `path` 的 **route**：

>注意，`{service}` 路径参数，为 upstream 的名字。

```
$ curl -X POST http://localhost:8001/services/demo-service/routes --data "name=demo-route" --data "paths[]=/demo-api"
```

![Kong route](Kong 极简入门（微服务网关）.assets/53.png)

![Kong route](Kong 极简入门（微服务网关）.assets/54.png)

③ 如上的配置，效果等同于如下 Nginx 的配置：

```
server {
    listen  80;
    location /demo-api/ {
        proxy_pass http://demo-upstream;
    }
}
```



### 3.3 简单测试

不断执行 `curl http://127.0.0.1:8000/demo-api/demo/echo` 命令，请求 Kong 网关来**负载均衡**转发到后端的 Spring Boot 项目，结果如下：

```
echo:18080
echo:18080
echo:28080

...
```

负载均衡请求到多个节点，符合预期~



### 4.4 简单小结

在上面，我们看到了 Kong 的 **upstream**、**target**、**service**、**route** 四个对象，一起来简单小节下：

![service + route + upstream + target](Kong 极简入门（微服务网关）.assets/55.png)

| Kong 组件    | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| **service**  | service 对应服务，可以直接指向一个 API 服务节点(`host` 参数设置为 ip + port)，也可以指定一个 upstream 实现负载均衡。简单来说，服务用于映射被转发的后端 API 的节点集合 |
| **route**    | route 对应路由，它负责匹配实际的请求，映射到 **service** 中  |
| **upstream** | upstream 对应一**组** API 节点，实现负载均衡                 |
| **target**   | target 对应一**个** API 节点                                 |

在 Kong 中，还有 Consumer、Plugin、Tag、Certificate、Target 等等对象，胖友可以阅读[《Kong 文档 —— Admin API》](https://docs.konghq.com/2.0.x/admin-api/)了解更多。

另外，胖友如果想用 Konga 通过图形界面来配置，可以参考阅读[《Kong 配置 service 和 route 实现简单 API 代理》](http://www.iocoder.cn/Fight/Configure-service-and-route-to-implement-simple-API-proxies-in-Kong)文章。





## 4. 限流限速

Kong 提供了 [Rate Limiting](https://docs.konghq.com/hub/kong-inc/rate-limiting) 插件，实现对请求的**限流**功能，避免过大的请求量过大，将后端服务打挂。

Rate Limiting 支持秒/分/小时/日/月/年多种**时间维度**的限流，并且可以组合使用。例如说：限制每秒最多 100 次请求，并且每分钟最多 1000 次请求。

Rate Limiting 支持 `consumer`、`credential`、`ip` 三种**基础维度**的限流，默认为 `consumer`。例如说：设置每个 IP 允许每秒请求的次数。计数的存储，支持使用 `local`、`cluster`、`redis` 三种方式进行存储，默认为 `cluster`：

- `local`：存储在 Nginx 本地，实现单实例限流。
- `cluster`：存储在 Cassandra 或 PostgreSQL 数据库，实现集群限流。
- `redis`：存储在 Redis 数据库，实现集群限流。

Rate Limiting 采用的限流算法是**计数器**的方式，所以无法提供类似[令牌桶算法](http://www.iocoder.cn/Eureka/rate-limiter/?self)的平滑限流能力。

> 友情提示：Kong 企业版提供了 [https://docs.konghq.com/hub/kong-inc/rate-limiting-advanced/](Rate Limiting Advanced) 插件，提供平滑限流的功能，且性能更加优秀。
>
> The Rate Limiting Advanced plugin for Kong Enterprise is a re-engineered version of the incredibly popular Kong Rate Limiting plugin, with greatly enhanced configuration options and performance.

下面，我们在[「3. 动态负载均衡」](https://www.iocoder.cn/Kong/install/#)小节的基础上，对名字为 `demo-service` 的 **service** 进行限流。



### 4.1 创建 Rate Limiting 插件

调用 Kong Admin API `services/${service}/plugins`，创建 Rate Limiting 插件的配置：

```
$ curl -X POST http://127.0.0.1:8001/services/demo-service/plugins \
    --data "name=rate-limiting"  \
    --data "config.second=1" \
    --data "config.limit_by=ip"
```

- `name` 参数，设置为 `rate-limiting` 表示使用 Rate Limiting 插件。
- `config.second` 参数，设置为 1 表示每秒允许 1 次请求。
- `config.limit_by` 参数，设置为 `ip` 表示使用 IP 基础维度的限流。

![Kong Rate Limiting 01](Kong 极简入门（微服务网关）.assets/60.png)

![Kong Rate Limiting 02](Kong 极简入门（微服务网关）.assets/61.png)

>友情提示：Rate Limiting 插件还支持给 **route**、**consumer** 设置限流配置，胖友可以阅读[《Kong 文档 —— Rate Limiting》](https://docs.konghq.com/hub/kong-inc/rate-limiting/)学习。



### 4.2 简单测试

快速使用 `curl http://127.0.0.1:8000/demo-api/demo/echo` 命令 **2** 次，会被 Kong 限流，返回结果如下：

```
{"message":"API rate limit exceeded"}
```



## 5. JWT 身份验证

略
