# 原子性操作
* 其他线程只能获取到执行操作之前的状态或者之后的状态
* 底层硬件保证原子性
* 更宽泛的不限于硬件指令，例如数据库事务

# wait free和lock free区别
* lock free允许某些线程保持饥饿，但是在运行一段时间至少有一个线程的任务有进展，这是wiki的定义，理解为在任意时间内总有一个线程获得了控制权在执行任务会好一些。 典型的例如CAS(compare and sawp)做循环重试的算法。

* wait free保证了吞吐量和饥饿程度，每个线程都能在有限的步骤中完成任务。参考http://www.cs.technion.ac.il/~erez/Papers/wfquque-ppopp.pdf

# date race的产生
* cpu是从cache中获取数据并把结果写入cache中，如果cache没有同步机制，在一个cpu操作数据后，其他cpu就会继续使用旧数据。

# trivially copyable类型
trivially copyable类似可以是原子的
满足以下条件是trivailly capyable类型
1. 连续的内存分布
2. 拷贝所有的字节就能直接拷贝整个对象（memcpy)
3. 没有虚方法，构造函数是不会跑出异常的(noexcept constructor)

# 哪些操作是原子的

![原子操作图片示意][1]
&=，^=也是原子操作
浮点数没有上述各种原子操作

CAS指compare and swap

# atomic和lock
![原子操作图片示意][2]
lock free并不一定快
依赖于cache和memory spinlock和atomic速度上会有不同。

# atomic operation
* 
* 在写入数据时 atomic 操作会等待，但是读不会(读到之前的？)
* 在同一个cache line上的两个atomic 操作会互相等待
* numa machines 还需要在不同的页表上
* compare_exchange_week在一定时间内没有获取到执行权限时也会返回false


# 内存屏障
* memory_order_relax,代码上写a,b,c对于真实的写入内存顺序可能不是这个顺序
* memory_order_acquire

![memory_order_acquire][3]

所有在内存屏障之后的全部读写不会移动屏障之前

* memory_order_release

![memory_order_release][4]

所有在内存屏障之前的全部读写不会移动屏障之后

通常一起使用生产者使用memory_order_release修改 atomic变量通知消息者所有数据准备好了，消费者使用memory_order_acquires读取同一个atomic变量，然后可以读取生产者生成的数据了。（数据指非atomic 变量)

* memory_order_acq_rel 全部读写不能跨越屏障
* memory_order_seq_cst sequential consistency 对不同原子数据进行保序
a,b,c 是原子性的,
a 写入数据,
b 写入数据,
c 写入标志 表示其他线程可以开始使用a和b了,
a和b 是relax，
c 是cst。

原子操作函数默认是cst，开销大。
# 参考
1. [C++ atomics, from basic to advanced](https://www.youtube.com/watch?v=ZQFzMfHIxng)

[1]: ./atomic/atomic_operation.png
[2]: ./atomic/atomic_and_lock.png
[3]: ./atomic/acquire.png
[4]: ./atomic/release.png
[link]: https://www.youtube.com/watch?v=ZQFzMfHIxng