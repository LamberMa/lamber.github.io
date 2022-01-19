> 使用BindExporter实现BindDNS的数据监控，具体参考[Github BindExporter](https://github.com/prometheus-community/bind_exporter/releases)

## 安装BindExporter

```shell
# 下载BindExporter的安装包
cd /opt
wget https://github.com/prometheus-community/bind_exporter/releases/download/v0.4.0/bind_exporter-0.4.0.linux-amd64.tar.gz
# 解压压缩包
tar xf bind_exporter-0.4.0.linux-amd64.tar.gz
mkdir -p /usr/local/exporters/
mv bind_exporter-0.4.0.linux-amd64 /usr/local/exporters/bind_exporter
```
编辑systemd的配置文件，因为我们的systemd是启动bindExporter的时候是以named用户来启动的，所以说要先安装bind，初始化好named用户和组
```shell
vim /usr/lib/systemd/system/bind_exporter.service
[Unit]
Description=bind_exporter
Documentation=https://github.com/digitalocean/bind_exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=named
Group=named
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/exporters/bind_exporter/bind_exporter \
  --bind.pid-file=/var/run/named/named.pid \
  --bind.timeout=20s \
  --web.listen-address=0.0.0.0:9119 \
  --web.telemetry-path=/metrics \
  --bind.stats-url=http://localhost:53/ \
  --bind.stats-groups=server,view,tasks

SyslogIdentifier=bind_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```
然后重载并重启systemd服务
```shell
chown -R named:named /usr/local/exporters/bind_exporter
systemctl daemon-reload
systemctl start bind_exporter
systemctl status bind_exporter
systemctl enable bind_exporter
```
!!! note "如果出现启动失败的错误"
    可以检查一下对应的/usr/local/exporters目录本身是否有other的x权限，如果没有权限的话，named用户是进不到这个目录的，自然BindExporter也是无法启动

接下来，在`/etc/named.conf`中添加如下内容，注意“statistics-channels”是与“options”并列的，而不是位于“options”内部

```shell
statistics-channels {
        inet 127.0.0.1 port 53 allow { 127.0.0.1; };
};
```

重启bind
```shell
named-checkconf
systemctl restart named
```

然后就可以测试一下是否正常暴露出来了对应的metrics

```shell
[root@cd-ocg-b-vm-48-42-bind-dns-master ~]# curl localhost:9119/metrics | grep bind_zone
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 22857    0 22857    0     0  72539      0 --:--:-- --:--:-- --:--:-- 72792
# HELP bind_zone_serial Zone serial number.
# TYPE bind_zone_serial counter
bind_zone_serial{view="_default",zone_name="0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.IP6.ARPA"} 0
bind_zone_serial{view="_default",zone_name="0.in-addr.arpa"} 0
bind_zone_serial{view="_default",zone_name="1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa"} 0
bind_zone_serial{view="_default",zone_name="1.0.0.127.in-addr.arpa"} 0
bind_zone_serial{view="_default",zone_name="10.IN-ADDR.ARPA"} 0
bind_zone_serial{view="_default",zone_name="100.100.IN-ADDR.ARPA"} 0
bind_zone_serial{view="_default",zone_name="100.51.198.IN-ADDR.ARPA"} 0
bind_zone_serial{view="_default",zone_name="101.100.IN-ADDR.ARPA"} 0
……………………
```

接下来记得调整对应的防火墙

```shell
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9119 -j ACCEPT
```

## Prometheus接入Exporter

