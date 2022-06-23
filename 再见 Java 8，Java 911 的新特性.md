# 再见 Java 8，Java 9~11 的新特性

## JDK 9

JDK 9 的发布时间是2017年9月。参考文档👉https://docs.oracle.com/javase/9/whatsnew/toc.htm

### 目录结构

JDK 9 的目录结构相较于JDK 8 有很大变化

![Java9目录结构](https://img.zhsong.cn/blog-image/image-20220521174959258.png)

![image-20220521175854645](https://img.zhsong.cn/blog-image/image-20220521175854645.png)

### 不可变的集合

List，Map和Set接口增加了.of()方法，可以创建**不可变的集合**，即创建后集合中的元素不可更改

```shell
List immutableList = List.of();
List<Integer> immutableList1 = List.of(1, 2);

Map<Integer, String> immutableMap = Map.of(1, "one", 2, "two");
```



### 模块化

模块（Module）是在Java包（`package`）的基础上又引入的一个新的抽象层

### jshell



