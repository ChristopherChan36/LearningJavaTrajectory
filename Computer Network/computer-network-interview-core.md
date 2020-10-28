# 计算机网络面试核心

## 网络基础知识

### OSI开放式互联参考模型（OSI七层网络协议）图示

![OSI七层网络协议](https://project-figure-bed.oss-cn-beijing.aliyuncs.com/pisces-image/blog/youdaoyun/%E4%B8%83%E5%B1%82%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE.gif)

![OSI解释图](https://images0.cnblogs.com/blog2015/776442/201508/272237504067568.jpg)

### TCP/IP

**OSI的“实现”：TCP/IP**

### TCP的三次握手

**传输控制协议TCP简介：**
- 面向连接的、可靠的、基于字节流的传输层通信协议
- 将应用层的数据流分割成报文段并发送给目标节点的TCP层
- 数据包都有序号，对方收到则发送ACK确认，未收到则重传
- 使用校验和来校验数据在传输过程中是否有误

TCP三次握手的流程图如下：

![TCP三次握手流程图](https://images2015.cnblogs.com/blog/880287/201608/880287-20160803151722918-1174987329.png)

- 第一次握手：建立连接时，客户端发送SYN包（seq=x）到服务器，并进入SYN_SEND状态，等待服务器确认；
- 第二次握手：服务器收到SYN包，必须确认客户的SYN（ack=x+1），同时自己也发送一个SYN包（seq=y），即SYN+ACK报，此时服务器进入SYN_RECV状态；
- 第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK（ack=y+1），此数据包发送完毕后，客户端和服务端进入ESTABLISHED状态，完成三次握手；

#### 为什么需要三次握手才能建立连接

- **为了初始化 Sequence Number 的初始值**
- **首次握手的隐患--SYN超时**
    - 问题起因分析：
        - Server收到Client的SYN，回复SYN+ACK的时候未收到ACK确认
        - Server会不断重试直至超时，Linux默认等待63秒才断开连接
    - 针对SYN Flood的防护措施
        - SYN队列满后，会通过tcp_syncookies参数回发SYN Cookie
        - 若为正常连接则Client会回发SYN Cookie，直接建立连接

#### 建立连接后，Client出现故障怎么办

**保活机制**
- 向对方发送保活探测报文，如果未收到响应则继续发送
- 尝试次数达到保活探测数仍未收到响应则中断连接


### TCP的四次挥手

TCP四次挥手的流程图如下：

![TCP四次挥手流程图](https://img-blog.csdn.net/20170606084851272?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXpjc3U=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传输，Client进入FIN_WAIT_1状态；
- 第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态；
- 第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传输，Server进入LAST_ACK状态；
- 第四次握手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手；

## 参考文章：
- [浅谈对【OSI七层协议】的理解](https://www.cnblogs.com/snoopylovefiona/p/4764853.html)
- [TCP的三次握手与四次挥手（详解+动图）](https://blog.csdn.net/qzcsu/article/details/72861891)