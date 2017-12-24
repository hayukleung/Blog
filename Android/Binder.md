## 进程间通信

Linux 已经拥有的进程间通信IPC手段包括：
- 管道（Pipe）
- 信号（Signal）和跟踪（Trace）
- 插口（Socket）
- 报文队列（Message）
- 共享内存（Share Memory）
- 信号量（Semaphore）

Android 基于自身的考量，设计了 Binder IPC。

## 各种 IPC 方式对比

| IPC                  | 数据拷贝次数 | 说明
|----------------------|--------------|-----
| 共享内存             | 0            | 但控制复杂，难以使用
| Binder               | 1            |Linux 内核实际上没有从一个用户空间到另一个用户空间直接拷贝的函数，需要先用 copy_from_user() 拷贝到内核空间，再用 copy_to_user() 拷贝到另一个用户空间。为了实现用户空间到用户空间的拷贝，mmap() 分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用 copy_from_user() 将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间
| Socket/管道/消息队列 | 2            |  数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程

## Binder 通信模型

![Binder 通信模型](http://ooun8fyfu.bkt.clouddn.com/2017/04/24/binder.jpg)

- Client 进程

> 使用服务的进程

- Server 进程

> 提供服务的进程

- ServiceManager 进程

> ServiceManager 的作用是将字符形式的 Binder 名字转化成 Client 中对该 Binder 的引用，使得 Client 能够通过 Binder 名字获得对 Server 中 Binder 实体的引用

- Binder 驱动

> 驱动负责进程之间 Binder 通信的建立，Binder 在进程之间的传递，Binder 引用计数管理，数据包在进程之间的传递和交互等一系列底层支持

## Android RPC 原理

![Android 的 RPC 原理](http://ooun8fyfu.bkt.clouddn.com/17-5-1/45313883-file_1493606998383_17cdd.png)

## 代理模式

![代理模式](http://ooun8fyfu.bkt.clouddn.com/17-5-1/69396724-file_1493606998686_174ca.png)

## Remote Method Invocation

![RMI](http://ooun8fyfu.bkt.clouddn.com/17-5-1/86191436-file_1493606998549_c808.png)