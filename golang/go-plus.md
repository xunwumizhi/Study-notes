# 调度GPM

Golang 调度器 GMP 原理与调度全分析 | Go 技术论坛
https://learnku.com/articles/41728

GMP 原理：把大量的 goroutine G 分配到少量线程 M 上去执行，并利用多核并行，实现更强大的并发。P 维护G队列，G0 代码逻辑负责调度运行 G （P本地G队列，全局G队列）；

防止空闲的 P 过多，占用 M 自旋空转，提供参数控制 P 数量 GOMAXPROCS；

P 对应一个 CPU proc

P 上面挂着 OS 线程 M；

M 上执行者 G 用户态的协程；

P 限制了同时能用到 N 个CPU，一个线程同时只能运行在一个 CPU 上。

P 同时也就只能用到 N 个 M 线程；每个 P 同一时间只能绑定一个 M，避免引起 M 切换，如果P里面G系统调用引起阻塞，P转移到其他空闲M

M0 main G协程对应的线程
G0 每个M关联的第一个G

# GPM 运用
sync.Pool 高性能设计之集大成者
https://mp.weixin.qq.com/s/ltSzjyRoSRlN-VMEmJ61Cg

