# API框架
go get github.com/go-courier/oas

c++部分使用zmq通信。

## zmq
起来像是一套嵌入式的网络链接库，是网络通信中新的一层，介于应用层和传输层之间（按照TCP/IP划分），其是一个可伸缩层，但工作起来更像是一个并发式的框架，分散在分布式系统间。它提供的套接字可以在多种协议中传输消息，如线程间、进程间、TCP、广播等。可以使用套接字构建多对多的连接模式，如扇出、发布-订阅、任务分发、请求-应答等。ZMQ的快速足以胜任集群应用产品。它的异步I/O机制能够构建多核应用程序，完成异步消息处理任务。ZMQ有着多语言支持，并能在几乎所有的操作系统上运行。
1. 可以在多种协议之间传递消息
2. 多对多的连接模式
3. 异步消息处理任务


## 跟 Socket 的区别
是：普通的 socket 是端到端的（1:1的关系），而 ZMQ 却是可以N：M 的关系，人们对 BSD 套接字的了解较多的是点对点的连接，点对点连接需要显式地建立连接、销毁连接、选择协议（TCP/UDP）和处理错误等，而 ZMQ 屏蔽了这些细节，让网络编程更为简单。ZMQ 用于 node 与 node 间的通信，node 可以是主机或者是进程