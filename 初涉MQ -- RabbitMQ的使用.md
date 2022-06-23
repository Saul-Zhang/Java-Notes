# 初涉MQ -- RabbitMQ的使用

## 消息队列

消息队列（Message Queue，简称MQ），指保存消息的一个容器，本质是个队列。

## 使用场景

消息队列的经典使用场景**异步**，**解耦**，**削峰**

### 异步

比如一个支付场景下，用户付款时还需要扣减优惠券，增减积分，短信通知，如果这整个流程是同步的，那么这个支付过程会花费大量的时间。其实整个支付过程，用户只要付过钱，接口就应该正常返回数据了，而不应该等着扣减优惠券，增减积分等操作

### 解耦

同样上边的支付场景，用户支付后通过MQ发送消息，其他短信通知，增减积分等系统只用监听MQ就可以了，不用把代码耦合在支付系统中

### 削峰

比如秒杀活动，当请求量非常大时，服务器没办法处理那么多请求，就把请求放在MQ中，服务器按能力消费。

## 相关概念

### Broker

RabbitMQ Server，服务器实体。

### Vhost

虚拟主机，一个 broker 里可以开设多个 vhost，用作不同用户的权限分离，连接到 RabbitMQ 默认就有一个名为 "/" 的 vhost 可用

![image-20210422164337661](http://saulimg.zsly.xyz/img/image-20210422164337661.png)

### Exchange

消息队列交换机。**按一定的规则将消息路由转发到某个队列**。

![image-20220124180428797](https://gitee.com/SaulZ/img/raw/master/img/image-20220124180428797.png)

**所有队列都有一个默认的交换机和路由键，默认的交换机名称是空，路由键跟队列同名**

#### Direct Exchange

直连交换机，此交换机需要绑定一个队列，要求**该消息与一个特定的路由键（Routing key）完全匹配**。简单点说就是一对一的，点对点的发送。

在provider中

```java
@Configuration
public class DirectRabbitConfig {

    @Bean
    public Queue rabbitmqDemoDirectQueue() {
        /**
         * 1、name:    队列名称
         * 2、durable: 是否持久化
         * 3、exclusive: 是否独享、排外的。如果设置为true，定义为排他队列。则只有创建者可以使用此队列。也就是private私有的。
         * 4、autoDelete: 是否自动删除。也就是临时队列。当最后一个消费者断开连接后，会自动删除。
         * */
        return new Queue(RabbitMQConfig.RABBITMQ_DEMO_TOPIC, true, false, false);
    }

    @Bean
    public DirectExchange rabbitmqDemoDirectExchange() {
        //Direct交换机
        return new DirectExchange(RabbitMQConfig.RABBITMQ_DEMO_DIRECT_EXCHANGE, true, false);
    }

    @Bean
    public Binding bindDirect() {
        //链式写法，绑定交换机和队列，并设置匹配键
        return BindingBuilder
                //绑定队列
                .bind(rabbitmqDemoDirectQueue())
                //到交换机
                .to(rabbitmqDemoDirectExchange())
                //并设置特定的路由键
                .with(RabbitMQConfig.RABBITMQ_DEMO_DIRECT_ROUTING);
    }
}
```

```java
@Service
public class RabbitMQServiceImpl implements RabbitMQService {
    //日期格式化
    private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Override
    public String sendMsg(String msg) throws Exception {
        try {
            Map<String, Object> map = getMessage(msg);
            rabbitTemplate.convertAndSend(RabbitMQConfig.RABBITMQ_DEMO_DIRECT_EXCHANGE, RabbitMQConfig.RABBITMQ_DEMO_DIRECT_ROUTING, map);
            return "ok";
        } catch (Exception e) {
            e.printStackTrace();
            return "error";
        }
    }
    
     private Map<String, Object> getMessage(String msg) {
        String msgId = UUID.randomUUID().toString().replace("-", "").substring(0, 32);
        String sendTime = sdf.format(new Date());
        Map<String, Object> map = new HashMap<>();
        map.put("msgId", msgId);
        map.put("sendTime", sendTime);
        map.put("msg", msg);
        return map;
    }
}
```

```java
@RestController
@RequestMapping("/rabbitmq")
public class RabbitMQController {
    @Resource
    private RabbitMQService rabbitMQService;

    @PostMapping("/sendMsg")
    public String sendMsg(@RequestParam(name = "msg") String msg) throws Exception {
        return rabbitMQService.sendMsg(msg);
    }
}
```



在consumer中

```java
@Component
//监听的队列的名字
@RabbitListener(queues = {RabbitMQConfig.RABBITMQ_DEMO_TOPIC})
public class RabbitMQConsumer {
    @RabbitHandler
    public void process(Map map) {
        System.out.println("队列[" + RabbitMQConfig.RABBITMQ_DEMO_TOPIC + "]收到消息：" + map.toString());
    }
}
```

![image-20210422165416975](http://saulimg.zsly.xyz/img/image-20210422165416975.png)

![image-20210422165435272](https://gitee.com/SaulZ/img/raw/master/img/image-20210422165435272.png)

#### Fanout exchange

扇型交换机，将队列绑定到交换机上。**一个发送到交换机的消息都会被转发到与该交换机绑定的所有队列上**。很像子网广播，每台子网内的主机都获得了一份复制的消息。简单点说就是发布订阅。

在provider中

```java
@Configuration
public class FanoutRabbitConfig {

    @Bean
    public Queue fanoutExchangeQueueA() {
        //队列A
        return new Queue(RabbitMQConfig.FANOUT_EXCHANGE_QUEUE_TOPIC_A, true, false, false);
    }

    @Bean
    public Queue fanoutExchangeQueueB() {
        //队列B
        return new Queue(RabbitMQConfig.FANOUT_EXCHANGE_QUEUE_TOPIC_B, true, false, false);
    }

    @Bean
    public FanoutExchange rabbitmqDemoFanoutExchange() {
        //创建FanoutExchange类型交换机
        return new FanoutExchange(RabbitMQConfig.FANOUT_EXCHANGE_DEMO_NAME, true, false);
    }

    @Bean
    public Binding bindFanoutA() {
        //队列A绑定到FanoutExchange交换机
        return BindingBuilder.bind(fanoutExchangeQueueA()).to(rabbitmqDemoFanoutExchange());
    }

    @Bean
    public Binding bindFanoutB() {
        //队列B绑定到FanoutExchange交换机,扇型交换机, 路由键无需配置,配置也不起作用
        return BindingBuilder.bind(fanoutExchangeQueueB()).to(rabbitmqDemoFanoutExchange());
    }
}
```

```java
    /**
     * 扇型交换机发送消息
     * @param msg 消息内同
     * @return 是否发送成功
     * @throws Exception e
     */
    @PostMapping("/publish")
    public String publish(@RequestParam(name = "msg") String msg) throws Exception {
        return rabbitMQService.sendMsgByFanoutExchange(msg);
    }
```

```java
    @Override
    public String sendMsgByFanoutExchange(String msg) throws Exception {
        Map<String, Object> message = getMessage(msg);
        try {
            rabbitTemplate.convertAndSend(RabbitMQConfig.FANOUT_EXCHANGE_DEMO_NAME, "", message);
            return "ok";
        } catch (Exception e) {
            e.printStackTrace();
            return "error";
        }
    }
```



![image-20210422172349440](http://saulimg.zsly.xyz/img/image-20210422172349440.png)

![image-20210422172324654](https://gitee.com/SaulZ/img/raw/master/img/image-20210422172324654.png)

#### Topic Exchange

直接翻译的话叫做主题交换机，如果从用法上面翻译可能叫通配符交换机会更加贴切。这种交换机是使用通配符去匹配，路由到对应的队列。通配符有两种："*" 、 "#"。需要注意的是通配符前面必须要加上"."符号。

`*` 符号：有且只匹配一个词。比如 `a.*`可以匹配到"a.b"、"a.c"，但是匹配不了"a.b.c"。

`#` 符号：匹配一个或多个词。比如"rabbit.#"既可以匹配到"rabbit.a.b"、"rabbit.a"，也可以匹配到"rabbit.a.b.c"。

在provider中

```java
@Configuration
public class TopicRabbitConfig {
    @Bean
    public Queue topicExchangeQueueA() {
        //创建队列1
        return new Queue(RabbitMQConfig.TOPIC_EXCHANGE_QUEUE_A, true, false, false);
    }

    @Bean
    public Queue topicExchangeQueueB() {
        //创建队列2
        return new Queue(RabbitMQConfig.TOPIC_EXCHANGE_QUEUE_B, true, false, false);
    }

    @Bean
    public Queue topicExchangeQueueC() {
        //创建队列3
        return new Queue(RabbitMQConfig.TOPIC_EXCHANGE_QUEUE_C, true, false, false);
    }

    @Bean
    public TopicExchange rabbitmqDemoTopicExchange() {
        //配置TopicExchange交换机
        return new TopicExchange(RabbitMQConfig.TOPIC_EXCHANGE_DEMO_NAME, true, false);
    }

    @Bean
    public Binding bindTopicA() {
        //队列A绑定到TopicExchange交换机，路由键为rabbit.#，所以消息携带的路由键以rabbit开头,都会匹配到队列A
        return BindingBuilder.bind(topicExchangeQueueA())
                .to(rabbitmqDemoTopicExchange())
                .with("rabbit#");
    }

    @Bean
    public Binding bindTopicB() {
        //队列B绑定到TopicExchange交换机，路由键为a.*，所以消息携带的路由键以开头并且a后边只有一位,都会匹配到队列b
        return BindingBuilder.bind(topicExchangeQueueB())
                .to(rabbitmqDemoTopicExchange())
                .with("a.*");
    }

    @Bean
    public Binding bindTopicC() {
        //队列C绑定到TopicExchange交换机
        return BindingBuilder.bind(topicExchangeQueueC())
                .to(rabbitmqDemoTopicExchange())
                .with("a.*");
    }
}
```

```java
 /**
     * 主题交换机发送消息
     * @param msg
     * @param routingKey
     * @return
     * @throws Exception
     */
    @PostMapping("/topicSend")
    public String topicSend(@RequestParam(name = "msg") String msg, @RequestParam(name = "routingKey") String routingKey) throws Exception {
        return rabbitMQService.sendMsgByTopicExchange(msg, routingKey);
    }
```

```java
 @Override
    public String sendMsgByTopicExchange(String msg, String routingKey) throws Exception {
        Map<String, Object> message = getMessage(msg);
        try {
            //发送消息
            rabbitTemplate.convertAndSend(RabbitMQConfig.TOPIC_EXCHANGE_DEMO_NAME, routingKey, message);
            return "ok";
        } catch (Exception e) {
            e.printStackTrace();
            return "error";
        }
    }
```



![image-20210422163106677](http://saulimg.zsly.xyz/img/image-20210422163106677.png)

![image-20210422163127623](https://gitee.com/SaulZ/img/raw/master/img/image-20210422163127623.png)

![image-20210422165527326](http://saulimg.zsly.xyz/img/image-20210422165527326.png)

![image-20210422165544157](https://gitee.com/SaulZ/img/raw/master/img/image-20210422165544157.png)

#### Headers Exchange

这种交换机用的相对没这么多。**它跟上面三种有点区别，它的路由不是用routingKey进行路由匹配，而是在匹配请求头中所带的键值进行路由**

在provider中

```java
@Configuration
public class HeadersRabbitConfig {

    @Bean
    public Queue headersQueueA() {
        return new Queue(RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_A, true, false, false);
    }

    @Bean
    public Queue headersQueueB() {
        return new Queue(RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_B, true, false, false);
    }

    @Bean
    public HeadersExchange rabbitmqDemoHeadersExchange() {
        return new HeadersExchange(RabbitMQConfig.HEADERS_EXCHANGE_DEMO_NAME, true, false);
    }

    @Bean
    public Binding bindHeadersA() {
        Map<String, Object> map = new HashMap<>();
        map.put("key_one", "java");
        map.put("key_two", "rabbit");
        //全匹配，只有消息头中包含{"key_one":"java","key_two": "rabbit"}才会匹配搭配队列A
        return BindingBuilder.bind(headersQueueA())
                .to(rabbitmqDemoHeadersExchange())
                .whereAll(map).match();
    }

    @Bean
    public Binding bindHeadersB() {
        Map<String, Object> map = new HashMap<>();
        map.put("headers_A", "coke");
        map.put("headers_B", "sky");
        //部分匹配
        return BindingBuilder.bind(headersQueueB())
                .to(rabbitmqDemoHeadersExchange())
                .whereAny(map).match();
    }
}
```

```java
@Override
    public String sendMsgByHeadersExchange(String msg, Map<String, Object> map) throws Exception {
        try {
            MessageProperties messageProperties = new MessageProperties();
            //消息持久化
            messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
            messageProperties.setContentType("UTF-8");
            //添加消息
            messageProperties.getHeaders().putAll(map);
            Message message = new Message(msg.getBytes(), messageProperties);
            rabbitTemplate.convertAndSend(RabbitMQConfig.HEADERS_EXCHANGE_DEMO_NAME, null, message);
            return "ok";
        } catch (Exception e) {
            e.printStackTrace();
            return "error";
        }
    }
```

```java
 @PostMapping("/headersSend")
    public String headersSend(@RequestParam(name = "msg") String msg,
                              @RequestParam(name = "json") String json) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        Map<String, Object> map = mapper.readValue(json, Map.class);
        return rabbitMQService.sendMsgByHeadersExchange(msg, map);
    }
```

在consumer中

```java
@Component
public class HeadersExchangeConsumer {
    @RabbitListener(queuesToDeclare = @Queue(RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_A))
    public void processA(Message message) throws Exception {
        MessageProperties messageProperties = message.getMessageProperties();
        String contentType = messageProperties.getContentType();
        System.out.println("队列[" + RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_A + "]收到消息：" + new String(message.getBody(), contentType));
    }

    @RabbitListener(queuesToDeclare = @Queue(RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_B))
    public void processB(Message message) throws Exception {
        MessageProperties messageProperties = message.getMessageProperties();
        String contentType = messageProperties.getContentType();
        System.out.println("队列[" + RabbitMQConfig.HEADERS_EXCHANGE_QUEUE_B + "]收到消息：" + new String(message.getBody(), contentType));
    }
}
```



![image-20210422175526148](http://saulimg.zsly.xyz/img/image-20210422175526148.png)

![image-20210422180102839](https://gitee.com/SaulZ/img/raw/master/img/image-20210422180102839.png)

### Routing key

绑定键，**指定当前消息被哪个队列接收**

### Binding

在 RabbitMQ 中是通过 routing key 把 queue 绑定到 exchange 上，这种绑定关系即 binding

## 源码

以上源码的地址https://github.com/Saul-Zhang/springboot-demo/tree/main/springboot-rabbitmq

## 参考

https://www.zhihu.com/question/54152397/answer/923992679

https://blog.csdn.net/qq_35387940/article/details/100514134
