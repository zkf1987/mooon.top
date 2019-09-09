# CoreDNS配置参数详解

CoreDNS是一个插件链式实现的DNS服务器/转发器，目前线上集群使用的coredns插件及其参数如下：

##  1. log

## 描述

通过使用*日志，*您可以在标准输出上转储所有查询（以及回复的部分）。存在一些选项来稍微调整输出。日志行上的日期/时间前缀是RFC3339格式，以**毫秒**为单位。

## 句法

```txt
log
```

- 如果没有参数，则查询日志条目将以通用日志格式写入*stdout*以用于所有请求

或者，如果您想要/需要更多控制：

```txt
log [NAMES...] [FORMAT]
```

- `NAMES` 是要匹配的名称列表以便记录
- `FORMAT`是要使用的日志格式（默认为通用日志格式），`{common}`用作通用日志格式的快捷方式。您还可以使用`{combined}`将查询操作码添加 `{>opcode}`到通用日志格式的格式。

您可以进一步指定记录的响应类：

```txt
log [NAMES...] [FORMAT] {
    class CLASSES...
}
```

- `CLASSES` 是一个空格分隔的应该记录的响应类列表

响应类具有以下含义：

- `success`：成功的回应
- `denial`：NXDOMAIN或nodata响应（名称存在，类型不存在）。nodata响应将返回代码设置为NOERROR。
- `error`：SERVFAIL，NOTIMP，REFUSED等。指示远程服务器不愿意解析请求的任何内容。
- `all`：默认值 - 未指定任何内容。使用此类意味着无论我们与“all”混合在一起，所有消息都将被记录。

如果未指定类，则默认为*all*。

## 日志格式

您可以使用任何占位符值指定自定义日志格式。Log支持请求和响应占位符。

支持以下占位符：

- `{type}`：请求的qtype
- `{name}`：请求的qname
- `{class}`：请求的qclass
- `{proto}`：使用的协议（tcp或udp）
- `{remote}`：客户端的IP地址，对于IPv6地址，这些地址括在括号中： `[::1]`
- `{local}`：服务器的IP地址，对于IPv6地址，这些地址括在括号中： `[::1]`
- `{size}`：请求大小（字节）
- `{port}`：客户端口
- `{duration}`：响应持续时间
- `{rcode}`：响应RCODE
- `{rsize}`：raw（未压缩），响应大小（客户端可能会收到较小的响应）
- `{>rflags}`：响应标志，将显示每个设置标志，例如“aa，tc”。这也包括qr位
- `{>bufsize}`：查询中公布的EDNS0缓冲区大小
- `{>do}`：是查询中设置的EDNS0 DO（DNSSEC OK）位
- `{>id}`：查询ID
- `{>opcode}`：查询OPCODE
- `{common}`：默认的通用日志格式。
- `{combined}`：带有查询操作码的通用日志格式。
- `{/LABEL}`：如果任何元数据标签位于`{/`和之间， 则将其接受为占位符，如果未定义标签`}`，则占位符将替换为相应的元数据值或默认值 `-`。有关更多信息，请参阅*元数据*插件。

默认的通用日志格式为：

```txt
`{remote}:{port} - {>id} "{type} {class} {name} {proto} {size} {>do} {>bufsize}" {rcode} {>rflags} {rsize} {duration}`
```

这些日志中的每一个都将输出`log.Infof`，因此典型示例如下所示：

```txt
2018-10-30T19:10:07.547Z [INFO] [::1]:50759 - 29008 "A IN example.org. udp 41 false 4096" NOERROR qr,rd,ra,ad 68 0.037990251s
~~~~

## Examples

Log all requests to stdout

~~~ corefile
. {
    log
    whoami
}
```

自定义日志格式，适用于所有区域（`.`）

```corefile
. {
    log . "{proto} Request: {name} {type} {>id}"
}
```

仅为example.org（及以下）记录拒绝（NXDOMAIN和nodata）

```corefile
. {
    log example.org {
        class denial
    }
}
```

以组合日志格式记录未成功解析的所有查询。

```corefile
. {
    log . {combined} {
        class denial error
    }
}
```

记录我们没有收到错误的所有查询

```corefile
. {
    log . {
        class denial success
    }
}
```

多个语句也可以进行OR编辑，例如，我们可以重写上述情况如下：

```corefile
. {
    log . {
        class denial
        class success
    }
}
```

# 2. errors

## 描述

查询处理期间遇到的任何错误都将打印到标准输出。特定类型的错误可以在一段时间内合并和打印一次。

## 句法

基本语法是：

```
errors
```

提供额外的正则语法：

```
errors {
	consolidate DURATION REGEXP
}
```

选项`consolidate`允许在**DURATION**期间收集与正则表达式**REGEXP**匹配的多个错误消息。在接收到第一个这样的消息之后的**DURATION**之后，合并的消息将被打印到标准输出，例如

```
2 errors like '^read udp .* i/o timeout$' occurred in last 30s
```

允许`consolidate`使用具有不同**DURATION**和**REGEXP的**多个选项。如果某些错误消息对应于几个定义的正则表达式，则该消息将与第一个适当的**REGEXP**相关联。

为了获得更好的性能，建议在通过前缀或后缀（例如或）过滤错误消息时在正则表达式中使用`^`或`$`元字符。`^failed to .*``.* timeout$`

## 例子

使用*whoami*响应查询并将错误记录到标准输出。

```corefile
. {
    whoami
    errors
}
```

使用*转发*通过8.8.8.8解析查询，并打印合并错误消息，以查找带有后缀“i / o timeout”或前缀“Failed to”的错误。

```corefile
. {
    forward . 8.8.8.8
    errors {
        consolidate 5m ".* i/o timeout$"
        consolidate 30s "^Failed to .+"
    }
}
```

# 3. health

## 描述

启用health endpoint 。当CoreDNS启动并运行时，它将返回200 OK HTTP状态代码。默认情况下，将在端口8080 / health上导出运行状况。

## 句法

```
health [ADDRESS]
```

可选地取一个地址; 默认是`:8080`。健康路径是固定的`/health`。当此服务器运行正常时，运行状况端点将返回200响应代码和单词“确定”。

可以使用此扩展语法设置额外选项：

```
health [ADDRESS] {
    lameduck DURATION
}
```

- `lameduck`将使进程标记为不健康并在进程彻底退出关闭之前等待DURATION时间。

如果有多个coredns服务器，则只能在其中一个服务器块中启用*运行状况*（因为它是进程范围的）。如果确实需要多个端点，则必须在不同端口上运行健康端点：

```corefile
com {
    whoami
    health :8080
}

net {
    erratic
    health :8081
}
```

这样做是受支持的，但是两个endponts“：8080”和“：8081”将导出完全相同的健康状况。

## 度量

如果启用了监控（通过*prometheus*指令），则会导出以下指标：

- `coredns_health_request_duration_seconds{}`- 处理对本地`/health`端点的HTTP查询的持续时间 。此持续时间的增加表明CoreDNS进程无法跟上其查询负载。

## 例子

在[http：// localhost：8091](http://localhost:8091/)上运行另一个运行时health endpoint。

```corefile
. {
    health localhost:8091
}
```

设置1秒的lameduck持续时间：

```corefile
. {
    health localhost:8092 {
        lameduck 1s
    }
}
```

# 4. autopath

## 描述

如果它与配置的搜索路径的第一个元素匹配，则*autopath*将搜索路径元素链并返回不是NXDOMAIN的第一个reply。如果发生任何故障，将返回原reply。因为*autopath会*返回一个不是原始reply的名称，所以它会添加一个CNAME，该CNAME从原始名称（其中包含搜索路径元素）指向此answer的名称。

## 句法

```
autopath [ZONE...] RESOLV-CONF
```

- **ZONES**区域*autopath*应具有权威性。
- **RESOLV-CONF**指向`resolv.conf`类似文件或使用特殊语法指向另一个插件。例如`@kubernetes`，将调用kubernetes插件（对于每个查询）来检索它应该使用的搜索列表。

如果插件实现了`AutoPather`接口，则可以使用它。

## 度量

如果启用了监控（通过*prometheus*指令），则会导出以下指标：

- `coredns_autopath_success_count_total{server}` - 成功自动验证查询的计数器。

该`server`标签在*metrics*插件文档中进行了解释。

## 例子

```
autopath my-resolv.conf
```

使用`my-resolv.conf`的文件，以获得搜索路径来源。这个文件只需要一行： `search domain1 domain2 ...`

```
autopath @kubernetes
```

使用搜索路径从*kubernetes*插件动态检索。

## 已知的问题

在Kubernetes中，*autopath*与从Windows节点运行的*pod*不兼容。

如果服务器侧搜索最终返回一个否定的answer（例如`NXDOMAIN`），则客户端会手动搜索所有的路径，从而否定了*autopath*优化。

# 5. prometheus

## 描述

使用*prometheus，*您可以从CoreDNS和任何包含它们的插件中导出指标。指标的默认位置是`localhost:9153`。指标路径是固定的`/metrics`。导出以下指标：

- `coredns_build_info{version, revision, goversion}` - 有关CoreDNS本身的信息。
- `coredns_panic_count_total{}` - 恐慌总数。
- `coredns_dns_request_count_total{server, zone, proto, family}` - 总查询次数。
- `coredns_dns_request_duration_seconds{server, zone}` - 处理每个查询的持续时间。
- `coredns_dns_request_size_bytes{server, zone, proto}` - 请求的大小（以字节为单位）。
- `coredns_dns_request_do_count_total{server, zone}` - 设置了DO位的查询
- `coredns_dns_request_type_count_total{server, zone, type}` - 每个区域和类型的查询计数器。
- `coredns_dns_response_size_bytes{server, zone, proto}` - 响应大小（字节）。
- `coredns_dns_response_rcode_count_total{server, zone, rcode}` - 每个区域和rcode的响应。
- `coredns_plugin_enabled{server, zone, name}` - 指示是否基于每个服务器和区域启用插件。

每个计数器都有一个标签`zone`，它是用于请求/响应的zonename。

使用的额外标签是：

- `server`正在识别负责该请求的服务器。这是一个格式化为服务器监听地址的字符串：`<scheme>://[<bind>]:<port>`。即，这是一个“正常”的DNS服务器`dns://:53`。如果您使用*绑定*插件，则包含IP地址，例如：`dns://127.0.0.53:53`。
- `proto` 它保存响应的传输（“udp”或“tcp”）
- `family`传输的地址族（）（1 = IP（IP版本4），2 = IP6（IP版本6））。
- `type`它包含查询类型。它包含最常见的类型（A，AAAA，MX，SOA，CNAME，PTR，TXT，NS，SRV，DS，DNSKEY，RRSIG，NSEC，NSEC3，IXFR，AXFR和ANY）和“其他”将所有其他类型混为一谈。
- 它`response_rcode_count_total`有一个额外的标签`rcode`，用于保存响应的rcode。

如果启用了监视，则不会进入插件链的查询将以伪名“已删除”的形式导出（没有结束点 - 这绝不是有效的域名）。

此插件每个服务器块只能使用一次。

## 句法

```
prometheus [ADDRESS]
```

对于要查看其指标的每个区域。

它可选地采用导出度量的绑定地址; 默认侦听`localhost:9153`。指标路径是固定的`/metrics`。

## 例子

使用一个默认的监听端口+地址：

```corefile
. {
    prometheus localhost:9253
}
```

或者通过环境变量（整个Corefile支持）:, `export PORT=9253`然后：

```corefile
. {
    prometheus localhost:{$PORT}
}
```

## 错误

重新加载时，Prometheus处理程序在新服务器实例启动之前停止。如果新服务器无法启动，则初始服务器实例仍然可用且仍然提供DNS查询，但Prometheus handler 保持关闭状态。在成功重新加载或完全重启CoreDNS之前，Prometheus不会回复HTTP请求。只有注册为Handler的插件才可见`coredns_plugin_enabled{server, zone, name}`。

# 6. cache

## 描述

随着*cache*启用，除了zone transfers和元数据记录的所有其他记录都将被缓存3600s。

## 句法

```txt
cache [TTL] [ZONES...]
```

- **TTL**最大TTL，以秒为单位。如果未指定，将使用最大TTL，NOERROR响应为3600，拒绝响应为1800。将TTL设置为300：`cache 300`将记录缓存最多300秒。
- 它应该缓存的**ZONES**区域。如果为空，则使用配置块中的区域。

缓存中的每个元素都根据其TTL进行缓存（**TTL**为最大值）。缓存分为256个分片，默认情况下每个分片最多可容纳39个项目 - 总大小为256 * 39 = 9984个项目。

如果你想要更多的控制：

```txt
cache [TTL] [ZONES...] {
    success CAPACITY [TTL] [MINTTL]
    denial CAPACITY [TTL] [MINTTL]
    prefetch AMOUNT [[DURATION] [PERCENTAGE%]]
}
```

- 如上所述的**TTL** 和**ZONES**。
- `success`，覆盖缓存成功响应的设置。**CAPACITY**表示在我们开始逐出（*随机*）之前缓存的最大数据包数。**TTL**会覆盖缓存最大TTL。 **MINTTL**会覆盖缓存最小TTL（默认值为5），这对于限制对后端的查询非常有用。
- `denial`，覆盖缓存拒绝存在响应的设置。**CAPACITY**表示在我们开始驱逐（LRU）之前缓存的最大数据包数。**TTL**会覆盖缓存最大TTL。 **MINTTL**会覆盖缓存最小TTL（默认值为5），这对于限制对后端的查询非常有用。还有第三个类别（`error`），但这些响应永远不会被缓存。
- `prefetch`当它们即将从缓存中清除时，它将预取热门项目。热门意味着已经看到**AMOUNT个**查询，它们之间没有**DURATION**或更多的差距。 **DURATION**默认为1米。当TTL降至**PERCENTAGE**以下时会发生预取，默认为`10%`TTL到期之前或最迟1秒。值应在范围内`[10%, 90%]`。请注意百分号是强制性的。**PERCENTAGE**被视为一个`int`。

## 能力和驱逐

如果*未*指定**CAPACITY** ，则每个缓存的默认缓存大小为9984。允许的最小高速缓存大小为1024.如果指定了**CAPACITY，***则*使用的实际高速缓存大小将向下舍入到可被256整除的最接近的数字（因此所有分片的大小相等）。

驱逐是每个碎片完成的。实际上，当分片达到容量时，项目将从该分片中逐出。由于分片不能完全均匀填充，因此在整个缓存达到满容量之前会发生驱逐。每个分片容量等于总缓存大小/分片数（256）。驱逐是随机的，而不是基于TTL的。具有0 TTL的条目将保留在缓存中，直到分片达到容量时随机逐出。

## 度量

如果启用了监控（通过*prometheus*指令），则会导出以下指标：

- `coredns_cache_size{server, type}` - 缓存类型的缓存中的总元素。
- `coredns_cache_hits_total{server, type}` - 缓存按缓存类型计数。
- `coredns_cache_misses_total{server}` - 缓存未命中的计数器。
- `coredns_cache_drops_total{server}` - 丢弃消息的计数器。

缓存类型是“拒绝”或“成功”。`Server`是处理请求的服务器，请参阅文档的metrics插件。

## 例子

为所有区域启用缓存，但将所有内容限制为10秒的TTL：

```corefile
. {
    cache 10
    whoami
}
```

代理到Google公共DNS，只缓存example.org（或更低版本）的响应。

```corefile
. {
    forward . 8.8.8.8:53
    cache example.org
}
```

为所有区域启用缓存，保持正缓存大小为5000，负缓存大小为2500：

```corefile
 . {
     cache {
         success 5000
         denial 2500
    }
 }
```

# 7. loop

## 描述

该*loop*插件将随机发送探针请求给自身，然后跟踪记录收到多少次探针请求。如果记录探针请求连续超过两次，coredns将将停止进程。

该插件将不停发送探针请求持续30秒。这样做是为了让CoreDNS有足够的时间启动。如果请求成功，*loop*将自身禁用。

发送的查询的`<random number>.<random number>.zone`类型设置为HINFO。

## 句法

```txt
loop
```

## 例子

在默认端口上启动服务器并加载*loop*和*forward*插件。*forward*插件转发到请求到自身。

```txt
. {
    loop
    forward . 127.0.0.1
}
```

CoreDNS启动后，它会在记录时停止进程：

```txt
plugin/loop: Loop (127.0.0.1:55953 -> :1053) detected for zone ".", see https://coredns.io/plugins/loop#troubleshooting. Query: "HINFO 4547991504243258144.3688648895315093531."
```

## 限制

此插件仅尝试在启动时找到简单的静态转发循环。要检测循环，必须满足以下条件：

- 循环必须在启动时出现。
- 必须为`HINFO`查询类型发生循环。

# 8. reload

## 描述

此插件允许自动重新加载已更改的*Corefile*。

## 句法

```txt
reload [INTERVAL] [JITTER]
```

- 该插件将检查每个**INTERVAL的**变化，但需要+/- **JITTER**持续时间
- **INTERVAL**和**JITTER**是Golang（持续时间）[ <https://golang.org/pkg/time/#ParseDuration> ]
- 默认**INTERVAL**为30s，默认**JITTER**为15s
- **INTERVAL的**最小值是2s，而**JITTER的**最小值是1s
- 如果**JITTER**超过**INTERVAL的**一半，它将被设置为**INTERVAL的**一半

## 例子

检查默认间隔：

```corefile
. {
    reload
    erratic
}
```

每10秒检查一次（在这种情况下抖动自动设置为10/2 = 5）：

```corefile
. {
    reload 10s
    erratic
}
```

# 9. loadbalance

## 描述

该*LOADBALANCE*将通过round-robin 算法随机负载A，AAAA和MX记录的answer。

## 句法

```
loadbalance [POLICY]
```

- **POLICY**是如何平衡，默认和唯一的选择，是“round_robin”。

## 例子

来自Google Public DNS的负载余额回复：

```corefile
. {
    loadbalance round_robin
    forward . 8.8.8.8 8.8.4.4
}
```

## 10. proxy

## 描述

作为反向代理连接多个后端，作为负载平衡实现多个策略，运行状况检查和故障转移。如果所有主机都未通过运行状况检查，则代理插件将故障返回以随机选择目标并向其发送数据包。

## 句法

在最基本的形式中，简单的反向代理使用以下语法：

```
proxy FROM TO
```

- **FROM**是要代理的请求匹配的基本域。
- **TO**是代理的目标端点。

但是，包括负载平衡在内的高级功能可以使用扩展语法：

```
proxy FROM TO... {
    policy random|least_conn|round_robin|sequential
    fail_timeout DURATION
    max_fails INTEGER
    health_check PATH:PORT [DURATION]
    except IGNORED_NAMES...
    spray
    protocol [dns [force_tcp]|grpc [insecure|CACERT|KEY CERT|KEY CERT CACERT]]
}
```

- **FROM**是要代理的请求匹配的名称。
- **TO**是代理的目标端点。至少需要一个，但可以指定多个。**TO**可以是IP：端口对，也可以引用resolv.conf格式的文件
- `policy`是要使用的负载平衡策略; 仅适用于多个后端。可以是random，least_conn，round_robin或sequential之一。默认是随机的。
- `fail_timeout`指定在后端失败后将后端视为停机的时间。当它关闭时，请求将不会路由到该后端。如果CoreDNS无法与之通信，后端将“关闭”。默认值为2秒（“2s”）。
- `max_fails`是在考虑后端关闭之前所需的fail_timeout内的失败次数。如果为0，则后端永远不会标记为关闭。默认值为1。
- `health_check`将在每个后端检查**PATH**（在**PORT上**）。如果后端返回状态代码200-399，则该后端标记为健康，使其达到健康检查持续时间的两倍。如果没有，则将其标记为不健康，并且不会将请求路由到它。如果未提供此选项，则禁用运行状况检查。默认持续时间为4秒（“4s”）。
- **IGNORED_NAMES** in `except`是一个以空格分隔的域列表，可以从代理中排除。将不会传递与这些名称都不匹配的请求。
- `spray`当所有后端都不健康时，随机选择一个发送流量。（这是一个故障保险。）
- `protocol`指定用于与上游通信的协议，`dns`（默认）是普通的旧DNS。该`grpc`选项将与已实现[DnsService](https://github.com/coredns/coredns/blob/master/pb/dns.proto)的服务器[通信](https://github.com/coredns/coredns/blob/master/pb/dns.proto)。

## 策略

有四种负载均衡策略可用：

- `random` （默认） - 随机选择后端
- `least_conn` - 选择活动连接最少的后端
- `round_robin` - 以循环方式选择后端
- `sequential` - 从左到右按照声明顺序选择第一个可用的后端
- `first` - 不推荐。请改用顺序

当*没有健康的*主机可用时，所有策略都会实施随机向后端主机喷洒数据包。这是为了抢占健康检查（作为一种机制）失败的情况。

## 上游协议

- `dns`

  使用标准DNS交换。`force_tcp`无论入站请求的协议如何，您都可以通过以确保代理连接是通过TCP执行的。

- `grpc`

  额外选项用于控制如何与gRPC服务器建立TLS连接。无 - 不使用客户端身份验证，系统CA用于验证服务器证书。`insecure` - 未使用TLS，连接以明文形式进行（生产效果不佳）。**CACERT** - 未使用客户端身份验证，文件**CACERT**用于验证服务器证书。**KEY** **CERT** - 客户端身份验证与指定的密钥/证书对一起使用。使用系统CA验证服务器证书。**KEY** **CERT** **CACERT** - 客户端身份验证与指定的密钥/证书对一起使用。使用**CACERT**文件验证服务器证书。

## 度量

如果启用了监控（通过*prometheus*指令），则会导出以下指标：

- `coredns_proxy_request_duration_seconds{server, proto, proto_proxy, family, to}` - 每上游交互的持续时间。
- `coredns_proxy_request_count_total{server, proto, proto_proxy, family, to}` - 每个上游的查询计数。

其中`proxy_proto`使用的协议（`dns`或`grpc`）和`to`是**TO** 在配置指定的，`proto`是由传入查询（“TCP”或“UDP”），家庭传输家族（“1”的IPv4使用的协议，和“2”为IPv6的）。`Server`是负责请求（和度量标准）的服务器。请参阅metrics插件中的文档。

## 例子

代理example.org中的所有请求。到后端系统：

```
proxy example.org 127.0.0.1:9005
```

在三个后端之间对所有请求进行负载均衡（使用RR策略）：

```corefile
. {
    proxy . 10.0.0.10:53 10.0.0.11:1053 10.0.0.12
}
```

与上述相同，但循环风格：

```corefile
. {
    proxy . 10.0.0.10:53 10.0.0.11:1053 10.0.0.12 {
        policy round_robin
    }
}
```

使用运行状况检查和代理头来传递主机名，IP和方案上游：

```corefile
. {
    proxy . 10.0.0.11:53 10.0.0.11:53 10.0.0.12:53 {
        policy round_robin
        health_check /health:8080
    }
}
```

除了对miek.nl或example.org的请求之外，代理所有内容

```
. {
    proxy . 10.0.0.10:1234 {
        except miek.nl example.org
    }
}
```

除了`example.org`使用主机的`resolv.conf`名称服务器之外，代理所有内容：

```corefile
. {
    proxy . /etc/resolv.conf {
        except example.org
    }
}
```

# 11. kubernetes

## 描述

该插件实现了[Kubernetes基于DNS的服务发现规范](https://github.com/kubernetes/dns/blob/master/docs/specification.md)。

运行kubernetes插件的CoreDNS可以用作kubernetes集群中kube-dns的替代品。有关[如何在Kubernetes中部署CoreDNS的](https://github.com/coredns/deployment/tree/master/kubernetes)详细信息，请参阅[部署](https://github.com/coredns/deployment)存储库。

[stubDomains和upstreamNameservers](https://kubernetes.io/blog/2017/04/configuring-private-dns-zones-upstream-nameservers-kubernetes/) 通过*forward*插件实现。请参阅以下示例。

## 句法

```
kubernetes [ZONES...]
```

只指定了指令，*kubernetes*插件将默认为服务器块中指定的区域。它将处理该区域中的所有查询并连接到集群中的Kubernetes。它不会提供服务的PTR记录或pod的A记录。如果使用**ZONES**，则指定插件应具有权威性的所有区域。

```
kubernetes [ZONES...] {
    resyncperiod DURATION
    endpoint URL
    tls CERT KEY CACERT
    kubeconfig KUBECONFIG CONTEXT
    namespaces NAMESPACE...
    labels EXPRESSION
    pods POD-MODE
    endpoint_pod_names
    ttl TTL
    noendpoints
    transfer to ADDRESS...
    fallthrough [ZONES...]
    ignore empty_service
}
```

- `resyncperiod`指定Kubernetes数据API **DURATION**期间。默认情况下，禁用重新同步（DURATION为零）。
- `endpoint`指定远程k8s API endpoint的**URL**。如果省略，它将使用群集服务帐户连接到群集中的k8s。
- `tls` **CERT** **KEY** **CACERT**是远程k8s连接的TLS证书，密钥和CA证书文件名。如果连接集群内（即未指定端点），则忽略此选项。
- `kubeconfig` **KUBECONFIG** **CONTEXT**使用kubeconfig文件验证与远程k8s群集的连接。它支持TLS，用户名和密码或基于令牌的身份验证。如果连接集群内（即未指定端点），则忽略此选项。
- `namespaces` **NAMESPACE [NAMESPACE ...]**仅公开列出的k8s名称空间。如果省略此选项，则会公开所有名称空间
- `namespace_labels` **EXPRESSION**仅公开与此标签选择器匹配的Kubernetes名称空间的记录。标签选择器语法在[Kubernetes用户指南 - 标签中描述](http://kubernetes.io/docs/user-guide/labels/)。仅公开标记为“istio-injection = enabled”的名称空间的示例将使用：`labels istio-injection=enabled`。
- `labels` **EXPRESSION**仅公开与此标签选择器匹配的Kubernetes对象的记录。标签选择器语法在 [Kubernetes用户指南 - 标签中描述](https://kubernetes.io/docs/user-guide/labels/)。仅在“staging”或“qa”环境中公开标记为“application = nginx”的对象的示例将使用：`labels environment in (staging, qa),application=nginx`。
- `pods` **POD-MODE**设置处理基于IP的pod A记录的模式，例如 `1-2-3-4.ns.pod.cluster.local. in A 1.2.3.4`。提供此选项是为了便于在直接连接到pod时使用SSL证书。**POD-MODE的**有效值：
  - `disabled`：默认。不要处理pod请求，总是返回`NXDOMAIN`
  - `insecure`：始终从请求中返回带有IP的A记录（不检查k8s）。如果与通配符SSL证书一起恶意使用，此选项很容易被滥用。提供此选项是为了向后兼容kube-dns。
  - `verified`：如果在具有匹配IP的相同名称空间中存在pod，则返回A记录。此选项需要比不安全模式更多的内存，因为它将在所有pod上保持监视。
- `endpoint_pod_names`使用端点所针对的pod的pod名称作为A记录中的端点名称，例如，`endpoint-name.my-service.namespace.svc.cluster.local. in A 1.2.3.4` 默认情况下，端点名称名称选择如下：使用端点的主机名，或者如果未设置hostname，则使用端点IP地址的虚线形式（例如`1-2-3-4.my-service.namespace.svc.cluster.local.`）如果包含此指令，则端点的名称选择更改如下：使用端点的主机名，或者如果未设置主机名，请使用由端点定位的窗格的窗格名称。端点。如果端点没有目标pod，请使用虚线IP地址表单。
- `ttl`允许您为响应设置自定义TTL。默认值为5秒。允许的最小TTL为0秒，最大值限制为3600秒。将TTL设置为0将阻止缓存记录。
- `noendpoints`将通过在端点上禁用监视来关闭端点记录的提供。所有端点查询和无头服务查询都将产生NXDOMAIN。
- `transfer`启用区域传输。它可以指定倍数。`To`发信号指示方向（仅`to`允许）。**ADDRESS**必须CIDR符号（127.0.0。来表示1 / 32等），或者仅仅作为纯地址。特殊的通配符`*`意味着：整个互联网。不支持发送DNS通知。子域中不推荐[使用的](https://github.com/kubernetes/dns/blob/master/docs/specification.md#26---deprecated-records) pod记录`pod.cluster.local`不会被传输。
- `fallthrough` **[ZONES ...]**如果对插件具有权威性的区域中的记录进行查询会导致NXDOMAIN，通常这就是响应。但是，如果指定此选项，则查询将在插件链上传递，该插件链可以包含另一个插件来处理查询。如果省略**[ZONES ...]**，则插件具有权威性的所有区域都会发生连贯。如果列出了特定区域（例如`in-addr.arpa`和`ip6.arpa`），则只有这些区域的查询才会受到影响。
- `ignore empty_service`返回NXDOMAIN以获取没有任何就绪端点地址的服务（例如，就绪pod）。这允许查询窗格继续在搜索路径中搜索服务。例如，搜索路径可以包括另一个Kubernetes集群。

## 例子

处理`cluster.local`区域中的所有查询。连接到群集中的Kubernetes。还处理所有 `in-addr.arpa` `PTR`请求`10.0.0.0/17`。在回答pod请求时验证pod是否存在。

```txt
10.0.0.0/17 cluster.local {
    kubernetes {
        pods verified
    }
}
```

或者您可以有选择地公开一些名称空间：

```txt
kubernetes cluster.local {
    namespaces test staging
}
```

使用在集群外部运行的CoreDNS连接到Kubernetes：

```txt
kubernetes cluster.local {
    endpoint https://k8s-endpoint:8443
    tls cert key cacert
}
```

## stubDomains和upstreamNameservers

这里我们使用*forward*插件来实现一个转发`example.local`到nameserver 的stubDomain `10.100.0.10:53`。还配置了一个upstreamNameserver `8.8.8.8:53`，用于解析不属于的名称`cluster.local` 或`example.local`。

```txt
cluster.local:53 {
    kubernetes cluster.local
}
example.local {
    forward . 10.100.0.10:53
}

. {
    forward . 8.8.8.8:53
}
```

上面的配置表示以下Kube-DNS stubDomains和upstreamNameservers配置。

```txt
stubDomains: |
   {“example.local”: [“10.100.0.10:53”]}
upstreamNameservers: |
   [“8.8.8.8:53”]
```

## AutoPath

所述*kubernetes*插件可与*autopath*插件结合使用。使用此功能可在Kubernetes群集中完成服务器端域搜索路径。注意：`pods`必须设置`verified`为此才能正常运行。

```
cluster.local {
    autopath @kubernetes
    kubernetes {
        pods verified
    }
}
```

## Federation

所述*kubernetes*插件可与结合使用*Federation*插件。使用此功能可以从Kubernetes群集中提供联合域。

```
cluster.local {
    federation {
        prod prod.example.org
        staging staging.example.org
    }
    kubernetes
}
```

## 通配符

某些查询标签接受通配符值以匹配任何值。如果标签是有效的通配符（*或单词“any”），则该标签将匹配所有值。接受通配符的标签是：

- *endpoint*在`A`录制请求：*端点* .service.namespace.svc.zone，例如，`*.nginx.ns.svc.cluster.local`
- *service* *在`A`录制请求：*服务* .namespace.svc.zone，例如，`*.ns.svc.cluster.local`
- *namespace* *中的`A`记录请求：服务。*namespace* .svc.zone，例如，`nginx.*.svc.cluster.local`
- *port and/or protocol* 在`SRV`请求中的*端口和/或协议*：*port_.protocol_.service.namespace.svc.zone。例如，`_http.\**service.ns.svc.cluster.local`
- 在单个查询中允许多个通配符，例如，`A`请求`*.*.svc.zone.`或`SRV`请求`*.*.*.*.svc.zone.`

例如，通配符可用于将服务的所有端点解析为`A`记录。例如：`*.service.ns.svc.myzone.local`将`service`在命名空间中返回服务中的端点IP `default`：

```
*.service.default.svc.cluster.local. 5	IN A	192.168.10.10
*.service.default.svc.cluster.local. 5	IN A	192.168.25.15
```

可以使用`loadbalance`插件随机化此响应

## 元数据

如果 还启用了*metadata* *插件，kubernetes插件将发布以下元*数据：

- kubernetes / endpoint：查询中的端点名称
- kubernetes / kind：查询中的资源种类（pod或svc）
- kubernetes / namespace：查询中的命名空间
- kubernetes / port-name：SRV查询中的端口名称
- kubernetes / protocol：SRV查询中的协议
- kubernetes / service：查询中的服务名称
- kubernetes / client-namespace：客户端pod的命名空间，如果`pods verified`启用了mode
- kubernetes / client-pod-name：客户端窗格的名称，如果`pods verified`启用了模式
