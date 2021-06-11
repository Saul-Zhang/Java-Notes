# 详解Filter

```java
@Component
@Order(1)
public class MyFilter implements Filter {

  @Override
  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
      FilterChain filterChain) throws IOException, ServletException {
    System.out.println("MyFilter");
    // 要继续处理请求，必须添加 filterChain.doFilter()，不加的话状态码还是200，但是不会返回任何数据
    filterChain.doFilter(servletRequest, servletResponse);
  }
}

```

- **@Order** 注解定义了组件的加载顺序，值越小越先加载，值是从`-2147483648(Ordered.LOWEST_PRECEDENCE)`到`2147483647(Ordered.HIGHEST_PRECEDENCE)`

