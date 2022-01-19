## 安装

```shell
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.30.0/prometheus-2.30.0.linux-amd64.tar.gz
tar xf prometheus-2.30.0.linux-amd64.tar.gz
cp -ra prometheus-2.30.0.linux-amd64 /usr/local/prometheus
groupadd prometheus
useradd -g prometheus -M -s /sbin/nologin prometheus
mkdir -p /usr/local/prometheus/data
chown prometheus.prometheus -R /usr/local/prometheus

# 检测配置文件是否合法
cd /usr/local/prometheus/
./promtool check config prometheus.yml
```
编辑systemd启动文件，文件位置为`/usr/lib/systemd/system/prometheus.service`，具体的内容为
```shell
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prometheus
Restart=on-failure
Type=simple

ExecStart=/usr/local/prometheus/prometheus \
  --config.file=/usr/local/prometheus/prometheus.yml \
  --storage.tsdb.path=/usr/local/prometheus/data \
  --storage.tsdb.retention.time=30d \
  --web.enable-lifecycle \
  --web.enable-admin-api

[Install]
WantedBy=multi-user.target
```
启动prometheus
```shell
systemctl daemon-reload
systemctl start prometheus
systemctl status prometheus
systemctl enable prometheus
```
接下来查看启动的端口
```shell
# netstat -ntlp | grep 9090
tcp6       0      0 :::9090                 :::*                    LISTEN      5398/prometheus
```
最后为了避免不能访问，在对应的iptables添加9090的记录
```shell
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9090 -j ACCEPT
```

## 初始化配置

```shell
cd /usr/local/prometheus/
mkdir -p ./rules/{alert_rules,record_rules}
chown -R prometheus:prometheus ./rules
# targets目录主要用来分门别类的保存各种类型的targets，当然不一定用得上
mkdir targets && chown -R prometheus:prometheus ./targets
```