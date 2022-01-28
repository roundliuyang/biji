## Lock和Condition（下）：Dubbo如何用管程实现异步转同

在上一篇文章中，我们讲到 Java SDK 并发包里的 Lock 有别于 synchronized 隐式锁的三个特性：能够响应中断、支持超时和非阻塞地获取锁。那今天我们接着再来详细聊聊 Java SDK 并发包里的 Condition，**Condition 实现了管程模型里面的条件变量**。

在[《08 | 管程：并发编程的万能钥匙》](https://time.geekbang.org/column/article/86089)里我们提到过 Java 语言内置的管程里只有一个条件变量，而 Lock&Condition 实现的管程是支持多个条件变量的，这是二者的一个重要区别。

在很多并发场景下，支持多个条件变量能够让我们的并发程序可读性更好，实现起来也更容易。例如，实现一个阻塞队列，就需要两个条件变量。

