# 初涉Dubbo---Spring Boot整合dubbo

## Dubbo

Apache Dubbo |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。



根据Dubbo[服务化最佳实践](!http://dubbo.apache.org/zh/docs/v2.7/user/best-practice/)的建议，将服务接口、服务模型、服务异常等均放在 API 包中，因为服务模型和异常也是 API 的一部分，这样做也符合分包原则：重用发布等价原则(REP)，共同重用原则(CRP)。

## 创建工程

先创建一个工程dubbo-springboot，然后创建三个模块

dubbo-provider模块作为服务提供者；dubbo-consumer工程作为服务消费者；dubbo-api主要放JavaBean，接口等公用公用的类，服务提供者和服务消费者可以在pom文件中引入这个模块；

![image-20201205173729245](http://saulimg.zsly.xyz/img/image-20201205173729245.png)



分别在dubbo-provider和dibbo-consumer的pom.xml中引入dubbo-api。

```xml
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

他跟dubbo-api中的pom是对应的

```xml
<groupId>com.example</groupId>
<artifactId>dubbo-api</artifactId>
<version>0.0.1-SNAPSHOT</version>
```

## 参考

[Dubbo2.7文档](!http://dubbo.apache.org/zh/docs/v2.7)

