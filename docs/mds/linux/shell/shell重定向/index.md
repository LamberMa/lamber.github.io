## Stdin,Stdout,Strerr

- stdin：标准输入（0）
- stdout：标准输出（1）
- stderr：标准错误输出（2）

常见的我们重定向命令的输出方式有
```shell
>/dev/null 2>&1
```
上面的命令可以分为两部分去看，其中`>/dev/null`，是把这些输出塞到/dev/null里面去。`2>&1`表示要操作的对象是stdout和stderr，所以上面这条命令的意思就是把标准输出和标准错误扔到/dev/null里面去。因为这里把标准输入直接丢弃了。


| Code | 说明 | FileDescription |
| --- | --- | --- |
| 0 | 标准输入  | /proc/self/fd/0 |
| 1 | 标准输出 | /proc/self/fd/1 |
| 2 | 错误输出 | /proc/self/fd/2 |

采用&可以将两个输出绑定在一起，就是错误输出将会和标准输出输出到同一个地方，下面针对两个命令进行简单的说明

| 命令| 标准输出 |错误输出|
| --- | --- | --- |
|>/dev/null 2>&1 |丢弃|丢弃
|2>&1 >/dev/null|丢弃|屏幕|

我们经常讲这些内容与nohup结合使用将应用放到后台启动，比如
```shell
nohup java -jar xxxx.jar >/dev/null 2>&1 &
```