# Spring Boot 自动装配初涉

## 前言

第一搭建Spring Boot项目的时候，我的内心是这样的，卧槽？！！这么简单！！完全不需要配置文件，只需要一个main方法就能跑起来。我陷入了沉思。。。经过一番学习，终于了解到了这要归功于Spring Boot的自动装配（Auto-configuration），也可以叫自动配置，但是叫装配好像高大上一点哈。

## 自动装配

> Spring Boot auto-configuration attempts to automatically configure your Spring application based on the jar dependencies that you have added.

![](http://saulimg.zsly.xyz/img/18ce9b90252e63bc25c1bafdb455185e_t.gif)

Spring Boot 官方文档上给了这么一个定义，初看并不容易理解。其实说白了，自动装配就是Spring Boot为我们自动创建了bean。

比如我们要在项目中使用redis

- 我们能先添加redis的依赖

  ```java
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

- 在配置文件中添加redis 的配置

  ```java
  spring.redis.host=localhost
  spring.redis.port=6379
  ```

- 然后我们就能通过@Autowired引入RedisTemplate对redis 进行操做了

  ```java
  @RestController
  public class RedisController {
  	@Autowired
  	private RedisTemplate<String, String> redisTemplate;
  
  	@GetMapping("/test")
  	public String test() {
  		redisTemplate.opsForValue().set("test", "test demo");
  		return "Test Demo";
  	}
  }
  ```

  这就是Spring Boot 的自动装配机制。我们并没有通过XML或者注解形式将RedisTemplate注入到IoC容器中，但是

## @SpringBootApplication注解

在Spring Boot项目的启动类中都会有这个注解

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

从这个注解CTRL+鼠标左键进入后看到的主要部分就是下边这个

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ....
}
```

可以看到其实@SpringBootApplication就是由**@SpringBootConfiguration，@EnableAutoConfiguration，@ComponentScan** 三个注解组成的，完全可以把@SpringBootApplication 换成这三个，就像@RestController 可以换成@Controller 和@ResponseBody

### @SpringBootConfiguration

``` java
@Configuration
public @interface SpringBootConfiguration {
}
```

可以看到**@SpringBootConfiguration 注解其实就是@Configuration注解**。@Configuration 用于修饰一个类，表示这个类的作用是配置Spring 容器，因为用@Bean 修饰这个类中的方法，方法的返回值作为一个bean装载到Spring 容器中。

### @ComponentScan

这个注解就是**指定了需要扫描package的根路径**，默认扫描当前包及其字包下的注解。会将@Controller @Service @Component @Repository 等注解加载到IOC容器中。

### @EnableAutoConfiguration

@EnableAutoConfiguration是这里最重要的注解，**他用来自动载入Spring Boot应用的所有默认配置**。

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    ...
}
```

#### @AutoConfigurationPackage

#### @Import

## 总结

Spring Boot自动装配的原理并不是非常复杂，其实背后的主要原理就是条件注解。 当我们使用@EnableAutoConfiguration注解激活自动装配时，实质对应着很多XXXAutoConfiguration类在执行装配工作，这些XXXAutoConfiguration类是在spring-boot-autoconfigure jar中的META-INF/spring.factories文件中配置好的，@EnableAutoConfiguration通过SpringFactoriesLoader机制创建XXXAutoConfiguration这些bean。XXXAutoConfiguration的bean会依次执行并判断是否需要创建对应的bean注入到Spring容器中。 在每个XXXAutoConfiguration类中，都会利用多种类型的条件注解@ConditionOnXXX对当前的应用环境做判断，如应用程序是否为Web应用、classpath路径上是否包含对应的类、Spring容器中是否已经包含了对应类型的bean。如果判断条件都成立，XXXAutoConfiguration就会认为需要向Spring容器中注入这个bean，否则就忽略。

## 参考

- [Spring Boot 2.1.15.RELEASE 文档](https://docs.spring.io/spring-boot/docs/2.1.15.RELEASE/reference/html/using-boot-auto-configuration.html)
- [Spring Boot自动装配详解](https://segmentfault.com/a/1190000019425477)

