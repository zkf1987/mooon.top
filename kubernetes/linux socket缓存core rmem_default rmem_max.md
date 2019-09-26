
```shell
之前讲到缓存大小的配置项:
/proc/sys/net/core/rmem_default
/proc/sys/net/core/rmem_max
 
使用udp接收数据时：
若没有调用setsockopt设置系统接收缓存,则接收缓存的大小为rmem_default.
若程序调用setsockopt设置系统接收缓存,设置值不能超过rmem_max.
 
系统会为每个udp socket申请一份缓存空间,而不是共用同一份缓存.
即每个udp socket都会有一个rmem_default大小的缓存空间(假设没有setsockopt设置).
```

## 测试代码：

```c++
//udp.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <netinet/in.h>
 
#define	SERVER_PORT	 1201
#define MAX_DATA_LEN 1024	//服务端每次发送数据大小
 
#define SERVER_SEND_SLEEP 1000*4	//服务端发送数据的间隔时间
#define CLIENT_RECV_SLEEP 1000*100	//客户端接收数据的间隔时间
 
//全局定义
static int serverfd  = 0;	//服务端UDP句柄
static int sendcount = 0;	//发送数据的次数
 
//服务端发送数据线程
void* thread_server_send(void *arg)
{
	static int threadidx = 0;
	threadidx++;
	char tname[256] = {0};
	snprintf(tname, sizeof(tname), "thread-%d", threadidx);
   	//填充发送数据
   	char data[MAX_DATA_LEN] = {0};
	int i = 0;
	for ( ; i < MAX_DATA_LEN; i++)
	{
		data[i] = '0' + (i % 10);
	}
	//拷贝客户端地址信息
	struct sockaddr_in cliaddr = {0};
	memcpy(&cliaddr, arg, sizeof(cliaddr));
	int printer = 0;
    while(1){  
    	if (sendcount <= 0){
    		usleep(1000*10);
    		if (printer >= 0) printer++; 
    		if (printer>250){
    			printf("%s is alive!\n", tname);
    			printer = -1; //不再打印
    		}
    		continue;
    	}
    	int count = sendcount;
    	sleep(2);//等待,让其他线程也读取到sendcount.
    	sendcount = 0;
    	printf("read new count value:%d\n", count);
		for ( i = 0; i < count; i++)
		{
			int result = sendto(serverfd, data, sizeof(data), 0, (struct sockaddr*)&cliaddr, sizeof(struct sockaddr_in));
			printf("tname:%s fd:%d sendlen:%d client:%s:%u counter:%d\n", tname, serverfd, result, inet_ntoa(cliaddr.sin_addr), ntohs(cliaddr.sin_port), i+1);
			usleep(SERVER_SEND_SLEEP); 
		}	
    }		 
}
 
int exec_udp_server()
{
	printf("######## exec udp server ########\n");
    if( (serverfd = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
    {
        printf("Socket error.. %d\n", errno);
        exit( EXIT_FAILURE );
    }
    struct sockaddr_in servaddr;
    bzero( &servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERVER_PORT);
   	//检查发送缓存大小
   	int buflen = 0;
   	int paramlen = sizeof(buflen);
    int ret = 0;
	//setsockopt(listenfd, SOL_SOCKET, SO_SNDBUF, (char*)&nZero, sizeof(nZero));   
    getsockopt(serverfd, SOL_SOCKET, SO_SNDBUF, &buflen, ¶mlen);
    printf("fd:%d send buffer length:%d result:%d\n", serverfd, buflen, ret); 	 
    getsockopt(serverfd, SOL_SOCKET, SO_RCVBUF, &buflen, ¶mlen);
    printf("fd:%d recv buffer length:%d result:%d\n", serverfd, buflen, ret);       	
  	//绑定端口
    if( bind(serverfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    {
        printf("Bind error... %d\n", errno);
        exit( EXIT_FAILURE );
    }
    printf("udp bind exec\n");
 	//接收客户端数据
 	while(1){
 		//使用说明:
 		//选择1之后,服务器这边会等待客户端数据.此时启动客户端程序,待服务端收到客户端数据后,该步骤才执行完.
 		//选择2之后,需要数据发送次数(每次发送数据为MAX_DATA_LEN),为所有已经连接连接上的客户端共用.
 		printf("menu info\n");
 		printf("1：waiting for client\n");
 		printf("2：ready to send data\n");
 		char input[256] = {0};
		gets(input);
		if (strcmp(input, "1") == 0){
			printf("waiting for client data now!\n");
			char recvbuf[1024] = {0};
			struct sockaddr_in cliaddr = {0};
 			paramlen = sizeof(struct sockaddr_in);
    		ret = recvfrom(serverfd, recvbuf, sizeof(recvbuf), 0, (struct sockaddr*)&cliaddr, ¶mlen);   
			printf("recv client data:%s len:%d\n", recvbuf, ret);
			int thread = 0;
   			pthread_create(&thread, NULL, thread_server_send, &cliaddr);
   			sleep(1);
		}else if (strcmp(input, "2") == 0){
			char input[256] = {0};
			printf("the count you want to send:");//对所有客户端有效
			gets(input);
			sendcount = atoi(input);
			printf("new count value:%d\n", sendcount);
		}
   	}
 
}
 
int exec_udp_client()
{
	printf("######## exec udp client ########\n");
    int	connfd;
    if( (connfd = socket(AF_INET, SOCK_DGRAM, 0)) == -1 )
    {
        printf("Socket error.. %d\n", errno);
        exit( EXIT_FAILURE );
    }
    struct sockaddr_in servaddr;
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = inet_addr("192.168.6.37");
    servaddr.sin_port = htons(SERVER_PORT);
    //检查发送缓存大小
   	int buflen = 0;
   	int paramlen = sizeof(buflen);
    int ret = 0;
    //setsockopt(listenfd, SOL_SOCKET, SO_SNDBUF, (char*)&nZero, sizeof(nZero));   
    getsockopt(connfd, SOL_SOCKET, SO_SNDBUF, &buflen, ¶mlen);
    printf("fd:%d send buffer length:%d result:%d\n", connfd, buflen, ret); 	 
    getsockopt(connfd, SOL_SOCKET, SO_RCVBUF, &buflen, ¶mlen);
    printf("fd:%d recv buffer length:%d result:%d\n", connfd, buflen, ret); 
    //发送数据到服务端
    char sendbuf[1024] = "hello server, start now!";
	int result = sendto(connfd, sendbuf, strlen(sendbuf), 0, (struct sockaddr*)&servaddr, sizeof(struct sockaddr_in));   
	printf("send server data:%s len:%d\n", sendbuf, result); 
    //接收缓存      
   	char data[MAX_DATA_LEN+1] = {0};
   	int counter = 0;
    while(1)
    {
        int len = sizeof(struct sockaddr_in);
        int result = recvfrom(connfd, data, MAX_DATA_LEN, 0, (struct sockaddr*)&servaddr, &len);
    	if (result > 0) counter++;   
        printf("fd:%d recvlen:%d server:%s:%u counter:%d\n", connfd, result, inet_ntoa(servaddr.sin_addr), ntohs(servaddr.sin_port), counter);
        usleep(CLIENT_RECV_SLEEP);
    }
    close(connfd);
}
```
```c
//main.c
//输入参数us表示执行服务端代码.
//输入参数us表示执行客户端代码
 
#include <stdio.h>
#include <stdlib.h>
#include <stdarg.h>
#include <time.h>
#include <string.h>
#include <unistd.h>
 
int main(int argc, char** argv[])
{
	if (strcmp(argv[1], "us") == 0){
		exec_udp_server();
	}else if (strcmp(argv[1], "uc") == 0){
		exec_udp_client();
	}
	return 0;
}
```

```shell
之前讲到缓存大小的配置项:
/proc/sys/net/core/rmem_default
/proc/sys/net/core/rmem_max
 
使用udp接收数据时：
若没有调用setsockopt设置系统接收缓存,则接收缓存的大小为rmem_default.
若程序调用setsockopt设置系统接收缓存,设置值不能超过rmem_max.
 
系统会为每个udp socket申请一份缓存空间,而不是共用同一份缓存.
即每个udp socket都会有一个rmem_default大小的缓存空间(假设没有setsockopt设置).
更新: 
经过测试发现,实际上这个缓存空间能存储的数据大小只有getsockopt&SO_RCVBUF值的一半.具体原因见下面测试过程-1.
 
 
测试过程-1:
1.启动服务端程序,选择菜单-1,等待客户端的连接.
2.设置客户端的接收缓存为16K 
  sysctl -w net.core.rmem_max=16384
  sysctl -w net.core.rmem_default=16384
3.启动客户端程序
4.服务端选择菜单-2,输入16.(即发送16K的数据给客户端)
 
服务端打印:
read new count value:16
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:1
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:2
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:3
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:4
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:5
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:6
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:7
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:8
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:9
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:10
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:11
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:12
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:13
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:14
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:15
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:58402 counter:16
 
客户端端打印:
fd:3 send buffer length:108544 result:0
fd:3 recv buffer length:16384 result:0
send server data:hello server, start now! len:24
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:1
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:2
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:3
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:4
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:5
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:6
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:7
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:8
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:9
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:10
 
可以发现虽然客户端的接收缓存有16K,但实际上并没有存储下16K的数据.而只有10K.
重复上面的测试步骤,把客户端的接收缓存修改成48K,服务端发送48次数据,客户端只收到30K.
测试得出一个结论: upd接收缓存最多能存储数据大小为: SO_RCVBUF * 0.625 (0.625是我测试得出的数据,不同的机器测试结果应该不同)
至于为什么如此? 我也想不明白.
通过不断测试,发现调用命令: sysctl -w net.core.rmem_max=xxxx 设置缓存时,
需要把xxxx设置为我们需要的缓存值的2倍,才能最大程度的保证不丢包.
但如果程序调用setsockopt设置SO_RCVBUF把缓存设置成xxxx,此时无需乘以2倍.
就能实现我们预期的效果.
为什么? 
Sets or gets the maximum socket receive buffer in bytes.  
The kernel doubles this value (to allow space for  book‐keeping  overhead)
when  it is set using setsockopt(2), and this doubled value is returned by getsockopt(2).  
The default value is set by the /proc/sys/net/core/rmem_default file, 
and the maximum allowed  value  is  set  by  the/proc/sys/net/core/rmem_max file.  
The minimum (doubled) value for this option is 256.
可以在程序里调用setsockopt马上调用getsockopt查询下缓存的大小.
 
 
测试过程-2:
1.启动服务端程序,选择菜单-1,等待客户端的连接.
2.设置客户端的接收缓存为16K 
  sysctl -w net.core.rmem_max=16384
  sysctl -w net.core.rmem_default=16384
3.启动客户端程序1
4.服务端程序再次选择菜单-1
5.启动客户端程序2
6.服务端选择菜单-2,输入16.(即发送16K的数据给客户端)
 
服务端打印:
read new count value:16
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:1
read new count value:16
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:1
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:2
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:2
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:3
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:3
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:4
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:4
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:5
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:5
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:6
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:6
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:7
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:7
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:8
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:8
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:9
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:9
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:10
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:10
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:11
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:11
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:12
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:12
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:13
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:13
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:14
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:14
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:15
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:15
tname:thread-2 fd:3 sendlen:1024 client:192.168.6.125:57408 counter:16
tname:thread-1 fd:3 sendlen:1024 client:192.168.6.125:55154 counter:16
 
客户端1打印:                                     
fd:3 send buffer length:108544 result:0                                   
fd:3 recv buffer length:16384 result:0                                    
send server data:hello server, start now! len:24                          
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:1                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:2                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:3                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:4                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:5                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:6                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:7                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:8                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:9                      
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:10
客户端2打印:
fd:3 send buffer length:108544 result:0
fd:3 recv buffer length:16384 result:0
send server data:hello server, start now! len:24
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:1
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:2
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:3
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:4
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:5
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:6
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:7
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:8
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:9
fd:3 recvlen:1024 server:192.168.6.37:1201 counter:10
 
由此可以得出结论:
/proc/sys/net/core/rmem_default
/proc/sys/net/core/rmem_max
每个udp socket都有自己的一份缓存,而不是共用
```