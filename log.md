

## 日志分类

这张图非常清晰地展示了日志框架体系，建议收藏

![img](https://gitee.com/SaulZ/img/raw/master/img/46f9eb50dfa3e14a5a7f58cf9dfb7301.png)

日志框架分为两类，**日志门面**(facade)和**日志实现**

### 日志门面

日志门面定义了一组日志的接口规范，它并不提供底层具体的实现逻辑，比如常用的**SLF4J**(The Simple Logging Facade for Java)，以及**Apache Commons Logging**

### 日志实现

日志实现则是日志具体的实现，包括日志级别控制、日志打印格式、日志输出形式（输出到数据库、输出到文件、输出到控制台等）。**`Log4j`**、**`Log4j2`**、**`Logback`** 以及 **`Java Util Logging`** 则属于这一类。

## SLF4J

使用`SLF4J`经常会看到这样一行代码

```java
private static final Logger logger = LoggerFactory.getLogger(Object.class);
```

**`Logger`是`SLF4J`中的一个接口，这行代码就是通过`LoggerFactory`去拿`SLF4J`提供的一个`Logger`接口的具体实现**，比如我们引入了`Logback`，这里就会拿到的实现类就是`Logback`

**如果引入了`Lombok`，就可以使用注解`@Slf4j`替代这行代码**

## Log4j 

`log4j1`十年前就停止维护了，github地址 ☞https://github.com/apache/logging-log4j1

`log4j2`是`log4j1`的升级版本，github地址 ☞ https://github.com/apache/logging-log4j2，2021 年 12 月被曝出有漏洞的就是`log4j2`，漏洞详情，☞https://www.cve.org/CVERecord?id=CVE-2021-44228。

现在一般说`Log4j` 都是指`log4j2`

```xml
<dependencies>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
  </dependency>
</dependencies>
```

这是使用`log4j2`要引入的maven依赖，如果`log4j-core`的版本低于2.15.0(除了2.12.2, 2.12.3, and 2.3.1)就有被黑的风险，就要升级了

## Logback

`Logback`是作为的`log4j 1.x`项目的继承者，对比`log4j 1.x`有很大改进

github地址 ☞https://github.com/qos-ch/logback

![image-20211230111936044](https://gitee.com/SaulZ/img/raw/master/img/image-20211230111936044.png)

`log4j1`，`SLF4J`和`Logback`看来都是同一个公司开发的，地址☞https://www.qos.ch/

- logback-core，logback的核心模块，另外两个模块要依赖这个模块
- logback-classic，这个模块实现了SLF4J的接口，可以搭配SLF4J使用；这个模块也可以看作是log4j 1.x的重大升级版本
- logback-access，这个模块与Servlet容器（例如Tomcat和Jetty）集成，以提供HTTP访问日志功能。它是作用在Servlet容器级别上，jar包要放在容器内，不是像logback-classic作用在Web应用级别。☞https://logback.qos.ch/access.html

## Spring Boot 中日志实现

创建一个Spring Boot 项目，版本是2.6.2，只有一个依赖`spring-boot-starter-web`

```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
```

这是这个项目日志相关的依赖图

![image-20211228161719739](https://gitee.com/SaulZ/img/raw/master/img/image-20211228161719739.png)

从依赖图中可以看出，默认引入了`logback-core`,说明了**Spring Boot默认使用的日志框架是`logback`**

这里也引入了`log4j-to-slf4`j和`log4j-api`,很多开发者看到这里有`log4j`，纷纷去给Spring Boot项目提issue害怕`log4j`的漏洞，Spring Boot的维护者出来解释，这两个包**只是从`Log4j2`的`api`到`SLF4J`的适配器**，是为了方便使用`log4j2`，但是其实**是没有引入`log4j-core`的**，所以Spring Boot是安全的

我把`log4j-to-slf4`j和`log4j-api`这两个依赖去掉发现项目仍然能够正常启动的

后来我又试了一下在Spring Boot默认依赖下，使用`log4j`的方式创建Logger，虽然这段代码导入的都是`log4j`的的包，但是我发现底层还是使用的`logback`

![image-20211230113338268](https://gitee.com/SaulZ/img/raw/master/img/image-20211230113338268.png)

## 参考

https://logback.qos.ch/

[Spring Boot 日志各种使用姿势，是时候捋清楚了！](![https://www.cnblogs.com/lenve/p/14142244.html)

