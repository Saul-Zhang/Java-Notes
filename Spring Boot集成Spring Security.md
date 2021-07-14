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

默认的用户名是`user`，密码会由Spring Security生成，就是一个UUID，可以从控制台看到，每次重启时密码都会改变

```控制台输出
Using generated security password: 1e3780a0-9ffb-434a-b54e-b45b76242ed2
```

输入用户名和密码就可以正常访问了，但是实际开发中，这样显示是不合理的，用户名和密码应该是由我们自己配置

## 配置用户名密码

### 使用配置文件配置

```yaml
spring:
  security:
    user:
      name: zhangsong
      password: zsly.xyz
```

只要在application.yml文件中加上这个配置，就可以使用自己定义的用户名和密码登录了

## 使用Java Configuration

```java
  /**
   * 在内存中指定用户
   */
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        //指定加密方法
        .passwordEncoder(passwordEncoder())
        // 用户名
        .withUser("zhangsong")
        // 密码
        .password(passwordEncoder().encode("zsly.xyz"))
        //角色
        .roles("USER")
        //使用and添加多个用户
        .and().withUser("admin").password(passwordEncoder().encode("1234")).roles("ADMIN");
  }
```

这样我们就新生成了两个用户，这样设置后配置文件就失效了，但是一般用户都是保存在数据库中的

### 数据库中获取用户

ORM框架我使用了Spring Data JPA，前边写过怎么用，https://zsly.xyz/archives/how-to-use-spring-data-jpa

新建一个def_user表，我只写了三个字段，然后要实现UserDetails接口

```java
@Data
@Entity(name = "def_user")
public class User implements UserDetails {

  @Id
  @JsonIgnore
  private Long id;

  private String username;

  @JsonIgnore
  private String password;

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return null;
  }

  @Override
  public boolean isAccountNonExpired() {
    return true;
  }

  @Override
  public boolean isAccountNonLocked() {
    return true;
  }

  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }

  @Override
  public boolean isEnabled() {
    return true;
  }
}

```

- UserDetails保存用户的核心信息，后续会将该接口提供的用户信息封装到认证对象`Authentication`中去

UserRepository中只需要写一个根据username获取用户信息就好了

```java
public interface UserRepository extends JpaRepository<User, Long> {

  User findByUsername(String username);
}
```



```java
@Service
public class JdbcUserDetailsService implements UserDetailsService {

  @Autowired
  private UserRepository userRepository;

  @Override
  public UserDetails loadUserByUsername(String input) {
    User user = userRepository.findByUsername(input);
    if (user == null) {
      throw new BadCredentialsException("authenticationFailure.AccountNotFoundException");
    }
    // 通过User类中isEnabled()，isAccountNonExpired()等方法验证用户是否有效
    new AccountStatusUserDetailsChecker().check(user);
    return user;
  }
}

```

- UserDetailsService接口只提供了一个方法，loadUserByUsername()根据username获取用户信息，重写这个方法我们就可以从数据库获取用户了

```java
@EnableWebSecurity
public class WebSecurityConfig{

  @Bean
  public UserDetailsService userDetailsService() {
    InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
    manager.createUser(User.withDefaultPasswordEncoder().username("zhangsong").password("zsly.xyz").roles("USER").build());
    return manager;
  }
}
```

要重写WebSecurityConfigurerAdapter的configure方法

```java
  @Override
  public void configure(AuthenticationManagerBuilder auth) throws Exception {
    // 从数据库中获取用户
    auth.userDetailsService(jdbcUserDetailsService).passwordEncoder(passwordEncoder());
  }

  /**
   * 加密方式
   */
  private PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }

```

从Spring5开始要指定一个加密方法对密码加密，所以数据库的密码一定是存密文

这样就可以读数据库的用户了

## 配置HTTP请求

Spring Security默认会拦截所有请求，当我们需要配置某些接口直接放行，或者自定义登录页时就要配置HttpSecurity了

```java
@Override
  protected void configure(HttpSecurity http) throws Exception {
    // 关闭csrf和cors
    http.csrf().disable().cors().disable()
        // 放行这几个请求
        .authorizeRequests().antMatchers("/login/**", "/logout", "/error/**").permitAll()
        // 其他请求都要验证
        .anyRequest().authenticated()
        .and()
        // 表明是基于表单的身份验证
        .formLogin();
    // 配置登录页面,就是可以把默认的登录页面换成自定义的
    //.loginPage("/login")
    // 登录表单的action
    //.loginProcessingUrl("/toLogin")
    // 登录成功后处理器，比如对于localhost:8080/hello?redirect=www.zsly.xyz，登录成功后重定向到redirect
    //.successHandler(new MySuccessHandler())
    // 登录失败重定向到这个请求
    //.failureUrl("/login?error")
    //.and()
    //.logout()
    //登出后跳转处理器
    //.logoutSuccessHandler(new MyLogoutSuccessHandler())

  }
```

## 代码

完整代码https://github.com/Saul-Zhang/springboot-demo/tree/main/springboot-springSecurity

