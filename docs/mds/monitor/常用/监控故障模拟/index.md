## CPU占用模拟

> 模拟高CPU使用率，引发故障告警

比如说使用python写一个简单的死循环
```python
while True:
    pass
```
或者可以使用如下的命令
```shell
cat /dev/urandom | gzip -9 > /dev/null
```
如果你想要更大的负荷，或者系统有多个核，那么只需要对数据进行压缩和解压就行了，像这样：
```shell
# 按下 CTRL+C 来终止进程。
cat /dev/urandom | gzip -9 | gzip -d | gzip -9 | gzip -d > /dev/null
```

## 内存占用模拟

> 模拟内存占用

下面命令会减少可用内存的总量。它是通过在内存中创建文件系统然后往里面写文件来实现的。你可以使用任意多的内存，只需要往里面写入更多的文件就行了。

```shell
mkdir testdir
mount -t ramfs ramfs testdir
```
然后我们切换到这个目录中去向testdir中写入数据的时候，就相当于向内存中写入数据了。
```shell
# bs表示块大小。可以是任何数字后面接上 B（表示字节），K（表示 KB），M（ 表示 MB）或者 G（表示 GB）。
# count= 要写多少个块。
dd if=/dev/zero of=./file bs=1M count=7500
```

## 磁盘IO

创建磁盘 I/O 的方法是先创建一个文件，然后使用 for 循环来不停地拷贝它。

下面使用命令 dd 创建了一个全是零的 1G 大小的文件：

```shell
dd if=/dev/zero of=loadfile bs=1M count=1024
```
下面命令用 for 循环执行 10 次操作。每次都会拷贝 loadfile 来覆盖 loadfile1：

```shell
for i in {1…10}; do cp loadfile loadfile1; done
```
通过修改 {1…10} 中的第二个参数来调整运行时间的长短。（LCTT 译注：你的 Linux 系统中的默认使用的 cp 命令很可能是 cp -i 的别名，这种情况下覆写会提示你输入 y 来确认。你可以使用 -f参数的 cp 命令来覆盖此行为，或者直接用 /bin/cp 命令。）

若你想要一直运行，直到按下 CTRL+C 来停止，则运行下面命令：

```shell
while true; do cp loadfile loadfile1; done
```

## 参考文件

- [Linux系统下简单模拟高CPU\高内存\高负载的方法](https://blog.csdn.net/heavenmark/article/details/82805260)