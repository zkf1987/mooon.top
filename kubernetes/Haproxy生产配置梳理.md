| 配置  | 参数  | 值   | 解释  |
| ------------ | ------------ | ------------ | ------------ |
| global  |  log |  127.0.0.1 local0 | 定义全局的syslog服务器  |
| global  | maxconn  | 65535  |  设定每个haproxy进程所接受的最大并发连接数 |
| global  | daemon  |   | 让haproxy以守护进程的方式工作于后台等同于“-D”选项的功能  |
| global  | uid  | 99  | 以指定的GID运行haproxy，建议使用专用于运行haproxy的GID，以免因权限问题带来风险  |
| global  | pid  | 99  | 同gid，不过指定的组名  |
|   |   |   |   |
| defaults  |log   | global  | 当前实例的日志系统参数同”global”段中的定义时，将使用此格式  |
| defaults  | timeout  | server 300s  |  客户端与服务端建立连接后，等待服务端的超时时长 |
|defaults   |timeout   | server 300s  |  定义HAProxy将客户端请求转发至后端服务器所等待的时长 |
| defaults  | timeout  | server 300s  |  客户端非活动状态的超时时长 |
|   |   |   |   |
| listen  | bind  | 0.0.0.0:port  |  此指令仅能用于frontend和listen区段，用于定义一个或几个监听的套接字。 |
| listen  |  mode | http  | 实例运行于纯TCP模式，在客户端和服务器端之间将建立一个全双工的连接，且不会对7层报文做任何类型的检查；通常用于SSL、SSH、SMTP等应用；  |
| listen  |  mode |  tcp | 实例运行于HTTP模式，客户端请求在转发至后端服务器之前将被深度分析，所有不与RFC格式兼容的请求都会被拒绝；此为默认模式；  |
| listen  |  mode | health  | 实例工作于health模式，其对入站请求仅响应“OK”信息并关闭连接，且不会记录任何日志信息；此模式将用于响应外部组件的健康状态检查请求；目前来讲，此模式已经废弃，因为tcp或http模式中的monitor关键字可完成类似功能；  |
| listen  |  option | httpchk  | 定义基于http协议健康状态检测机制  |
| listen  |  option | tcp-check  | 定义基于tcp协议健康状态检测机制  |
| listen  |  option | smtpchk  |  定义基于smtp协议健康状态检测机制 |
| listen  | option  | mysql-check  | 定义mysql健康状态检测机制  |
| listen  |  option | pgsql-check  | 定义pgsql健康状态检测机制  |
| listen  | option  | ssl-hello-chk  | 定义使用ssl证书的连接健康状态检测机制  |
| listen  |  option | http-close  | HAProxy会针对客户端的第一条请求的返回添加cookie并返回给客户端，客户端发送后续请求时会发送此cookie到HAProxy，HAProxy会针对此cookie分发到上次处理此请求的服务器上。  |
| listen  |  option |  forwardfor | 如果服务器上的应用程序想记录发起请求的客户端的IP地址，需要在HAProxy上配置此选项，这样HAProxy会把客户端的IP信息发送给服务器，在HTTP请求中添加"X-Forwarded-For"字段。  |
|  listen |  balance | roundrobin  |  基于权重进行轮询，在服务器的处理时间保持均匀分布时，这是最平衡、最公平的算法。此算法是动态的，这表示其权重可以在运行时进行调整，不过，在设计上，每个后端服务器仅能最多接受4128个连接；并支持慢启动。 |
|  listen |  cookie |  SERVERNAME insert nocache | 为指定server设定cookie属性名为SERVERNAME值，此处指定的值将在请求入站时被检查，第一次为此值挑选的server将在后续的请求中被选中，其目的在于实现持久连接的功能；  |
|  frontend | default-backend  |  backend name |  设定监听转发的后端 |
| backend  | stick-table  |  type ip size 200k expire 30m | HAProxy会话保持机制，创建一个以源IP地址为key的stick table，该表允许20W条记录，30分钟的记录过期时长，并且不记录任何额外数据。  |
| backend  |  stick on | src  | 存储指定内容，并在请求到达时匹配该内容，只有配置了stick on后，haproxy才能根据匹配的结果决定是否存储到stick table中，以及如何筛选待分派的后端。  |
|backend   | server  | check port  | 启动对此server执行健康状态检查  |
| backend  | server  | inter  |设定健康状态检查的时间间隔，单位为毫秒，默认为2000   |
| backend  |  server | upon-marked-down shutdown-sessions  | 表示当该服务器被认为是 shutdown的时候，关闭全部与该服务器的请求连接。  |
|backend   | server  |fall 3   | 确认server从正常状态转换为不可用状态需要检查的次，当前生产配置3次  |
| backend  | server  | rise 3  |  设定健康状态检查中，某离线的server从离线状态转换至正常状态需要成功检查的次数，当前生产配置为3次 |
|  backend | server  |  observe <mode> | 通过观察服务器的通信状况来判定其健康状态，默认为禁用，其支持的类型有“layer4”和“layer7”，“layer7”仅能用于http代理场景；  |
| backend  | server  |  weight <value> | 权重，默认为1，最大值为256，0表示不参与负载均衡（不被调度）  |