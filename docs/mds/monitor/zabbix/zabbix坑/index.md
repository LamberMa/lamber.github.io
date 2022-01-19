## 坑1 ZabbixServer拿不到监控数据

!!! 背景
    在A市部署了ZabbixServer作为监控数据汇聚的中心点，其他城市都部署ZabbixProxy作为代理节点收集数据。但是发现ZabbixServer无法收集到外地分支的数据

排查过程如下：

1. 排查代理节点到被监控终端的网络状况（pass）
2. 排查代理节点是否能通过snmpwalk抓取到对应的oid的数据（pass，说明代理节点部分没问题）
3. 排查zabbixServer和zabbixProxy服务是否有明显的日志报错（pass，无日志报错）
4. 排查zabbixServer和proxy之间的网络问题（是否有网络故障，pass）
5. 排查server和proxy版本是否一致（pass）
6. 排查server端和proxy的时间是否一致（check，发现时间没有同步差了8小时，非时区问题）

解决：由于代理节点chronyd服务设置的server为内部ntp服务器，但是dns设置不正确，导致无法解析ntpserver的ip地址。也就导致虽然chronyd服务正常，但是无法同步数据。调整dns解析，手动ntpdate以后数据恢复正常。但是之前的数据并没有补回来，丢掉了。


## 坑2 ZabbixServer拿不到数据2

确保Server和Proxy的版本要一直，否则有可能拿不到数据。