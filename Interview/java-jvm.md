# Java Virtual Machine

## Pre-foundation

### Java 内存模型（JMM）

> Java  内存模型 -> 原子性、可见性、有序性 -> volatile -> happens-before / 内存屏障

#### Java 内存模型的理解

![Java 内存模型](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/08/2020-10-22-140248.png)

#### Java 内存模型中的原子性、可见性以及有序性

Java的内存模型是分为主内存和线程的工作内存两部分进行工作的，线程的工作内存从主内存 read 数据，load 到工作内存中，线程对数据进行 use，然后将数据assign 到工作内存，从工作内存 store 处理过的数据，最后将新数据 write 进主内存。 默认的这种情况下，不同线程并发对同一个数据进行操作是没有**可见性**的，会发生数据错误的情况。如果能保证**可见性**就是在线程1修改了数据后，线程2立马能将自己工作线程的数据刷新进行后续操作。 原子性是指：线程1对数据读取并操作是一个原子过程，线程1在处理过程中，其他线程不能对数据进行操作。 java 虚拟机会对写好的代码进行**指令重排**，在多线程情况就可能会因为代码顺序调整出现问题。比如单例模式中的双重校验，指令重排会导致单例生成失败返回 null。如果指令重排序就可能导致线程1还未准备好数据，线程2就开始执行业务操作，发生错误。有序性是指：通过一定手段，保证不会对代码进行指令重排序。

#### volatile 关键字的原理

volatile主要是用来解决多线程场景下变量的可见性以及顺序性。 如何实现可见性呢？如果在一个线程中修改了被volatile修饰的变量值，那么会让其他线程的工作内存中的变量值失效（基于处理器遵循的一致性协议），在下一次需要用到该变量的时候，重新去主存中去加载最新的变量值。

#### 指令重排以及 happens-before 原则

编译器、指令器可能对代码重排序，乱排，要守一定的规则，happens-before原则，只要符合happens-before的原则，那么就不能胡乱重排，如果不符合这些规则的话，那就可以自己排序。

Happens-before 原则：

1. **程序次序规则**：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
2. **锁定规则**：一个unLock操作先行发生于后面对同一个锁的lock操作，比如说在代码里有先对一个lock.lock()，lock.unlock()，lock.lock()
3. **volatile变量规则**：对一个volatile变量的写操作先行发生于后面对这个volatile变量的读操作，volatile变量写，再是读，必须保证是先写，再读
4. **传递规则**：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
5. **线程启动规则**：Thread对象的start()方法先行发生于此线程的每个一个动作，thread.start()，thread.interrupt()
6. **线程中断规则**：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
7. **线程终结规则**：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
8. **对象终结规则**：一个对象的初始化完成先行发生于他的finalize()方法的开始

总的来说，happens-before 规则制定了在一些特殊情况下，不允许编译器、指令器对代码进行指令重排，必须保证你的代码的有序性.

#### volatile 底层是如何基于内存屏障保障可见性和有序性

（1）lock指令：volatile保证可见性

对volatile修饰的变量，执行写操作的话，JVM会发送一条lock前缀指令给CPU，CPU在计算完之后会立即将这个值写回主内存，同时因为有MESI缓存一致性协议，所以各个CPU都会对总线进行嗅探，自己本地缓存中的数据是否被别人修改

如果发现别人修改了某个缓存的数据，那么CPU就会将自己本地缓存的数据过期掉，然后这个CPU上执行的线程在读取那个变量的时候，就会从主内存重新加载最新的数据了 

lock前缀指令 + MESI缓存一致性协议

（2）内存屏障：volatile禁止指令重排序

volatille是如何保证有序性的？加了volatile的变量，可以保证前后的一些代码不会被指令重排，这个是如何做到的呢？指令重排是怎么回事，volatile就不会指令重排，简单介绍一下，内存屏障机制是非常非常复杂的，如果要讲解的很深入

Load1：

int localVar = this.variable

Load2：

int localVar = this.variable2

LoadLoad屏障：Load1；LoadLoad；Load2，确保Load1数据的装载先于Load2后所有装载指令，他的意思，Load1对应的代码和Load2对应的代码，是不能指令重排的

Store1：

this.variable = 1

StoreStore屏障

Store2：

this.variable2 = 2 

StoreStore屏障：Store1；StoreStore；Store2，确保Store1的数据一定刷回主存，对其他cpu可见，先于Store2以及后续指令

LoadStore屏障：Load1；LoadStore；Store2，确保Load1指令的数据装载，先于Store2以及后续指令

StoreLoad屏障：Store1；StoreLoad；Load2，确保Store1指令的数据一定刷回主存，对其他cpu可见，先于Load2以及后续指令的数据装载

volatile的作用是什么呢？

volatile variable = 1

this.variable = 2 => store操作

int localVariable = this.variable => load操作 

对于volatile修改变量的读写操作，都会加入内存屏障

每个volatile写操作前面，加StoreStore屏障，禁止上面的普通写和他重排；每个volatile写操作后面，加StoreLoad屏障，禁止跟下面的volatile读/写重排

每个volatile读操作后面，加LoadLoad屏障，禁止下面的普通读和voaltile读重排；每个volatile读操作后面，加LoadStore屏障，禁止下面的普通写和volatile读重排



































