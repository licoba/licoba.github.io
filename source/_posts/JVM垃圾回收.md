---
title: JVM
date: 2024-08-28 21:41:14
tags: [JVM,Java]
---

# JVM垃圾回收

### 什么情况会导致内存泄露？
总的来说，长生命周期的对象（如单例）持有短生命周期对象的引用，导致短生命周期对象无法被回收。

### 怎么判断一个对象死亡？

一个对象死亡，也就意味着，它不能再通过任何途径被使用。

#### 一、引用计数法

添加一个计数器，用来统计对象的引用数量，每当有一个地方引用这个对象的时候，计数器就加一，失去引用时，计数器就减一。

缺点：A引用了B，B引用了A，这样会有循环引用的问题。

JVM在两个对象循环引用的时候，也会将对象回收，说明JVM并不是采用的这种简单的引用计数法来判断对象是否可以回收的。

#### 二、可达性分析算法

重要概念：GC Roots，GC Roots可以是栈中的引用对象，也可以是方法区的字符串常量池、或者静态变量等。

如果从GC Roots到这个对象是不可达的（这个对象与GC Roots之间没有任何联系，是分开的）那么代表这个对象就可以回收了。（这里有错误，不是可回收了，看下面）

**如果一个对象不可达了，那么它将被进行一次标记，如果它被标记了两次，那么它才会被回收。**

为什么要标记？：标记是为了防止对象的finalize方法没有被执行，就已经被JVM回收了

如果这个对象没有finalize方法或者finalize方法已经执行完毕了，并且对象仍然不可达GC Roots，那么这个对象就是可回收的。

如果这个对象有finalize方法，那么这个对象会被移动到一个专门放finalize对象的队列中，执行这个队列的线程优先级比较低，并且这个队列是不保证finalize方法一定执行完毕的（因为finalize方法里面可能有耗时操作，或者有死循环）过一段时间，系统会再次检测这个对象，如果这个时候，对象还是不能到达GC Roots，这个对象就要被回收了，这个时候的对象，才是真正被判定为「死亡」的对象。当然，如果对象在finalize方法里面，与GC Roots又再次建立起了链接，那么垃圾回收的时候，这个对象将不会被回收（即：对象可以完成「自我拯救」）。

### Java里面的四种引用

Java里面的引用分为四个：强引用、软引用、弱引用、虚引用

- 强引用：Object o = new Objtect() 其中的o就是强引用，默认的都是强引用，只要引用关系还存在，垃圾回收器就不会回收这个对象。
- 软引用（Soft）：如果虚拟机内存不够了，虚拟机会对软引用的对象进行第二次回收，如果回收这些对象之后，内存还是不够，才会发生内存溢出。
- 弱引用（Weak）：用来描述非必须的对象，强度比软引用还要若一些。弱引用的对象只能存活到JVM的下一次GC的时刻，当下一次GC操作进行的时候，无论虚拟机内存是不是够，弱引用的对象都会被回收。
- 虚引用（PhantomReference ）：没有办法通过一个虚引用来得到它指定的对象，它存在的唯一目的，只是为了让它所指向的对象，在被GC回收时候，能得到一个通知。

### 垃圾回收算法

#### 分代收集理论

分代收集理论将堆分成两个大堆：新生代和老年代，新生代就是频繁有大量对象创建和销毁的区域，老年代就是对象的创建和回收没有那么频繁的区域。在每次内存回收后，新生代的对象会慢慢往老年代移动（因为存活得越久的对象就越难被回收，所以系统也会去降低GC访问标记的频率，这是分代假说里面的知识）

### 1、标记-清除算法

![image.png](https://pic.imgdb.cn/item/66d1a086d9c307b7e9af164c.png)

标记-清除算法就是基于上面所说的，判定对象是否可回收的标记。这个 **基础** 的算法，有两个缺点：

一是如果对象过多的话，会产生过多的标记，这样会对垃圾回收的效率产生影响。

二是对象存储的控件大多都是不连续的，清除的时候，会产生大量的内存空间碎片，这些碎片不连续，如果下次要申请一大段内存空间的话，不太好利用，需要再触发一次内存回收。

### 2、标记-复制算法

![](https://pic.imgdb.cn/item/66d1a0a9d9c307b7e9af7c2a.png)

将空间分为两片，一片用来回收，回收的时候将依然存活的对象，复制到另外一块上面去。

优点：解决了空间碎片的问题

缺点：内存空间浪费得比较大

为了解决空间浪费的问题，存在一个假说：新生代中的对象有98%都熬不过第一轮垃圾回收，所以不需要按照1:1的比例来分配空间。

解决方案：按照8:1:1的比例来划分空间，Eden 和 Survivor，Eden占8，两个Survivor各占1。每次回收的时候，将 Eden 和Survivor 中仍然存活的对象一次性复制到另外 一块 Survivor 空间上，然后直接清理掉 Eden 和已用过的那块 Survivor 空间。

当然也有例外情况，Survivor放不下存活对象时，这些对象将会进入老年代，也就是老年代负责“担保“。

### 3、标记-整理算法

![](https://pic.imgdb.cn/item/66d1a0cbd9c307b7e9afc937.png)

和标记-清除算法的区别就是，这个是移动式的算法，在垃圾回收后，会将存活对象往前移动，然后一次性清理这个区域之外的内存。

缺点：对于老年代这种存活对象过多的区域，每次都移动这些对象，并且更新这些对象的引用，消耗过大。需要暂停用户线程，来等待GC线程的操作完成。

### 新生代和老年代

- 新生代：存放刚被new出来的对象，容易被回收的对象
- 老年代：存放不会被轻易回收的对象
## 【相关问题】
### 哪些对象可以是GC Roots？
两栈两方法：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象

### 新生代区域比例？
Eden：Survivor from：Survivor ro = 8:1:1
新生代：老年代 = 1:2，也就是说，新生代占整个堆内存的1/3，而老年代占整个堆内存的2/3。
### ![](https://pic.imgdb.cn/item/66d1a0e1d9c307b7e9b00885.png)

- 在新生代，采用标记-复制算法效率比较高，因为很多对象存活时间都非常短
- 在老年代，采用标记-整理算法效率比较高，因为没有额外的空间对老年代的对象做担保了。
### **新生代的垃圾回收过程？**

- 当eden区满了的时候，此时会触发一次MinorGC，把被标记还活着的对象复制到from区中，然后将eden清空；然后当eden区再次满了的时候，此时会将eden和from区中被标记还活着的对象复制到to区，然后将eden和from清空。（此时我们将from和to区交换一下）
- 然后接下来的每一次垃圾回收，都可以看做是一个eden+from复制到to区的一个过程。当GC年龄超过**15**岁还存活的话，则会被送进老年区。
### System.GC调用后会立马回收吗？

- 不是的，只是程序通知JVM，用户希望触发一次GC回收，GC回收的时间，是否会回收，都是不确定的。它只是向 JVM 发出一个建议，实际的垃圾回收行为由 JVM 的垃圾回收器决定

### 标记时为什么要标记两次？
在并发垃圾回收算法中，垃圾回收器和应用程序线程是并发运行的。标记一次可能会导致以下问题：
对象复活：在第一次标记之后，应用程序线程可能会重新引用那些被标记为不可达的对象。如果只标记一次，这些对象可能会被误回收。
不一致性：在并发环境中，标记一次可能无法准确反映对象的可达性状态，导致不一致性。
通过标记两次，可以确保在第一次标记之后，应用程序线程没有重新引用这些对象，从而**避免误回收**。

