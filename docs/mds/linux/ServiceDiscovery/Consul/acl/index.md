

## ACL系统

## ACL规则

### 规则语法说明

!!! 参考
    [官方参考文档](https://www.consul.io/docs/security/acl/acl-rules#service-rules)
    [Consul集群配置ACL](https://www.bilibili.com/read/cv10442154)

简单来讲ACL的规则匹配有两种，一种是前缀匹配（Prefix based rules），另外一种是精确匹配（Exact Matching Rules）

指定Rule规则的书写结构为：
```hcl
<resource> "<segment>" {
  policy = "<policy disposition>"
}
```

- resource：代表的是我们可以操作的资源，比如说key，service，agent，node等。
- policy：策略，如read，write，deny，list。read代表只读，write代表读写，deny表示不能读写，list表示可以访问当前resource segment下的Consul KV的所有key，注意这个策略只能和key_prefix联合使用，并且`acl.enable_key_list_policy`必须设置为true。

### 添加一个ACL

#### 命令行添加


#### web界面添加

