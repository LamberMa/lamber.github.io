

## Sed在文本的行位或者行首添加字符串

在每行的头添加字符，比如"HEAD"，命令如下：

```shell
sed 's/^/HEAD&/g' test.file
```

在每行的行尾添加字符，比如“TAIL”，命令如下：

```shell
sed 's/$/&TAIL/g' test.file
```
