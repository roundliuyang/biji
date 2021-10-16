# Linux 命令

标签（空格分隔）： 未分类

---

## Cat
cat（“ concatenate ”的缩写）命令是Linux / Unix等操作系统中最常用的命令之一。cat命令允许我们创建单个或多个文件，查看文件包含，连接文件以及在终端或文件中重定向输出。在本文中，我们将发现cat命令及其在Linux中的示例的便捷用法。

前面一章我们讲到了LS命令示例

一般语法
cat [OPTION] [FILE]...
### 1.显示文件内容
在下面的示例中，它将显示/ etc / passwd文件的内容。
```shell
# cat /etc/passwd
 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
narad:x:500:500::/home/narad:/bin/bash
```

### 2.在终端中查看多个文件的内容
在下面的示例中，它将在终端中显示test和test1文件的内容。

```shell
# cat test test1
 
Hello everybody
Hi world,
```
### 3.使用Cat命令创建文件

我们将使用以下命令创建一个名为test2文件的文件。

```shell
# cat >test2
```

等待用户输入，键入所需的文本，然后按CTRL + D（按住Ctrl键并键入“ d ”）退出。文本将写入test2文件中。您可以使用以下cat命令查看文件的内容。

```shell
# cat test2
hello everyone, how do you do?
```


### 4.将Cat命令与更多或更少的选项一起使用
如果具有大量内容的文件无法容纳在输出终端中，并且屏幕快速滚动，则可以通过cat命令使用越来越多的参数，如上所示。

```shell
# cat song.txt | more
# cat song.txt | less
```

### 5.在文件中显示行号
使用-n选项，您可以在输出终端中看到文件song.txt的行号。
```shell
# cat -n song.txt
1  "Heal The World"
2  There's A Place In
3  Your Heart
4  And I Know That It Is Love
5  And This Place Could
6  Be Much
7  Brighter Than Tomorrow
8  And If You Really Try
9  You'll Find There's No Need
10  To Cry
11  In This Place You'll Feel
12  There's No Hurt Or Sorrow
```
### 6.在文件末尾显示$

在下面，您可以使用-e选项看到' $ '出现在行尾，如果各段之间有间隙，则显示' $ '。此选项对于将多行压缩为一行很有用。

```shell
# cat -e test
 
hello everyone, how do you do?$
$
Hey, am fine.$
How's your training going on?$
$
```
### 7.在文件中显示制表符分隔的行
在下面的输出中，我们可以看到TAB空间被' ^ I '字符填充。

```shell
# cat -T test
 
hello ^Ieveryone, how do you do?
 
Hey, ^Iam fine.^I^IHow's your training ^Igoing on?
Let's do ^Isome practice in Linux.
```
### 8.一次显示多个文件
在下面的示例中，我们有三个文件test，test1和test2，并且能够查看这些文件的内容，如上所示。我们需要用;分隔每个文件；（半冒号）。

```shell
# cat test; cat test1; cat test2
 
This is test file
This is test1 file.
This is test2 file.
```
### 9.将标准输出与重定向运算符一起使用
我们可以将文件的标准输出重定向到新文件，或者使用' > '（大于）符号将其重新存在。小心，test1的现有内容将被测试文件的内容覆盖。

```shell
# cat test > test1shell
```
### 10.使用重定向运算符附加标准输出
在现有文件中附加' >> '（大于1的符号）。这里，测试文件的内容将附加在test1文件的末尾。

```shell
# cat test >> test1
```

### 11.使用重定向运算符重定向标准输入
当您将重定向与标准输入' < '（小于符号）一起使用时，它将文件名test2用作命令的输入，并且输出将显示在终端中。

```shell
# cat < test2
 
This is test2 file.
```

### 12.重定向单个文件中包含的多个文件
这将创建一个名为test3的文件，所有输出都将重定向到新创建的文件中。

```shell
# cat test test1 test2 > test3
```
### 13.在单个文件中对多个文件的内容进行排序
这将创建一个文件test4，并将cat命令的输出通过管道传递到进行排序，结果将重定向到新创建的文件中。

```shell
# cat test test1 test2 test3 | sort > test4
```
本文介绍了cat命令的基本命令。在下一篇文

章中，我们将介绍更高级的cat命令。



