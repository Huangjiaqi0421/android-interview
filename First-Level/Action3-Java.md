## 问题1：HashMap原理？

## 问题2：对Thread的理解？线程状态？阻塞和运行状态区别？锁的种类，什么是自旋锁，ReentrantLock？

答：
 - JVM中允许并发执行的单元，是操作系统对cpu任务处理的一种抽象，通过Thread可以构建一个线程实例
 - 线程的状态可参考Thread.State的枚举定义（注意，android中jvm对线程状态的定义多于java，这个在一些系统日志比如anr日志文件可得到验证诸如处于native状态的线程）
 - 运行状态的线程意味着正在被当前cpu执行，阻塞状态的线程可被cpu执行，前提是获取到使他处于block状态的monitor lock
 - 锁的种类，还是参考这个吧，这个文章详细[点击这里](https://www.cnblogs.com/qifengshi/p/6831055.html)  [自选锁](http://ifeve.com/java_lock_see1/)
 - [ReentrantLock源码解析](https://www.cnblogs.com/leesf456/p/5383609.html)
 - 锁的问题，本质上看是一个读写问题(volatile的读写安全)，在实际代码实现时，能不用锁尽量不用/使用细粒度的锁/加锁顺序一致/线程安全发布等等
 - 线程不安全产生的bug一般表现为非必现（视线程竞争和任务执行状态而定，比如高端手机必现，低端手机不出现）

## 问题3：HashMap的哈希散列实现，线程安全吗，为什么？

## 问题4：ArrayList和Vector扩容的区别HashTable，ConcurrentHashMap怎么实现线程安全

## 问题5：ava8对hashmap的优化hashmap和hashset区别，hash怎么散列

## 问题6：为什么`wait` , `notify`是在Object类中的

## 问题7：简述类加载机制，触发类初始化的时机，被动引用的时机

## 问题8：GC的机制