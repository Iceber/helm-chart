# Thanos
提供 query, store-gateway 组件的 helm chart  
默认没有开启 store-gateway 组件

---

暂未实现 tls 相关功能，比如额外挂载卷等


## 运行

```
helm install -n <namespace> -f values.yml thanos ./thanos 
```
## 更新

```
helm upgrade -n <namespace> -f values.yml thanos ./thanos
```

## 删除

```
helm uninstall -n <namespace> thanos
```


## 配置 query 组件

```
query:
    logLevel: debug

    replicaLabels: []
    stores:
      - dnssrv+_grpc._tcp.thanos-store-grpc-headless.prome.svc
      - dnssrv+_grpc._tcp.prom-thanos-prometheus-prome-thanos-grpc-headless.prome.svc

    service:
      type: NodePort
      nodePort: 30902
```

主要需要配置的是需要收集的 store api 和 replicaLabels

```
    replicaLabels: []
    stores: []
```

#### replicaLabels 通过副本标签可以删除掉重复数据    
> replicaLabels 可以匹配 Prometheus 配置文件的 global 中 extraLabels 下的标签


#### stores 可以使用 Headless Services，格式为
```
dnssrv+_grpc._tcp.<service-name>.<namespace>.svc
```

> 如果一个 Namespace 下存在多个 store-api 集群服务则需要把他们都加入到 stores 参数中  
> 可以通过对这个提供 store-api 的集群增加标签，组建新的 headless service

#### TODO （如果有这个需要的话):
自动匹配指定标签，汇总 Namespace 下所有提供store-api的pod，组建新的服务

## 使用 store-gateway
#### 将对象存储的配置文件保存到 ConfigMap 中

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: thanos-objstore-s3-config
data:
    s3.yml: |
        type: S3
        config:
            bucket: "def"
            endpoint: "10.8.1.73:10110"
            access_key: "adsf"
            secret_key: "ddf"
            insecure: true
```
###### 具体的 Thanos 对象存储配置参考：https://thanos.io/storage.md
#### 使用 [fakes3](https://github.com/jubos/fake-s3) 来进行 s3 存储测试
#### 开启 store-gateway
```
store:
    enabled: true
    logLevel: debug
    objstore:
        configMapName: "thanos-objstore-s3-config"
        configFileName: "s3.yml"
    grpcHeadless:
        servicePort: 10901
        annotations: {}
```
###### 自动创建 grpc 的 Headless Service
