





## Linux 系统介绍

> 为什么用 Linux

### 1、开源

​	因为是开源的，所以==免费==，不需要任何授权费；而且没有后门，完全可控，用的放心。

2、稳定

​    系统可以正常运行4，5年不用重启，友商 Windows Server 长时间不重启很容易卡或其他问题。

3、高效

​    1核1G的虚拟机，跑字符界面的 Linux，没有压力。

4、生态

​    需要的服务组件，配置源后，大都可以直接安装，而且也免费。



> 文件类型

```shell
[root@testhotel temp]# ll
total 127016
-rw-r--r--  1 root    root         998 Sep 15 16:56 BACK.log
-rw-r--r--  1 root    root         263 Sep 15 18:28 backvideo.txt
drwxrwxr-x  5 root    root        4096 Sep 13 19:33 bill
-rw-r--r--  1 root    root      171501 Sep 10 14:34 bill-1.1.0-4df3a16.html5.zip
-rw-r--r--  1 root    root      171484 Sep 11 16:22 bill-1.1.1-3815961.html5.zip
#文件属性  文件个数  所属者  所属组  大小  月  日  分/年  文件名
```



文件属性的第一个字符，代表文件类型：

- 普通文件

- 目录文件

- 特殊文件（该类有 5 个文件类型）

- - 链接文件
    - 字符设备文件
    - Socket 文件
    - 命名管道文件
    - 块文件

`–`　普通文件以连接号`-`开头。

`d`　目录文件以英文字母`d`开头。

`l`　链接文件以英文字母`l`开头。

​	链接文件分为`软连接`和`硬链接`

​	硬链接： 与普通文件没什么不同，索引节点都指向同一个文件在硬盘中的区块，如果想，可以代替 cp 命令。
​	软链接： 保存了其代表的文件的绝对路径，是另外一种文件，在硬盘上有独立的区块，访问时替换自身路径，相当于 Windows 的快捷方式。

`c`　字符设备文件以英文字母`c`开头。

​    Linux 字符设备提供连续的数据流，应用程序可以顺序读取，通常不支持随机存取。相反，此类设备支持按字节/字符来读写设备。举例来说，键盘，串口，调制解调器都是典型的字符设备。

`s`　Socket 文件以英文字母`s`开头。

​    socket 的原意是“插座”，在计算机通信领域，socket 被翻译为“套接字”，它是计算机之间进行通信的一种约定或一种方式。通过 socket 这种约定，一台计算机可以接收其他计算机的数据，也可以向其他计算机发送数据。

`p`　命名管道文件以英文字母`p`开头。

​    管道分为无名管道和有名管道两种管道，管道文件是建立在内存之上可以同时被两个进程访问的文件。

`b`　块文件以英文字母`b`开头。

​    系统中能够随机（不需要按顺序）访问固定大小数据片（chunks）的设备被称作块设备。这些数据片就称作块。最常见的块设备是硬盘，除此以外，还有软盘驱动器、CD-ROM 驱动器和闪存等等许多其他块设备。



> 属主属组

![image-20210917195613301](Linux%E5%9F%BA%E7%A1%80.assets/image-20210917195613301.png)

权限：

`r`	读

​	对于文件，是查看文件内容

​	对于目录，是查看目录下内容

`w`	写

​	对于文件，是向文件内添加、修改和删除内容

​	对于目录，是对目录下添加和删除文件或目录

`x`	执行

​	对于文件，是直接执行，一般用于 shell 脚本和二进制命令文件

​	对于目录，是能否进入目录



`root` 属于超级账号，拥有系统所有权限，而且不受读、写权限影响，可以读取或修改任意文件/目录，只受执行权限影响。



## Linux 常用命令

参考文档：[Linux 常用命令学习 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/linux-common-command-2.html)



### 基础常用命令

> #### ls

```shell
# 查询目录的内容

-a		# 显示隐藏文件
-t		# 按文件修改时间排序
-r		# 反向排序，经常和 -t、-S 一起使用
-S		# 按文件大小排序
-l		# 显示详细信息，别名是 ll
```

#### vim

参考文档：[Linux vi/vim | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-vim.html)

[vim快速操作 - Linux程 - 博客园 (cnblogs.com)](https://www.cnblogs.com/chengjiawei/p/9339951.html)



​	vi 是老式的字处理器，所有 Linux 都会默认安装。vim 是其升级版，最明显的区别是语法高亮，方便排错。

​	vim 可能需要 Centos 使用 yum 额外安装。

​	vim 是程序开发工具，不仅仅是文本处理工具。

​	vim 分为**命令模式**、**输入模式**、**底行（底线）命令模式**。

![img](Linux%E5%9F%BA%E7%A1%80.assets/vim-vi-workmodel.png)



> 命令模式

```shell
# 执行 vim，就会先进入命令模式
vim /data/statics/fixed/style.json

i		# 当前光标处，进入输入模式，会进入输入模式
I		# 当前行首，进入输入模式，会进入输入模式
a		# 当前光标的下一个字符处，会进入输入模式
A		# 当前行尾，进入输入模式，会进入输入模式
0 / ^	# 光标移动到行首（Shift + 6）
$		# 光标移动到行尾（Shift + 4）
o		# 在当前行后插入一个新行，会进入输入模式
O		# 在当前行前插入一个新行，会进入输入模式
r		# r + 5，将当前光标所在的字符，改为5
x		# 删除当前光标的字符
w		# 光标移动到下一个单词的开头（默认上来说，一个单词由字母，数字和下划线组成）
gg		# 光标移动到第一行第一个字符
G		# 光标移动到最后一行第一个字符
u		# 撤销上一步的修改
Ctrl + r  # 反撤销
dw		# 剪切当前光标所在位置到单词结尾（可以删除整个单词或者空格）
dd		# 剪切当前所在行，可以当删除使用
d + 行数 + 上/下  # 比如，d3上，就是剪切当前光标行 + 上数3行的内容（共剪切4行）
行数 + d + 上/下  # 同上
yy		# 复制当前所在行
p		# 粘贴剪切的内容到当前光标后，如果剪切的是整行，就会粘贴到当前行下方
P		# 粘贴剪切的内容到当前光标前（Shift + p）
ggdG 	# 清空文件，光标先移动到第一行，然后剪切到最后一行
Gdgg	# 清空文件，光标先移动到最后一行，然后剪切到第一行
/		# 进入输入模式，可以输入想查找的字符串，如果结果有多个，n 下一个，N 上一个
```



多行添加注释：

`Ctrl + v`，移动光标选中需要注释的所有行，`I`，将光标移动到行首并进入输入模式，输入注释符号`#`，输入完毕后，`ESC`两次，vim 会自动多行添加注释。

取消多行注释：

`Ctrl + v`，移动光标选中需要取消注释的所有行的注释符号，`x`。





> 输入模式

​	就是使用键盘输入，想咋输入咋输入。

​	使用`ESC`退出输入模式，进入命令模式。



> 底行模式

在命令模式按下`:`（英文冒号）就进入了底线命令模式。

```shell
q		# 退出
w		# 保存
wq		# 保存退出
x		# 退出，需要保存时会保存
q!		# 强制退出，忽略文件的修改
n		# 光标跳转到指定行行首

# 按 ESC 可以退出底线模式
```



#### cd

```shell
# 切换目录

.		# 当前目录
..		# 上级目录
-		# 上次的目录
~		# 当前登录用户的家目录
```



> 相对路径、绝对路径

`相对路径`：相对自己当前位置，目标文件的路径，如 temp/cms-v1.40.2t.e723e3b9.zip

`绝对路径`：文件在系统中的完整路径，如 /data/temp/cms-v1.40.2t.e723e3b9.zip



#### pwd

```shell
# 显示当前目录绝对路径
```



#### mv

```shell
# 剪切、改名

-f		# 强制执行，忽略提示信息
mv file1 file2 file3 dir  # 将多个文件剪切到 dir 目录
```



#### rm

```shell
# 删除

rm -rf file1 file2 dir1  # 同时删除多个文件和目录
-r		# 删除目录，如果不需要删除目录，建议不要使用
-f		# 强制执行，忽略提示信息
```



#### 通配符

```shell
*		# 匹配任意长度的任意字符
?		# 匹配任意单个字符
[]		# 匹配中括号指定范围内的任意单个字符
[a-Z]	# 任意单个字母
[0-9]	# 任意单个数字
```



#### touch

```shell
# 创建文件

touch file1 file2 file3  # 可以同时创建多个文件
```



#### mkdir

```shell
# 创建目录

mkdir dir1 dir2 dir3  # 可以同时创建多个目录
-p		# 递归创建目录，即创建目录的路径有某些目录未创建，会一次性创建
```



#### cp

```shell
# 复制

cp -r file1 dir1 dir2  # 将多个文件或目录复制到 dir2 目录中
-r		# 复制目录及目录内所有内容
-a		# 保留文件权限，时间，并且会复制目录及目录内所有内容
\cp		# 强制复制，不提示，\ 会不使用命令别名
```



#### cat

```shell
# 查看文件所有内容

-n		# 显示行号
```



#### head

```shell
# 从文件开头开始，查看文件内容

-n 10	# 展示10行（默认），可以简写为 -10
```



#### tail

```shell
# 从文件结尾开始，查看文件内容

-n 10	# 展示10行（默认），可以简写为 -10
-f		# 监控文件，和 tailf 命令相似
```



#### date

```shell
# 查看时间

date +"%F"  # 指定格式输出时间（2021-09-17）
date +"%Y%m%d"  # 指定格式输出时间（20210917）
```



#### echo

```shell
# 输出语句

echo 123 >index.html  # 通过重定向符 '>'，对 index.html 文件写入字符串 '123'
>		# 重定向符，输入，没有文件会创建文件，并清空文件内容，然后写入重定向内容
>>		# 重定向符，追加，没有文件会创建文件，不会清空文件内容，会在文件最后追加重定向内容
```



#### ssh

```shell
# 远程连接

ssh -p22 root@192.168.10.123  # 使用 root 用户，通过22端口，连接服务器192.168.10.123
-p		# 指定 sshd 服务端口，使用默认端口22时，可以省略此选项
```



#### curl

```shell
# 字符端综合传输工具

curl "baidu.com"
--resolve html5.mycool.tv:80:127.0.0.1  # 指定域名 html5.mycool.tv 解析为本机
```



#### wget

```shell
# 网络下载工具

# 下载指定 HTTP 地址的视频
wget -t0 -c "http://videos.mycool.tv/videos/20210912/08fcd149a82c2bb561d445904d46f9d0bbe4562b.mp4"
-t0		# 断点续传重试次数，0代表无穷次
-c		# 断点续传，如果下载过程中网络中断，再次下载时，从上次中断的地方继续下载
```



#### scp

```shell
# 服务器间文件拷贝

# 将本机 /data/vidoes/20210901 目录，复制到124服务器一份
scp -r /data/vidoes/20210901 root@192.168.20.124:/data/videos/

-r		# 拷贝目录及目录下所有内容
```



#### yum

```shell
# 软件包管理器

yum install -y lrzsz  # 安装指定服务

-y		# 同意安装，不用与 yum 交互
```



#### rz && sz

```shell
# lrzsz 服务安装之后的两个命令（lrzsz 服务，centos7默认不安装，需要手动安装）

rz		# 从 Windows 上传文件到当前目录，最大上传文件不能超过4G，也可以直接将文件按住拖进来
sz file	# 从 Linux 传输指定文件到 windows
```



#### ip a / ifconfig

```shell
# 查看系统 IP 地址
```



#### find

```shell
# 文件搜索工具

# 查找 /data/videos 目录下指定文件的路径
find /data/videos -name "08fcd149a82c2bb561d445904d46f9d0bbe4562b.mp4"

# 在当前目录查找时，可以省略目录参数
cd /data/videos
find -name "08fcd149a82c2bb561d445904d46f9d0bbe4562b.mp4"
```



#### tar

```shell
# 压缩和解压

# 压缩指定目录，z 使用 gzip 压缩格式，c 打包，v 显示过程，f 指定压缩包名称
tar zcvf cms.tar.gz cms

# 解压指定 tar 包
tar xf cms.tar.gz -C /data/temp

-C		# 指定解压目的目录，解压到当前目录时可以省略该参数
```



#### zip && unzip

```shell
# zip 格式的压缩和解压缩

# 将 /data/temp/cms 目录打个 zip 包
zip -r cms.zip /data/temp/cms

-r		# 递归处理目录及其子文件

# 解压 zip 包
unzip cms.zip
```



#### df -h

```shell
# 显示系统磁盘空间使用情况

-h		# 使用更易读的单位显示文件大小（默认 KB 为单位）
```

<img src="Linux%E5%9F%BA%E7%A1%80.assets/image-20210918085449921.png" alt="image-20210918085449921" style="zoom:67%;" />



#### du -sh

```shell
# 查看文件和目录的磁盘使用情况

# 查看当前目录下所有文件或目录大小
du -sh *

-s		# 仅显示总计
-h		# 使用更易读的单位显示文件大小（默认 KB 为单位）
```



#### free -m

```shell
# 查看系统内存使用情况

-m		# 以 MB 为单位显示文件大小
```



#### ps

```shell
# 查看系统进程

ps aux		# 显示当前所有进程
ps -ef		# 显示当前所有进程，和上面的命令结果格式稍微有区别

# 查看 tomcat 进程
ps -ef | grep tomcat
```



#### |

```shell
# 管道符，将第一个命令的输出为第二个命令的输入

# 监听 nginx 访问日志，并筛选404
tail -f /data/logs/nginx/access.log | grep " 404 "
```



#### grep

```shell
# 文本搜索工具

# 搜索 redis 服务密码
cat /opt/soft/redis/conf/redis-6379.conf | grep "requirepass"
grep "requirepass" /opt/soft/redis/conf/redis-6379.conf

# nginx 配置文件中，搜索包含指定域名的配置文件
grep "images.mycool.tv" /opt/soft/nginx/conf/vhosts/*
# 或者
cd /opt/soft/nginx/conf/vhosts/
grep -R "images.mycool.tv"

-R		# 递归查找文件夹
```



注：练习命令时，==不能==在生产环境，也==不建议==在测试环境使用，自己用虚拟机或者 docker 开一个 centos7.4 环境，随便搞。



### 常用服务重启命令

#### service

```shell
# service 是 Centos6 和6之前版本的系统服务管理工具，Centos7 也能用
# service 能正常重启的服务，一般也能用 systemctl 重启

# 查看 nginx 配置文件语法是否正常，并重新加载
/etc/init.d/nginx configtest
/etc/init.d/nginx reload

# 或者
service nginx configtest
service nginx reload

# 上面的命令两者完全相同，service 命令本质也是执行 /etc/init.d/nginx 脚本

# 或者使用 systemctl，不过 nginx 的 confitest 选项没配置
systemctl reload nginx

# 如果重启
systemctl restart nginx
```



#### systemctl

```shell
# systemctl 是 Centos7 的系统服务管理工具，比 service 性能更好

# 重启网卡
systemctl restart network

# 停掉 tomcat，再启动
systemctl stop tomcat
systemctl start tomcat
```



#### supervisord

```shell
# Supervisor 是用 Python 开发，是 Linux/Unix 系统下的一个进程管理工具，不支持 Windows 系统。它可以很方便的监听、启动、停止、重启一个或多个进程。用 Supervisor 管理的进程，当一个进程意外被杀死，supervisort 监听到进程死后，会自动将它重新拉起，很方便的做到进程自动恢复的功能，不再需要自己写 shell 脚本来控制。
# 比如说，可以把 java 的 jar 包通过配置文件，放入 Supervisor 的托管
# 不是 Centos7 自带的，需要额外安装

# 查看正在托管的服务
[root@testhotel ~]# supervisorctl status
account                          RUNNING   pid 19228, uptime 5 days, 5:52:59
c-agent                          RUNNING   pid 19234, uptime 5 days, 5:52:59
ctrld                            RUNNING   pid 19231, uptime 5 days, 5:52:59
harvest                          RUNNING   pid 19259, uptime 5 days, 5:52:59
hotelservice                     RUNNING   pid 19230, uptime 5 days, 5:52:59
hsyncd                           RUNNING   pid 19237, uptime 5 days, 5:52:59
mrp                              RUNNING   pid 19232, uptime 5 days, 5:52:59
nmagent                          RUNNING   pid 19229, uptime 5 days, 5:52:59
pmsservice                       RUNNING   pid 19233, uptime 5 days, 5:52:59
ssdpd                            RUNNING   pid 31629, uptime 3 days, 16:55:20
triggerserver                    RUNNING   pid 19239, uptime 5 days, 5:52:59
viking                           RUNNING   pid 6318, uptime 0:00:09

# 重启 supervisorctl 下托管的服务，比如 viking
supervisorctl restart viking

# 或者交互式
[root@testhotel ~]# supervisorctl 
account                          RUNNING   pid 19228, uptime 5 days, 5:58:00
c-agent                          RUNNING   pid 19234, uptime 5 days, 5:58:00
ctrld                            RUNNING   pid 19231, uptime 5 days, 5:58:00
harvest                          RUNNING   pid 19259, uptime 5 days, 5:58:00
hotelservice                     RUNNING   pid 19230, uptime 5 days, 5:58:00
hsyncd                           RUNNING   pid 19237, uptime 5 days, 5:58:00
mrp                              RUNNING   pid 19232, uptime 5 days, 5:58:00
nmagent                          RUNNING   pid 19229, uptime 5 days, 5:58:00
pmsservice                       RUNNING   pid 19233, uptime 5 days, 5:58:00
ssdpd                            RUNNING   pid 31629, uptime 3 days, 17:00:21
triggerserver                    RUNNING   pid 19239, uptime 5 days, 5:58:00
viking                           RUNNING   pid 6976, uptime 0:00:08
supervisor> restart viking  # 交互式可以 tab 补全服务名称

# exit 或者 Ctrl + c 退出交互
```



## 网络通讯

### netstat

**netstat -tunlp** 用于显示 tcp，udp 的端口和进程等相关情况。

netstat 查看端口占用语法格式：

```
netstat -tunlp | grep 端口号
```

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名

例如查看 8000 端口的情况，使用以下命令：

```
# netstat -tunlp | grep 8000
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      26993/nodejs   
```

更多命令：

```
netstat -ntlp   //查看当前所有tcp端口
netstat -ntulp | grep 80   //查看所有80端口使用情况
netstat -ntulp | grep 3306   //查看所有3306端口使用情况
```





## nginx

### 配置

参考文档：[nginx_百度百科 (baidu.com)](https://baike.baidu.com/item/nginx/3817705?fr=aladdin)、[Nginx中文文档](https://www.nginx.cn/doc/)、[nginx系列之一：nginx入门_王道长的技术博客-CSDN博客_nginx](https://blog.csdn.net/qq_29677867/article/details/90112120)

​	*Nginx* 是一个高性能的 HTTP 和反向代理 web 服务器。

​	因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名，其特点是占有内存少，并发能力强。

​	中国大陆使用 nginx 网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

![img](Linux%E5%9F%BA%E7%A1%80.assets/4ffce04d92a4d6cb21c1494cdfcd6dc1.jpg)





> 简单的 nginx 配置解析

```shell
​```
server {
    listen       80;  # 监听的端口
    server_name  html5.mycool.tv;  # 这个 server 标签，监听的域名
    charset utf-8;  # 字符集
    set $HIT "HIT";  # 设置变量，日志使用

    location /html5 {  # 浏览器请求 http://html5.mycool.tv/html5/ 时
        alias  /data/html5;  # 使用服务器 /data/html5 目录，响应请求
    }
}
​```


​```
    set_by_lua_block $hotelId { return hotelId }  # 通过 lua，设置变量

    location = /images/offlineIcon/hotel.gif {  # 浏览器访问指定文件时
        # 重定向到 /images/offlineIcon/$hotelId.gif 文件
        rewrite ^(.*)/hotel.gif $1/$hotelId.gif last;
    }
​```
```



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

### 命令

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

## tomcat

### 将tomcat服务注册到service中，使用service tomcat start启动

linux下有的软件启动很麻烦，跟一大堆参数，比如指定配置文件路径、以何种模式启动神马的，等等。而我们装上appache或者mysql后，就可以使用service httpd start来启动，很是方便，service命令其实是跑一个shell脚本来管理，这样的话，我们自己手动写个shell脚本就可以实现service anything doanything了。另外，用chkconfig命令设置开机自动启动一个服务，该服务必须是系统服务，否则用chkconfig设置是会报错的。这样的话，把一些服务注册为系统服务，确实还是蛮必须的。而注册成系统服务，就是这个service…

当我们输入service命令时，linux会去/etc/rc.d/init.d下去找这个脚本运行。init.d下面放的就是很多脚本，比如service tomcatd start时，就去/etc/rc.d/init.d下找tomcatd这个脚本文件，如果这个文件不存在，则会提示不存在这个服务。所以，这个就好办了，只要在init.d目录下写个脚本，就可以用service命令在任何地方执行了。

```shell
进入init.d目录
cd /etc/rc.d/init.d

创建tomcat脚本
vim tomcatd
习惯上我们习惯在服务的名称后加上d，d代表daemon，即服务的意思

编写启动脚本
#!/bin/bash
case "$1" in
"start")
        echo "$0正在启动";
        /home/devworker/webtest_tomcat8080/bin/startup.sh
        echo "$0启动成功";
esac
```



> 说明：

- 1代表接收的第二个输入，servicetomcatdstart则1代表接收的第二个输入，servicetomcatdstart则1代表着start

- 而$0代表着第一个输入参数即tomcatd

  

> tomcat catalina.sh中添加 JAVA_HOME 、JRE_HOME 环境变量，大约在100行处

```
export JAVA_HOME=/home/devworker/jdk1.8.0_60
export JRE_HOME=/home/devworker/jdk1.8.0_60/jre
```



> 文件tomcatd增加执行权限

chmod +x tomcatd



> 启动tomcat服务

```
service  tomcatd start
```



### 关闭

> 首先，进入Tomcat下的bin目录

```Shell
./shutdown.sh
```



> 查看Tomcat是否以关闭

```Shell
ps -ef|grep java
```



### 启动

```shell
./startup.sh
```

## supervisor重新启动



> 重新启动

```Shell
supervisorctl reload
```



> 查看进程

```Shell
supervisorctl status
```



> 启动某个进程

```Shell
supervisorctl start xxxx
```



> 停止某个进程

```shell
supervisorctl stop xxxx
```



> 重启某个进程

```shell
supervisorctl restart xxxx
```

