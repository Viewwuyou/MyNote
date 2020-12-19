# Java面试题（自己总结搜集版）

### 1、Object类中包含哪些方法，以及其基本用途是什么

Object类中包含:

```java
protected Object clone(); // 创建并返回当前对象的副本
public boolean equals(Object obj); // 比较当前对象是否与obj对象相等
protected void finalize(); // Java9已经删除，作用是提醒JVM删除该对象
public Class<?> getClass(); // 获得当前对象的运行时类对象
public int hashCode(); // 返回当前对象Hash值
public void notify(); // 唤醒一个单独的等待该监视器的线程
public void notifyAll(); // 唤醒所有等待该监视器的线程
public String toString(); // 返回代表当前对象的字符串

public void wait();
public void wait(long timeoutMillis);
public void wait(long timeoutMillis,
                int nanos);
// 让当前线程进入阻塞状态直到唤醒
```

