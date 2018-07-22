## volatile和synchronized

java中volatile和synchronized

1. JMM

   并发过程中如何处理可见性、原子性和有序性的问题

   Runnable、Thread来创建线程

   并发编程中2个关键问题：

   a.线程间如何通信

   ​	共享内存（隐士通信）

   ​	消息传递 显示通信   wait  notify

   b.线程之间如何同步

     	在共享内存的并发模型中，同步是显示做的 synchronized

   ​	在消息传递的并发模型中，由于消息的发送必须在消息的接受之前，所以同步是隐士的

    java多线程通信是共享内存隐士通信

   

2. 定位内存可见性问题

   什么对象是内存共享，堆内存 进程级别的  主内存

   什么不是  本地缓存 线程级别

   主内存数据对所有可见，线程本地内存对其他线程不可见

   volatile  synchronized

   volatile功能 ，对于声明了volatile的变量进行写操作的时候，JVM会向处理器发送一条lock前缀的指令，会吧这个变量在多小缓存行的数据写回到主内存

   在多处理器的情况下，保证每个处理器缓存一致行的特点，实现缓存一致协议

3. 区别

   synchronized 可重入锁 互斥性 可见性

   volatile 可以做到原子性，可见性 不能做到符合操作的原子性

   i++

   

   synchronized   汇编源码：monitorenter  moniterexist

   volatile：内存可见性   synchronized：实现锁