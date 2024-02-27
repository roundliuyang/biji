# Redis HyperLogLog：数据统计的轻量级解决方案



## **引言**

在现代数据驱动的应用中，[Redis](https://cloud.tencent.com/product/crs?from_column=20065&from=20065) 以其出色的性能和灵活性成为了不可或缺的工具。

特别是在统计大量数据时，传统的计数方法往往既耗时又占用大量存储空间。

这次，阿七将介绍一种名为 HyperLogLog 的算法，它在 Redis 中的实现让大规模数据统计变得简单且高效。



## **理解 HyperLogLog**



### **1、HyperLogLog 基础**

HyperLogLog 是一种用于**估计集合中唯一元素数量的算法**，它通过概率统计方法，在极小的内存空间内提供近似的计数结果。这种方法特别适用于需要统计巨[大数据](https://cloud.tencent.com/solution/bigdata?from_column=20065&from=20065)集中唯一元素数量的场景。



### **2、HyperLogLog 与传统方法对比**

与传统的精确计数方法相比，**HyperLogLog 在处理大数据集时占用极少的内存**。例如，一个包含数亿唯一元素的数据集可能只需要几百字节的内存来估算其大小。且最大只会使用 12 KB 的内存。



## **Redis 中的 HyperLogLog**



### **1、Redis 与 HyperLogLog**

在 Redis 中，HyperLogLog 提供了一些基本命令来处理这种类型的数据结构。以下是一些基本的 Redis 命令：

- `PFADD key element [element ...]`: 向 HyperLogLog 中添加元素。
- `PFCOUNT key [key ...]`: 计算 HyperLogLog 中的唯一元素数量。
- `PFMERGE destkey sourcekey [sourcekey ...]`: 合并多个 HyperLogLog。

而且，HyperLogLog 提供了惊人的精度与性能平衡。通常，它的标准误差为 0.81%，这对于大多数应用来说已经足够准确。



### **2、代码示例:**

```java
// Redis HyperLogLog 操作示例
Jedis jedis = new Jedis("localhost");
String key = "page_views";

// 添加元素
jedis.pfadd(key, "user1");
jedis.pfadd(key, "user2");

// 获取估算的唯一元素数量
long count = jedis.pfcount(key);
System.out.println("Estimated unique elements: " + count);

// 合并 HyperLogLog
String otherKey = "more_page_views";
jedis.pfadd(otherKey, "user3");
jedis.pfmerge(key, otherKey);

// 再次获取估算数量
long mergedCount = jedis.pfcount(key);
System.out.println("Estimated unique elements after merge: " + mergedCount);
```



### **3、实际应用场景**

1、计算网站某个功能的 UV，比如说某个网站的日访客数据。比如：有多少独立用户播放过这首歌？这一天该页面的独立访问次数有多少？有多少独立用户观看过该视频？

2、社交媒体平台可以用它来估算独特用户的参与度。



## **案例研究**

在这部分，我们可以探讨一个基于真实数据的案例，展示如何在一个 ToC 业务中计算某个功能的使用 UV（唯一访问用户数），使用 Redis HyperLogLog 来实现。

要使用 Redis HyperLogLog 来统计每天展示的 UV，并根据用户手机的设备 UID 进行跟踪，你可以按照以下步骤实现：

设置 Redis HyperLogLog: 对于每个用户访问，你可以使用 HyperLogLog 数据结构来跟踪 UID。

业务ID + 日期为键: 使用日期作为键的一部分，这样你可以对每天的访问进行独立计数。

Java 代码实现: 使用 Jedis，这是一个流行的 Java Redis 客户端，来与 Redis 进行通信。

```java
import redis.clients.jedis.Jedis;

public class UVCounter {
    private Jedis jedis;
    private String static final String BUSINESS_ID = "business_id";
    
    public UVCounter(String host, int port) {
        this.jedis = new Jedis(host, port);
    }

    public void addVisit(String date, String deviceUID) {
        String key = "uv:" + date;
        jedis.pfadd(key, deviceUID);
    }

    public long getUVCount(String date) {
        String key = BUSINESS_ID + ":" + "uv:" + date;
        return jedis.pfcount(key);
    }

    public static void main(String[] args) {
        UVCounter uvCounter = new UVCounter("localhost", 6379);

        // 假设这是今天的日期
        String today = "2023-12-16";

        // 模拟一些用户访问
        uvCounter.addVisit(today, "device1");
        uvCounter.addVisit(today, "device2");
        uvCounter.addVisit(today, "device3");
        uvCounter.addVisit(today, "device1"); // 重复的设备 UID

        // 获取今天的 UV 数
        long uvCount = uvCounter.getUVCount(today);
        System.out.println("Unique Visitors Today: " + uvCount);
    }
}

```



### 总结**

Redis Bloom filter 大部分都知道，毕竟属于面试八股文中很重要的一个知识点。它可以用来解决缓存穿透的问题，可以判断 Redis key 是否在 DB 中，从而避免请求 DB 中不存在的数据，造成 DB 压力。

它可以使用很小的空间，存储大规模的数据。它的特点是：**判断存在不一定存在，但是判断不存在，一定不存在！**

但是 Redis HyperLogLog，很多人都不知道，但是在计算大规模数据的唯一数据量级的场景下，这是一个既高效又节省空间的方法。

Redis 还提供了很多好用的工具，阿七后面会为大家继续介绍，大家可以关注我，追更不迷路



转自：https://cloud.tencent.com/developer/article/2372401