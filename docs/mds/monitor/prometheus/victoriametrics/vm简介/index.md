!!! note
    vm 是一个快速高效可扩展的时序数据库，它可以被用来作为rometheus的一个长期存储。当然这里官网建议当数据获取的速率低于每秒百万级的话，使用单节点版本。




## Cluster集群版安装

```shell
# 下载vmutils，主要是工具集，包含vmagent，vmalert，vmauth，vmbackup，vmctl以及vmrestore
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.69.0/vmutils-amd64-v1.69.0.tar.gz
tar xf vmutils-amd64-v1.69.0.tar.gz -C /bin
# 下载vm cluster主程序
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.69.0/victoria-metrics-amd64-v1.69.0-cluster.tar.gz
tar xf victoria-metrics-amd64-v1.69.0-cluster.tar.gz -C /bin/
```

vmstorage会监听三个端口，给自己用的，给vminsert用的，给vmselect用的。
```shell
  -vminsertAddr string
    	TCP address to accept connections from vminsert services (default ":8400")
  -vmselectAddr string
    	TCP address to accept connections from vmselect services (default ":8401")
```
