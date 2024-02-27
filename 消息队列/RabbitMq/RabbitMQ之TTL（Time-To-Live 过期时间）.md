# RabbitMQ之TTL（Time-To-Live 过期时间）

## **1. 概述**

RabbitMQ可以对消息和队列设置TTL. 目前有两种方法可以设置。第一种方法是通过队列属性设置，队列中所有消息都有相同的过期时间。第二种方法是对消息进行单独设置，每条消息TTL可以不同。如果上述两种方法同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。消息在队列的生存时间一旦超过设置的TTL值，就称为dead message， 消费者将无法再收到该消息。


## **2. 设置队列属性**

通过队列属性设置消息TTL的方法是在queue.declare方法中加入x-message-ttl参数，单位为ms.
例如：

```java
package com.vms.test.zzh.rabbitmq.self;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import org.apache.commons.collections.map.HashedMap;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeoutException;

/**
 * Created by hidden on 2017/2/7.
 */
public class RBttlTest {
    public static final String ip = "xx.xx.xx.73";
    public static final int port = 5672;
    public static final String username = "root";
    public static final String password = "root";

    public static final String queueName = "queue.ttl.test";
    public static final String exchangeName = "exchange.ttl.test";
    public static final String routingKey = "ttl";
    public static final Boolean durable = true;
    public static final Boolean exclusive = false;
    public static final Boolean autoDelete = false;

    public static void main(String[] args) {
        try {
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost(ip);
            factory.setPort(port);
            factory.setUsername(username);
            factory.setPassword(password);

            Connection connection = factory.newConnection();
            Channel channel = connection.createChannel();

            Map<String, Object>  argss = new HashMap<String, Object>();
            argss.put("vhost", "/");
            argss.put("username","root");
            argss.put("password", "root");
            argss.put("x-message-ttl",6000);
            channel.queueDeclare(queueName, durable, exclusive, autoDelete, argss);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

通过RabbitMQ的管理页面可以看到有新的queue生成，并标记为TTL（上面的代码同时会将此queue设置为durable=true,以及包含相关参数，比如vhost=/），如下图所示：

![这里写图片描述](RabbitMQ之TTL（Time-To-Live 过期时间）.assets/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMjA3MjEwMzU5OTk5.png)

另外也可以同rabbitmq的命令行模式来设置：

```shell
rabbitmqctl set_policy TTL ".*" '{"message-ttl":60000}' --apply-to queues
```

还可以通过HTTP接口调用：

```shell
$ curl -i -u guest:guest -H "content-type:application/json"  -XPUT -d'{"auto_delete":false,"durable":true,"arguments":{"x-message-ttl": 60000}}' 
http://localhost:15672/api/queues/{vhost}/{queuename}
```

如果不设置TTL,则表示此消息不会过期。如果将TTL设置为0，则表示除非此时可以直接将消息投递到消费者，否则该消息会被立即丢弃，这个特性可以部分替代RabbitMQ3.0以前支持的immediate参数，之所以所部分代替，是应为immediate参数在投递失败会有basic.return方法将消息体返回（这个功能可以利用死信队列来实现）。



## **3. 设置消息属性**

针对每条消息设置TTL的方法是在basic.publish方法中加入expiration的属性参数，单位为ms.
关键代码：

```java
        AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
        builder.deliveryMode(2);
        builder.expiration("6000");
        AMQP.BasicProperties  properties = builder.build();

        channel.basicPublish(exchangeName,routingKey,mandatory,properties,"ttlTestMessage".getBytes());

```

也可以写成：

```shell
AMQP.BasicProperties properties = new AMQP.BasicProperties();
properties.setExpiration("60000");
channel.basicPublish(exchangeName,routingKey,mandatory,properties,"ttlTestMessage".getBytes());
```

具体代码如下所示：

```java
public static void main(String[] args) throws InterruptedException {
    sendTTLMessage();
    TimeUnit.SECONDS.sleep(5);
    consumeTTLMessage();
}

public static void sendTTLMessage(){
    try {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(ip);
        factory.setPort(port);
        factory.setUsername(username);
        factory.setPassword(password);

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
        builder.deliveryMode(2);
        builder.expiration("6000");
        AMQP.BasicProperties  properties = builder.build();

        channel.basicPublish(exchangeName,routingKey,mandatory,properties,"ttlTestMessage".getBytes());

        channel.close();
        connection.close();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (TimeoutException e) {
        e.printStackTrace();
    }
}

public static void consumeTTLMessage(){
    try {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(ip);
        factory.setPort(port);
        factory.setUsername(username);
        factory.setPassword(password);

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(queueName, true, consumer);
        QueueingConsumer.Delivery delivery = consumer.nextDelivery();
        String message = new String(delivery.getBody());
        System.out.println(" [X] Received '" + message + "'");

        channel.close();
        connection.close();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (TimeoutException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

```

当TimeUnit.SECONDS.sleep(5);设置为5s时可以消费到消息，当设置为7s时，则消费不到消息，因为此时已经超时了。

还可以通过HTTP接口调用如下：

```shell
$ curl -i -u guest:guest -H "content-type:application/json"  -XPOST -d'{"properties":{"expiration":"60000"},"routing_key":"routingkey","payload":"my body","payload_encoding":"string"}'  http://localhost:15672/api/exchanges/{vhost}/{exchangename}/publish
```



## **4. 对比**

对于第一种设置队列TTL属性的方法，一旦消息过期，就会从队列中抹去，而第二种方法里，即使消息过期，也不会马上从队列中抹去，因为每条消息是否过期时在即将投递到消费者之前判定的，为什么两者得处理方法不一致？因为第一种方法里，队列中已过期的消息肯定在队列头部，RabbitMQ只要定期从队头开始扫描是否有过期消息即可，而第二种方法里，每条消息的过期时间不同，如果要删除所有过期消息，势必要扫描整个队列，所以不如等到此消息即将被消费时再判定是否过期，如果过期，再进行删除。



## **5. Queue TTL**

queue.declare 命令中的 x-expires 参数控制 queue 被自动删除前可以处于未使用状态的时间。未使用的意思是 queue 上没有任何 consumer ，queue 没有被重新声明，并且在过期时间段内未调用过 basic.get 命令。该方式可用于，例如，RPC-style 的回复 queue, 其中许多 queue 会被创建出来，但是却从未被使用。

服务器会确保在过期时间到达后 queue 被删除，但是不保证删除的动作有多么的及时。在服务器重启后，持久化的 queue 的超时时间将重新计算。

用于表示超期时间的 x-expires 参数值以毫秒为单位，并且服从和 x-message-ttl 一样的约束条件，且不能设置为 0 。所以，如果该参数设置为 1000 ，则表示该 queue 如果在 1s之内未被使用则会被删除。

下面的 Java 示例创建了一个 queue ，其会在 30 分钟不使用的情况下判定为超时。

```java
Map<String, Object> args = new HashMap<String, Object>();  
args.put("x-expires", 1800000);  
channel.queueDeclare("myqueue", false, false, false, args);  
```



参考资料

1. [Time-To-Live Extensions](http://www.rabbitmq.com/ttl.html)
2. [RabbitMQ 之 TTL 详解（翻译）](https://my.oschina.net/moooofly/blog/101765)
3. [RabbitMQ（三）RabbitMQ消息过期时间（TTL）



转自：https://blog.csdn.net/u013256816/article/details/54916011

