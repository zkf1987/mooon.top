## 问题描述

k8s之前的job完成后，如果不用cronjob管理，都不会自动删除，该job对象和其相关的pod对象都会保存以便查看记录。

然而在1.12版本之后，k8s提出了通过TTL自动删除Job的特性，当前仅对job生效，对 Complete 和 Failed 状态的Job都会自动删除，以后会逐步对所有的其他资源对象生效。

关于spec.ttlSecondsAfterFinished属性的三种含义
```shell
ttlSecondsAfterFinished: 0 #在job执行完时马上删除
ttlSecondsAfterFinished: 100 #在job执行完后，等待100s再删除
若不设置 ttlSecondsAfterFinished这个属性则不会自动删除
```

## 操作实践
这个特性现在在v1.12版本是alpha阶段，而且默认关闭的，需要手动开启。

需要修改的组件包括apiserver、controller还要scheduler。

我的都是apiserver、controller还要scheduler都是以pod的形式运行的，所以直接修改/etc/kubernetes/manifests下面对应的三个.yaml静态文件，加入- --feature-gates=TTLAfterFinished=true命令，然后令对应的pod重新运行即可。

例如修改后的kube-scheduler.yaml的spec部分如下，kube-apiserver.yaml和kube-controller-manager.yaml也在spec部分加入- --feature-gates=TTLAfterFinished=true即可。
```shell
apiVersion: v1
...
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    - --feature-gates=TTLAfterFinished=true
 ...
```

然后等一会，查看pod，可以发现apiserver、controller、scheduler都已经重新运行了，他们AGE和其他pod明显不一样
```shell
$ kubectl get po -nkube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-bnmc8                1/1     Running   4          19d
coredns-576cbf47c7-zh7nb                1/1     Running   4          19d
etcd-udo                                1/1     Running   6          154d
kube-apiserver-udo                      1/1     Running   0          56m
kube-controller-manager-udo             1/1     Running   0          56m
kube-flannel-ds-amd64-blqvw             1/1     Running   4          154d
kube-proxy-vqfvz                        1/1     Running   4          154d
kube-scheduler-udo                      1/1     Running   0          55m
kubernetes-dashboard-65c76f6c97-xtnvf   1/1     Running   14         19d
```
    
声明一个如下的job文件kube-lykops-job.yaml，ttl设为100，即在它运行完后等待100s，k8s就会把这个job及其对应的pod都自动删除
```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
        resources:
          requests:
            memory: "128Mi"
            cpu: "1"
          limits:
            memory: "256Mi"
            cpu: "1"
      restartPolicy: Never
  backoffLimit: 4
```

操作

```shell
$ kubectl create -f /home/karl/Desktop/kube-lykops-job.yaml 
job.batch/pi-with-ttl created
```
然后查看这个job具体描述有没有"ttlSecondsAfterFinished": 100 这个属性，如果没有则代表TTL这个特性没开启成功
```shell
$ kubectl get job pi-with-ttl -o json
```

最后不断查看这个job和它对应的pod资源，会发现等job complete之后100s，job以及pod资源对象都会被删掉。
```shell
$ kubectl get jobs
NAME          COMPLETIONS   DURATION   AGE
pi-with-ttl   0/1           4s         4s

$ kubectl get jobs
No resources found.
```