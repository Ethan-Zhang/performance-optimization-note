# performance-optimization-note
性能优化笔记

## CPU
### 指定进程或线程运行的CPU
通过Linux命令或系统API，将运行中的某个进程/线程绑定到指定的CPU
#### 优点
1. 减少重要进程/线程的切换
2. 提高CPU Cache的命中率
#### 缺点

#### 方法
1. taskset命令   
2. numactl命令
3. sched_setaffinity系统API

#### 适用场景

#### 参考
http://www.cnblogs.com/liuhao/archive/2012/06/21/2558069.html
http://www.cnblogs.com/highway-9/p/5494977.html
http://coolshell.cn/articles/7490.html


## 内存
### 预分配
分配内存时刻早于程序需要的时间或分配内存的容量超过程序某次实际需要的大小
#### 优点
#### 缺点
#### 适用场景
#### 参考

### 锁定
通过mlock等系统调用，允许程序在物理内存上锁住它的部分或全部地址空间，阻止Linux操作系统将这些被锁住的页换出。
#### 优点
1. 减少内存页面的调入调出带来的延迟
2. 防止内存中敏感数据的泄露
#### 缺点
1. 操作系统可用内存减少，易宕机
#### 适用场景
#### 参考
http://www.cnblogs.com/sky-heaven/p/7019977.html

### Huge Page
Linux标准内存页的大小为4KB，通过设置Huge Page，可以允许程序使用
#### 优点
1. 减少操作系统访问内存页表TLB的开销
2. 减少内存页换入换出的延迟
#### 缺点
由于内存页page size增大，多CPU写同一个内存页容易冲突（加锁）
#### 适用场景
对于大量热点内存数据比较有效果，但是会造成多CPU写同一块page的冲突，一般像MongoDB这类采用多线程随机读写的数据库都要求关闭huge page。如果是单线程的程序，因为减少了多CPU的写冲突，可以利用到Huge Page的特性。
#### 参考
http://hsbxxl.blog.51cto.com/181620/1075166
http://cenalulu.github.io/linux/huge-page-on-numa/

## 网络
### Zero Copy
“零拷贝”是指计算机操作的过程中，CPU不需要为数据在内存之间的拷贝消耗资源。而它通常是指计算机在网络上发送文件时，不需要将文件内容拷贝到用户空间（User Space）而直接在内核空间（Kernel Space）中传输到网络的方式。   
Zero Copy的模式中，避免了数据在用户空间和内存空间之间的拷贝，从而提高了系统的整体性能。Linux中的sendfile()以及Java NIO中的FileChannel.transferTo()方法都实现了零拷贝的功能，而在Netty中也通过在FileRegion中包装了NIO的FileChannel.transferTo()方法实现了零拷贝。
![normal](http://static.oschina.net/uploads/space/2014/0108/104700_qc4H_859646.png)
![Zero-Copy](http://static.oschina.net/uploads/space/2014/0108/104957_UW6E_859646.png)
#### 优点
1. 避免数据在用户态空间和内核态空间之间的拷贝
#### 缺点
1. 不同于常用socket的read、write调用，使用sendfile调用，需要重写底层网络库的代码。（目前已经有许多框架实现，如Netty）
### Non-Blocking

## 磁盘
### 使用Ext4或xfs文件系统
如运维可以支持，建议将生产系统的操作系统尽量升级至高版本以支持ext4或xfs等新的文件系统。
#### 优点（以Ext4为例）
1. 更大的inode
2. 快速fsck（inode表中增加未使用inode表）
3. 引入Extends。可以分配连续的Block，提高大文件的写入效率
4. 多块分配
5. 延迟分配
#### 参考
http://misujun.blog.51cto.com/2595192/883949

## JVM调优
### 管理JVM堆外内存
通过sun.misc.unsafe管理JVM的堆外内存。不使用JVM的垃圾回收而手动管理内存数据。具体可参见flink中的实现。
#### 优点
#### 缺点
#### 适用场景
1. Flink
#### 参考
http://www.cnblogs.com/mickole/articles/3757278.html
http://wuchong.me/blog/2016/04/29/flink-internals-memory-manage/
