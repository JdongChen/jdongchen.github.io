---
title: "IO模型"
subtitle: ""
date: 2021-10-24T11:58:08+08:00
lastmod: 2021-10-24T11:58:08+08:00
draft: false
hidden: true
description: ""
tags: ["LinuxIPC"]
categories: []
---

<!--more-->

## 五种I/O模型
1. 阻塞式I/O
默认情况下，所有套接字均为阻塞，阻塞时会让出CPU，导致该进程处理效率低下。
阻塞(UDP)分为两个过程：数据包准备(等待数据)、从内核复制到用户空间(复制数据)。

多进程阻塞式I/O，一旦连接完成，则fork进程，服务器将连接交由子进程处理。该方案优势在于父进程仅需要“监听”，子进程仅需要“连接”，但是该方案要注重完成请求后子进程需要退出回收资源，同时不能处理过多的连接，因为进程的上下文切换需要占据资源。

多线程阻塞式I/O，一旦连接完成，则pthread_create创建新线程，服务器将连接交由子线程处理。然后将已连接的socket文件描述符交由子线程，接着进行通信，达到并发处理的目的，CPU尽可能减少闲置。频繁的创建与销毁线程，也会导致系统开销。

线程池方式避免线程的频繁创建于销毁，即提前创建若干线程，当新连接建立时，将socket放入队列，然后线程池里的线程负责从队列取出socket进程处理。由于该队列是全局的，因此线程在操作队列前需要加锁。

2. 非阻塞式I/O
进程可将套接字设置成非阻塞：当所请求的I/O操作需把本进程投入睡眠时，直接返回错误，即数据包未准备好(等待数据)时直接返回错误而不阻塞。进程需要循环读取，占据时间片。

3. I/O复用(select、poll、epoll)
进程受阻于select调用，等待可能多个套接字中任意一个变为可读(等待数据)，可读之后进行系统调用将数据从内核复制到应用缓冲区，该期间阻塞。

使用select优势在于可以等待多个文件描述符就绪，对比多线程阻塞式I/O，使用select阻塞在多个文件描述符上。

4. 信号驱动式I/O(SIGIO)
首先开启套接字的信号驱动式I/O功能，并通过sigaction系统调用安装信号处理函数，该系统调用将立即返回(未阻塞)。当数据报准备好后，内核递交SIGIO，可在信号处理函数中进行系统调用读取数据报。
该模型的优势在于等待数据报到达期间可以继续执行，只要等待来自信号处理函数的通知：既可以是数据已准备好并被处理，也可以是数据准备好被读取。

5. 异步I/O(POSIX的aio_系列函数)
该系列函数的工作机制：告知内核启动操作，并让内核在整个操作(等待数据、复制数据)完成后通知应用程序，应用程序进行信号处理。
异步I/O与信号驱动I/O的区别：
- 信号驱动I/O由内核告知应用程序启动I/O操作
- 异步I/O模型由内核告知应用程序操作何时完成

## select函数 
函数原型：
~~~ c
int select(int maxfdpl, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout);
~~~
参数struct timeval *timeout，该结构体如下：
~~~ c
struct timeval {
    long tv_sec; // seconds
    long tv_usec; // microseconds
}
~~~
1. 永远等待下去：设定为空指针
2. 等待固定时间：设定timeval值
3. 轮询(不等待)：指向timeval值为0
前两种情形将会在进程等待期间捕获信号中断，并从信号处理函数返回。

中间三个fd_set参数指定内核测试读、写、异常描述符。
select 采用固定长度的BitMap，表示文件描述符的集合，而且所支持的文件描述符的个数是有限制。
select函数需要两次遍历文件描述符集合，一次在内核态(遍历、拷贝)，一次在用户态。
select返回-1，表示出错；返回0，表示超时；返回正数，正常进行遍历。

## poll函数
函数原型：
~~~ c
int poll(strunct pollfd *fdarray, unsigned long nfds, int timeout)
~~~
第一个参数是指向一个结构数组的第一个元素的指针，每个数组元素都是pollfd结构，用于指定测试某个给定描述符的fd的条件。
~~~ c
struct pollfd {
    int fd;        // descriptor to check
    short events;  // events of interest on fd
    short revents; // events that occurred on fd
}
要测试的条件由events成员指定，在revents成员中返回该描述符状态。
结构体长度由nfds指定，timeout参数指定poll函数返回前等待多长世间。

poll不再使用BitsMap来存储文件描述符，取而采用动态数组(链表)来组织，突破了文件描述的数量限制，除此外均相似。


## epoll函数
创建epoll函数原型：
~~~ c
int epoll_create(int size);
~~~
修改epoll函数原型：
~~~c
int epoll_clt(int epfd, int op, struct epoll_event *event);
~~~
op 包含增、删、改三个操作，对应以下宏的集合：
- EPOLL_CTL_ADD：注册新的fd到epfd中；
- EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
- EPOLL_CTL_DEL：从epfd中删除一个fd；

events可以是以下几个宏的集合：
- EPOLLIN：触发该事件，表示对应的文件描述符上有可读数据。(包括对端SOCKET正常关闭)；
- EPOLLOUT：触发该事件，表示对应的文件描述符上可以写数据；
- EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
- EPOLLERR：表示对应的文件描述符发生错误；
- EPOLLHUP：表示对应的文件描述符被挂断；
- EPOLLRDHUP: 表示对端断开连接
- EPOLLET：将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的；
- EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。


等待epoll函数原型：
~~~c
int epoll_wait(int epfd, struct epoll_event *event, int maxevents, int timeout);
~~~
参数：
epfd:由epoll_create 生成的epoll专用的文件描述符；
epoll_event:用于回传代处理事件的数组；
maxevents:每次能处理的事件数；
timeout:等待I/O事件发生的超时值(单位我也不太清楚)；-1相当于阻塞，0相当于非阻塞。一般用-1即可
返回值:发生事件数。


epoll 优势：
1. 在内核中使用红黑树跟踪进程所有待检测的文件描述符，将需要监控的socket 通过epoll_clt函数加入内核中的红黑树。每次操作时仅需传入待检测的socket，减少了内核和用户空间的大量的数据拷贝与内存分配。
2. 采用事件驱动的机制，内核中维护链表记录就绪事件，当某个socket 有事件发生时，通过回调函数内核会将其加入就绪事件列表中，用户调用epoll_wait函数时，只会返回有事件发生的文件描述符的个数，而不需要像select/poll轮询整个文件描述符集合。

epoll事件触发模式：
1. 边缘触发(edge-triggered,ET):高效，使用边缘触发模式，当被监控的socket描述符上有可读事件发生时，服务器端只会从epoll_wait中苏醒一次，因此需要保证一次性将内核缓冲区市局读取完。
2. 水平触发(level-triggered,LT):普通，服务器端不断从epoll_wait中苏醒，直到缓冲区数据读取结束。

边缘触发一般和非阻塞I/O搭配使用，尽可能多读取数据；水平触发不必一次执行尽可能多的读写操作。
select/poll 本质上只有水平触发模式，epoll可根据特定场景设置为边缘触发模式。


## select、poll、epoll对比
1、select、poll
- select、poll每轮循环都需要从用户空间拷贝所有描述符和事件到内核空间；
- 内核以轮询的方式检查描述符上有没有事件产生，时间复杂度O(n)；
- select、poll 返回后，需要遍历所有描述符才能找到就绪的文件描述符，时间复杂度O(n).。

2、epoll
- 创建内核事件表，每个描述符只添加一次（不要每次都拷贝数据到内核中）;
- 内核实现：注册回调函数方式实现，时间复杂度O(1);
- 直接返回就绪的文件描述符 ，时间复杂度O(1)。
