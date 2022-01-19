## alertmanger安装

```shell
cd /opt
wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
tar xf alertmanager-0.23.0.linux-amd64.tar.gz
mv alertmanager-0.23.0.linux-amd64 /usr/local/alertmanager
chown -R prometheus:prometheus /usr/local/alertmanager
```

alertmanager systemd service文件

```shell
# /usr/lib/systemd/system/alertmanager.service
[Unit]
Description=Alertmanager
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/alertmanager/alertmanager \
  --config.file=/usr/local/alertmanager/alertmanager.yml \
  --storage.path=/data/alertmanager \
  --web.listen-address=<ip>:9093 \
  --cluster.listen-address=<ip>:8001
Restart=on-failure

[Install]
WantedBy=multi-user.target
```



alertmanager配置文件参考

```yaml

```