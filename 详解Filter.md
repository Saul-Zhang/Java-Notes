# SpringBoot 中使用 Filter

## 过滤器是什么

> A filter is an object that performs filtering tasks on either the request to a resource (a servlet or static content), or on the response from a resource, or both.  
>
> Filters perform filtering in the doFilter method. Every Filter has access to a FilterConfig object from which it can obtain its initialization parameters, a reference to the ServletContext which it can use, for example, to load resources needed for filtering tasks.
>

过滤器是一个对象，它对资源的请求或来自资源的响应执行过滤任务，或两者都执行，这里的资源是指动态资源Servlet以及静态资源css，js，图片等

过滤器在doFilter()方法中执行过滤。每个Filter都可以访问FilterConfig对象，它可以从该对象获得其初始化参数，例如，它可以使用ServletContext引用来加载过滤任务所需的资源

## 过滤器的用途

以下下是Filter接口源代码中列举出的过滤器的一些用途

-  Authentication Filters 身份认证过滤，比如判断用户是否登录
-  Logging and Auditing Filters  日志过滤，比如可以记录特殊用户的特殊请求
-  Image conversion Filters
- Data compression Filters 
- Encryption Filters
- Tokenizing Filters
-  Filters that trigger resource access events 
-  XSL/T filters 
- Mime-type chain Filter

## 快速开始

### @Component

```java
@Component
@Order(1)
public class MyFilter implements Filter {

  @Override
  public void init(FilterConfig filterConfig) throws ServletException {
    System.out.println("MyFilter--init");
  }

  @Override
  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
      FilterChain filterChain) throws IOException, ServletException {
    System.out.println("MyFilter--start");
    // 要继续处理请求，必须添加 filterChain.doFilter()，不加的话状态码还是200，但是不会返回任何数据
    filterChain.doFilter(servletRequest, servletResponse);
    System.out.println("MyFilter--end");
  }

  @Override
  public void destroy() {
    System.out.println("MyFilter--destroy");
  }
}

```

这是Spring Boot使用filter的最简单方式，**只要实现Filter接口中的doFilter()方法，加上@Component注解就可以了**，但是这种方式会拦截所有请求，不能通过配置去拦截指定的 URL。

- **@Order** 注解定义了组件的加载顺序，**值越小越先加载**，值是从`-2147483648(Ordered.LOWEST_PRECEDENCE)`到`2147483647(Ordered.HIGHEST_PRECEDENCE)`，不是必须的

Filter 接口中只有三个方法

- `void init(FilterConfig filterConfig)  当容器初始化 Filter 时调用，该方法在 Filter 的生命周期**只会被调用一次**，一般在该方法中初始化一些资源，FilterConfig 是容器提供给 Filter 的初始化参数，在该方法中可以抛出 ServletException 。init 方法必须执行成功，否则 Filter 可能不起作用，出现以下两种情况时，web 容器中 Filter 可能无效： 1）抛出 ServletException 2）超过 web 容器定义的执行时间。
- void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`   Web 容器每一次请求都会调用该方法。该方法将容器的请求和响应作为参数传递进来， FilterChain 用来调用下一个 Filter。
- `void destroy()`   当容器销毁 Filter 实例时调用该方法，可以在方法中销毁资源，该方法在 Filter 的生命周期只会被调用一次。

### FilterRegistrationBean

实际开发中下边这种写法更常用一些

```java
public class TokenFilter implements Filter {

  @Override
  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
      FilterChain filterChain) throws IOException, ServletException {
    System.out.println("TokenFilter");
    // 要继续处理请求，必须添加 filterChain.doFilter()，不加的话状态码还是200，但是不会返回任何数据
    filterChain.doFilter(servletRequest, servletResponse);
  }
}

```

```java
@Configuration
public class FilterConfig {

  @Bean
  public FilterRegistrationBean<TokenFilter> registerMyFilter() {
    FilterRegistrationBean<TokenFilter> bean = new FilterRegistrationBean<>();
    bean.setOrder(2);
    bean.setFilter(new TokenFilter());
    // 匹配"/hello/"下面的所有url
    bean.addUrlPatterns("/hello/*");
    return bean;
  }
}
```

### @WebFilter ，@ServletComponentScan

**这种方式可以指定过滤的url但是不能指定顺序**

```java
@WebFilter(urlPatterns = "/*")
public class AuthenticationFilter implements Filter {

  @Override
  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
      FilterChain filterChain) throws IOException, ServletException {
    System.out.println("AuthenticationFilter");
    // 要继续处理请求，必须添加 filterChain.doFilter()，不加的话状态码还是200，但是不会返回任何数据
    filterChain.doFilter(servletRequest, servletResponse);
  }
}

```

```java
@SpringBootApplication
//启动类加上@ServletComponentScan
@ServletComponentScan
public class SpringbootFilterApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringbootFilterApplication.class, args);
  }
}
```

## 源码
demo地址https://github.com/Saul-Zhang/springboot-demo/tree/main/springboot-filter

## 参考

https://segmentfault.com/a/1190000025152370

[https://www.jianshu.com/p/596b1b2bab3d](https://segmentfault.com/a/1190000025152370)