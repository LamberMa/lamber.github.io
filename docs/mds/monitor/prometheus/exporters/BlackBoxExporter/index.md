## 安装BlackBoxExporter

```shell
cd /opt/
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz
tar xf blackbox_exporter-0.19.0.linux-amd64.tar.gz
mv blackbox_exporter-0.19.0.linux-amd64 /usr/local/exporters/blackbox_exporter
chown -R prometheus:prometheus /usr/local/exporters/
```

编辑systemd配置文件

```shell
# /usr/lib/systemd/system/blackbox_exporter.service
[Unit]
Description=blackbox_exporter
After=network.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecStart=/usr/local/exporters/blackbox_exporter/blackbox_exporter \
    --config.file=//usr/local/exporters/blackbox_exporter/blackbox.yml \
    --web.listen-address "[你的机器的ip，建议不要监听在0.0.0.0]:9115"

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

配置文件保持默认就好

??? note "blackbox.yml sample"
    ```yaml
    modules:
      http_2xx:
        prober: http
        timeout: 5s
        http:
          method: GET
          preferred_ip_protocol: "ip4"
      http_post_2xx:
        prober: http
        http:
          method: POST
          preferred_ip_protocol: "ip4"
      tcp_connect:
        prober: tcp
      pop3s_banner:
        prober: tcp
        tcp:
          query_response:
          - expect: "^+OK"
          tls: true
          tls_config:
            insecure_skip_verify: false
      ssh_banner:
        prober: tcp
        tcp:
          query_response:
          - expect: "^SSH-2.0-"
          - send: "SSH-2.0-blackbox-ssh-check"
      irc_banner:
        prober: tcp
        tcp:
          query_response:
          - send: "NICK prober"
          - send: "USER prober prober prober :prober"
          - expect: "PING :([^ ]+)"
            send: "PONG ${1}"
          - expect: "^:[^ ]+ 001"
      icmp:
        prober: icmp
    ```

然后我们启动blackboxExporter

```shell
systemctl daemon-reload
systemctl enable blackbox_exporter
systemctl start blackbox_exporter
systemctl status blackbox_exporter
```

## 配置Prometheus对接BlackBox

