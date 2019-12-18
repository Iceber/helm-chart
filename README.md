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

## Configuration
Parameter | Description | Default
--------- | ----------- | -------
`image.repository` | `Thanos container 镜像仓库` | `thanosio/thanos`
`image.tag` | `Thanos container 镜像标签` | `master-2019-12-14-bec86666`
`image.pullPolicy` | `Thanos 镜像 pull policy` | `IfNotPresent`
`query.name` | `Thanos query 容器名称` | `thanos-query`
`query.replicaCount` | `Thanos query pods 副本数量` | 1
`query.logLevel` | `Thanos query log level` | ""
`query.podLabels` | `增加到 Thanos query pods 的标签` | {}
`query.replicaLabels` | `Thanos query 用来区分 store-api 副本的标签` | []
`query.stores` | `Thanos store api 的地址` | []
`query.service.type` | `Thanos http service 类型` | ClusterIP
`query.service.annotations` | `Thanos http service annotations` | {}
`query.service.labels` | `Thanos http service labels` | {}
`query.service.servicePort` | `Thanos http service servicePort` | 10902
`query.service.nodePort` | `Thanos http service node port 当 query.service.type 为 NodePort 时才有效` | 0
`query.resource` | `Thanso query pod resource request & limits` | {}
`query.nodeSelector` | `Thanos query nodeSelector` | {}
`query.toleration` | `Thanos query toleration` | {}
`query.affinity` | `Thanos query toleration` | {}
`store.enabled` | `是否开启Thanos store gateway` | false
`store.name` | `Thanso store 容器名称` | "thanos-store"
`sotre.logLevel` | `Thanos store log level" | debug
`store.replicaCount` | `Thanos store pods 副本数量` | 1
`store.podLables` | `增加到 Thanos sotre pods 的标签` | {}
`store.objstore.configFileName` | `Thanos store 对象存储配置文件名称` | ""
`store.objstore.configMapName` | `Thanos store 对象存储配置文件的ConfigMap` | ""
`store.grpcHeadless.servicePort` | `Thanos store grpc Headless Service port` | 10901
`store.grpcHeadless.annotations` | `Thanos store grpc Headless Service annotations` | {}
`store.resource` | `Thanos store pod resource reuqest & limits` | {}
`sotre.nodeSelector` | `Thanos sotre nodeSelector` | {}
`sotre.toleration` | `Thanos sotre toleration` | {}
`sotre.affinity` | `Thanos sotre toleration` | {}
