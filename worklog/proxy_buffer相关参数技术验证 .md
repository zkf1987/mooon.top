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
