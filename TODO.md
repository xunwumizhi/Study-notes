## 网络

net/http/httptrace
https://github.com/davecheney/httpstat

# 性能测试

link1st/go-stress-testing: go 实现的压测工具，ab、locust、Jmeter压测工具介绍【单台机器100w连接压测实战】: https://github.com/link1st/go-stress-testing

自定义压测
https://github.com/rakyll/hey

```
ab -n 10000 -c 500
```

## 操作系统

epoll fd文件句柄：

深入理解 Linux 的 epoll 机制：
https://mp.weixin.qq.com/s/Bhoym1M8Pa_G3926p7_czg



# 网络

HTTP客户端使用了连接池，避免频繁建立带来的大开销，

HTTP服务端的路由只是一个静态索引匹配，对于动态路由匹配支持的不好，并且每一个请求都会创建一个gouroutine进行处理，海量请求到来时需要考虑这块的性能瓶颈；

# 数据结构设计

## 连续内存

go slice内存分配_anssummer的博客-CSDN博客_slice是什么时候分配内存的
https://blog.csdn.net/yizhou35/article/details/103408133

底层为数组，连续内存，读取快速。但是空间不够会扩容，连续内存扩容也很快

Go 在 append 和 copy 方面的开销是可预知+可控的，应用上简单的调优有很好的效果。这个世界上没有免费的动态增长内存，各种实现方案都有设计权衡。

在go语言中slice是很灵活的，大部分情况都能表现的很好，但也有特殊情况。当程序要求slice的容量超大并且需要频繁的更改slice的内容时，就不应该用slice，改用list更合适。

## 内存对齐

编译器使用「字节对齐」的方式分配内存时，虽然 char 类型只占一个字节，但是由于成员变量里有 int 类型，它占用了 4 个字节，所以在成员变量为 char 类型分配内存时，会分配 4 个字节，其中这多余的 3 个字节是为了字节对齐而分配的，相当于有 3 个字节被浪费掉了。

# 定时任务、延时队列

一口气说出 6 种实现延时消息的方案: https://mp.weixin.qq.com/s/b8YzfjyhMjHnoBCKv592xA

如何设计一个合适的延时队列: https://mp.weixin.qq.com/s/hqPy-SOo2qh0WOJOSilIbA

# go

## 并发

通俗易懂！图解Go协程原理及实战
https://mp.weixin.qq.com/s/VtwXu0aerBwgaDJV7i3guw

Go中看似简单的WaitGroup源码设计，竟然暗含这么多知识？
https://mp.weixin.qq.com/s/PykMWANuSkDWavW5c6L2gg

使用信号量 semap 阻塞主goroutine

## 数据结构

Go是如何设计Map的
https://mp.weixin.qq.com/s?__biz=MzkyMzIyNjIxMQ==&mid=2247484572&idx=1&sn=93953c678184e3b1b3508c8956e84016&source=41#wechat_redirect

一文读懂channel设计
https://mp.weixin.qq.com/s?__biz=MzkyMzIyNjIxMQ==&mid=2247484573&idx=1&sn=805f9cd855fc55b53ddb8500a93a401f&source=41#wechat_redirect

## 垃圾回收

Golang GC 从原理到优化
https://mp.weixin.qq.com/s/35i61m-kXzOpiwOaJGO4Xg
