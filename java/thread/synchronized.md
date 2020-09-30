synchronized关键字是一种原子性内置锁，线程进入synchronized代码块前会自动获取监视器锁，这时其他线程再访问该同步代码块是会被阻塞挂起。

前面的共享变量内存可见性问题主要是线程的工作内存导致的，而synchronized的内存语义可以解决此问题。

synchronized的内存语义：

- 进入synchronized块：把synchronized块中使用到的变量从线程的工作内存中清除，这样当synchronized块中要使用该变量时，就会从主存中去取。
- 离开synchronized块：把synchronized块中对共享变量的修改刷新到主内存。

### 示例

```java
public class ThreadSafeInteger {    
    private int value;
    public synchronized int get() {
        return value;    
    }    
    public synchronized void set(int value) {
        this.value = value;
    }
}
```

> 注1：get()方法虽然只是读操作，但仍要加上synchronized来实现value的内存可见性。
>
> 注2：使用synchronized虽然解决了共享变量value的内存可见性问题，但由于synchronized是独占锁，同时只能有一个线程调用get()方法，其他调用线程则会被阻塞，同时存在线程切换、调度的开销，效率并不高。

