## TCP探针


## HTTP探针

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: statistics-xueguan-service-deploy
  namespace: default
  labels:
    app: statistics-xueguan-service-deploy
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: statistics-xueguan-service
    spec:
      containers:
      - name: nginx
        image: registry-vpc.cn-beijing.aliyuncs.com/limi-public/nginx:v1
        livenessProbe:
          httpGet:
            # 向该端口发起一个http get请求
            # 如果返回一个正常的response code那么就证明是没问题的。
            path: /index/health
            port: 8080
          # 在第一次探测之前需要先等待15s
          initialDelaySeconds: 15
          # 探测周期为3s
          periodSeconds: 3
```

## UDP探针

官方没有提供UDP的探测，但是我们可以使用一些命令的方式去探测，比如说针对dns的容器，我们可以用nslookup

```yaml
livenessProbe:
  exec:
    command:
    - "/bin/sh"
    - "-c"
    # The health check succeeds by virtue of not hanging. It'd be nice
    # to also check local services are known, but if that's broken then
    # etcd or kube2sky has to be restarted, not skydns.
    - "nslookup foobar 127.0.0.1 &> /dev/null; echo ok"
  initialDelaySeconds: 30
  timeoutSeconds: 5
```