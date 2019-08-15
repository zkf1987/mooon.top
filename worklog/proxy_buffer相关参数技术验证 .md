## 环境准备
nginx   版本为：1.16.0
nginx相关配置参数如下，以下测试中主要对proxy_buffer_size、proxy_buffer_size、
proxy_busy_buffers_size参数进⾏调整，输出相关验证结果。
```nginx
http{
    log_format main '$time_local||$request_method||$request_uri||$status||$upstream_status||$request _time||$upstream_response_time||$upstream_addr||$request_length||$upstream_respo nse_length||$remote_addr'; 
    access_log logs/access.log main; 
    error_log logs/error.log debug;#debug⽇志 
    upstream demo{ 
        server 172.18.65.59:8888 max_fails=0 fail_timeout=0s;
    }
    server { 
        listen 80; 
        server_name 172.18.65.59; 
        proxy_buffer_size 2k;
        proxy_buffers 5 1k;
        proxy_busy_buffers_size 3k; 
        location /demo { 
              proxy_pass http://demo; 
        }
    }
}
```

## upstream
⼀个简单的springboot⼯程，模拟upstream响应，主要对header、body字节数组⻓度进⾏调整，模拟 不同⼤⼩的header、body进⾏技术验证，具体代码如下：
```java
@RestController
@RequestMapping("/demo") 
public class DemoController { 
    @GetMapping("/order") 
    public void list(HttpServletResponse httpServletResponse) { 
        try {
            long start = System.currentTimeMillis(); 
            int _1KB = 1024;//1KB 
            int _1MB = _1KB * _1KB;//1MB 
            //response header⼤⼩
            byte[] header = new byte[1 * _1KB]; 
            //response body⼤⼩ 
            byte[] body = new byte[4 * _1KB]; 
            //设置header 
            httpServletResponse.setHeader("name", new String(header));
             //设置响应体 
             httpServletResponse.getOutputStream().write(body); 
             } catch (IOException e) {
                  e.printStackTrace(); 
            } 
        }
}
```

## 参数意义
nginx官⽹对这个⼏个参数只是简单的描述说明，但并未指明这⼏个参数到底是如何控制upstream的响 应报⽂传输。下⾯通过对各个参数的调整，通过抓包及模拟upstream响应报⽂进⾏分析其意义。
### proxy_buffer_size
此参数主要控制upstream响应头的⼤⼩，若响应头超过此⼤⼩，nginx会打印error⽇志。⽇志如下： 
```nginx
2019/08/13 16:27:46 [error] 28493#0: *178 upstream sent too big header while reading response header from upstream, client: 172.18.65.59, server: 172.18.65.59, request: "GET /demo/order HTTP/1.1", upstream: "http://172.18.65.59:8888/demo/order", host: "172.18.65.59"
```
### 当proxy_buffer_size⼩于upsteam的header
相关配置及代码如下，已省略⽆关配置及代码： 
```java
proxy_buffer_size 1k;#设置header⼤⼩限制为1KB 
proxy_buffers 5 2k; 
proxy_busy_buffers_size 3k;
```
```java
int _1KB = 1024; 
int _1MB = _1KB * _1KB; 
byte[] header = new byte[1 * _1KB]; 
byte[] body = new byte[4 * _1KB]; 
httpServletResponse.setHeader("name", new String(header));//发送header⼤⼩为1KB 
httpServletResponse.getOutputStream().write(body);
```
虽然header中key为name的⼤⼩为1KB，但总的header会略⼤于1KB的，因为还有⼀些http标准响应 头。当请求nginx时，nginx相关⽇志如下：
### error.log
```nginx
2019/08/13 17:02:50 [error] 31979#0: *185 upstream sent too big header while reading response header from upstream, client: 172.18.65.59, server: 172.18.65.59, request: "GET /demo/order HTTP/1.1", upstream: "http://172.18.65.59:8888/demo/order", host: "172.18.65.59"
```
### access.log
```nginx
13/Aug/2019:17:02:50 +0800||GET||/demo/order||502||502||0.003||0.000 ||172.18.65.59:8888||230||0||172.18.65.59
```
### wireshark
![1.png](https://i.loli.net/2019/08/15/h2CyflUM1b849ZH.png)
### 当proxy_buffer_size⼤于upsteam的header
调整nginx的proxy_buffer_size为2k,upstram代码不变。
```nginx
proxy_buffer_size 2k;#设置header⼤⼩限制为2KB 
proxy_buffers 5 2k; 
proxy_busy_buffers_size 3k;
```
### error.log
⽆任何⽇志。
### access.log
```nginx
13/Aug/2019:17:08:52 +0800||GET||/demo/order||200||200||0.004||0.004 ||172.18.65.59:8888||230||4096||172.18.65.59
```
### wireshark
![2.png](https://i.loli.net/2019/08/15/fgYXsLME91ktuG2.png)
## 结论
从以上结果可以看到，当upstream的响应头⼤于proxy_buffer_size时，nginx会输出error⽇志，同 时返回502状态码给client。当*upstream的响应头⼩于proxy_buffer_size时，nginx能正常返回给
client


## proxy_busy_buffers_size
⽤于限制nginx单次返回client数据包⼤⼩，此参数建议为偶数。
### 参数设置规则
在设置此参数时，与proxy_buffer_size、proxy_buffers⼤⼩关系，配置关系不正确，⽆法reload⽣ 效。

proxy_busy_buffers_size⼤于等于proxy_buffer_size的值，同时也必须⼩于proxy_buffers第2个参数 的值，也就是单个buffer空间⼤⼩。 
并且
 proxy_busy_buffers_size ⼩于 proxy_buffers中(number-1)*size的⼤⼩。
 > 务必按照此规则配置proxy_busy_buffers_size的值。

 ### proxy_busy_buffers_size值验证
 调整proxy_busy_buffers_size=2K，upstream响应报⽂为4MB
 ```nginx
 proxy_buffer_size 2k; 
 proxy_buffers 5 2k; 
 proxy_busy_buffers_size 2k;
 ```
 ```nginx
 int _1KB = 1024; 
 int _1MB = _1KB * _1KB; 
 byte[] header = new byte[1 * _1KB]; 
 byte[] body = new byte[4 * _1MB]; 
 httpServletResponse.setHeader("name", new String(header)); 
 httpServletResponse.getOutputStream().write(body);
 ```
 wireshark抓包如下，可以看到nginx每个包响应client的⼤⼩为2055，因为还要包含⼀些协议本⾝的报 ⽂，所以略⼤于2048

![3png.png](https://i.loli.net/2019/08/15/osUryE6SFQ1CDxu.png)
 调整proxy_busy_buffers_size=3K，upstream响应报⽂⼤⼩不变，nginx每个包响应⼤⼩为2055
![4.png](https://i.loli.net/2019/08/15/kACSBtTh5VyPZua.png)
 调整proxy_busy_buffers_size=4K，upstream响应报⽂⼤⼩不变，nginx每个包响应⼤⼩为4104
![5.png](https://i.loli.net/2019/08/15/LixPkjV3JwQqEWm.png)
 调整proxy_busy_buffers_size=5K，upstream响应报⽂⼤⼩不变，nginx每个包响应⼤⼩为4104
![6.png](https://i.loli.net/2019/08/15/IqyfTAxB4QNZk96.png)
 调整proxy_busy_buffers_size=6K，upstream响应报⽂⼤⼩不变，nginx每个包响应⼤⼩为6152
![7.png](https://i.loli.net/2019/08/15/BlRWaJcAModDOeU.png)
 后续proxy_busy_buffers_size依次增加1K进⾏测试，发现只有偶数时才⽣效，当为奇数n时，单次响应 数据包⼤⼩为n-1

 ### 结论
 proxy_busy_buffers_size参数主要限制nginx每次返回给client数据包⼤⼩，且此参数应设置为偶 数。但此参数不能调太⼤，因为如果太⼤了，当响应报⽂过⼤时，需要达到此缓存窗⼝后才会向client
发送，所以会影响耗时。经过调整此参数⼤⼩，在保证不出现warn的情况下，发现默认8K缓冲区⼤ ⼩，要⽐200K缓冲区响应快点。

## proxy_buffers
在proxy_buffering 开启的情况下，nginx将会尽可能的读取所有的upstream端传输的数据到
proxy_buffers，直到proxy_buffers被写满或者数据被读取完(EOF)。此时nginx开始向客户端传输数 据，将整个buffers中的数据返回给client。如果response的内容很⼤的话，nginx会接收并把他们写⼊
到temp_file⾥去。⼤⼩由proxy_max_temp_file_size控制。如果busy的buffer传输完了会从temp_file
⾥⾯接着读数据，直到传输完毕。

 此参数是动态的，因为proxy_buffers会不断的填充、清空，主要取决于client的ack速度，或者说是
client的响应速度，这取决于client端tcp的read_buffer⼤⼩。如果要测试精确的upstream响应报⽂⻓ 度，只有⼿写⼀个tcp客户端，控制客户端不进⾏read,直到client的read_buffer填满，才可以精确看到 这个值。

 我们将upstream的响应体增⼤到20M，可以很容易看到buffers被打满，往temp_file中写。
```nginx
 2019/08/13 20:59:27 [warn] 7860#0: *229 an upstream response is buffered to a temporary file /home/zhuzh/middle/nginx/proxy_temp/9/01/0000000019 while reading upstream, client: 172.18.65.59, server: 172.18.65.59, request: "GET /demo/order HTTP/1.1", upstream: "http://172.18.65.59:8888/demo/order", host: "172.18.65.59"
 
 
 13/Aug/2019:20:59:27 +0800||GET||/demo/order||200||200||0.186||0.176 ||172.18.65.59:8888||230||20971520||172.18.65.59
```
![8.png](https://i.loli.net/2019/08/15/wq4mB1dRazeVQGi.png)
![9.png](https://i.loli.net/2019/08/15/Hr5ZqhGCmlUvEAL.png)
 经过多次测试可以看到，当nginx出现warn⽇志时，通过wireshark抓包看到，client⽆法快速响应
(ack)nginx发出的数据包,同时tcp滑动窗⼝已被打满。也就是说当往temp_file中写响应报⽂时，说明
upstream的响应体已经⼤到缓存区⽆法承载，另外即使没有出现warn，缓存区也可能已多次被
upstream填满。扩⼤缓冲区只是可以规避这个warn，但本质问题还是upstream单次响应过⼤，⽽且 ⼀味的调⼤buffers也存在⼀定的数据延迟，因为只有buffers满时，nginx才会往client发数据，类似与 线程池的阻塞队列⼤⼩。所以此参数⼀定要设置的合适，⽽不是过⼤或过⼩。
```
### 结论
当调⼤此参数时，warn⽇志没有了，响应时间也短了，因为不⾛⽂件IO了，直接⾛内存。

## 如何解决容器nginx的问题
1. 根据容器中error.log的⽇志，an upstream response is buffered to a temporary file，可以确定 此错误是由于upstream响应体过⼤出现的问题，所以与proxy_buffer_size参数⽆关，建议此参数 保持默认值不变，也可以通过getconf PAGE_SIZE命令查看系统内存⻚⼤⼩，进⾏显⽰设置。 
2. 在nginx的log_format中添加$upstream_response_length，获取upstream实际响应体⼤⼩，同 时调整proxy_buffers的值，建议第2个参数维持系统内存⻚⼤⼩，只调整number的个数，然后进 ⾏测试，达到不出现warn告警的最⼩值为合适值。
 3. 调整proxy_busy_buffers_size的值，建议保持跟proxy_buffer_size的值⼀致，也就是⼀个内存⻚ ⼤⼩，因为此参数控制nginx单次响应client的⼤⼩。但根据tcp/ip协议栈，mtu跟mss的关系，tcp的最⼤报⽂⻓度也就是1460。⽆论单次nginx响应client的报⽂有多⼤，最终在传说时也会进⾏分 ⽚。另外上⾯已经提到过，数值太⼤时，如果upstream的响应报⽂过⼤，需要堆积够proxy_busy_buffers_size,nginx才会发送给client，必然会导致⼀定的延迟。此问题的根本问题还 是在upstream的响应体过⼤，所以最直接的解决办法还是业务⽅的接⼝响应的内容是啥，是否可以改造为多次请求，⽽不是⼀次返回这么多数据。

 >MTU(Maximum Transmission Unit) - ⼀种通信协议的某⼀层上⾯所能通过的最⼤数据包⼤ ⼩.以太⽹为 1500，这也是为何路由器或 PC 上默认都是 1500，因此数据包⼀旦超过此⼤ ⼩，就会进⾏分包，这也就有了 IP 分⽚的过程。
>MSS就是TCP数据包每次能够传输的最⼤数据分段。为了达到最佳的传输效能 TCP协议在建⽴连接的时候通常要协商双⽅的MSS值，这个值TCP协议在实现的 时候往往⽤MTU值代替（需要减去IP数据包包头的⼤⼩20Bytes和TCP数据段的 包头20Bytes）所以往往MSS为1460。通讯双⽅会根据双⽅提供的MSS值得最⼩ 值确定为这次连接的最⼤MSS值。