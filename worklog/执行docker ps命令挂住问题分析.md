查找/var/log/messages，发现有错误信息：XFS: possible memory allocation deadlock in kmem_alloc (mode:0x250)

查找linux内核，搜索"kernel: XFS: possible memory allocation deadlock in kmem_alloc (mode:0x250)

发现源代码如下：
```java
void *
kmem_alloc(size_t size, xfs_km_flags_t flags)
{
   int    retries = 0;
   gfp_t  lflags = kmem_flags_convert(flags);
   void   *ptr;
 
   do {
      ptr = kmalloc(size, lflags);
      if (ptr || (flags & (KM_MAYFAIL|KM_NOSLEEP)))
         return ptr;
      if (!(++retries % 100))
         xfs_err(NULL,
      "possible memory allocation deadlock in %s (mode:0x%x)",
               __func__, lflags);
      congestion_wait(BLK_RW_ASYNC, HZ/50);
   } while (1);
}
```
分析代码：

+ 当尝试申请到内存时，直接返回；
+ 申请不到内存会等待HZ/50 的jiffies后，继续尝试，直到申请成功才会退出；
+ 每尝试申请失败100次后，会打印possible memory allocation deadlock in %s (mode:0x%x)
+ 说明申请不到内存时，会卡在这里一直循环下去
mode的含义是所申请的内存标记为， GFP_IO|GFP_WAIT|GFP_NOWARN，如下:

结合日志来看，有很长一段时间都在打印“XFS: possible memory allocation deadlock in kmem_alloc (mode:0x250)”


之前只是偶尔打印“XFS: possible memory allocation deadlock in kmem_alloc 
(mode:0x250)”

用到的文件系统是XFS，查找到一段资料，里面描述：

我们可以看到这里比较大块的连续的page 是基本没有的. 因为在xfs 的申请内存操作里面我们看到有这种连续的大块的内存申请的操作的请求, 比如:
```shell
6000:   map = kmem_alloc(subnex * sizeof(*map), KM_MAYFAIL | KM_NOFS);
```
因此比较大的可能是线上虽然有少量的空闲内存, 但是这些内存非常的碎片, 因此只要有一个稍微大的的连续内存的请求都无法满足。

由于已经做了echo 1 > /proc/sys/vm/drop_caches 操作，查看下另一台机器的情况：

 /proc/buddyinfo是linuxbuddy系统管理物理内存的debug信息。

在linux中使用buddy算法解决物理内存的外碎片问题，其把所有空闲的内存，以2的幂次方的形式，分成11个块链表，分别对应为1、2、4、8、16、32、64、128、256、512、1024个页块。

而Linux支持NUMA技术，对于NUMA设备，NUMA系统的结点通常是由一组CPU和本地内存组成，每一个节点都有相应的本地内存，因此buddyinfo 中的Node0表示节点ID；而每一个节点下的内存设备，又可以划分为多个内存区域（zone），因此下面的显示中，对于Node0的内存，又划分类DMA、Normal、HighMem区域。而后面则是表示空闲的区域。

此处以Normal区域进行分析，第二列值为100，表示当前系统中normal区域，可用的连续两页的内存大小为100*2*PAGE_SIZE；第三列值为52，表示当前系统中normal区域，可用的连续四页的内存大小为52*2^2*PAGE_SIZE
```shell
cat /proc/buddyinfo 
Node 0, zone      DMA     23     15      4      5      2      3      3      2      3      1      0 
Node 0, zone   Normal    149    100     52     33     23      5     32      8     12      2     59 
Node 0, zone  HighMem     11     21     23     49     29     15      8     16     12      2    142 
```

可以看到从第5列开始，只剩下44*16*PAGE_SIZE的页块，后面剩下的分别是1 * 32 *PAGE_SIZE, 1 * 64 *PAGE_SIZE, 2 *128 * PAGE_SIZE，剩下256,512的页块都没有了


因此推测，导致这个问题出现的时候，该机器的内存碎片很多，当某个应用执行时，在xfs 的申请内存中有这种连续的大块的内存申请的操作的请求, 比如:
 
```shell
6000:   map = kmem_alloc(subnex * sizeof(*map), KM_MAYFAIL | KM_NOFS);
```
就会导致内存一直分配不到。例如执行docker ps,docker exec这些命令时，会一直阻塞在kmem_alloc的循环中，反复申请内存，由于内存碎片没有被组合，因此就一直申请不到，执行这些命令也会卡住，这也就验证了执行某些命令如ls,ssh都不会失败（因为需要内存的PAGE不是那么大）。

而执行echo 1 > /proc/sys/vm/drop_caches 操作会把碎片化的PAGE重新分配，之后在申请大块的PAGE内存就可以申请到，不阻塞了。

然而不能总是输入这个命令解决问题，于是找到了下面的方法：
 

参考

http://www.cnblogs.com/itfriend/archive/2011/12/14/2287160.html

Linux 提供了这样一个参数min_free_kbytes，用来确定系统开始回收内存的阀值，控制系统的空闲内存。值越高，内核越早开始回收内存，空闲内存越高。

```shell
设置/proc/sys/vm/min_free_kbytes的值为4G bytes

echo 4194304 > /proc/sys/vm/min_free_kbytes

目前为90M bytes：

```
查看内存碎片情况，发现有明显改善。

按此方法修改后，再没有出现docker ps命令挂住的现象。
 