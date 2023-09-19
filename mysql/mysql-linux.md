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

