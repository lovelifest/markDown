Java分为两种线程：用户线程和守护线程

所谓守护线程是指在程序运行的时候在后台提供一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者，并且这种线程并不属于程序中不可或缺的部分。因 此，当所有的非守护线程结束时，程序也就终止了，同时会杀死进程中的所有守护线程。反过来说，只要任何非守护线程还在运行，程序就不会终止。

守护线程和用户线程的没啥本质的区别：唯一的不同之处就在于虚拟机的离开：如果用户线程已经全部退出运行了，只剩下守护线程存在了，虚拟机也就退出了。 因为没有了被守护者，守护线程也就没有工作可做了，也就没有继续运行程序的必要了。

将线程转换为守护线程可以通过调用Thread对象的setDaemon(true)方法来实现。

1.如果不设置setDaemon(true)

```java
public class DaemonThreadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Thread threadNormalThread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println(Thread.currentThread().getName() + " running ... ");
			}
		}, "threadNormalThread");

		Thread threadDaemonThread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				while (true) {
					System.out.println(Thread.currentThread().getName() + " running ... ");
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}, "threadDaemonThread");

		threadNormalThread.start();
		//threadDaemonThread.setDaemon(true);
		threadDaemonThread.start();
	}

}

```

执行结果：

```java
threadNormalThread running …
threadDaemonThread running …
threadDaemonThread running …
threadDaemonThread running …
threadDaemonThread running …
threadDaemonThread running …
…
```

threadDaemonThread会一致执行下去，不会随threadNormalThread结束而结束

threadDaemonThread会一致执行下去，不会随threadNormalThread结束而结束

如果设置为true：

```java
public class DaemonThreadTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Thread threadNormalThread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println(Thread.currentThread().getName() + " running ... ");
			}
		}, "threadNormalThread");

		Thread threadDaemonThread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				while (true) {
					System.out.println(Thread.currentThread().getName() + " running ... ");
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		}, "threadDaemonThread");

		threadNormalThread.start();
		threadDaemonThread.setDaemon(true);
		threadDaemonThread.start();
	}

}
```

执行结果：

```java
threadDaemonThread running ... 
threadNormalThread running ... 
12
```

随着threadNormalThread结束，threadDaemonThread也结束了。同时进程结束，程序执行完毕。