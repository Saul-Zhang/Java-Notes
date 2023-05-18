# Shell常用命令

# 2>&1

- 这里1，2都是文件描述符
- `1`表示标准输出(stdout)，比如Java中System.out
- `2`表示标准错误(stderr)，比如Java中System.err
- `>`表示重定向，`echo hello > 1.txt` 表示将"hello"写入到1.txt中
- `>>`也是重定向，它会将内容**追加**到文件中，而不是直接覆盖
- `&`可以看作是转义字符，因为如果直接写`2>1`表示将标准错误写到文件名为1的文件中了

## 常用用法

```shell
nohup python main.py > my.log 2>&1 &
```

- nohup 表示将程序以忽略挂起信号的方式运行起来，就算退出ssh连接，命令仍然正常执行

- 最后一个&表示将命令放到后台执行
- python main.py > my.log 是把标准输出写到my.log文件中，2>&1是指把标准错误写到标准输出中，这样就相当于标准错误也写到my.log文件中了。注意2>&1一定是写在python main.py > my.log后面的

参考 https://blog.csdn.net/zhaominpro/article/details/82630528

# /dev/null

`/dev/null` 是一个特殊的设备文件，它丢弃一切写入其中的数据 可以将它视为一个黑洞, 它等效于只写文件, **写入其中的所有内容都会消失, 尝试从中读取或输出不会有任何结果**

# 磁盘相关

## df

用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

```shell
df -h 
```

- -h，--human-readable 以K，M，G为单位，提高信息的可读性

## du

查看文件和目录磁盘使用的空间

```shell
du -h # 查看当前目录下所有文件夹的大小(多层) 
```

- -h ，--human-readable以可读性较高的方式来显示信息

```shell
du -sh # 查看当前目录大小
```

- -s ，--summarize 仅显示总计，只列出最后加总的值

```shell
du -h * 或者 du -sh *  # 查看当前目录下所有文件夹和文件大小（多层）
```

- `*`应该是类似通配符，加上后可以匹配当前目录下所有文件夹和文件，也可以指定具体目录，如/home/sf/*

```shell
du -h --max-depth=1
```

- --max-depth=N 计算的目录深度为N

## ls

list的缩写，用来显示目标列表

```shell
ls -l
```

- -l 以长格式显示目录下的内容列表 可用别名ll代替上边的命令

```shell
ll -t
```

- -t 按时间信息排序
- -r 逆序排列

## find

在指定目录下查找文件

```shell
find . #列出当前目录及子目录下所有文件和文件夹
```

```shell
find /data -mtime +10 # 列出/data目录下10天前修改的文件
```

```shell
find . -name "*.log" -mtime -10 -exec  rm -rfv {} \; # 删除当前目录下10天内修改的以.log结尾的文件
```

- -exec find命令获取到值之后就执行 exec后边的命令
- {} 表示前面find命令的结果项

```shell
find . -type f -mtime +30 -name "*.log" -exec cp {} old \; #将30天前的.log文件移动到old目录中
```

- type 文件类型
  - f 普通文件
  - d 目录
  - l 符号连接

```shell
find . -type f-size +50M -exec rm {} \; # 删除当前目录下大于50M的文件
```



## 参考

强烈推荐Linun命令搜索☞https://wangchujiang.com/linux-command/list.html





