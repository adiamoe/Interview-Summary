# 操作系统

#### [缺页中断](https://www.cnblogs.com/sunsky303/p/9214223.html)

软件试图访问已映射在虚拟地址空间中，但是目前并未被加载在物理内存中的一个分页时，由中央处理器的内存管理单元所发出的中断。

#### [进程间的通信方式](https://blog.csdn.net/m0_37907797/article/details/103188294)

1. 管道
2. 消息队列
3. 共享内存
4. 信号量
5. Socket

#### 线程的6种状态

1. New
2. Runnable(分为Running和Ready)
3. Blocked(**进入 synchronized 关键字保护的代码，但是没有获取到 monitor 锁**)
4. Waiting
5. Timed Waiting
6. Terminated

**处于 Runnable 状态的线程并不一定在运行**。

**Blocked 在等待其他线程释放 monitor 锁，而 Waiting 则是在等待某个条件，比如 join 的线程执行完毕，或者是 notify ()/notifyAll ()**。

Timed Waiting与 Waiting 状态的区别在于：**有没有时间限制，Timed Waiting 会等待超时，由系统自动唤醒，或者在超时前被唤醒信号唤醒**。

#### 进程和线程有什么区别

1. 进程是内存分配的最小单位，线程是调度的最小单位
2. 进程有自己的独立地址空间，而线程则是共享本线程的数据空间。因此CPU切换和创建一个线程的花费远小于进程

#### 进程和线程的上下文切换

