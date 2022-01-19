## Prometheus对接Consul

### 配置Consul

!!! 参考
    [Prometheus consul_sd_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#consul_sd_config)

在使用Prometheus对接Consul之前，我们先想Consul中插入一条测试数据，直接访问接口，你这里使用curl或者postman都可以。我这里直接使用curl。

```shell
curl -X PUT -H "Content-Type: application/json" -H "x-consul-token: <consul token>" -d \
'{
    "id": "node-exporter-192.168.1.1",
    "name": "node-exporter",
    "address": "192.168.1.1",
    "port": 9100,
    "tags": ["node_exporter"],
    "Meta": {
        "area": "beijing",
        "service": "kvm",
        "env": "prod",
        "type": "pm",
        "owner": "lamber",
        "monitor":"node_exporter",
        "desc":"我的kvm节点"
    },
    "checks": [
        {
            "http": "http://192.168.1.1:9100/metrics",
            "interval":"25s"
        }
    ]
}' http://yourconsul.example.com/v1/agent/service/register
```

注册成功以后，我们开始使用Prometheus对接Consul，配置的语法很简单，如下：
```yaml
- job_name: "consul-prometheus"
  consul_sd_configs:
    - server: "<your server address domain or ip>"
      token: "<这里填写token字符串>"
      datacenter: "dc-xeq"
      services: []
```

这里Consul的地址我利用了一个LB层，转发到了后端consul的8500端口，当然你也可以用lvs，nginx，haproxy等工具做代理。当然如果说你的consul是单点的，那么就直接写对应的`ip:8500`端口就可以了。

为了保证相对来讲的安全性，我这里还使用了consul的acl，所以说在Prometheus连接consul的时候就需要consul下发的一个token，datacenter可以指定dc名称。

但是上面配置完以后存在几个问题。

1. Prometheus会把默认的consul服务加载出来，这个我可能并不需要。
2. job标签只有两个，一个instance，一个job。tag太少的话后期针对很多数据都是不好管理的。但是我们在Prometheus中发现，我们添加的其他标签并不是没有，而是在`Before relabeling`中以`__meta_consul_service_metadata_xxx`的形式存在。这种形式下存在的数据我是无法当做tag用来筛选的。
3. 如果需要自定义标签，我该如何进行操作。
4. 是否所有类型的Exporter都注册到这个consul-prometheus组中，如果说注册进去，我应该如何进行区分？

### relabel_configs

!!! 参考
    - [relabel_config官方说明](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config)

relabel_configs允许用户在任务采集设置中，通过relabel_configs来添加自定义的relabeling，实现对标签进行指定规则的重写。Prometheus 加载 Targets 后，这些 Targets 会自动包含一些默认的标签，Target 以 `__` 作为前置的标签是在系统内部使用的，这些标签不会被写入到样本数据中。眼尖的会发现，每次增加 Target 时会自动增加一个 instance 标签，而 instance 标签的内容刚好对应 Target 实例的 `__address__` 值，这是因为实际上 Prometheus 内部做了一次标签重写处理，默认 `__address__` 标签设置为 `<host>:<port>` 地址，经过标签重写后，默认会自动将该值设置为 instance 标签，所以我们能够在页面看到该标签。

接下来简单介绍一下常见的`relabel_action`的作用

- **replace**: 根据`regex`的配置匹配`source_labels`标签的值（注意：多个 source_label 的值会按照 separator 进行拼接），并且将匹配到的值写入到 target_label 当中，如果有多个匹配组，则可以使用 ${1}, ${2} 确定写入的内容。如果没匹配到任何内容则不对`target_label`进行重新， 默认为 replace。
- **keep**: 丢弃 source_labels 的值中没有匹配到 regex 正则表达式内容的 Target 实例
- **drop**: 丢弃 source_labels 的值中匹配到 regex 正则表达式内容的 Target 实例
- **hashmod**: 将 target_label 设置为关联的 source_label 的哈希模块
- **labelmap**: 根据 regex 去匹配 Target 实例所有标签的名称（注意是名称），并且将捕获到的内容作为为新的标签名称，regex 匹配到标签的的值作为新标签的值
- **labeldrop**: 对 Target 标签进行过滤，会移除匹配过滤条件的所有标签
- **labelkeep**: 对 Target 标签进行过滤，会移除不匹配过滤条件的所有标签

接下来，依次处理上述问题：

#### 过滤掉Consul的service

过滤`__meta_consul_tags`中包含exporter的服务，丢弃`__meta_consul_tags`中不包含`exporter`的服务。默认的consul服务并不带该标签，因此可以使用这种方式实现过滤。

```yaml
- job_name: "consul-prometheus"
  consul_sd_configs:
    - server: "<your server address domain or ip>"
      token: "<这里填写token字符串>"
      datacenter: "dc-xeq"
      services: []
  relabel_configs:
    - source_labels: [__meta_consul_tags]
      regex: .*exporter.*
      action: keep
```

#### 自定义tag

第二个问题和第三个问题属于一类问题，需要将`Before relabeling`中的自定义标签，转化为可视化的标签。方便后续的分组和告警。在服务进行注册的时候，我们需要将自定义的标签添加到`MetaData`中，如最一开始的那个curl命令，详见[Consul Service Agent HTTP API](https://www.consul.io/api/agent/service)

我们在最一开始注册的时候，向Meta中保存了很多数据，比如说area，owner等：

```json
"Meta": {
    "area": "beijing",
    "service": "kvm",
    "env": "prod",
    "type": "pm",
    "owner": "lamber",
    "monitor":"node_exporter",
    "desc":"我的kvm节点"
},
```
最后这些标签都会以以`__meta_consul_service_metadata_xxx`的形式出现在Target的`Before Relabeling`中，比如area对应的就是`__meta_consul_service_metadata_area`。接下来修改Prometheus的配置文件。

```yaml
- job_name: "consul-prometheus"
  consul_sd_configs:
    - server: "<your server address domain or ip>"
      token: "<这里填写token字符串>"
      datacenter: "dc-xeq"
      services: []
  relabel_configs:
    - source_labels: [__meta_consul_tags]
      regex: .*exporter.*
      action: keep
      action: keep
    - regex: __meta_consul_service_metadata_(.+)
      action: labelmap
```

