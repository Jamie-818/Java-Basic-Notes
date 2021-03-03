# 简述 BIO, NIO, AIO 的区别

## 同步和异步

- 同步：发起一个调用后，被调用者未处理完请求之前，调用不返回。
- 异步：发起一个请求后，被调用者会通过回调等机制通知调用者其返回结果。

## 阻塞和非阻塞

- 阻塞：发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才会继续
- 非阻塞：发起一个请求，调用者不用一直等着结果返回，可以去干其他事情。

## BIO

- 同步阻塞IO

- 一个连接一个线程
- 线程发起IO处理，不管内存是否准备好IO操作，从发起请求起，线程一直阻塞，直到操作完成。
- 一般通过连接池进行改善

## NIO

- 同步非阻塞
- 一个请求一个线程
- 客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有IO请求时才会启动一个线程处理

## AIO

- 异步非阻塞
- 一个有效请求一个线程
- 线程发起IO请求，立即返回。内存做好IO操作的准备之后，做IO操作，直到操作完成或者失败，通过调用注册的回调函数通知线程做IO操作完成或者失败。

## 适用场景分析

- BIO方式适用于连接数目比较少且固定的架构，这种方式对服务器资源要求比较高，并发局限在应用，JDK1.4以前的唯一选择，但程序直观简单易理解。
- NIO方式使用多连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限在应用中，编程比较复杂，JDK1.4开始支持。
- AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。