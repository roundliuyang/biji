# mysql-linux

## 要允许MySQL的`root`用户从任何主机访问

1、使用合适的MySQL客户端连接到MySQL服务器，通常可以使用以下命令：

```shell
mysql -u root -p
```

这将提示你输入`root`用户的密码。

2、一旦成功连接到MySQL服务器，你可以运行以下SQL命令来修改`root`用户的主机访问权限：

```shell
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
```

- `GRANT ALL PRIVILEGES ON *.*` 授予`root`用户对所有数据库和所有表的所有权限。
- `'root'@'%'` 表示`root`用户可以从任何主机连接。
- `IDENTIFIED BY '你的密码'` 指定了`root`用户的密码。请将`'你的密码'`替换为你想要设置的密码。

3、刷新MySQL权限以使更改生效：

```shell
FLUSH PRIVILEGES;
```





## 优化器追踪示例



1. 查看优化器状态
   - show variables like 'optimizer_trace';

2. 会话级别临时开启
   - set session optimizer_trace="enabled=on",end_markers_in_json=on;

3. 设置优化器追踪的内存大小

   - set OPTIMIZER_TRACE_MAX_MEM_SIZE=1000000;

4. 执行自己的SQL
   - select host,user,plugin from user;

5. information_schema.optimizer_trace表
   - SELECT trace FROM information_schema.OPTIMIZER_TRACE;

6. 导入到一个命名为xx.trace的文件，然后用JSON阅读器来查看（如果没有控制台权限，或直接交由运维，让他把该 trace 文件，输出给你就行了。 ）。
   - `SELECT TRACE INTO DUMPFILE "E:\\test.trace" FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;`

**注意：不设置优化器最大容量的话，可能会导致优化器返回的结果不全。**
