## memory
内存资源控制器可以将一个组的内存访问行为和系统其它部分隔离开来，其特性包括：

* 审计匿名页、文件缓存、交换缓存用量，并限制之
* 页面按照Cgroup创建LRU，不使用全局LRU
* memory + swap总量可以被审计和限制
* 软限制支持
* 用量阈值、内存压力通知器
* 禁用oom-killer的开关，oom通知器
* 根控制组不受到限制
### 文件
|文件|	说明|
:---:|:---
cgroup.procs|	显示进程列表|
cgroup.event_control|	event_fd()的接口
memory.usage_in_bytes	|显示当前内存用量
memory.memsw.usage_in_bytes	|显示当前内存 + Swap用量
memory.limit_in_bytes	|设置/显示内存用量配额
memory.memsw.limit_in_bytes	|设置/显示内存 + Swap用量配额
memory.soft_limit_in_bytes|设置/显示内存用量的软限制软限制用于实现更多的内存共享，它允许控制组使用尽可能多的内存，前提是： 没有内存争用    不超过硬限制  当系统检测到内存不足或争用时，控制组被push back到软限制。软限制是一种Best-effort特性，不做绝对保证
memory.failcnt	|显示内存用量到达限额的次数
memory.memsw.failcnt	|显示内存 + Swap用量到达限额的次数
memory.max_usage_in_bytes	|显示记录到的内存用量峰值
memory.memsw.max_usage_in_bytes	|显示记录到的内存 + Swap用量峰值
memory.stat	|显示多种统计信息：cache 页面缓存用量 rss 匿名页 + Swap缓存内存用量，包括透明巨页rss_huge 匿名透明巨页用量mapped_file 映射文件内存用量，包括tmpfs/shmemswap 交换分区用量dirty 脏页（等待回写磁盘）用量writeback 等待回写磁盘的量inactive_anon 非活动LRU中的匿名+ Swap用量active_anon2活动LRU中的匿名+ Swap用量unevictable 不可回收内存量（例如mlocked）
memory.use_hierarchy	|设置/显示层次性审计
memory.pressure_level	设置内存压力通知
memory.swappiness	设置/显示vmscan的swappiness参数，参考sysctl的vm.swappiness
memory.oom_control	
设置/显示OOM控制。设置为1可以禁用OOM-killer

你可以在OOM发生后获得通知：

调用eventfd(2)创建文件描述符
打开memory.oom_control文件
写入<event_fd> <fd of memory.oom_control>到cgroup.event_control
当OOM发生后通过eventfd通知应用程序

当OOM-killer被禁用后，组中任务请求过量内存时会在OOM-waitqueue中挂起/休眠。放开限制后任务可以继续运行

memory.numa_stat |	显示各NUMA节点的内存用量统计信息
memory.kmem.limit_in_bytes	设置/显示内核内存的硬限制
memory.kmem.usage_in_bytes	显示当前内核内存用量
memory.kmem.failcnt	显示内核内存用量到达限额的次数
memory.kmem.max_usage_in_bytes	显示记录到的内核内存用量峰值
memory.kmem.tcp.limit_in_bytes	显示/设置TCP缓冲内存限额
memory.kmem.tcp.usage_in_bytes	显示当前TCP缓冲内存用量
memory.kmem.tcp.failcnt	显示TCP缓冲内存用量到达限额的次数
memory.kmem.tcp.max_usage_in_byte	显示记录到的TCP缓冲内存用量峰值