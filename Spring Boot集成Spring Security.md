# Spring Boot集成Spring Security

## Spring Security

> Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
>
> 
>
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

## 快速开始

### 导入依赖

新建一个Spring Boot项目，直接导入Spring Security的依赖

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
```

### 访问页面

访问`http://localhost:8080/`，会被重定向到`http://localhost:8080/login`，出现一个登录页面

默认的用户名是`user`，密码会由Spring Security生成，可以从控制台看到，每次重启时密码都会改变

```java
Using generated security password: 1e3780a0-9ffb-434a-b54e-b45b76242ed2
```

输入用户名和密码就可以正常访问了，但是实际开发中，这样显示是不合理的，用户名和密码应该是由我们自己配置

## 