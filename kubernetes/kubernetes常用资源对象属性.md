## strategy
```yaml
spec:
    replicas: 3
    strategy:
        # POD全部停止，然后在全部启动  
        type: Recreate      #不建议使用该策略
    
spec:
  replicas: 3
  strategy:
    type: RollingUpdate    #滚动更新 生产环境建议使用
    rollingUpdate:
      maxSurge: 2        # 一次可以添加多少个Pod
      maxUnavailable: 1  # 滚动更新期间最大多少个Pod不可用

    #版本1提供服务
    #部署版本2
    #等待直到所有副本都被版本2替换完成

```

## 镜像拉取策略
```yaml
containers:
    - name: coredns
    image: coredns/coredns:1.2.0
    imagePullPolicy: IfNotPresent
#IfNotPresent ：如果本地存在镜像就优先使用本地镜像。
#Never：直接不再去拉取镜像了，使用本地的；如果本地不存在就报异常了。
#Always  每次都去镜像仓库拉取，拉取不到就报异常。
```
## POD重启策略
```yaml
spec:
    restartPolicy: Always
#Always: 当容器失效时, 由kubelet自动重启该容器
#OnFailure: 当容器终止运行且退出码不为0时, 由kubelet自动重启该容器
#Never: 不论容器运行状态如何, kubelet都不会重启该容器
```

## POD声明周期（策略）
```yaml
#挂起（Pending）：Pod 已被 Kubernetes 系统接受，但有一个或者多个容器镜像尚未创建。等待时间包括调度 Pod 的时间和通过网络下载镜像的时间，这可能需要花点时间。
#运行中（Running）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
#成功（Succeeded）：Pod 中的所有容器都被成功终止，并且不会再重启。
#失败（Failed）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。
#未知（Unknown）：因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败。
```