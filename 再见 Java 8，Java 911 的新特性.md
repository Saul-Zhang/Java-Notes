---
title: "文章标题"
---



# 再见 Java 8，Java 9~11 的新特性

## JDK 9

JDK 9 的发布时间是2017年9月，不是长期支持的版本。参考文档👉https://docs.oracle.com/javase/9/whatsnew/toc.htm

### 不可变的集合

List，Map和Set接口增加了.of()及Map.ofEntries()方法，可以创建**不可变的集合**，即创建后集合中的元素不可更改（包括添加、删除、替换、 排序），否则会抛出`java.lang.UnsupportedOperationException` 异常

```java
List immutableList = List.of();
List<Integer> immutableList1 = List.of(1, 2);

Map<Integer, String> immutableMap = Map.of(1, "one", 2, "two");

Map<Integer, String> map = Map.ofEntries(
        entry(1, "a"),
        entry(2, "b"));
```

### 接口支持私有方法

支持private的私有方法和私有静态方法

```java
public interface TestInterface {
    
// 私有方法肯定是要有方法体的
  private int method1() {
    return 1;
  }

  private static void method2() {
    System.out.println("method2");
  }

  static void method3() {
    System.out.println("method3");
  }

  default void method4() {
    System.out.println(method1());
  }
  
  void method5();

}
```

### Optional 类改进

`Optional` 类中新增了 `ifPresentOrElse()`、`or()` 和 `stream()` 等方法

#### `ifPresentOrElse()` 

接受两个参数 `Consumer` 和 `Runnable` ，如果 `Optional` 不为空调用 `Consumer` 参数，为空则调用 `Runnable` 参数。

```java
/**
 * If a value is present, performs the given action with the value,
 * otherwise performs the given empty-based action.
 *
 * @since 9
 */
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) {
    if (value != null) {
        action.accept(value);
    } else {
        emptyAction.run();
    }
}
```
```java
Optional.ofNullable(list).ifPresentOrElse(System.out::println,()-> System.out.println("list is null"));
```

#### `or()`

接受一个 `Supplier` 参数 ，如果 `Optional` 为空则返回 `Supplier` 参数指定的 `Optional` 值。

```java
    public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier) {
        Objects.requireNonNull(supplier);
        if (isPresent()) {
            return this;
        } else {
            @SuppressWarnings("unchecked")
            Optional<T> r = (Optional<T>) supplier.get();
            return Objects.requireNonNull(r);
        }
    }
```

```java
Optional<Object> objectOptional = Optional.empty();
objectOptional.or(() -> Optional.of("java")).ifPresent(System.out::println);//java
```

### Stream API的改进

`Stream` 中增加了新的方法 `ofNullable()`、`dropWhile()`、`takeWhile()` 以及 `iterate()` 方法的重载方法。

#### ofNullable()

Java 9 中的 `ofNullable()` 方 法允许我们创建一个单元素的 `Stream`，可以包含一个非空元素，也可以创建一个空 `Stream`。 而在 Java 8 中则不可以创建空的 `Stream` 。

```java
Stream<String> stringStream = Stream.ofNullable("Java");
System.out.println(stringStream.count());// 1
Stream<String> nullStream = Stream.ofNullable(null);
System.out.println(nullStream.count());//0
```

#### `takeWhile()` 

可以从 `Stream` 中依次获取满足条件的元素，直到不满足条件为止结束获取。

```java
List<Integer> integerList = List.of(11, 33, 66, 8, 9, 13);
integerList.stream().takeWhile(x -> x < 50).forEach(System.out::println);// 11 33
```

#### `dropWhile()` 

效果和 `takeWhile()` 相反。

```java
List<Integer> integerList2 = List.of(11, 33, 66, 8, 9, 13);
integerList2.stream().dropWhile(x -> x < 50).forEach(System.out::println);// 66 8 9 13
```

#### `iterate()` 

新重载方法提供了一个 `Predicate` 参数 (判断条件)来决定什么时候结束迭代

```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
}
// 新增加的重载方法
public static<T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> next) {

}
```

两者的使用对比如下，新的 `iterate()` 重载方法更加灵活一些。

```java
// 使用原始 iterate() 方法输出数字 1~10
Stream.iterate(1, i -> i + 1).limit(10).forEach(System.out::println);
// 使用新的 iterate() 重载方法输出数字 1~10
Stream.iterate(1, i -> i <= 10, i -> i + 1).forEach(System.out::println);
```

### 模块化

模块（Module）是在Java包（`package`）的基础上又引入的一个新的抽象层。JPMS（Java Platform Module System）是Java 9发行版的核心亮点。JDK 本身也已经被模块化了

![image-20221228110602493](https://img.zhsong.cn/blog-image/image-20221228110602493.png)

在IDEA中使用new module可以新建一个模块，模块描述文件`module-info.java`必须位于Sources Root目录下

![image-20221228160133497](https://img.zhsong.cn/blog-image/image-20221228160133497.png)



![image-20221228160216797](https://img.zhsong.cn/blog-image/image-20221228160216797.png)

模块化的好处在于可以实现**强封装性**，模块中的包只有在显式地导出后才可以被其他模块所使用，模块必须显式地声明它需要的模块后才能使用这些模块下的包；可以**简化类库**， 比如可以使用`jlink` 工具创建自定义的JRE，只引入我们需要的模块，避免写一个“hello world”还需要一个200MB的JRE

进一步了解，👉https://www.jianshu.com/p/50f0cd1860a9

### jshell

一个交互式的编程环境，可以运行代码片段

```
D:\software\java\java9.0.4\jdk\bin>jshell
|  欢迎使用 JShell -- 版本 9.0.4
|  要大致了解该版本, 请键入: /help intro

jshell> public class Test{
   ...> public void test(){
   ...> System.out.println("hello world");
   ...> }
   ...> }
|  已创建 类 Test

jshell> Test t = new Test();
t ==> Test@59494225

jshell> t.test();
hello world
```

```
jshell> System.out.println("hello world");
hello world
```

```
jshell> (3+4)*5
$5 ==> 35
```

### String 存储结构优化

Java 8 及之前的版本，`String` 一直是用 `char[]` 存储。在 Java 9 之后，`String` 的实现改用 `byte[]` 数组存储字符串。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    @Stable
    private final byte[] value;
}
```

### try-with-resources的改进

### 设置G1为JVM默认垃圾收集器

在 Java 8 的默认垃圾回收器是 Parallel Scavenge（新生代）+Parallel Old（老年代）。到了 Java 9，G1（Garbage-First Garbage Collector）成为了默认垃圾回收器。

### 支持http2.0和websocket的API

### 多版本兼容Jar包

### 进程 API

Java 9 增加了 `java.lang.ProcessHandle` 接口来实现对原生进程进行管理，尤其适合于管理长时间运行的进程。

```java
// 获取当前正在运行的 JVM 的进程
ProcessHandle currentProcess = ProcessHandle.current();
// 输出进程的 id
System.out.println(currentProcess.pid());
// 输出进程的信息
System.out.println(currentProcess.info());
```

## JDK 10

发布于2018年3月，不是长期支持版本

### 局部变量类型推断

类似JS可以通过var来修饰局部变量，编译之后会推断出值的真实类型

### 不可变集合的改进

`List`，`Set`，`Map` 提供了静态方法`copyOf()`返回入参集合的一个不可变拷贝。



### 并行全垃圾回收器 G1，来优化G1的延迟

### 线程本地握手，允许在不执行全局VM安全点的情况下执行线程回调，可以停止单个线程，而不需要停止所有线程或不停止线程

### Optional 增强

新增orElseThrow()方法

```java
Optional.ofNullable(cache.getIfPresent(key))
        .orElseThrow(() -> new PrestoException(NOT_FOUND, "Missing entry found for key: " + key));

```

### 类数据共享

### Unicode 语言标签扩展

### 根证书

## JDK 11

发布于2018年9月，是长期支持版本

### 增加一些字符串处理方法

Java 11 增加了一系列的字符串处理方法

```java
//判断字符串是否为空
" ".isBlank();//true
//去除字符串首尾空格
" Java ".strip();// "Java"
//去除字符串首部空格
" Java ".stripLeading();   // "Java "
//去除字符串尾部空格
" Java ".stripTrailing();  // " Java"
//重复字符串多少次
"Java".repeat(3);             // "JavaJavaJava"
//返回由行终止符分隔的字符串集合。
"A\nB\nC".lines().count();    // 3
"A\nB\nC".lines().collect(Collectors.toList());
```



### 用于 Lambda 参数的局部变量语法

从 Java 10 开始，便引入了局部变量类型推断这一关键特性。类型推断允许使用关键字 var 作为局部变量的类型而不是实际类型，编译器根据分配给变量的值推断出类型。

Java 10 中对 var 关键字存在几个限制

- 只能用于局部变量上
- 声明时必须初始化
- 不能用作方法参数
- 不能在 Lambda 表达式中使用

Java11 开始允许开发者在 Lambda 表达式中使用 var 进行参数声明。

```java
// 下面两者是等价的
Consumer<String> consumer = (var i) -> System.out.println(i);
Consumer<String> consumer = (String i) -> System.out.println(i);
```

### Http Client重写，支持HTTP/1.1和HTTP/2 ，也支持 websockets

### 可运行单一Java源码文件，如：java Test.java

### ZGC：

可伸缩低延迟垃圾收集器，ZGC可以看做是G1之上更细粒度的内存管理策略。由于内存的不断分配回收会产生大量的内存碎片空间，因此需要整理策略防止内存空间碎片化，在整理期间需要将对于内存引用的线程逻辑暂停，这个过程被称为"Stop the world"。只有当整理完成后，线程逻辑才可以继续运行。（并行回收）

### 支持 TLS 1.3 协议

### Flight Recorder（飞行记录器），基于OS、JVM和JDK的事件产生的数据收集框架

### 对Stream、Optional、集合API进行增强



