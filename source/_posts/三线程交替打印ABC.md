---
title: 三线程交替打印ABC
date: 2024-09-03 11:00:00
tags: [多线程, Java]
---

## 问题描述
给定三个线程，分别命名为A、B、C，要求这三个线程按照顺序交替打印ABC，每个字母打印100次，最终输出结果为
```
ABCABCABC...ABCABC
```

## 解决方法

### 1. Synchrognized + Lock的wait/notify

思路是new一个Lock，然后用Synchrognized获取这个锁，如果当前没轮到自己（n%3!=0）就用while等待，然后执行一遍打印，然后调用lock.notifyAll方法

```


class ThreeThreadPrinter1 {
    // 创建一个对象作为锁对象
    private val lock = Object()
    private var num = 0
    fun printA() {
        for (i in 0..100) {
            synchronized(lock) {
                while (num % 3 != 0) {
                    lock.wait()
                }
                println("A")
                num++
                lock.notifyAll()
            }
        }
    }

    // 打印B的方法
    fun printB() {
        for (i in 0..100) {
            synchronized(lock) {
                while (num % 3 != 1) {
                    lock.wait()
                }
                println("B")
                num++
                lock.notifyAll()
            }
        }
    }

    // 打印C的方法
    fun printC() {
        for (i in 0..100) {
            synchronized(lock) {
                while (num % 3 != 2) {
                    lock.wait()
                }
                println("C")
                num++
                lock.notifyAll()
            }
        }
    }

    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            val printer = ThreeThreadPrinter1()
            // 创建三个线程，分别调用printA, printB, printC方法
            val threadA = Thread { printer.printA() }
            val threadB = Thread { printer.printB() }
            val threadC = Thread { printer.printC() }
            // 启动三个线程
            threadA.start()
            threadB.start()
            threadC.start()
            // 等待3个线程执行完了之后，再退出main线程
            threadA.join()
            threadB.join()
            threadC.join()
        }
    }
}

```

### 2. ReeentrantLock + Condition  

创建可重入锁，然后用condition从更细的维度上来控制，可重入锁和condition是配套使用的，condition是lock内的一个接口，通过lock.newCondition方法来创建一个condition，condition的等待方法，是**await**，唤醒方法是**signalAll**，需要记住这两个方法名关键代码：
```
// 创建一个可重入锁
private final ReentrantLock lock = new ReentrantLock();
// 创建一个Condition对象，用于线程间的协调
private final Condition condition = lock.newCondition();
// 用于跟踪当前应该打印哪个字符的状态变量
private int state = 0;

// 打印A的方法
public void printA() {
    for (int i = 0; i < 10; i++) {
        lock.lock(); // 获取锁
        try {
            // 如果当前状态不是0，则等待
            while (state % 3 != 0) {
                condition.await();
            }
            System.out.println("A");
            state++;
            // 唤醒所有等待的线程
            condition.signalAll();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            lock.unlock(); // 释放锁
        }
    }
}
```

### 3. 使用信号量Semaphore
信号量基础知识
```
Semaphore(1)：表示信号量初始时有1个许可。通常用于实现类似互斥锁的功能，即一次只允许一个线程访问某个资源。
Semaphore(0)：表示信号量初始时没有许可。任何调用 acquire() 的线程都会被阻塞，直到有其他线程调用 release() 增加许可。
Semaphore(n)：表示信号量初始时有 n 个许可。最多允许 n 个线程同时访问某个资源。
```

信号量的获取函数是**acquire**，增加函数是**release** ，Semaphore(0).relase = Semaphore(1)，也就是说调用release函数，就会给信号量的许可证数量+1。

基本逻辑是创建一个Class，三个参数：curSemaphore，nextSemaphore，message，同时创建三个semaphore，像指针一样，A指向B，B指向C，C指向A，一直循环打印。

