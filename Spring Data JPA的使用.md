# Spring Data JPA的使用

## JPA

**JPA (Java Persistence API)** 是 Sun 官方提出的 Java 持久化规范。他只是一个规范

## Spring Data JPA

**Spring Data JPA** 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套 JPA 应用框架，底层使用了 Hibernate 的 JPA 技术实现，可使开发者用极简的代码即可实现对数据的访问和操作。

在复杂查询不多的情况下使用Spring Data JPA是极好的

## 快速开始

这部分是基于Spring Boot，Java8，以一个最简单的方式开始使用Spring Data JPA，然后逐渐进阶，使用高级功能

### 导入jar包

引入Spring Data JPA 和 MySQL的jar包，

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 添加yml配置文件

配置一下数据库

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/kg?useUnicode=true&characterEncoding=utf8
    username: root
    password: 123456
```

### 定义实体类

简单定义一个User类，这里要建一个kg库，添加一张def_user表，有三个字段。后边会将怎么自动生成数据表

```java
@Entity
@Data
public class DefUser {
  @Id
  private String id;
  private String name;
  private String email;
}
```

### Repository接口

定义一个UserRepository接口

```java
@Repository
public interface UserRepository extends JpaRepository<DefUser, String> {
}
```

### 数据库操作

定义一个sever，就可以使用UserRepository接口中的方法了，findAll()可以获取表中所有的user。可以写个controller或者单元测试看一下结果，这里就不贴代码了，可以去看我写的demo

```java
@Service
public class UserService {

  @Autowired
  private UserRepository userRepository;

  public List<DefUser> getAll() {
      // 获取user表中所有数据
    return userRepository.findAll();
  }
```

至此就可以简单使用Spring Data JPA了，然后就是详细的介绍了

## 配置文件

```yaml
spring:
  jpa:
    properties:
      hibernate:
#        用MySQL5InnoDBDialect生成的表的引擎才是InnoDB
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      naming:
#        物理名称命名策略，表名，字段为小写，当有大写字母的时候会添加下划线分隔符号，这是默认策略
#        如果是org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl，直接映射，不会做过多的处理
        physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
#      update： 每次运行，没有表会新建表，表内有数据不会清空，只会更新
#      create： 先删除表，在重新生成表
#      create-drop：每次程序结束的时候会清空表
#      validate：运行程序会校验数据与数据库的字段类型是否相同，不同会报错
#      none: 什么也不做
      ddl-auto: update
#      控制台数据sql语句
    show-sql: true
```



## 注解

### 实体类

- **@Entity：表明该类是对应数据库的实体类，必选**
  - name: 只有一个name属性，可选。name对应数据库的table名
- **@Table：用来指定数据库中的表，可选**
  - name：对应数据库的table名，如果表名跟实体名不一致则必选
  - catalog，schema：在MySQL中，没有catalog，schema其实是数据库名。配置文件中指定了数据库名，所以schema应该也用不上
  - uniqueConstraints：用来指定唯一索引，uniqueConstraints = {@UniqueConstraint(columnNames={"uid","email"})}
- **@Id：用在主键上**
- @GeneratedValue：用于标注主键的生成策略，可选
  - strategy：使用方法@GeneratedValue(strategy = GenerationType.AUTO)
    - GenerationType.AUTO：默认策略，会从以下三个中选一个合适的
    - **GenerationType.IDENTITY：主键自增**
    - GenerationType.TABLE：使用数据库中的表提供主键，与@TableGenerator配合使用
    - GenerationType.SEQUENCE：在某些数据库中,比如Oracle,不支持主键自增长,提供了一种叫做"序列(sequence)"的机制生成主键
- **@Column：指定数据库字段的属性**
  - **name：数据库字段名**
  - unique：是否有唯一索引
  - length：字段的长度，仅对字符串类型的列生效，默认255
  - nullable：字段是否能为空，默认false
  - insertable，updatable：插入，更新时是否包含该列，默认true
  - columnDefinition：生成列的 DDL 时使用的 SQL 片段。示例 columnDefinition = "char(11) NOT NULL"
  - table：当前列所属的表名
  - precision，scale：仅对十进制数值有效，分别表示有效数值的总位数，小数位的总位数
  - @Temporal：用在日期类型上
    - TemporalType.DATE：日期格式 "yyyy-mm-dd"。示例@Temporal(TemporalType.DATE)
    - TemporalType.TIME：日期格式 "HH:MM:ss"
    - TemporalType.TIMESTAMP：日期格式 "yyyy-mm-dd HH:MM:ss"
  - @Enumerated：用在枚举类型上
    - EnumeType.STRING：保存枚举类型的实际值，要加@Columm设置非空
    - EnumeType.ORDINAL：枚举类型的索引值，从0开始
  - @Lob
    - 数据库字段为大文本（longtext），属性类型用String
    - 数据库字段为longblob，属性类型用byte[]
    - 如果不加@Lob，属性类型用byte[]，数据库应该对应tinyblob
  - @Basic(fetch=FetchType.LAZY) ：延迟加载，只有在调用这个属性的getter方法时才把数据加载到内存中，主要是对大文件，需要的时候才加载到内存
  - **@Transient：这个属性不作为持久化字段**

### 一对多

**一对多或者多对一的关系维护方通常是多的一方，多的一方用用外键跟一的一方关联**

比如用户跟书籍的关系是一对多的关系，可以在book类中添加@ManyToOne，这样就会在book中生成一个外键user_id

```java
  @ManyToOne
  private DefUser user;
```

如果也需要查书籍的时候带出用户信息，可以在user中加上@OneToMany

```java
  @OneToMany(mappedBy = "user",fetch = FetchType.EAGER)
  private Set<Book> books;
```



### 多对多

比如一个用户与权限是多对多的关系

如果只是需要查用户信息的时候带出权限，直接在user类中加上下边的代码就可以了，role类可以什么都不加

```java
  @ManyToMany(fetch = FetchType.EAGER)
  @JoinTable(name = "user_role",
      joinColumns = {@JoinColumn(name = "user_id", referencedColumnName = "id")},
      inverseJoinColumns = {@JoinColumn(name = "role_id", referencedColumnName = "id")})
  private Set<Role> roles;
```

- @ManyToMany：表示多对多
-  @JoinTable：多对多的中间表的一些信息
  - name：中间表的表名
  - joinColumns：关系维护方的信息
    - @JoinColumn 中 referencedColumnName 表示user表中的字段id，name表示user_role中的字段user_id，对应user.id
  - inverseJoinColumns
    - @JoinColumn 中 referencedColumnName 表示role表中的字段id，name是id对应的user_role中的字段role_id，对应role.id

## 简单查询

### JpaRepository

JpaRepository接口中提供了一些常用的数据操作方法，比如

```java
List<T> findAll();
List<T> findAllById(Iterable<ID> ids);
<S extends T> List<S> saveAll(Iterable<S> entities);
void deleteAllByIdInBatch(Iterable<ID> ids);
T getById(ID id);
```

写一个BookRepository继承JpaRepository，第一个类型表示这个Repository对应的实体类，第一个表示主键的数据类型

```java
@Repository
public interface BookRepository extends JpaRepository<Book,String> {
}
```

在service中就可以直接调用JpaRepository接口中的方法了

```java
@Service
public class BookService {
  @Autowired
  private BookRepository bookRepository;

  public List<Book> findAllBook() {
      // 返回book表的所有数据
    return bookRepository.findAll();
  }
}
```

### 自定义简单查询

自定义的简单查询就是根据方法名来自动生成SQL，主要的语法是`findXXBy,readAXXBy,queryXXBy,countXXBy, getXXBy`后面跟属性名称，举几个例子：

```java
User findByUserName(String userName);

User findByUserNameOrEmail(String username, String email);

Long deleteById(Long id);

Long countByUserName(String userName);

List<User> findByEmailLike(String email);

User findByUserNameIgnoreCase(String userName);

List<User> findByUserNameOrderByEmailDesc(String email);
```



### @Query

可以在@Query注解中写SQL语句

可以用`?`加数字表示第一个占位符，数字要从1开始，只要一个参数的话可以不加数字。也可以使用@Param注解，@Param中的字符串跟冒号后的字符串要相同

```java
  @Query(value = "select b from Book b where b.price>?1 and b.price<?2 ")
  List<Book> findByPriceRange(long price1, long price2);
```

```java
@Query(value = "select b from Book b where b.price>:price1 and b.price<:price2 ")
 List<Book> findByPriceRange(@Param("price1")long price1, @Param("price2") long price2);
```

使用nativeQuery时表示原生SQL语句，原生SQL语句才可以使用`*`，而且`form`后是表名，不是Java对象，上边的SQL语句是 不能这么写的

```java
 @Query(value = "select * from book b where b.name like %?%", nativeQuery = true)
  List<Book> findByName(String name);
```



## 复杂查询

要实现动态SQL或者其他复杂查询最好是Repository**继承JpaSpecificationExecutor接口**

```java
@Repository
public interface BookRepository extends JpaRepository<Book, String>, JpaSpecificationExecutor<Book> {
    }
```

这个接口中有一些方法，传入Specification对象，Specification是一个接口

```java
List<T> findAll(@Nullable Specification<T> spec);
Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable);
```

使用示例

```java
@Test
  public List<Book> getBook(){
    String bookId = "e123w";
    String bookName = "世界";
    Specification<Book> specification = (root, criteriaQuery, criteriaBuilder) -> {
      List<Predicate> predicates = new ArrayList<>();
      // 条件1：bookId等于book表的id
      predicates.add(criteriaBuilder.equal(root.get("id"), bookId));
      // 条件2：根据bookName模糊查找
      predicates.add(criteriaBuilder.like(root.get("name"), "%"+bookName +"%"));
       // 还可以加其他条件，比如between，大于小于，关联表等
      return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
    };
    return bookRepository.findAll(specification);
  }
```



## 可能遇到的问题

### 一对多双向映射 ，结果对象相互迭代 ，造成堆栈溢出

这大概有两种情况，

- 一种是向前端传数据时出错，转换json时忽略循环字段，jackson包对应的相关注解： `@JsonIgnoreProperties`、`@JsonIgnore`

fastjson包对应的相关注解： `@JSONField(serialize = false)`

- 调用toString()方法时出错，比如使用System.out.println()输出，这时候要把toString()方法中循环引用的对象去掉就好了。这个问题在使用lombok的@data注解时不容易被发现，@data是会自动生成toString()方法的

## 源码
demo地址https://github.com/Saul-Zhang/springboot-demo/tree/main/springboot-jpa

