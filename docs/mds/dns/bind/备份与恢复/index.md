在未启用chroot的情况下，主要需要备份的文件包含如下的文件

named配置文件

- /etc/named.comf
- /etc/named.*
- /etc/rndc.key
- /etc/rndc.conf

区域数据文件

- /var/named/*

## 手工备份

> 其实在每次推送的时候进行手动备份就可以，每一个文件没有那么大，所以说理论来讲也不用进行打包压缩。直接使用文件目录+软连接的方式就可以了。


