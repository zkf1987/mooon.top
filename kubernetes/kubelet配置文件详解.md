|参数名称	|配置含义
| -| -| 
|address: 0.0.0.0	| 地址是Kubelet要使用的IP地址  默认为0.0.0.0
|apiVersion: kubelet.config.k8s.io/v1beta1	|组名称和版本号
|authentication	|	权限认证
|anonymous：enabled: false		|  "启用对Kubelet服务器的匿名请求。未被其他身份验证方法拒绝的请求被视为匿名请求。匿名请求的用户名是system: Anonymous，组名是system:unauthenticated，默认为true" 
| cacheTTL: 2m0s	| 缓存来自webhook令牌验证器的响应的持续时间。(默认2 m0)|
|x509: clientCAFile: /etc/kubernetes/pki/ca.crt	"|设置客户端的CA证书文件，一旦设置该文件，则将对所有客户端请求进行鉴权，验证客户端证书的CommonName信息"
|cacheAuthorizedTTL: 5m0s	|cacheAuthorizedTTL是缓存来自webhook授权器的“授权”响应的持续时间
|cacheUnauthorizedTTL: 30s	|cacheUnauthorizedTTL是缓存来自webhook授权器的“未授权”响应的持续时间
|cgroupDriver: cgroupfs	|用于操作本机cgroup的驱动模式，支持的选项包括groupfs或systemd，默认值为cgroups
|cgroupsPerQOS: true	|设置为true表示启用创建QOS cgroup hierarchy，默认值为true
|clusterDNS:   - 172.22.242.239	| "clusterDNS是集群DNS服务器的IP地址列表。如果设置,kubelet将配置所有容器，将其用于DNS解析，而不是主机的DNS服务器"
|clusterDomain: cluster.local	|"clusterDomain是这个集群的DNS域。如果设置好，kubelet将配置除主机搜索域之外的所有容器来搜索此域"
|configMapAndSecretChangeDetectionStrategy: Watch	 |"ConfigMapAndSecretChangeDetectionStrategy是一种运行配置映射和秘密管理器的模式。默认值:watch"
|containerLogMaxFiles: 5	|"可以为容器显示的容器日志文件的最大数量。动态Kubelet配置(beta):如果动态更新这个字段，请考虑降低它可能会导致日志文件被删除"
|containerLogMaxSize: 10Mi	|"可以为容器显示的容器日志文件的最大量。动态Kubelet配置(beta):如果动态更新这个字段，请考虑降低它可能会导致日志文件被删除。"
|contentType: application/vnd.kubernetes.protobuf	|"contentType是发送给apiserver的请求的contentType。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能会影响Kubelet与API服务器通信的能力。如果Kubelet由于该字段的更改而与API服务器失去联系，则无法通过动态Kubelet配置恢复更改。默认配置为： application/vnd.kubernetes.protobuf"
|cpuCFSQuota: true	|设置为true表示启用CPU CFS quota，用于设置容器的CPU限制，默认值为true
|cpuCFSQuotaPeriod: 100ms	|"CPUCFSQuotaPeriod是CPUCFS配额周期值CPU . cfs__us。动态Kubelet配置(beta):如果动态更新此字段，请考虑容器设置的限制将导致不同的cpu。cfs_quota设置。这将在重新配置的节点上触发容器重新启动。默认值:“100 ms"
|cpuManagerPolicy: none	|要求启用CPUManager特性门。动态Kubelet配置(beta):在没有完全重新引导节点的情况下，不应该更新此字段。将此值与本地配置保持相同是最安全的。默认值:“没有"
|cpuManagerReconcilePeriod: 10s	|CPU管理器调整周期。要求启用CPUManager特性门。动态Kubelet配置(beta):如果动态更新该字段，请考虑缩短周期可能会带来性能影响。默认时间为10s"
|enableControllerAttachDetach: true	|"管理附件/超然的卷将这个节点,和禁用kubelet执行任何附加/分离操作动态kubelet配置(β):如果动态更新这一领域,认为改变哪个组件负责卷管理现场节点上可能导致卷拒绝分离如果不排水节点更新前,如果豆荚定于volumes.kubernetes前的节点。io/controller-managed-attach-detach注释由Kubelet更新。通常，将这个值设置为本地配置是最安全的。"
|enableDebuggingHandlers: true |enableDebuggingHandlers支持服务器端点，用于日志访问和容器和命令的本地运行，包括exec、attach、日志和portforward特性。动态Kubelet配置(beta):如果动态更新该字段，请考虑禁用它可能会破坏与Kubelet服务器交互的组件。默认值:true"
|enforceNodeAllocatable: -pods	|此标志指定Kubelet需要执行的各种节点可分配强制。此标志接受选项列表。可接受的选项是“none”、“pods”、“system-reserved”&“kube-reserved”。如果指定“none”，则不能指定其他选项。更多信息请参考[Node Allocatable](https://git.k8s.io/community/contributors/design-proposal als/node/nodes -allocatable.md)文档。动态Kubelet配置(beta):如果动态更新该字段，请考虑删除强制措施可能会降低节点的稳定性。或者，增加强制措施可能会降低使用超过保留资源数量的组件的稳定性;例如，如果kube-reserved使用的资源超过了所保留的资源，则强制执行kubelet可能会导致OOM;如果系统保留的守护进程使用的资源超过了所保留的资源，则强制执行系统保留的守护进程可能会导致系统守护进程离开房间。默认为 pods"
|eventBurst: 10	|"eventBurst是突发事件创建的最大大小，暂时允许事件创建突发到这个数字，但仍然不超过eventRecordQPS。仅在eventRecordQPS > 0时使用。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能通过更改事件创建产生的通信量来影响可伸缩性。默认值:10"
|eventRecordQPS: 5	|"eventBurst是突发事件创建的最大大小，暂时允许事件创建突发到这个数字，但仍然不超过eventRecordQPS。仅在eventRecordQPS > 0时使用。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能通过更改事件创建产生的通信量来影响可伸缩性。默认值:10"
|evictionHard:|将信号名称映射到定义软清除阈值的量。例如:{“记忆。”:“300 mi”}。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能触发或延迟Pod的清除，并可能更改节点报告的可分配值。默认值:无"
|imagefs.available: 15%	|将信号名称映射到定义硬清除阈值的量。例如:{“记忆。”:“300 mi”}。若要显式禁用，请在任意资源上通过0%或100%阈值。动态Kubelet配置(beta版):如果动态更新此字段，请考虑它可能触发或延迟Pod驱逐。默认的15%"
|memory.available: 100Mi	|将信号名称映射到定义硬清除阈值的量。例如:{“记忆。”:“300 mi”}。若要显式禁用，请在任意资源上通过0%或100%阈值。动态Kubelet配置(beta版):如果动态更新此字段，请考虑它可能触发或延迟Pod驱逐。默认的15%"
|nodefs.available: 10%	|将信号名称映射到定义硬清除阈值的量。例如:{“记忆。”:“300 mi”}。若要显式禁用，请在任意资源上通过0%或100%阈值。动态Kubelet配置(beta版):如果动态更新此字段，请考虑它可能触发或延迟Pod驱逐。默认的15%"
nodefs.inodesFree: 5%	|将信号名称映射到定义硬清除阈值的量。例如:{“记忆。”:“300 mi”}。若要显式禁用，请在任意资源上通过0%或100%阈值。动态Kubelet配置(beta版):如果动态更新此字段，请考虑它可能触发或延迟Pod驱逐。默认的15%"
evictionPressureTransitionPeriod: 5m0s	|最大允许宽限期(秒)使用时，终止豆荚，以响应软驱逐阈值被满足。这个值在软驱逐期间有效地限制了Pod的terminationgraceseconds值。注意:由于问题#64530，该行为有一个bug，该值当前只是覆盖了软驱逐期间的宽限期，这可以从Pod上设置的宽限期中增加宽限期。这个bug将在将来的版本中修复。动态Kubelet配置(beta):如果动态更新这个字段，请考虑降低它会减少在软驱逐期间杀死pod之前必须优雅地清理的时间。默认值:0"
failSwapOn: true	|failSwapOn告诉Kubelet，如果节点上启用了swap，则启动失败。动态Kubelet配置(beta):如果动态更新该字段，请考虑将其设置为true将导致启用swap时Kubelet崩溃循环。默认值:true"
fileCheckFrequency: 20s	|fileCheckFrequency是检查新数据动态Kubelet配置(beta)的配置文件之间的持续时间:如果动态更新该字段，请考虑缩短持续时间将导致Kubelet更频繁地重新加载本地静态Pod配置，这可能会对性能产生负面影响。默认:20s"
hairpinMode: promiscuous-bridge	|hairpinMode指定Kubelet应该如何为发夹包配置容器桥接。设置此标志允许服务中的端点在尝试访问自己的服务时将loadbalance返回给自己。值:“杂乱-桥接”:使容器桥接杂乱。“发夹-veth”:在容器veth接口上设置发夹标志。“没有”:什么也不做。通常，必须设置—hairpin-mode=hairpin-veth才能实现hairpin NAT，因为乱接桥假设存在名为cbr0的容器桥。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能需要重新启动节点，这取决于网络插件。默认值:“promiscuous-bridge"
healthzBindAddress: 127.0.0.1	|healthzBindAddress是healthz服务器在动态Kubelet配置(beta)上服务的IP地址:如果动态更新这个字段，请考虑它可能会破坏监视Kubelet健康状况的组件。默认值:“127.0.0.1
healthzPort: 10248	|healthzPort是本地主机healthz端点的端口(设置为0以禁用)动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能会破坏监视Kubelet健康状况的组件。默认值:10248"
httpCheckFrequency: 20s	|httpCheckFrequency是检查http以获取新数据动态Kubelet配置(beta)之间的持续时间:如果动态更新该字段，请考虑缩短持续时间将导致Kubelet更频繁地轮询staticPodURL，这可能会对性能产生负面影响。默认:20s"
imageGCHighThresholdPercent: 85|magegchigh阈值百分比是始终运行映像垃圾收集的磁盘使用量的百分比。百分比计算为100中的这个字段值。动态Kubelet配置(beta):如果动态更新此字段，请考虑它可能触发或延迟垃圾收集，并可能更改节点上的映像开销。默认值:85"
imageGCLowThresholdPercent: 80	|imageg小丑阈值百分比是在此之前从未运行图像垃圾收集的磁盘使用率的百分比。垃圾收集到的磁盘使用量最低。百分比计算为100中的这个字段值。动态Kubelet配置(beta):如果动态更新此字段，请考虑它可能触发或延迟垃圾收集，并可能更改节点上的映像开销。默认值:80"
imageMinimumGCAge: 2m0s	|"imageMinimumGCAge是未使用的映像在被垃圾收集之前的最小年龄。动态Kubelet配置(beta):如果动态更新此字段，请考虑它可能触发或延迟垃圾收集，并可能更改节点上的映像开销。默认值:“2 m"
iptablesDropBit: 15	|"iptablesDropBit是iptables fwmark空间的位，用于标记丢弃数据包。值必须在[0,31]范围内。必须与其他标记位不同。动态Kubelet配置(beta):如果动态更新该字段，请考虑它需要与其他组件(如kube-proxy)进行协调，并且只有启用了akeIPTablesUtilChains，更新才会有效。默认值:15"
iptablesMasqueradeBit: 14	|iptables masquebit是iptables fwmark空间的位，用于标记SNAT值的位必须在[0,31]范围内。必须与其他标记位不同。警告:请匹配kube-proxy中相应参数的值。TODO:清理kube-proxy Dynamic Kubelet Config (beta)中的iptablesmasqu根除ebit:如果动态更新这个字段，请考虑它需要与其他组件(如kube-proxy)协调，并且只有启用了MakeIPTablesUtilChains，更新才会有效。默认值:14"
kind: KubeletConfiguration	|
kubeAPIBurst: 10	|kubeAPIBurst是在与kubernetes apiserver Dynamic Kubelet Config (beta)对话时允许使用的突发事件:如果动态更新这个字段，请考虑它可能会通过更改Kubelet发送给API服务器的通信量来影响可伸缩性。默认值:10"
kubeAPIQPS: 5	|kubeAPIQPS是在与kubernetes apiserver Dynamic Kubelet Config (beta)对话时使用的QPS:如果动态更新该字段，请考虑它可能会通过更改Kubelet发送给API服务器的通信量来影响可伸缩性。默认值:5"
makeIPTablesUtilChains: true	|iptablesDropBit是iptables fwmark空间的位，用于标记丢弃数据包。值必须在[0,31]范围内。必须与其他标记位不同。动态Kubelet配置(beta):如果动态更新该字段，请考虑它需要与其他组件(如kube-proxy)进行协调，并且只有启用了akeIPTablesUtilChains，更新才会有效。"
maxOpenFiles: 1000000	|maxOpenFiles是库贝列进程可以打开的文件数量。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能会影响Kubelet与节点文件系统交互的能力。默认值:1000000"
maxPods: 110	|maxPods是可以在这个Kubelet上运行的pod的数量。动态Kubelet配置(beta):如果动态更新此字段，请考虑更改可能导致在Kubelet重启时pod无法通过，并可能更改Node.Status.Capacity中报告的值[v1]。，从而影响未来的调度决策。增加这个值还可能降低性能，因为可以将更多的pod打包到一个节点中。默认值:110"
nodeLeaseDurationSeconds: 40	|nodeLeaseDurationSeconds是Kubelet在启用NodeLease特性时对其相应租约设置的持续时间。该特性通过让Kublet在kube-node-lease名称空间中创建并定期更新以节点命名的租约，从而提供节点健康状况的指示器。如果租约到期，则可以认为节点不健康。租约目前每10秒续签一次，按KEP-0009执行。在未来，可以根据租赁期限设定租赁续期。要求启用NodeLease特性门。动态Kubelet配置(beta):如果动态更新该字段，请考虑缩短持续时间可能会降低对暂时阻止Kubelet更新租约的问题的容忍度(例如，短期网络问题)。默认值:40"
nodeStatusReportFrequency: 1m0s	|nodeStatusReportFrequency是kubelet在不更改节点状态的情况下将节点状态发布到master的频率。如果检测到任何更改，Kubelet将忽略此频率并立即发布节点状态。它只在启用节点租赁特性时使用。nodeStatusReportFrequency的默认值是1m。但是，如果显式地设置了nodeStatusUpdateFrequency，则为了向后兼容，nodeStatusReportFrequency的默认值将被设置为nodeStatusUpdateFrequency。默认值:1m"
nodeStatusUpdateFrequency: 10s|nodeStatusUpdateFrequency是kubelet计算节点状态的频率。如果没有启用节点租赁特性，那么它也是kubelet向master发布节点状态的频率。注意:当节点租约功能未启用时，更改常量时要小心，它必须与nodecontroller中的nodeMonitorGracePeriod一起工作。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能会影响节点的可伸缩性，并且节点控制器的nodeMonitorGracePeriod必须设置为N*NodeStatusUpdateFrequency，其中N是节点控制器标记节点不健康之前重试的次数。默认值:10s"
oomScoreAdj: -999	|oomScoreAdj是kubelet过程的oom-score-adj值。值必须在[-1000,1000]范围内。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能会影响节点在内存压力下的稳定性。默认值:-999"
|podPidsLimit: -1	|PodPidsLimit是任何pod中pid的最大数量。要求启用SupportPodPidsLimit特性门。动态Kubelet配置(beta):如果动态更新这个字段，请考虑降低它可以防止容器进程在更改后分叉。默认值:-1"
|port: 10250	|port是kubelet服务的港口。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能会破坏与Kubelet服务器交互的组件。默认值:10250"
registryBurst: 10	|注册量暴是突发拉力的最大大小，暂时允许拉力暴到这个数字，但仍不超过注册量暴。仅在registryPullQPS > 0时使用。动态Kubelet配置(beta版):如果动态更新这个字段，请考虑它可能会通过更改由图像提取产生的通信量来影响可伸缩性。默认值:10"
|registryPullQPS: 5	|registryPullQPS是每秒对注册表进行拉取的限。设为0没有极限。动态Kubelet配置(beta版):如果动态更新这个字段，请考虑它可能会通过更改由图像提取产生的通信量来影响可伸缩性。默认值:5"
|resolvConf: /etc/resolv.conf	|"ResolverConfig是用作容器DNS解析配置基础的解析器配置文件。动态Kubelet配置(beta):如果动态更新该字段，请考虑更改只会对更新后创建的pod生效。建议在更改此字段之前清空节点。默认值:/etc/resolv.conf"
|rotateCertificates: true	|"rotateCertificates支持客户端证书的旋转。Kubelet将从证书中请求一个新的证书。io API。这需要审批者批准证书签名请求。必须启用RotateKubeletClientCertificate特性。动态Kubelet配置(beta):如果动态更新该字段，请考虑禁用它可能会在当前证书过期后破坏Kubelet通过API服务器进行身份验证的能力。默认值:false"
|runtimeRequestTimeout: 2m0s	|"runtimerRequestTimeout是除长时间运行的请求(pull、log、exec和attach)之外的所有运行时请求的超时。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能会破坏与Kubelet服务器交互的组件。默认值:2 m"
|serializeImagePulls: true	|"当启用serializeImagePulls时，告诉Kubelet一次拉出一个图像。我们建议在运行版本< 1.9或Aufs存储后端docker守护进程的节点上*不*更改默认值。第10959期报道。动态Kubelet配置(beta):如果动态更新这个字段，请考虑它可能会影响图像拉取的性能。默认值:true"
|staticPodPath: /etc/kubernetes/manifests	|"staticPodPath是指向包含要运行的本地(静态)pod的目录的路径，或者指向单个静态pod文件的路径。动态Kubelet配置(beta):如果动态更新该字段，请考虑在新路径中指定的一组静态pod可能与Kubelet最初启动时的pod不同，这可能会破坏您的节点。默认值:"
|streamingConnectionIdleTimeout: 4h0m0s	|"streamingConnectionIdleTimeout是流连接在自动关闭之前空闲的最长时间。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能会影响依赖于不频繁更新的组件，这些组件通过流连接到Kubelet服务器。默认值:4 h"
|syncFrequency: 1m0s	|"同步频率是同步运行容器和配置之间的最大周期。动态Kubelet配置(beta):如果动态更新该字段，请考虑缩短此持续时间可能会对性能产生负面影响，特别是当节点上的pod数量增加时。或者，增加这个持续时间将导致更长的ConfigMaps和secret刷新时间。默认值:“1m"
|volumeStatsAggPeriod: 1m0s	|"kubelet计算所有pod和volume的磁盘使用情况聚合值的时间间隔，默认为1m0s。设置为0表示不启用该计算功能"
|readOnlyPort: 10255	|"readOnlyPort是Kubelet在没有身份验证/授权的情况下使用的只读端口。动态Kubelet配置(beta):如果动态更新该字段，请考虑它可能会破坏与Kubelet服务器交互的组件。默认值:0"
