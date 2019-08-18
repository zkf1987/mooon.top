### netstat -s 命令参数说明

1. IP

   | 参数                               | 含义                     | 备注 |
   | ---------------------------------- | ------------------------ | ---- |
   | 818608 total packets received      | 接收到的IP数据包总数     |      |
   | 0 forwarded                        | 转发数                   |      |
   | 0 incoming packets discarded       | 丢弃数                   |      |
   | 805474 incoming packets delivered  | 接收数                   |      |
   | 774311 requests sent out           | 发出请求数               |      |
   | 40 outgoing packets dropped        | 丢包数                   |      |
   | 1 dropped because of missing route | 由于路由缺失导致的丢包数 |      |

   

2. Icmp

   | 参数                            | 含义                   | 备注 |
   | ------------------------------- | ---------------------- | ---- |
   | 232 ICMP messages received      | ICMP报文接收数         |      |
   | 5 input ICMP message failed     |                        |      |
   | ICMP input histogram:           | ICMP接收历史           |      |
   | 1. destination unreachable: 225 | 目标不可达的icmp报文数 |      |
   | 2. echo requests: 1             | 回显请求（ping）报文数 |      |
   | 3. echo replie: 6               | 回显应答（ping）报文数 |      |
   | 262 ICMP messages sent          | ICMP消息发出数         |      |
   | 0 ICMP messages failed          | ICMP消息发送失败数     |      |
   | ICMP output histogram:          | ICMP发出历史           |      |
   | 1. destination unreachable: 255 | 目标不可达的icmp报文数 |      |
   | 2. echo requests: 6             | 回应请求报文数         |      |
   | 3. echo replie: 1               | 回应回复报文数         |      |

   

3. IcpmMsg

   | 参数          | 含义                      | 备注 |
   | ------------- | ------------------------- | ---- |
   | InType0: 6    | type0类型的icmp报文接收数 |      |
   | InType3: 225  | type3类型的icmp报文接收数 |      |
   | InType8: 1    | type1类型的icmp报文接收数 |      |
   | OutType0: 1   | type0类型的icmp报文发出数 |      |
   | OutType3: 225 | type3类型的icmp报文发出数 |      |
   | OutType8: 6   | type8类型的icmp报文发出数 |      |

   附：（ICMP类型号含义对应）

   | TYPE | CODE | Description                                            | Query                        | 备注 |
   | ---- | ---- | ------------------------------------------------------ | ---------------------------- | ---- |
   | 0    | 0    | Echo Reply                                             | 回显应答（Ping应答）         |      |
   | 3    | 0    | Network Unreachable                                    | 网络不可达                   |      |
   | 3    | 1    | Host Unreachable                                       | 主机不可达                   |      |
   | 3    | 2    | Protocol Unreachable                                   | 协议不可达                   |      |
   | 3    | 3    | Port Unreachable                                       | 端口不可达                   |      |
   | 3    | 4    | Fragmentation needed but no frag. bit set              | 需要进行分片但设置不分片比特 |      |
   | 3    | 5    | Source routing failed                                  | 源站选路失败                 |      |
   | 3    | 6    | Destination network unknown                            | 目的网络未知                 |      |
   | 3    | 7    | Destination host unknown                               | 目的主机未知                 |      |
   | 3    | 8    | Source host isolated (obsolete)                        | 源主机被隔离（作废不用）     |      |
   | 3    | 9    | Destination network administratively prohibited        | 目的网络被强制禁止           |      |
   | 3    | 10   | Destination host administratively prohibited           | 目的主机被强制禁止           |      |
   | 3    | 11   | Network unreachable for TOS                            | 由于服务类型TOS，网络不可达  |      |
   | 3    | 12   | Host unreachable for TOS                               | 由于服务类型TOS，主机不可达  |      |
   | 3    | 13   | Communication administratively prohibited by filtering | 由于过滤，通信被强制禁止     |      |
   | 3    | 14   | Host precedence violation                              | 主机越权                     |      |
   | 3    | 15   | Precedence cutoff in effect                            | 优先中止生效                 |      |
   | 4    | 0    | Source quench                                          | 源端被关闭（基本流控制）     |      |
   | 5    | 0    | Redirect for network                                   | 对网络重定向                 |      |
   | 5    | 1    | Redirect for host                                      | 对主机重定向                 |      |
   | 5    | 2    | Redirect for TOS and network                           | 对服务类型和网络重定向       |      |
   | 5    | 3    | Redirect for TOS and host                              | 对服务类型和主机重定向       |      |
   | 8    | 0    | Echo request                                           | 回显请求（Ping请求）         |      |
   | 9    | 0    | Router advertisement                                   | 路由器通告                   |      |
   | 10   | 0    | Route solicitation                                     | 路由器请求                   |      |
   | 11   | 0    | TTL equals 0 during transit                            | 传输期间生存时间为0          |      |
   | 11   | 1    | TTL equals 0 during reassembly                         | 在数据报组装期间生存时间为0  |      |
   | 12   | 0    | IP header bad (catchall error)                         | 坏的IP首部（包括各种差错）   |      |
   | 12   | 1    | Required options missing                               | 缺少必需的选项               |      |
   | 13   | 0    | Timestamp request (obsolete)                           | 时间戳请求（作废不用）       |      |
   | 14   |      | Timestamp reply (obsolete)                             | 时间戳应答（作废不用）       |      |
   | 15   | 0    | Information request (obsolete)                         | 信息请求（作废不用）         |      |
   | 16   | 0    | Information reply (obsolete)                           | 信息应答（作废不用）         |      |
   | 17   | 0    | Address mask request                                   | 地址掩码请求                 |      |
   | 18   | 0    | Address mask reply                                     | 地址掩码应答                 |      |

   

4. Tcp

   | 参数                             | 含义               | 备注 |
   | -------------------------------- | ------------------ | ---- |
   | 7004 active connections openings | 主动连接打开数     |      |
   | 9140 passive connection openings | 被动连接打开数     |      |
   | 535 failed connection attempts   | 连接尝试失败数     |      |
   | 139 connection resets received   | 连接重置接收数     |      |
   | 3 connections established        | 连接建立           |      |
   | 753907 segments received         | 数据段接收数       |      |
   | 773481 segments send out         | 数据段发出数       |      |
   | 193 segments retransmited        | 数据段重传数       |      |
   | 0 bad segments received.         | 错误数据段接收数   |      |
   | 730 resets sent                  | 重置（命令）发送数 |      |

   

5. Udp

   | 参数                                 | 含义                                  | 备注 |
   | ------------------------------------ | ------------------------------------- | ---- |
   | 5374 packets received                | UDP数据包接收数                       |      |
   | 96 packets to unknown port received. | 接收到位置端口的数据包数量            |      |
   | 0 packet receive errors              | 数据包接收错误，若不为空说明有UDP丢包 |      |
   | 449 packets sent                     | UDP数据包发送数                       |      |
   | IgnoredMulti: 45905                  | 忽略组播数据包数量                    |      |
   | 0 receive buffer errors              | 因为 UDP 的接收缓存太小导致丢包的数量 |      |
   | 0 send buffer errors                 | 因为 UDP 的发送缓存问题导致丢包的数量 |      |

   

6. UdpLite】

7. TcpExt】

   | 参数                                                         | 含义                                         | 备注                                                         |
   | ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------ |
   | 9086 TCP sockets finished time wait in fast timer            | 完成时间在快速计时器中等待的socket数         | 不确定                                                       |
   | 31875 delayed acks sent                                      | 发出的延迟确认包数                           |                                                              |
   | 1 delayed acks further delayed because of locked socket      | 由于socket阻塞导致的延迟确认包               |                                                              |
   | Quick ack mode was activated 31 times                        | 快速确认模式被激活的次数                     |                                                              |
   | 126242 packet headers predicted                              | 预测数据包头部                               |                                                              |
   | 43860 acknowledgments not containing data payload received   | 接收到的不含payload的确认包                  |                                                              |
   | 170103 predicted acknowledgments                             | 预测确认                                     |                                                              |
   | 17 times recovered from packet loss by selective acknowledgements | 通过选择性确认从数据包丢失中恢复次数         |                                                              |
   | Detected reordering 8 times using SACK                       | 检测到使用SACK重新排序次数                   |                                                              |
   | 9 congestion windows recovered without slow start by DSACK   | 通过dsack恢复拥塞窗口的个数，但没有慢启动    |                                                              |
   | 7 congestion windows recovered without slow start after partial ack | 部分确认后，拥塞窗口恢复的个数，没有慢速启动 |                                                              |
   | 15 fast retransmits                                          | 快速重传次数                                 |                                                              |
   | 132 other TCP timeouts                                       | 其他TCP超时次数                              |                                                              |
   | TCPLossProbes: 55                                            | TCP丢包探测到的次数                          |                                                              |
   | TCPLossProbeRecovery: 4                                      | TCP丢包探测恢复次数                          |                                                              |
   | 31 DSACKs sent for old packets                               | 对就数据包重复选择确认的次数                 |                                                              |
   | 29 DSACKs received                                           | 接收到的重复选择确认次数                     |                                                              |
   | 8 DSACKs for out of order packets received                   | 对接收到的乱序数据包的重复选择确认次数       |                                                              |
   | 95 connections reset due to unexpected data                  | 由于意外数据造成的连接重置次数               |                                                              |
   | TCPDSACKIgnoredNoUndo: 5                                     | DSACK忽略不撤销                              | tcp_sacktag_write_queue(): undo_marker为０并且接收到非法D-SACK块时，加１，即SACK中的序号太旧 |
   | TCPSackShiftFallback: 101                                    | 选择确认移位回退                             | tcp_ack()->tcp_sacktag_write_queue()->tcp_sacktag_walk()->tcp_shift_skb_data()；不支持GSO； prev skb不完全是paged的； SACK的序号已经ACK过，等等 |
   | TCPDeferAcceptDrop: 8309                                     | 由于延迟接受的丢包数                         |                                                              |
   | TCPRcvCoalesce: 8908                                         | TCP接收合并                                  | 不确定                                                       |
   | TCPOFOQueue: 3736                                            | Tcp_ofo_queue大小                            | 不确定                                                       |
   | TCPSpuriousRtxHostQueues: 21                                 | 虚假通讯主机队列大小                         | 不确定                                                       |
   | TCPAutoCorking: 6007                                         | tcp自动阻塞数量                              |                                                              |
   | TCPSynRetrans: 16                                            | tcp同步重传数                                |                                                              |
   | TCPOrigDataSent: 558910                                      | 原始tcp数据发送量                            |                                                              |
   | TCPHystartTrainDetect: 10                                    | 混合慢启动检测                               | 不确定                                                       |
   | TCPHystartTrainCwnd: 196                                     | 混合慢启动拥塞窗口                           | 不确定                                                       |
   | TCPKeepAlive: 2468                                           | tcp长连接数                                  |                                                              |

   

8. IpExt

   | 参数                   | 含义                | 备注 |
   | ---------------------- | ------------------- | ---- |
   | InMcastPkts: 5107      | 接收的组播包数      |      |
   | OutMcastPkts: 65       | 发出的组播包数      |      |
   | InBcastPkts: 45905     | 接收的广播包数      |      |
   | OutBcastPkts: 6        | 发出的广播包数      |      |
   | InOctets: 358455270    | 接收的总字节数      |      |
   | OutOctets: 292538471   | 发出的总字节数      |      |
   | InMcastOctets: 447338  | 接收的组播总字节数  |      |
   | OutMcastOctets: 8511   | 发出的组播总字节数  |      |
   | InBcastOctets: 9311193 | 接收的广播总字节数  |      |
   | OutBcastOctets: 284    | 发出的广播总字节数  |      |
   | InNoECTPkts: 848367    | 接收的非ECT数据包数 |      |
   | InECT0Pkts: 5          | 接收的ECT0数据包数  |      |

   

