# Nginx





## 配置

参考文档：[nginx_百度百科 (baidu.com)](https://baike.baidu.com/item/nginx/3817705?fr=aladdin)、[Nginx中文文档](https://www.nginx.cn/doc/)、[nginx系列之一：nginx入门_王道长的技术博客-CSDN博客_nginx](https://blog.csdn.net/qq_29677867/article/details/90112120)

​	*Nginx* 是一个高性能的 HTTP 和反向代理 web 服务器。

​	因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名，其特点是占有内存少，并发能力强。

​	中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

![img](Linux%E5%9F%BA%E7%A1%80.assets/4ffce04d92a4d6cb21c1494cdfcd6dc1.jpg)





> 简单的 nginx 配置解析

~~~shell
```
server {
    listen       80;  # 监听的端口
    server_name  html5.mycool.tv;  # 这个 server 标签，监听的域名
    charset utf-8;  # 字符集
    set $HIT "HIT";  # 设置变量，日志使用

    location /html5 {  # 浏览器请求 http://html5.mycool.tv/html5/ 时
        alias  /data/html5;  # 使用服务器 /data/html5 目录，响应请求
    }
}
```


```
    set_by_lua_block $hotelId { return hotelId }  # 通过 lua，设置变量

    location = /images/offlineIcon/hotel.gif {  # 浏览器访问指定文件时
        # 重定向到 /images/offlineIcon/$hotelId.gif 文件
        rewrite ^(.*)/hotel.gif $1/$hotelId.gif last;
    }
```
~~~



> 展示目录

<img src="Linux%E5%9F%BA%E7%A1%80.assets/image-20210918134259629.png" alt="image-20210918134259629" style="zoom:50%;" />

```shell
server {
    listen       443;  # 监听443，默认的 https 端口
    server_name  videos.mycool.tv;
    include vhosts/ssl.conf;  # 导入写好的配置
    charset utf-8;
    set $HIT "HIT";

    location /video_list {
        autoindex on;  # 允许列出目录内容，常用于一些资源下载网站
        alias /data/videos;
    }
```



> account.mycool.tv.conf

```shell
server {
    listen 443 ssl;
    server_name account.mycool.tv;
    include vhosts/ssl.conf;

    location /account {
        add_header Access-Control-Allow-Origin $http_origin always;  # 跨域头
        fastcgi_hide_header Set-Cookie;  # 
        add_header Access-Control-Allow-Credentials true;  # 跨域头
        add_header Access-Control-Allow-Headers Cookie,Set-Cookie,X-Requested-With,Content-Type;  # 跨域头
        add_header Access-Control-Allow-Methods GET,POST,OPTIONS;  # 跨域头
        if ($request_method = 'OPTIONS') {  # 不知道
            return 204;
        }

        proxy_set_header Host $host;  # 将原 http 请求的域名也转发到代理服务
        # 被代理的服务获取真实请求 ip，而不是 nginx 服务器的 ip
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:8080;  # 反向代理到本机的8080端口（tomcat）
    }

    location = / {
        # 请求 https://account.mycool.tv/ 时，重定向到 https://account.mycool.tv/account/，并改变浏览器中的 url
        # 所以登录后台，输入 https://account.mycool.tv 就行
        rewrite ^ https://account.mycool.tv/account/ redirect;
    }
}
```

## 命令

#### 启动

```shell
./nginx  或者
./nginx -c /usr/local/nginx/nginx/conf/nginx.conf
```

说明：

●　其中/usr/local/nginx/nginx/conf/nginx.conf是你自己的nginx.conf路径。

●　-c参数指定了要加载的nginx配置文件路径。

#### 重启

```shell
./nginx -s reload
```





## 实战操作

### 查看[nginx](https://so.csdn.net/so/search?q=nginx&spm=1001.2101.3001.7020)服务路径

```java
ps aux|grep nginx
```

nginx的路径为：/usr/sbin/nginx



### 查看nginx配置文件路径

使用nginx的 `-t` 参数进行配置检查，即可知道实际调用的配置文件路径及是否调用有效。

```shell
/usr/sbin/nginx -t
```



### nginx配置文件include其他配置文件

nginx 的配置很灵活,支持`include`配置文件,如果配置内容过多， 这个文件就会比较乱, 也影响管理和阅读.所以直接拆分出来,分成不同的配置文件；怎么实现呢 ？直接include：

```shell
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites/pstest.conf;
```

所以，如果在主配置文件里找不到想要的配置信息，可以看看是否include其他配置文件；