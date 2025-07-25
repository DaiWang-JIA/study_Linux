<center> linux网络编程</center>

# 套接字通信基础

## 概念

- 网络设计模式
  - B/S
    - broswer-浏览器->客户端
      - html
      - css
      - js
    - server->服务器
    - 优势：
      1. 跨平台
      2. 开发成本低
    - 缺点：
      1. 网络通信时，必须要使用http协议
         - HTTP/HTTPS（加密）->应用层协议
      2. 不能在磁盘缓存或者从磁盘加载大量数据
  - C/S
    - client->桌面应用程序 （QQ，微信，迅雷）->QT
    - server->服务器
    - 特点：
      1. 优点：
         - 使用的协议可以随意选择
         - 可以在本地缓存或者加载大量数据
      2. 缺点：
         - 研发成本高-不同平台不同的客户端版本

- 服务器：

  - 硬件：配置比较高的主机
    - 买阿里云，百度云服务器
  - 软件：有一台主机，主机上运行了一个进程，该进程可以处理网络协议，称这台主机是一个服务器
    - nginx
  - 服务器开发：
    - 工作不是去开发web服务器
    - 我们做的工作：
      - 在一台装有服务器的主机上开发应用程序
        - 斗地主
        - 文件服务器

- IP和端口

  - IP地址分类

    - 公网IP
      - 可以访问Interface，公网IP是唯一的
    - 局域网IP
      - 小的网络，比如路由器对应家里的网段
        - 192.168.6.xxxx
      - 在这个最小的网络中IP是唯一的
      - 在这个网络中的主机可以相互通信
      - 如果局域网和外网连接，那么通过局域网也可以连接外网

  - IP协议

    - IPV4-"Internet Protocol Version 4"

      - 现在应用很广泛
      - IP地址的点分十进制字符串
        - 192.168.1.100
      - 本质是整型数，4字节，32位
        - 通过3个点分成4分，每份一个字节
          - 字节取值范围 0-255
          - 最大IP地址：255.255.255.255
      - 可用IP地址：
        - 2的32次方-1
      - IPv4非常不够用

    - IPV6-"Internet Protocol Version 6"

      - 将来主推的一种协议

      - 本质也是整型数 16字节 128 位（bit）

      - IP地址表示：

        - 分为8分，每份2字节，使用16进制数字表示

        - ```shell
          fe80::20d4:1a84:6918:546a
          ```

        - ```shell
          fe80:0000:0000:0000:20d4:1a84:6918:546a
          ```

      - Linux相关的命令

      - ```shell
        #linux
        $ ifconfig
        # windows
        $ ipconfig
      
      - ![image-20250703142706481](Linux网络编程.assets/image-20250703142706481.png)
      
      - 只要是192.168.x.x格式都是局域网IP
      
      - 公网IP：11.13.35.67
      
      - 测试：ping 域名  / ping IP地址
      
      - IP地址个数：
      
        - 2的128次方-1
  
- 端口

  ```shell
  #1.端口本质
  无符号短整型数->unsigned short
  # 2.端口的取值范围
  1.可以有多少端口：2的16次方 
  2.取值范围：0-65535
  # 3.端口的作用
  定位某台主机上运行的进程
  # 4. 如何给自己编写的应用程序分配端口
  -0 端口一般不用
  - 1-1024：系统使用
  - 1014-5000：系统预装的程序/一些常用的协议使用
  - 5001-65525：留给用于自定义的端口
  # 5.所有的程序都要有端口吗？
  如果进程不通信则不需要
  如果通信就需要给程序进程绑定端口
       -bind()
  ```

- IP和端口的使用

- ```shell
  # 通过IP定位网络环境中的主机，通过端口定位主机上的进程
  # 比如通过浏览器进行网络服务器访问
  # 服务器有IP地址/域名，服务器需要绑定端口
  # b/s ->协议http ->使用的端口是80,https协议端口->443
  # 如果服务器使用的就是协议的默认端口，访问时候端口可以省略不写
  http://www.baidu.com:80
  https://www.baidu.com:443
  
  http://192.168.1.100:9999
  
  # 域名：特殊的字符串，通过字符串可以访问到web服务器了
  # 为什么？
      - 域名需要先申请。然后需要和web服务器的外网IP绑定
      - 绑定成功之后，才能通过域名访问到web服务器
      - IP为什么要和域名关联？->方便记忆网站地址
      - 通过域名取访问web服务器->连接dns服务器->查询域名和IP的对应关系->得到了域名对应的IP地址->通过IP地址访问web服务器
      IP地址->通过IP地址访问web服务器



- OSI/ISO 网络分层模型

![image-20250703153154905](Linux网络编程.assets/image-20250703153154905.png)



- 网络协议是什么

![image-20250703153518341](Linux网络编程.assets/image-20250703153518341.png)



- 数据在网络环境中的发送和接受过程

![image-20250703172027920](Linux网络编程.assets/image-20250703172027920.png)

![image-20250703172636524](Linux网络编程.assets/image-20250703172636524.png)

==程序员只需要处理应用层，应用层以下不需要我们处理==



- **==套接字通信==**

![image-20250703172941091](Linux网络编程.assets/image-20250703172941091.png)

```c
# 套接字通信就是网络通信，跟语言无关，因为通信是基于协议的，所有的编程都需要基于协议对数据进行处理
// 1.套接字是什么
1.套接字就是一套网络通信接口，这个接口封装了传输层协议（tcp/udp)
2.接口就是api,就是一套函数

//2.socket（插座），套接字通信组成部分？
1.服务器端，插座的作用
        -被动接受的角色，不会主动发起连接的
        -绑定固定IP和端口，这样客户端才能连接

2.客户端
         -主动连接服务器
         -连接服务器需要地址：
               -IP+端口
               
//3.怎么用？  ->所有编程语言的通信流程都是固定的
    -服务器有通信流程
    -客户端有通信流程
    
// 4.套接字通信过程中数据要使用网络字节序
字节序：字符在内存中的存储顺序，单位是字节，char类型不需要研究字节序问题
    字符串没有字节序问题
字节序的分类：
      - 网络字节序
      - 主机字节序
    
   char -> 1字节
   int ->  4字节
 
```



- 字节序

![image-20250703174610924](Linux网络编程.assets/image-20250703174610924.png)

 - 概念：
   - little-endian
     - 小端，也称之为主机字节序
     - 在内存的低地址位，存储数据的低位字节，在内存的高地址位存储数据的高位字节
       - 低低高高
     - pc机都是小端存储，跟CPU架构有关
   - big-endian
     - 大端，也称之为网络字节序
     - 在内存的低地址位，存储数据的高位字节，在内存的高地址位存储数据的低位字节
     - 网络通信时，要使用大端字节序

- 大小端存储举例

![image-20250703180544506](Linux网络编程.assets/image-20250703180544506.png)



## IP和端口大小端转换函数

- 函数

![image-20250703181600416](Linux网络编程.assets/image-20250703181600416.png)

- 字符串类型IP地址转换

1. int inet_pton()  主机字节序的字符串类型IP->大端的整型数

![image-20250703182439583](Linux网络编程.assets/image-20250703182439583.png)

2. const char * inet_ntop()   大端的整型数->主机字节序的字符串类型IP

![image-20250703182925407](Linux网络编程.assets/image-20250703182925407.png)





# TCP通信

## TCP特点

![image-20250703185844541](Linux网络编程.assets/image-20250703185844541.png)



## 套接字通信里面的文件描述符结构

![image-20250703190216729](Linux网络编程.assets/image-20250703190216729.png)

![image-20250703191113843](Linux网络编程.assets/image-20250703191113843.png)



## 套接字通信服务端的通信流程

```c
/*
	在服务器端有两类文件描述符：
		1.监听的
		- 检测有没有新的客户端连接服务器
		-服务器端一个就够了
		2.通信的
		-负责和建立连接的客户端通信
		-和多少个客户端建立了连接，就有多少个通信文件描述符
*/

//1.创建一个用于监听的套接字，这个套接字就是一个文件描述符
//   类似管道中的文件描述符，对应是内核中的内存。通过文件描述符操作就可以写读内核中的内存数据
int lfd=socket();

//2.让监听的文件描述符和本地IP+端口进行绑定，为了让客户端找到服务器
//     绑定成功之后，lfd就可以检测到有没有客户端连接请求了
bind();

//3.g给绑定成功的套接字设置监听
listen();

//4.等待并接受客户端连接，得到一个新的用于通信的文件描述符
int cfd=accept();

//5.使用accept返回值对应的通信文件描述符和客户端通信
//接受数据
read();
recv();

//发送数据
write();
send();

//6.断开连接，关闭文件描述符
//关闭通信的文件描述符，也能关闭监听的文件描述符
close();

```



## 基于TCP的客户端的通信流程

```c
//在TCP的客户端文件描述符只有一种：通信的文件描述符

//1.创建用于通信的套接字==（文件描述符）
int fd=socket();

//2.使用创建的通信文件描述符连接服务器，通过服务器绑定的IP和端口进行连接
connect();

//3.连接成功之后，通信
//接受数据
read();
recv();

//发送数据
write();
send();

//4.断开和服务器连接
close();

```



## 文件描述符对应的内核缓冲区和读写操作之间的关系

![image-20250709194516795](Linux网络编程.assets/image-20250709194516795.png)

![image-20250709195749190](Linux网络编程.assets/image-20250709195749190.png)

![image-20250709195722643](Linux网络编程.assets/image-20250709195722643.png)



## 创建套接字函数

```
// 创建一个套接字（文件描述符），用于通信和监听都可以
int socket(int domain,int type,int protocol);
参数：
-domain：
 -AF_INET:使用ipv4网络协议
 -AF_INET6:使用ipv6网络协议
 
-type:
 -SOCK_STREAM:使用流式传输协议
 -SOCK_DGRAM:使用报式传输协议
 
-protacol:默认写0
  -流式协议，默认使用TCP
  -报式协议，默认使用udp
  
返回值：
成功：返回一个文件描述符
失败：-1
```



## 绑定函数

![image-20250710212723834](Linux网络编程.assets/image-20250710212723834.png)

![image-20250710201944837](Linux网络编程.assets/image-20250710201944837.png)

![image-20250710202159681](Linux网络编程.assets/image-20250710202159681.png)



## 设置监听

![image-20250710202620045](Linux网络编程.assets/image-20250710202620045.png)



## 等待并接受客户端连接请求-accept（）函数

![image-20250710203246694](Linux网络编程.assets/image-20250710203246694.png)



## 接收数据-read(),recv()函数

![image-20250710204534822](Linux网络编程.assets/image-20250710204534822.png)



## 发送数据和连接函数

![image-20250710205926117](Linux网络编程.assets/image-20250710205926117.png)



## IP和端口需要使用的网络字节序

 绑定时IP和端口都需要使用大端转换

连接时也是大端（网络字节序）





## 基于tcp的服务器端程序

![image-20250710215056680](Linux网络编程.assets/image-20250710215056680.png)

![image-20250710215124485](Linux网络编程.assets/image-20250710215124485.png)

![image-20250710215144029](Linux网络编程.assets/image-20250710215144029.png)



## 基于tcp的客户端实现

![image-20250710222147013](Linux网络编程.assets/image-20250710222147013.png)

![image-20250710222219187](Linux网络编程.assets/image-20250710222219187.png)









# 三次握手四次挥手

## 知识点概述



## tcp协议

![image-20250711081415749](Linux网络编程.assets/image-20250711081415749.png)

![image-20250711082223382](Linux网络编程.assets/image-20250711082223382.png)



## 三次握手过程

![image-20250711083011848](Linux网络编程.assets/image-20250711083011848.png)

![image-20250711082533314](Linux网络编程.assets/image-20250711082533314.png)

![image-20250711085041016](Linux网络编程.assets/image-20250711085041016.png)

![image-20250711085241638](Linux网络编程.assets/image-20250711085241638.png)

![image-20250711085526855](Linux网络编程.assets/image-20250711085526855.png)



## 四次挥手过程

![image-20250711140948864](Linux网络编程.assets/image-20250711140948864.png)

![image-20250711141001386](Linux网络编程.assets/image-20250711141001386.png)

![image-20250711142537315](Linux网络编程.assets/image-20250711142537315.png)

![image-20250711143128472](Linux网络编程.assets/image-20250711143128472.png)



## TCP滑动窗口

![image-20250711144405558](Linux网络编程.assets/image-20250711144405558.png)

## 滑动窗口如何控制发送端阻塞

![image-20250711162051227](Linux网络编程.assets/image-20250711162051227.png)

![image-20250711162821028](Linux网络编程.assets/image-20250711162821028.png)



## TCP通信关键字

![image-20250711162916372](Linux网络编程.assets/image-20250711162916372.png)

![image-20250711163205029](Linux网络编程.assets/image-20250711163205029.png)



## tcp通信的全部过程分析

![image-20250711163325748](Linux网络编程.assets/image-20250711163325748.png)

![image-20250711181056166](Linux网络编程.assets/image-20250711181056166.png)

![image-20250711181317717](Linux网络编程.assets/image-20250711181317717.png)

![image-20250711182105527](Linux网络编程.assets/image-20250711182105527.png)

![image-20250711183240416](Linux网络编程.assets/image-20250711183240416.png)

![image-20250711184140635](Linux网络编程.assets/image-20250711184140635.png)









# 套接字并发服务器



## 上面套接字服务器的弊端

**无法处理多个客户端**



## 如何通过多进程的方式完成服务器端的并发

![image-20250712091359001](Linux网络编程.assets/image-20250712091359001.png)

- 多进程版的服务器开发

![image-20250712161018091](Linux网络编程.assets/image-20250712161018091.png)

![image-20250712091525849](Linux网络编程.assets/image-20250712091525849.png)

## 多进程服务器

![image-20250712161811577](Linux网络编程.assets/image-20250712161811577.png)

![image-20250712161846194](Linux网络编程.assets/image-20250712161846194.png)

![image-20250712161912474](Linux网络编程.assets/image-20250712161912474.png)

![image-20250712161942976](Linux网络编程.assets/image-20250712161942976.png)



## 多线程版tcp服务器思路处理

![image-20250712164422789](Linux网络编程.assets/image-20250712164422789.png)

![image-20250712164505473](Linux网络编程.assets/image-20250712164505473.png)

![image-20250712203440785](Linux网络编程.assets/image-20250712203440785.png)

![image-20250712203504925](Linux网络编程.assets/image-20250712203504925.png)

![image-20250712203528110](Linux网络编程.assets/image-20250712203528110.png)

![image-20250712203547247](Linux网络编程.assets/image-20250712203547247.png)







# TCP状态转换

![image-20250712212530019](Linux网络编程.assets/image-20250712212530019.png)

## 三次握手状态变化

```c
//tcp通信状态的变化
///////////三次握手////////////
//先启动服务器->绑定->设置监听 listen()
  服务器的状态变化： 无状态->LISTEN
//状态变化是从三次握手开始的（客户端发起连接）
第一次握手：
      客户端：给服务端发送SYN，无状态->SYN_SENT
      服务器：接受数据：LISTEN
      
第二次握手：
      服务器：给客户端回复ACK，并发送连接请求SYN，状态：LISTEN->SYN_RVCD
      客户端:受到ACK，状态：SYN_SENT->ESTABLISED
 
第三次握手：
       客户端：回复ACK，状态没有变化
       服务器：收到ACK，状态：SYN_RVED-ESTABLISED
      
```

- 在双向连接建立之后，通信过程，tcp状态不会发生变化



## 四次挥手状态变化

```c
///////四次挥手////////
第一次挥手:
	客户端：
	1.调用了close（）函数，相当于在tcp协议中将FIN设为1
	2.状态变化：ESTABLISED->FIN_WAIT_1;
	服务器：
	状态无变化：ESTABLISED
	
第二次挥手：
	服务器：收到FIN，回复ACK，状态：ESTABLISED->CLOSE_WAIT
	客户端：收到ACK，状态变化：FIN_WAIT_1->FIN_WAIT_2
	
第三次挥手：
	服务器：给客户端发生断开连接请求，FIN设为1，状态变化：CLOSE_WAIT->LAST_ACK
	客户端：收到服务器断开连接的请求FIN,状态变化：FIN_WAIT_2->TIME_WAIT
	
第四次挥手：
	客户端：回复ACK，状态没变
        -TIME_WAIT会持续一段时间，时间到达之后，周期结束
	服务器：收到ACK，LAST_ACK->无状态
	
	
```



## 处于TIME_WAIT的进程等待2MSL的原因

![image-20250713142825111](Linux网络编程.assets/image-20250713142825111.png)

![image-20250713142804667](Linux网络编程.assets/image-20250713142804667.png)



## 半关闭

![image-20250713145408951](Linux网络编程.assets/image-20250713145408951.png)

## 半关闭函数

![image-20250713145800630](Linux网络编程.assets/image-20250713145800630.png)



## 通过netstat命令查看进程的网络通信

![image-20250713151227467](Linux网络编程.assets/image-20250713151227467.png)





## 端口复用

![image-20250713161136405](Linux网络编程.assets/image-20250713161136405.png)

![image-20250713161952938](Linux网络编程.assets/image-20250713161952938.png)









# IO多路转接之select



## IO多路转接

![image-20250714150110962](Linux网络编程.assets/image-20250714150110962.png)

![image-20250714150325601](Linux网络编程.assets/image-20250714150325601.png)

![image-20250714150838262](Linux网络编程.assets/image-20250714150838262.png)

![image-20250714151124897](Linux网络编程.assets/image-20250714151124897.png)



## select

![image-20250714151158820](Linux网络编程.assets/image-20250714151158820.png)

![image-20250714173331070](Linux网络编程.assets/image-20250714173331070.png)==**函数细节**==

![image-20250716123821599](Linux网络编程.assets/image-20250716123821599.png)

![image-20250716151932300](Linux网络编程.assets/image-20250716151932300.png)




- 文件描述符集合操作函数

```c
 //fd_set类型数据操作函数
//将文件描述符fd从set集合中删除
void FD_CLR(int fd, fd_set *set);

//判断文件描述符fd是不是在set集合中，如果在返回1，如果不在返回0
int  FD_ISSET(int fd, fd_set *set);

//将文件描述符fd添加到set集合中
void FD_SET(int fd, fd_set *set);

//清空set中设置的所有数值，用于初始化
void FD_ZERO(fd_set *set);

```





## select的fd_set和文件描述符表关系

![image-20250716153723927](Linux网络编程.assets/image-20250716153723927.png)

![image-20250716154727360](Linux网络编程.assets/image-20250716154727360.png)



## 使用select处理服务端通信

![image-20250716163137428](Linux网络编程.assets/image-20250716163137428.png)

![image-20250716163212870](Linux网络编程.assets/image-20250716163212870.png)







# IO多路转接之epoll



## poll

![image-20250717215016449](Linux网络编程.assets/image-20250717215016449.png)



- 函数

![image-20250718191348653](Linux网络编程.assets/image-20250718191348653.png)

![image-20250718192050761](Linux网络编程.assets/image-20250718192050761.png)



- 部分代码

![image-20250718192250408](Linux网络编程.assets/image-20250718192250408.png)





## epoll

![image-20250718193058051](Linux网络编程.assets/image-20250718193058051.png)

**三者比较：**

![image-20250718193734598](Linux网络编程.assets/image-20250718193734598.png)

![image-20250718194447395](Linux网络编程.assets/image-20250718194447395.png)

![image-20250718193800658](Linux网络编程.assets/image-20250718193800658.png)

![image-20250718194604185](Linux网络编程.assets/image-20250718194604185.png)



## epoll的使用

![image-20250718194657504](Linux网络编程.assets/image-20250718194657504.png)



- 函数

 ![image-20250718200209663](Linux网络编程.assets/image-20250718200209663.png)

![image-20250718201027893](Linux网络编程.assets/image-20250718201027893.png)

![image-20250718201206968](Linux网络编程.assets/image-20250718201206968.png)



## epoll的检测函数-epoll_wait()

![image-20250718202441090](Linux网络编程.assets/image-20250718202441090.png)





## 基于epoll的tcp服务器的伪代码

![image-20250718203808567](Linux网络编程.assets/image-20250718203808567.png)

![image-20250718203827065](Linux网络编程.assets/image-20250718203827065.png)

![image-20250718203948979](Linux网络编程.assets/image-20250718203948979.png)

 



## epoll的水平模式

![image-20250718211356689](Linux网络编程.assets/image-20250718211356689.png)

- LT模式：通知频率高

![image-20250718212002245](Linux网络编程.assets/image-20250718212002245.png)





## epoll的边沿模式

![image-20250718213022853](Linux网络编程.assets/image-20250718213022853.png)



- ET模式

![image-20250718214150785](Linux网络编程.assets/image-20250718214150785.png)

- 如何设置边沿模式

![image-20250718214836631](Linux网络编程.assets/image-20250718214836631.png)



![image-20250718220950328](Linux网络编程.assets/image-20250718220950328.png)



![image-20250718222349326](Linux网络编程.assets/image-20250718222349326.png)



![image-20250718225051160](Linux网络编程.assets/image-20250718225051160.png)









# udp通信

## udp特点

![image-20250719194736486](Linux网络编程.assets/image-20250719194736486.png)

![image-20250719154626180](Linux网络编程.assets/image-20250719154626180.png)



## udp通信流程

![image-20250719193105063](Linux网络编程.assets/image-20250719193105063.png)

- 服务器端

![image-20250719193334968](Linux网络编程.assets/image-20250719193334968.png)



- 客户端

![image-20250719193501417](Linux网络编程.assets/image-20250719193501417.png)



## sendto和recvfrom函数

- 发送数据函数

![image-20250719195440611](Linux网络编程.assets/image-20250719195440611.png)



- 接收函数

![image-20250719194119372](Linux网络编程.assets/image-20250719194119372.png)

==接收和发送数据的函数默认是阻塞的==



## udp服务器程序代码

![image-20250719202545910](Linux网络编程.assets/image-20250719202545910.png)

![image-20250719202609378](Linux网络编程.assets/image-20250719202609378.png)



## udp客户端代码

![image-20250719204843535](Linux网络编程.assets/image-20250719204843535.png)

![image-20250719204933677](Linux网络编程.assets/image-20250719204933677.png)



## udp应用场景

![image-20250720074057257](Linux网络编程.assets/image-20250720074057257.png)



## 广播

![image-20250720080921029](Linux网络编程.assets/image-20250720080921029.png)



- 特点

![image-20250720080819035](Linux网络编程.assets/image-20250720080819035.png)

![image-20250720082527521](Linux网络编程.assets/image-20250720082527521.png)



- 通信流程

![image-20250720081337897](Linux网络编程.assets/image-20250720081337897.png)

![image-20250720081503082](Linux网络编程.assets/image-20250720081503082.png)



- 设置广播属性

![image-20250720095521397](Linux网络编程.assets/image-20250720095521397.png)



- 数据发送端代码

![image-20250720100016087](Linux网络编程.assets/image-20250720100016087.png)



- 数据接收端代码

![image-20250720100104730](Linux网络编程.assets/image-20250720100104730.png)

![image-20250720100133226](Linux网络编程.assets/image-20250720100133226.png)

![image-20250720101807147](Linux网络编程.assets/image-20250720101807147.png)



## 组播（多播）

![image-20250720110300747](Linux网络编程.assets/image-20250720110300747.png)

![image-20250720095245942](Linux网络编程.assets/image-20250720095245942.png)





- 组播地址

![image-20250720105107331](Linux网络编程.assets/image-20250720105107331.png)



- 组播通信流程

![image-20250720110205256](Linux网络编程.assets/image-20250720110205256.png)

![image-20250720110151156](Linux网络编程.assets/image-20250720110151156.png)



- 设置组播属性

![image-20250720114900459](Linux网络编程.assets/image-20250720114900459.png)



- 加入到组播地址

![image-20250720115344963](Linux网络编程.assets/image-20250720115344963.png)

![image-20250720115650838](Linux网络编程.assets/image-20250720115650838.png)

![image-20250720115220400](Linux网络编程.assets/image-20250720115220400.png)



- 组播特点总结

![image-20250720121908402](Linux网络编程.assets/image-20250720121908402.png)





# 本地套接字

 ![image-20250721082331788](Linux网络编程.assets/image-20250721082331788.png)



## 通信流程-基于tcp

![image-20250721083151489](Linux网络编程.assets/image-20250721083151489.png)



- 服务器端

![image-20250721083506036](Linux网络编程.assets/image-20250721083506036.png)

![image-20250721083418005](Linux网络编程.assets/image-20250721083418005.png)



- 客户端

![image-20250721095559577](Linux网络编程.assets/image-20250721095559577.png)



## 代码

- 服务器端

![image-20250721095944875](Linux网络编程.assets/image-20250721095944875.png)

![image-20250721100104889](Linux网络编程.assets/image-20250721100104889.png)

![image-20250721100212140](Linux网络编程.assets/image-20250721100212140.png)

![image-20250721100346116](Linux网络编程.assets/image-20250721100346116.png)

![image-20250721100427488](Linux网络编程.assets/image-20250721100427488.png)



- 客户端

![image-20250721100708538](Linux网络编程.assets/image-20250721100708538.png)

![image-20250721102059526](Linux网络编程.assets/image-20250721102059526.png)

![image-20250721102227833](Linux网络编程.assets/image-20250721102227833.png)



- 进程间通信的场景

![image-20250721102824481](Linux网络编程.assets/image-20250721102824481.png)

![image-20250721103626927](Linux网络编程.assets/image-20250721103626927.png)







# Libevent

## 特点

![image-20250722193423002](Linux网络编程.assets/image-20250722193423002.png)

![image-20250722195229407](Linux网络编程.assets/image-20250722195229407.png)

![image-20250722195213779](Linux网络编程.assets/image-20250722195213779.png)



![image-20250722202045742](Linux网络编程.assets/image-20250722202045742.png)





## 事件处理框架的创建----event_base

![image-20250723130113511](Linux网络编程.assets/image-20250723130113511.png)



- API函数

![image-20250723131603935](Linux网络编程.assets/image-20250723131603935.png)



## 启动事件循环函数--event_base_dispatch()函数

![image-20250723131837496](Linux网络编程.assets/image-20250723131837496.png)



- 启动事件循环

![image-20250723132550139](Linux网络编程.assets/image-20250723132550139.png)



![image-20250723132736008](Linux网络编程.assets/image-20250723132736008.png)



## 事件的终止函数

- 终止事件循环

![image-20250723133037407](Linux网络编程.assets/image-20250723133037407.png)





## 事件的创建和销毁

![image-20250723140556593](Linux网络编程.assets/image-20250723140556593.png)

![image-20250723140510654](Linux网络编程.assets/image-20250723140510654.png)

![image-20250723140446891](Linux网络编程.assets/image-20250723140446891.png)

==事件被创建之后，不能被事件处理框直接检测==



## 事件的添加和删除

![image-20250723153549344](Linux网络编程.assets/image-20250723153549344.png)




## 事件和事件处理框架之间的关系

![image-20250723141137487](Linux网络编程.assets/image-20250723141137487.png)

![image-20250723141531772](Linux网络编程.assets/image-20250723141531772.png)



## 通过event事件写管道

- 写端

![image-20250723151431437](Linux网络编程.assets/image-20250723151431437.png)

![image-20250723151517928](Linux网络编程.assets/image-20250723151517928.png)

![image-20250723151745379](Linux网络编程.assets/image-20250723151745379.png)

![image-20250723151806059](Linux网络编程.assets/image-20250723151806059.png)



- 读端

 ![image-20250723153051925](Linux网络编程.assets/image-20250723153051925.png)

![image-20250723153114379](Linux网络编程.assets/image-20250723153114379.png)

![image-20250723153148197](Linux网络编程.assets/image-20250723153148197.png)

![image-20250723153234960](Linux网络编程.assets/image-20250723153234960.png)





## 带缓冲区的事件

![image-20250723171706522](Linux网络编程.assets/image-20250723171706522.png)

![image-20250723165050732](Linux网络编程.assets/image-20250723165050732.png)

![image-20250723160558337](Linux网络编程.assets/image-20250723160558337.png)

![image-20250723165153692](Linux网络编程.assets/image-20250723165153692.png)

![image-20250723165217414](Linux网络编程.assets/image-20250723165217414.png)





## 创建带缓冲区的事件

![image-20250723172111794](Linux网络编程.assets/image-20250723172111794.png)

 



## 连接服务器函数--buffferevent_socket_connect()

![image-20250723172708151](Linux网络编程.assets/image-20250723172708151.png)





## bufferevent缓冲区的读写函数

![image-20250723175134999](Linux网络编程.assets/image-20250723175134999.png)



## 给bufferevent设置回调函数bufferevent_setcb()



![image-20250723180411074](Linux网络编程.assets/image-20250723180411074.png)



## bufferevent回调函数原型

![image-20250723181254390](Linux网络编程.assets/image-20250723181254390.png)





## 基于bufferevent的套接字客户端处理流程

![image-20250723213701279](Linux网络编程.assets/image-20250723213701279.png)



![image-20250723213738589](Linux网络编程.assets/image-20250723213738589.png)







## 套接字通信的客户端代码实现

![image-20250724144134433](Linux网络编程.assets/image-20250724144134433.png)

![image-20250724144211125](Linux网络编程.assets/image-20250724144211125.png)

![image-20250724144423571](Linux网络编程.assets/image-20250724144423571.png)

![image-20250724143803339](Linux网络编程.assets/image-20250724143803339.png)

![image-20250724143903614](Linux网络编程.assets/image-20250724143903614.png)







## 链接监听器

![image-20250724145811169](Linux网络编程.assets/image-20250724145811169.png)



- 创建和释放evconnlistener

![image-20250724153354504](Linux网络编程.assets/image-20250724153354504.png)

![image-20250724153927698](Linux网络编程.assets/image-20250724153927698.png)

![image-20250724153950171](Linux网络编程.assets/image-20250724153950171.png)



- 链接监听器的回调函数

![image-20250724155309332](Linux网络编程.assets/image-20250724155309332.png)



![image-20250724155715955](Linux网络编程.assets/image-20250724155715955.png)





## 套接字服务器端处理流程

![image-20250724160247681](Linux网络编程.assets/image-20250724160247681.png)

![image-20250724160315844](Linux网络编程.assets/image-20250724160315844.png)





## 套接字服务器端代码实现

![image-20250724161939488](Linux网络编程.assets/image-20250724161939488.png)

![image-20250724162026800](Linux网络编程.assets/image-20250724162026800.png)

![image-20250724162157408](Linux网络编程.assets/image-20250724162157408.png)

![image-20250724162246161](Linux网络编程.assets/image-20250724162246161.png)

![image-20250724162348591](Linux网络编程.assets/image-20250724162348591.png)

![image-20250724162420079](Linux网络编程.assets/image-20250724162420079.png)

![image-20250724162527520](Linux网络编程.assets/image-20250724162527520.png)
