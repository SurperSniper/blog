title: TCP/IP
date: 2015-06-18 14:45:16
categories: Study Notes
tags: [Tcp/Ip]
---
# TCP/IP连接三次握手
在TCP/IP协议中，TCP协议提供可靠的连接服务，采用三次握手建立一个连接。


1. 建立连接时，客户端发送syn包(syn=x)到服务器，并进入SYN_SEND状态，等待服务器确认；
2. 服务器收到syn包，必须确认客户的SYN（ack=x+1），同时自己也发送一个SYN包（syn=y），即SYN+ACK包，此时服务器进入SYN_RECV状态；
3. 客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=y+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

![](http://7xjkgu.com1.z0.glb.clouddn.com/image/TCP-IP.jpg)

# TCP/IP断开四次握手

1. Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态。
2. Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。
3. Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。
4. Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。