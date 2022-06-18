





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

#### ls

```shell
# 查询目录的内容

-a		# 显示隐藏文件
-t		# 按文件修改时间排序
-r		# 反向排序，经常和 -t、-S 一起使用
-S		# 按文件大小排序
-l		# 显示详细信息，别名是 ll
-d      #查看当前目录的详细信息

检索（例如检索 profile）：ls -l | grep profile
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



#### proc

```
//查看redis进程ps -aux | grep redis 或者ps -ef|grep redis
//假设得到redis的进程号123，然后使用以下命令查看安装位置。ll /proc/123/cwd
```



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

#### whoami

查看当前用户

#### su

- 切换用户：

  ```
  su -用户名
  ```

   

  或者

   

  ```
  su 用户名
  ```

  （从其他用户切换到 root 用户需要密码，从 root 用户切换到任何其他用户不需要密码）

  - 切换到 root 用户：su
  - 切换到 root 用户后使用命令：exit 切换到普通用户

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



#### passwd

修改密码：修改 root 自己的密码，直接输入 `passwd`，输入两遍新密码即可。若修改其他用户，如 oracle 的密码，可直接输入 `passwd oracle`，输入两遍性新密码即可。

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

#### chmod

获取权限：`chmod 777 文件名`

#### locale

查看系统编码：`locale`

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



#### sysctl -p

修改配置文件后立即生效：`sysctl -p`

#### env

查看系统环境变量：`env`

#### free -m/g

查看内存：`free -m/g`

#### tree

`tree`：树形查看当前目录结构



#### lsof

列出谁在使用 3306 端口：`lsof -i:3306`

#### dirname 

`dirname` 用于取指定路径所在的目录，如：`dirname /home/ikidou`，结果为：`/home`

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

[curl](https://curl.haxx.se/)是一种命令行工具，作用是发出网络请求，然后得到和提取数据，显示在"标准输出"（stdout）上面。

它支持多种协议，下面举例讲解如何将它用于网站开发。

一、**查看网页源码**

直接在curl命令后加上网址，就可以看到网页源码。我们以网址www.sina.com为例（选择该网址，主要因为它的网页代码较短）：

```shell
curl www.sina.com
```

```html
　　<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
　　<html><head>
　　<title>301 Moved Permanently</title>
　　</head><body>
　　<h1>Moved Permanently</h1>
　　<p>The document has moved <a href="http://www.sina.com.cn/">here</a>.</p>
　　</body></html>
```

如果要把这个网页保存下来，可以使用`-o`参数，这就相当于使用wget命令了。

**二、自动跳转**

有的网址是自动跳转的。使用`-L`参数，curl就会跳转到新的网址。

```shell
 curl -L www.sina.com
```

键入上面的命令，结果就自动跳转为www.sina.com.cn。

**三、显示头信息**

`-i`参数可以显示http response的头信息，连同网页代码一起。

```shell
　　$ curl -i www.sina.com
```

```shell
HTTP/1.0 301 Moved Permanently
　　Date: Sat, 03 Sep 2011 23:44:10 GMT
　　Server: Apache/2.0.54 (Unix)
　　Location: http://www.sina.com.cn/
　　Cache-Control: max-age=3600
　　Expires: Sun, 04 Sep 2011 00:44:10 GMT
　　Vary: Accept-Encoding
　　Content-Length: 231
　　Content-Type: text/html; charset=iso-8859-1
　　Age: 3239
　　X-Cache: HIT from sh201-9.sina.com.cn
　　Connection: close

　　<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
　　<html><head>
　　<title>301 Moved Permanently</title>
　　</head><body>
　　<h1>Moved Permanently</h1>
　　<p>The document has moved <a href="http://www.sina.com.cn/">here</a>.</p>
　　</body></html>
```

`-I`参数则是只显示http response的头信息。

**四、显示通信过程**

`-v`参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。

```shell
　　$ curl -v www.sina.com
```

```shell
　* About to connect() to www.sina.com port 80 (#0)
　　* Trying 61.172.201.195... connected
　　* Connected to www.sina.com (61.172.201.195) port 80 (#0)
　　> GET / HTTP/1.1
　　> User-Agent: curl/7.21.3 (i686-pc-linux-gnu) libcurl/7.21.3 OpenSSL/0.9.8o zlib/1.2.3.4 libidn/1.18
　　> Host: www.sina.com
　　> Accept: */*
　　>
　　* HTTP 1.0, assume close after body
　　< HTTP/1.0 301 Moved Permanently
　　< Date: Sun, 04 Sep 2011 00:42:39 GMT
　　< Server: Apache/2.0.54 (Unix)
　　< Location: http://www.sina.com.cn/
　　< Cache-Control: max-age=3600
　　< Expires: Sun, 04 Sep 2011 01:42:39 GMT
　　< Vary: Accept-Encoding
　　< Content-Length: 231
　　< Content-Type: text/html; charset=iso-8859-1
　　< X-Cache: MISS from sh201-19.sina.com.cn
　　< Connection: close
　　<
　　<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
　　<html><head>
　　<title>301 Moved Permanently</title>
　　</head><body>
　　<h1>Moved Permanently</h1>
　　<p>The document has moved <a href="http://www.sina.com.cn/">here</a>.</p>
　　</body></html>
　　* Closing connection #0
```

如果你觉得上面的信息还不够，那么下面的命令可以查看更详细的通信过程。

```
curl --trace output.txt www.sina.com
```

或者

```shell
　　$ curl --trace-ascii output.txt www.sina.com
```

运行后，请打开output.txt文件查看。

**五、发送表单信息**

发送表单信息有GET和POST两种方法。GET方法相对简单，只要把数据附在网址后面就行。

```shell
　$ curl example.com/form.cgi?data=xxx
```

POST方法必须把数据和网址分开，curl就要用到--data参数。

```shell
$ curl -X POST --data "data=xxx" example.com/form.cgi
```

如果你的数据没有经过表单编码，还可以让curl为你编码，参数是`--data-urlencode`。

```shell
　　$ curl -X POST--data-urlencode "date=April 1" example.com/form.cgi
```

**六、HTTP动词**

curl默认的HTTP动词是GET，使用`-X`参数可以支持其他动词。

```shell
$ curl -X POST www.example.com
```

```shell
$ curl -X DELETE www.example.com
```

**七、文件上传**

假定文件上传的表单是下面这样：

```shell
　<form method="POST" enctype='multipart/form-data' action="upload.cgi">
　　　　<input type=file name=upload>
　　　　<input type=submit name=press value="OK">
　　</form>
```

你可以用curl这样上传文件：

```shell
$ curl --form upload=@localfilename --form press=OK [URL]
```

**八、Referer字段**

有时你需要在http request头信息中，提供一个referer字段，表示你是从哪里跳转过来的。

```shell
　$ curl --referer http://www.example.com http://www.example.com
```

**九、User Agent字段**

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

iPhone4的User Agent是

```shell
　　Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293 Safari/6531.22.7
```

curl可以这样模拟：

```shell
$ curl --user-agent "[User Agent]" [URL]
```

**十、cookie**

使用`--cookie`参数，可以让curl发送cookie。

```shell
$ curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从http response头信息的`Set-Cookie`字段中得到。

`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```shell
　$ curl -c cookies http://example.com
　$ curl -b cookies http://example.com
```

**十一、增加头信息**

有时需要在http request之中，自行增加一个头信息。`--header`参数就可以起到这个作用。

```shell
　$ curl --header "Content-Type:application/json" http://example.com
```

**十二、HTTP认证**

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```shell
　$ curl --user name:password example.com
```

例子：

```shell
-A
-A参数指定客户端的用户代理标头，即User-Agent。curl 的默认用户代理字符串是curl/[version]。


$ curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
上面命令将User-Agent改成 Chrome 浏览器。


$ curl -A '' https://google.com
上面命令会移除User-Agent标头。

也可以通过-H参数直接指定标头，更改User-Agent。


$ curl -H 'User-Agent: php/1.0' https://google.com
-b
-b参数用来向服务器发送 Cookie。


$ curl -b 'foo=bar' https://google.com
上面命令会生成一个标头Cookie: foo=bar，向服务器发送一个名为foo、值为bar的 Cookie。


$ curl -b 'foo1=bar;foo2=bar2' https://google.com
上面命令发送两个 Cookie。


$ curl -b cookies.txt https://www.google.com
上面命令读取本地文件cookies.txt，里面是服务器设置的 Cookie（参见-c参数），将其发送到服务器。

-c
-c参数将服务器设置的 Cookie 写入一个文件。


$ curl -c cookies.txt https://www.google.com
上面命令将服务器的 HTTP 回应所设置 Cookie 写入文本文件cookies.txt。

-d
-d参数用于发送 POST 请求的数据体。


$ curl -d'login=emma＆password=123'-X POST https://google.com/login
# 或者
$ curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
使用-d参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法，因此可以省略-X POST。

-d参数可以读取本地文本文件的数据，向服务器发送。


$ curl -d '@data.txt' https://google.com/login
上面命令读取data.txt文件的内容，作为数据体向服务器发送。

--data-urlencode
--data-urlencode参数等同于-d，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码。


$ curl --data-urlencode 'comment=hello world' https://google.com/login
上面代码中，发送的数据hello world之间有一个空格，需要进行 URL 编码。

-e
-e参数用来设置 HTTP 的标头Referer，表示请求的来源。


curl -e 'https://google.com?q=example' https://www.example.com
上面命令将Referer标头设为https://google.com?q=example。

-H参数可以通过直接添加标头Referer，达到同样效果。


curl -H 'Referer: https://google.com?q=example' https://www.example.com
-F
-F参数用来向服务器上传二进制文件。


$ curl -F 'file=@photo.png' https://google.com/profile
上面命令会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。

-F参数可以指定 MIME 类型。


$ curl -F 'file=@photo.png;type=image/png' https://google.com/profile
上面命令指定 MIME 类型为image/png，否则 curl 会把 MIME 类型设为application/octet-stream。

-F参数也可以指定文件名。


$ curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
上面命令中，原始文件名为photo.png，但是服务器接收到的文件名为me.png。

-G
-G参数用来构造 URL 的查询字符串。


$ curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
上面命令会发出一个 GET 请求，实际请求的 URL 为https://google.com/search?q=kitties&count=20。如果省略--G，会发出一个 POST 请求。

如果数据需要 URL 编码，可以结合--data--urlencode参数。


$ curl -G --data-urlencode 'comment=hello world' https://www.example.com
-H
-H参数添加 HTTP 请求的标头。


$ curl -H 'Accept-Language: en-US' https://google.com
上面命令添加 HTTP 标头Accept-Language: en-US。


$ curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
上面命令添加两个 HTTP 标头。


$ curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
上面命令添加 HTTP 请求的标头是Content-Type: application/json，然后用-d参数发送 JSON 数据。

-i
-i参数打印出服务器回应的 HTTP 标头。


$ curl -i https://www.example.com
上面命令收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码。

-I
-I参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。


$ curl -I https://www.example.com
上面命令输出服务器对 HEAD 请求的回应。

--head参数等同于-I。


$ curl --head https://www.example.com
-k
-k参数指定跳过 SSL 检测。


$ curl -k https://www.example.com
上面命令不会检查服务器的 SSL 证书是否正确。

-L
-L参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。


$ curl -L -d 'tweet=hi' https://api.twitter.com/tweet
--limit-rate
--limit-rate用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。


$ curl --limit-rate 200k https://google.com
上面命令将带宽限制在每秒 200K 字节。

-o
-o参数将服务器的回应保存成文件，等同于wget命令。


$ curl -o example.html https://www.example.com
上面命令将www.example.com保存成example.html。

-O
-O参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。


$ curl -O https://www.example.com/foo/bar.html
上面命令将服务器回应保存成文件，文件名为bar.html。

-s
-s参数将不输出错误和进度信息。


$ curl -s https://www.example.com
上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。

如果想让 curl 不产生任何输出，可以使用下面的命令。


$ curl -s -o /dev/null https://google.com
-S
-S参数指定只输出错误信息，通常与-s一起使用。


$ curl -s -o /dev/null https://google.com
上面命令没有任何输出，除非发生错误。

-u
-u参数用来设置服务器认证的用户名和密码。


$ curl -u 'bob:12345' https://google.com/login
上面命令设置用户名为bob，密码为12345，然后将其转为 HTTP 标头Authorization: Basic Ym9iOjEyMzQ1。

curl 能够识别 URL 里面的用户名和密码。


$ curl https://bob:12345@google.com/login
上面命令能够识别 URL 里面的用户名和密码，将其转为上个例子里面的 HTTP 标头。


$ curl -u 'bob' https://google.com/login
上面命令只设置了用户名，执行后，curl 会提示用户输入密码。

-v
-v参数输出通信的整个过程，用于调试。


$ curl -v https://www.example.com
--trace参数也可以用于调试，还会输出原始的二进制数据。


$ curl --trace - https://www.example.com
-x
-x参数指定 HTTP 请求的代理。


$ curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
上面命令指定 HTTP 请求通过myproxy.com:8080的 socks5 代理发出。

如果没有指定代理协议，默认为 HTTP。


$ curl -x james:cats@myproxy.com:8080 https://www.example.com
上面命令中，请求的代理使用 HTTP 协议。

-X
-X参数指定 HTTP 请求的方法。


$ curl -X POST https://www.example.com
上面命令对https://www.example.com发出 POST 请求。
```



**一、参数说明**

使用 curl 发送 POST 请求

**格式：** `curl -H 请求头 -d 请求体 -X POST 接口地址`

| 参数              | 内容     | 格式                                                         |
| ----------------- | -------- | ------------------------------------------------------------ |
| -H(或者 --header) | 请求头   | “Content-Type: application/json”                             |
| -d                | POST内容 | ‘{“id”: “001”, “name”:“张三”, “phone”:“13099999999”}’ 或者 ‘id=001&name=张三&phone=13099999999’ |
| -X                | 请求协议 | POST、GET、DELETE、PUSH、PUT、OPTIONS、HEAD                  |

二、**示例说明**

**1、application/x-www-form-urlencoded**
最常见的一种 POST 请求，用 [curl](https://so.csdn.net/so/search?q=curl) 发起这种请求也很简单。

```shell
$ curl  -X POST -d 'name=张三'  http://localhost:2000/api/basic
```

**2、application/json**

跟发起 `application/x-www-form-urlencoded` 类型的 POST 请求类似，-d 参数值是 `JSON 字符串`，并且多了一个 `Content-Type: application/json` 指定发送内容的格式。

```shell
$ curl -H "Content-Type: application/json" -X POST -d '{"id": "001", "name":"张三", "phone":"13099999999"}'  http://localhost:2000/api/json
```

**3、multipart/form-data**

这种请求一般涉及到文件上传。后端对这种类型请求的处理也复杂一些。

```shell
$ curl -F raw=@raw.data -F name=张三 http://localhost:2000/api/multipart
```

**4、把文件内容作为要提交的数据**

如果要提交的数据不像前面例子中只有一个 name: 张三 键值对，数据比较多，都写在命令行里很不方便，也容易出错，那么可以把数据内容先写到文件里，通过 -d @filename 的方式来提交数据。这是 -d 参数的一种使用方式，所以前面用到 -d 参数的地方都可以这样用。
实际上就是把 -d 参数值写在命令行里，变成了写在文件里。跟 multipart/form-data 中上传文件的 POST 方式不是一回事。@ 符号表明后面跟的是文件名，要读取这个文件的内容作为 -d 的参数。

例如，有一个 JSON 文件 data.json 内容如下:

```shell
{
    "id": "001",
    "name":"张三",
    "phone":"13099999999"
}
```

就可以通过

```shell
$ curl -H "Content-Type: application/json" -X POST -d @data.json  http://localhost:2000/api/json
```

来提交数据。

参考：https://dev.to/getd/x-www-form-urlencoded-or-form-data-explained-in-2-mins-5hk6

参考：https://catonmat.net/cookbooks/curl



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



#### source 

在 [linux](https://so.csdn.net/so/search?q=linux) 里，当我们修改了配置文件，不想让linux重启时就可以通过重载配置文件使配置文件生效。
例如，刚修改 `/etc/profile`文件，我想让刚刚作出的修改立刻看到效果，但又不愿意重启，这时，就可以通过重载配置文件来使配置文件立即生效：

```shell
方式一：
source /etc/profile

方式二：
. /etc/profile

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



### 文件操作指令

#### mkdir

```shell
# 创建文件夹

mkdir dir1 dir2 dir3  # 可以同时创建多个目录
-p		# 递归创建目录，即创建目录的路径有某些目录未创建，会一次性创建
```



#### touch

创建文件：`touch [路径] + 文件名`

- `touch [路径] + 文件名1 [路径] + 文件名2`（可以同时创建多个文件，空格隔开）



#### copy

- 复制（copy）文件：`cp 源文件 目标文件夹`（正常情况下使用绝对路径） -复制文件夹：`cp -r 源文件 目标文件夹`



#### move

- 移动（move）文件：`mv 源文件 目标文件夹`（正常情况下使用绝对路径）
  - 移动文件夹：`mv 源文件夹 目标文件夹`
  - 重命名：`mv aa.txt aaaa.txt`

#### remove

- 删除（remove）文件：`rm [路径] + 文件名` （可以删除多个文件，每个文件用空格隔开）
  - 删除文件夹：`rm -r 文件夹1 文件夹2`（递归 recursive 删除文件夹1和文件夹2下的所有内容，每删除一个会提示）
  - `rm -rf 文件夹`（强制 force 删除文件夹下的所有内容，不提示删除）删除后无法还原
  - 删除文件夹或者： `rmdir 文件夹`（文件夹必须为空）
  - 删除所有内容：`rm -rf *`



### 文件读写指令

#### echo

```shell
echo "Hello World" >> a.txt
```

将字符串 Hello World追加到文件 a.txt 中。

- `echo "Hello World" > a.txt` 将文件 a.txt 中的内容`替换`为字符串 Hello World。

#### cat

查看文件内容：`cat a.txt`

#### head

`head services` 查看 services 文件前 10 行的内容（默认前 10 行） `head -20 services` （前20行的内容）

#### tail

`tail services` 查看 services 文件结尾 10 行的内容 `tail -20f services`（滚动显示结尾 20 行的内容）

#### more

`more services` 分页查看 services 文件中的内容，按空格或 f 切换下一页，回车下一行,q 退出。（文件内容较多时使用）

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



### 网络通讯

#### netstat

**netstat -tunlp** 用于显示 tcp，udp 的端口和进程等相关情况。

netstat 查看端口占用语法格式：

```
netstat -tunlp | grep 端口号
```

- -t (tcp) 仅显示tcp相关选项top
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



## 

