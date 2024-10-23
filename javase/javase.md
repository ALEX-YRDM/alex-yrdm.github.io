# Java 查漏补缺

## 你平时遇到的异常类有哪些?

- java.util.ConcurrentModificationException  并发修改异常
- java.lang.OutOfMemoryError  OOM
- java.lang.StackOverflowError  栈溢
- java.lang.IllegalMonitorStateException  当线程试图调用 `wait()`, `notify()`, `notifyAll()` 而没有持有相关对象的锁时抛出。
- java.lang.InterruptedException：当线程在等待、睡眠或其他操作中被中断时抛出。
- NullPointerException
- ArrayIndexOutOfBoundsException

## CopyOnWrite

- 是一种并发编程的设计模式,适合读多写少
- **读操作**：多个线程可以并发地读取数据，而不需要锁定集合。
- **写操作**：每当有写操作（如添加、删除、更新元素）时，都会创建一个新的集合副本。写入操作不会修改原始集合，而是在新副本上执行操作，执行完后再将这个新的副本替换掉原有的数据引用

## 读写分离了解吗?

Mycat sharingJDBC

## HashSet底层了解吗

就是hashmap只用了key