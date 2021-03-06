
## 一、如何制定性能调优标准

### 1. 为什么要做性能优化

> 好的系统性能调优不仅仅可以提高系统的性能，还能为公司节省资源

### 2. 什么时候开始介入调优

在项目开发的初期，我们没有必要过于在意性能优化，这样反而会让我们疲于性能优化，我们只需要在代码层面保证有效的编码，比如，减少磁盘 I/O 操作、降低竞争锁的使用以及使用高效的算法等等

***在系统编码完成之后，我们就可以对系统进行性能测试了***，根据产品经理提供的预期数据，进行压测，通过性能分析、统计工具来统计各项性能指标，看是否在预期范围之内

### 3. 有哪些参考因素可以体现系统的性能

先了解一下哪些计算机资源会成为系统的性能瓶颈

#### CPU
> 有的应用需要大量计算，他们会长时间、不间断地占用 CPU 资源，导致其他资源无法争夺到 CPU 而响应缓慢，从而带来系统性能问题<br>
例如，代码递归导致的无限循环，正则表达式引起的回溯，JVM 频繁的 FULL GC，以及多线程编程造成的大量上下文切换等，这些都有可能导致 CPU 资源繁忙

#### 内存
> Java堆内存的读写速度非常快，所以基本不存在读写性能瓶颈。但是由于内存成本要比磁盘高，相比磁盘，内存的存储空间又非常有限。所以当内存空间被占满，对象无法回收时，就会导致内存溢出、内存泄露等问题

#### 磁盘 I/O
>磁盘相比内存来说，存储空间要大很多，但磁盘 I/O 读写的速度要比内存慢，虽然目前引入的 SSD 固态硬盘已经有所优化，但仍然无法与内存的读写速度相提并论

#### 网络
> 网络带宽过低的话，对于传输数据比较大，或者是并发量比较大的系统，网络就很容易成为性能瓶颈

#### 异常
> Java 应用中，抛出异常需要构建异常栈，对异常进行捕获和处理，这个过程非常消耗系统性能。如果在高并发的情况下引发异常，持续地进行异常处理，那么系统的性能就会明显地受到影响

#### 数据库
> 大部分系统都会用到数据库，而数据库的操作往往是涉及到磁盘 I/O 的读写。大量的数据库读写操作，会导致磁盘 I/O 性能瓶颈，进而导致数据库操作的延迟性

#### 锁竞争
> 锁的使用可能会带来上下文切换，从而给系统带来性能开销。JDK1.6 之后，Java 为了降低锁竞争带来的上下文切换，对 JVM 内部锁已经做了多次优化。而如何合理地使用锁资源，优化锁资源，就需要了解更多的操作系统知识、Java 多线程编程基础，积累项目经验，并结合实际场景去处理相关问题

### 4. 衡量系统性能的指标

#### 响应时间

![响应时间](../docs/响应时间.jpg)

我们可以把响应时间自下而上细分为以下几种，其中数据库操作往往是整个请求链中最耗时的，一般一个接口的响应时间是在毫秒级
，但如果你的客户端嵌入了大量的逻辑处理，消耗的时间就有可能变长，从而成为系统的瓶颈


#### 吞吐量

吞吐量越大，性能越好，我们可以把吞吐量分为磁盘吞吐量和网络吞吐量

- 磁盘吞吐量
> 1. IOPS: 每秒的读写次数，关注的是随机读写性能，对于随机读写频繁的应用，是关键衡量指标
> 2. 数据吞吐量：指单位时间内可以成功传输的数据量，对于大量顺序读写频繁的应用，这是关键衡量指标

- 网络吞吐量
> 不仅仅跟带宽有关系，还跟 CPU 的处理能力、网卡、防火墙、外部接口以及 I/O 等紧密关联

- TPS、QPS
> 1. TPS是单位时间内处理事务的数量，代表一个事务的处理，可以包含了多次请求
> 2. QPS是单位时间内请求的数量,一次请求代表一个接口的一次请求到服务器返回结果
> 3. 区别：当一次用户操作只包含一个请求接口时，TPS和QPS没有区别。当用户的一次操作包含了多个服务请求时，这个时候TPS作为这次用户操作的性能指标就更具有代表性了


#### 计算机资源分配使用率
通常由 CPU利用率、内存使用率、磁盘 I/O、网络 I/O 来表示资源使用率，任何一项分配不合理，对整个系统性能的影响都是毁灭性的。


#### 负载承受能力
当系统压力上升时，你可以观察，系统响应时间的上升曲线是否平缓，这项指标能直观地反馈给你，系统所能承受的负载压力极限

系统负载指单位时间内系统正在运行或者等待的进程或线程数，代表系统的繁忙程度

***负载的数值代表的是 CPU 还没处理完的进程的数目，平均负载值是用移动平均法得出的，有n核，满载的负载值就是n***



### 小问题

#### 1. 避免业务异常生成栈追踪信息

遇到业务异常，只需要用字符串描述异常信息即可，自定义异常继承RuntimeException，将writableStackTrace设置为false
```
protected RuntimeException(String message, Throwable cause,
                               boolean enableSuppression,
                               boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
```

#### 2. 端口被大量CLOSE_WAIT占用

正常的关闭流程是：服务端在接收到客户端发送的关闭请求FIN后，会进入CLOSE_WAIT状态，同时发送ACK回去。在完成与客户端直接的通信操作之后，再向客户端发送FIN，进入LAST_ACK状态

- 原因一：可能是服务端在关闭连接之前还有逻辑未处理完或出错，导致没有发FIN包

- 原因二：服务设置超时时间过短，服务已经主动关闭，但是上游还未超时没有发送FIN包


## 二、如何制定性能调优策略
### 1. 性能测试应注意的问题
#### I. 热身
当代码被频繁执行时，会被虚拟机判定为热点代码，随后虚拟机便通过JIT把热点代码编译、优化，并存储在内存，之后的运行就直接从内存中获取代码

这也是为什么刚开始运行时，速度会比较慢

#### II. 性能测试不稳定
主要影响因素有网络波动、其他进程占用资源或Java垃圾回收。

我们可以通过多次测试，只要保证平均值在合理范围内即可，当波动不大时，性能测试就是通过的

### 2. 合理分析

#### I. 分析需要的数据
需要用到测试接口的平均、最大和最小吞吐量，响应时间。服务器的CPU、内存、I/O、网络I/O使用率，JVM的GC频率

#### II. 查找问题
可以用自下而上的方式查找问题：
> 1. 系统层面。观察系统的CPU、内存、I/O、网络，如有异常再通过命令查找异常日志并分析
> 2. JVM层面。 主要观察GC频率以及内存分配是否存在异常
> 3. 业务层面。 例如编程是否存在不合理的地方，读写数据是否存在瓶颈等

#### III. 调优策略
解决性能问题则可以采用自上而下的方式逐级优化：
> 1. 优化代码。应用层的问题代码往往会因耗尽系统资源而暴露出来，如内存溢出、for循环LinkedList等。可以用JProfile进行代码分析
> 2. 优化设计。可以使用合适的设计模式来精简代码和提高整体性能
> 3. 优化算法。合适的算法能大大提升系统性能
> 4. 时间换空间。 适用对查询速度没有较高要求而对存储空间苛刻的情况。例如分页查询
> 5. 空间换时间。 通过牺牲部分空间来到达预期的获取速度。比如缓存技术、数据冗余等等
> 6. 参数调优。 JVM的堆栈大小、垃圾收集器、容器线程池等


### 3. 兜底策略
无论我们的系统优化得有多好，还是会存在承受极限，所以为了保证系统的稳定性，我们还需要采用一些兜底策略
#### I. 限流
根据测试接口的TPS对系统的入口设置最大访问限制，同时采取熔断措施，友好地返回没有成功的请求

#### II. 智能扩容
当访问量达到某个阈值时，系统可以实现自动扩容。可以使用Kubernetes来实现


## 三、Java字符串性能优化

### 1. 如何构建超大字符串
用加号把字符串和变量拼接时，会被编译器优化成StringBuilder

在多线程编程时，StringBuilder会存在线程安全问题，可以使用StringBuffer，不过性能会下降

### 2. 使用String.intern()节省内存
每次赋值时使用String的intern方法，如果常量池中有相同值，就会重复使用该对象

如果对空间要求高于时间要求，且存在大量重复字符串时，可以考虑使用常量池存储。

常量池的实现类似HashTable。存储的数据越多，遍历的时间复杂度就会增加

### 3. 复习巩固
JDK8以后，字符串常量池存在于Java堆中，唯一由java.lang.String管理。它和运行时常量池、类文件常量池无关


## 四、ArrayList与LinkedList

### 1. 为什么ArrayList中的elementData属性会被transient修饰
主要属性：size、elementData、default_capacity

因为ArrayList是基于数组实现的，由于数组是基于动态扩展的，所以不是所有被分配的空间都存储了元素。

为了避免序列化没有存储数据的内存空间，它内部会提供writeObject和readObject来完成序列化

>使用 transient 修饰数组，是防止对象数组被其他外部方法序列化


### 2. 通过构建函数指定数组初始大小，有助于减少扩容
当清楚存储数据大小时，指定数组初始大小，并使用数组未尾添加元素，那么在大量新增元素的场景下，性能反而比List集合要好

数据删除元素时，需要重组，当元素越靠前，重组的开销就越大

### 3. LinkedList的实现
LinkedList 就是由 Node 结构对象连接而成的一个双向链表，主要属性有size、first、last，它们都被transient修饰。

Node结构包含元素内容item以及前后指针

> 1.7之前，LinkedList中只包含一个Entry结构的header属性，并在初始化的时候初始化一个空的Entry，它的前后指针
>都指向自己，形成一个循环双向链表


> 1.7以后，Entry结构换成了Node结构，并新增了Node结构的first属性以及last属性
>
>好处：清晰表达了链头链尾的概念，并且链头和链尾的插入删除操作更加便捷了

由于LinkedList内存不连续，所以不能实现快速随机访问，自然不能实现RandomAccess接口

first、last被transient关键字修饰是因为序列化时不会只对首尾进行序列化，LinkedList也是自行实现writeObject和readObject的


### 4. 为什么ArrayList没有负载因子而HashMap有
HashMap有负载因子是基于折衷的考虑，因为数组太短会导致哈希冲突增加，从而链表增长导致查询效率下降。而数组太长会导致新增元素性能下降

### 5. ArrayList遍历比LinkedList快的底层原因
数组的实现是在内存当中是一块连续的内存空间，而链表所有元素可能分布在内存的不同位置，对于数组这种数据结构来说对CPU读是非常友好的，不管是CPU从内存读数据读到高速缓存还是线程从磁盘读数据到内存时，都不只是读取需要的那部分数据，而是读取相关联的某一块地址数据，这样的话对于在遍历数组的时候，在一定程度上提高了CPU高速缓存的命中率，减少了CPU访问内存的次数从而提高了效率


### 注意
在集合中进行remove操作时，不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator方式，如果并发操作，需要对 Iterator 对象加锁。



## 四、如何解决高并发下I/O瓶颈

### 1. 信息最小的存储单元是字节，为什么I/O还分字符流操作
因为字符到字节必须经过转码，这个过程十分耗时，如果不知道编码类型很容易出现乱码问题。
所以I/O直接提供了操作字符的接口

### 2. 传统I/O性能问题
#### I. 多次内存复制

- JVM会发出read()系统调用，并通过read系统调用向内核发起读请求
- 内核向硬件发送读指令，并等待读就绪
- 内核把要读的数据复制到指向的内核缓存中
- 操作系统内核将数据复制到用户空间缓存区，然后read系统调用返回
![多次内存复制](../docs/IO_data_copy.jpg)
数据从外部设备复制到内核空间，再从内核空间复制到用户空间，这发生了两次内存复制。

这种操作会导致不必要的上下文切换和数据拷贝，影响性能

#### II. 阻塞
传统I/O中的read()是一个while循环操作，如果数据没有就绪，这个读操作会一直被挂起来，用户线程处于阻塞状态。

当有大量线程请求时，一旦发生线程阻塞，这些线程将会不断地抢夺 CPU 资源，导致大量的上下文切换，增加系统性能的开销

### 3. 如何优化I/O操作
#### I. 使用缓存区优化读写操作

NIO是以块(Block)为基本单位处理数据，其中还有两个重要的组件是缓冲区(Buffer)和通道(Channel)

Buffer 是一块连续的内存块，是 NIO 读写数据的中转地。Channel 表示缓冲数据的源头或者目的地，它用于读取缓冲或者写入数据，是访问缓冲的接口

#### II. 使用DirectBuffer减少内存复制


在Java中，首先从Java堆内存中拷贝到临时的直接内存，再拷贝到内核空间，再到输出设备。
经历了三次内存拷贝，其中前两次为用户空间拷贝


而DirectBuffer直接将数据保存到非堆内存，从而减少一次数据拷贝

另外，MappedByteBuffer通过本地类进行文件内存映射，map() 系统调用方法会直接将文件从硬盘拷贝到用户空间


### 4. Tomcat参数调优
Tomcat 中，BIO、NIO 是基于主从 Reactor 线程模型实现的。

***在BIO中***，Acceptor只负责监听新的连接，一旦建立连接监听到I/O操作，将会交给Worker线程中，它专门负责I/O读写操作

***在NIO中***，Tomcat新增了一个Poller线程池，Acceptor监听到连接后，会将请求发送给Poller缓冲队列。在Poller中，维护了一个Selector对象，
当Poller从队列取出一个连接后，注册到该Selector中，然后通过遍历 Selector，找出其中就绪的 I/O 操作，并使用 Worker 中的线程处理相应的请求

> acceptorThreadCount:该参数代表 Acceptor 的线程数量，在请求客户端的数据量非常巨大的情况下，可以适当地调大该线程数量来提高处理请求连接的能力，默认值为 1

> maxThreads：专门处理 I/O 操作的 Worker 线程数量，默认是 200，可以根据实际的环境来调整该参数，但不一定越大越好

> acceptCount: Tomcat 的 Acceptor 线程是负责从 accept 队列中取出该 connection，然后交给工作线程去执行相关操作，这里的 acceptCount 指的是 accept 队列的大小。
>
> 当 Http 关闭 keep alive，在并发量比较大时，可以适当地调大这个值。而在 Http 开启 keep alive 时，因为 Worker 线程数量有限，Worker 线程就可能因长时间被占用，而连接在 accept 队列中等待超时。如果 accept 队列过大，就容易浪费连接

> maxConnections：表示有多少个 socket 连接到 Tomcat 上。在 BIO 模式中，一个线程只能处理一个连接，一般 maxConnections 与 maxThreads 的值大小相同；在 NIO 模式中，一个线程同时处理多个连接，maxConnections 应该设置得比 maxThreads 要大的多，默认是 10000
## 五、避免使用Java序列化
### 1. Java的缺陷
1. 无法跨语言：Java 序列化目前只适用基于 Java 语言实现的框架，其它语言大部分都没有使用 Java 的序列化框架

2. 易被攻击：readObject()可以将类路径上几乎所有实现了Serializable接口都实例化，也就是说该方法可以执行任何类型的代码

3. 序列化之后的流太大：序列化后的二进制数组越大，占用的存储空间就越多，存储硬件的成本就越高。如果我们是进行网络传输，则占用的带宽就更多，这时就会影响到系统的吞吐量

4. 序列化性能太差：序列化的速度慢，就会影响网络通信的效率，从而增加系统的响应时间

### 2. 版本号在序列化中的作用

在Class类文件中默认会有一个serialNo作为序列化对象的版本号，无论在序列化方还是在反序列化方的class类文件中都存在一个默认序列号，在序列化时，会将该版本号加载进去，在反序列化时，会校验该版本号


### 3. 如何优化RPC通信

1. 选择合适的通信协议：为了保证数据的可靠性，通常采用TCP协议。

2. 基于TCP实现Socket通信：客户端多，则使用短连接避免请求长时间占用连接，浪费系统资源。客户端少可以使用长连接，省去大量的TCP建立和关闭操作

3. 使用Netty来优化Socket通信

4. 使用优秀的序列化框架进行编码解码

5. 调整Linux的TCP参数，可用sysctl -a | grep net.xxx查看

![Linux_TCP参数](../docs/Linux_TCP.jpg)

## 六、深入了解同步锁的优化方法

### 1. 自旋锁
在锁竞争不激烈且锁占用时间非常短的场景下，自旋锁可以提高系统性能。一旦锁竞争激烈或锁占用的时间过长，自旋锁将会导致大量的线程一直处于 CAS 重试状态，占用 CPU 资源，反而会增加系统性能开销。

高负载、高并发的场景下，我们可以通过设置 JVM 参数来关闭自旋锁，优化系统性能
> 1. -XX:-UseSpinning //参数关闭自旋锁优化(默认打开) 
> 2. -XX:PreBlockSpin //参数修改默认的自旋次数。JDK1.7后，去掉此参数，由jvm控制


### 2. Synchronized和Lock的选择
从性能方面上来说，在并发量不高、竞争不激烈的情况下，Synchronized 同步锁由于具有分级锁的优势，性能上与 Lock 锁差不多；但在高负载、高并发的情况下，Synchronized 同步锁由于竞争激烈会升级到重量级锁，性能则没有 Lock 锁稳定。

### 3. 处理器如何保证复杂内存操作的原子性
当处理器要操作一个共享变量的时候，其在总线上会发出一个 Lock 信号，这时其它处理器就不能操作共享变量了，该处理器会独享此共享内存中的变量。但总线锁定在阻塞其它处理器获取该共享变量的操作请求时，也可能会导致大量阻塞，从而增加系统的性能开销。

于是，后来的处理器都提供了缓存锁定机制，也就说当某个处理器对缓存中的共享变量进行了操作，就会通知其它处理器放弃存储该共享资源或者重新读取该共享资源。目前最新的处理器都支持缓存锁定机制。

### 4. 多少线程数量合适。
在并发程序中，并不是启动更多的线程就能让程序最大限度地并发执行

线程数量设置太小，会导致程序不能充分地利用系统资源；线程数量设置太大，又可能带来资源的过度竞争，导致上下文切换带来额外的系统开销

### 5. 监控上下文切换
可以使用 Linux 内核提供的 vmstat 命令，来监视 Java 程序运行过程中系统的上下文切换频率

vmstat 1 3 代表 每秒收集一次性能指标，总共获取3次。


如果是监视某个应用的上下文切换，就可以使用 pidstat 命令监控指定进程的 Context Switch 上下文切换

jstack 最常用的功能就是使用 jstack pid 命令查看线程堆栈信息，通常是结合 pidstat -p pid -t 一起查看具体线程的状态，也经常用来排查一些死锁的异常。

### 6. 竞争锁优化
多线程对锁资源的竞争会引起上下文切换，还有锁竞争导致的线程阻塞越多，上下文切换就越频繁，系统的性能开销也就越大。由此可见，在多线程编程中，锁其实不是性能开销的根源，竞争锁才是

#### I. 减少锁的持有时间
#### II. 降低锁的粒度（锁分离、锁分段）
#### III. 非阻塞乐观锁替代竞争锁

### 7. 优化wait()/notify()的使用，减少上下文切换
使用 Lock 锁结合 Condition 接口替代 Synchronized 内部锁中的 wait / notify，实现等待／通知。这样做不仅可以解决上述的 Object.wait(long) 无法区分的问题，还可以解决线程被过早唤醒的问题

### 8. 减少垃圾回收的频率可以有效地减少上下文
年轻代是部分对象复制过程，是不会存在stop the world的发生。如果存在对象移动，使用对象的线程是会被挂起的，这个过程存在上下文切换。

### 9. volatile的读写不会导致上下文切换
反编译文件我们可以看到通过volatile修饰的共享变量，在写入操作的时候会多一个Lock前缀这样的指令，当操作系统执行时会由于这个指令，将当前处理器缓存的数据写回系统内存中，并通知其他处理器中的缓存失效

### 10. Synchronized和Lock释放获取锁时的上下文切换
AQS挂起是通过LockSupport中的park进入阻塞状态，这个过程也是存在进程上下文切换的。但被阻塞的线程再次获取锁时，不会产生进程上下文切换，而synchronized阻塞的线程每次获取锁资源都要通过系统调用内核来完成，这样就比AQS阻塞的线程更消耗系统资源了

AQS阻塞线程再次获取锁时，是通过state以及CAS操作判断，只有没有竞争成功时，才会再次被挂起，这样可以尽量减少上下文切换

## 七、并发容器的使用
HashTable、ConcurrentHashMap是基于HashMap实现的，对于小数据量的存取比较有优势

ConcurrentHashMap在大数据量的情况下，链表会转换为红黑树，红黑树在并发的情况下，删除和插入会有个平衡的过程，会涉及到大量节点，因此锁竞争的代价相对较高

ConcurrentSkipListMap是基于TreeMap的设计原理实现的，前者基于跳表实现的，后者是基于红黑树的。
ConcurrentSkipListMap 的特点是存取平均时间复杂度是 O（log（n）），适用于大数据量存取的场景，最常见的是基于跳跃表实现的数据量比较大的缓存

跳表的操作针对局部，需要锁住的节点少，因此并发性能会好点。在非线程安全的Map容器中，，基于红黑树实现的 TreeMap 在单线程中的性能表现得并不比跳跃表差，所以TreeMap容器适合存放大数据



### 1. HashTable vs ConcurrentHashMap
Hashtable 使用 Synchronized 同步锁修饰了 put、get、remove 等方法，因此在高并发场景下，读写操作都会存在大量锁竞争，给系统带来性能开销。

ConcurrentHashMap 在保证线程安全的基础上兼具了更好的并发性能。

在强一致的场景中 ConcurrentHashMap 就不适用，原因是 ConcurrentHashMap 中的 get、size 等方法没有用到锁，ConcurrentHashMap 是弱一致性的，因此有可能会导致某次读无法马上获取到写入的数据



![容器对比](../docs/容器对比.jpg)

### 2. 使用什么队列来实现抢购
可以用ConcurrentLinkedQueue

1. 抢购一般都是写多读少，该队列是基于链表实现的，所以新增和删除元素性能较高
2. 写数据时通过cas操作，性能较高
3. 该队列无界，需要控制容量

### 3. CopyOnWriteArrayList
适合于读多写少的场景，它实现了读无锁操作，写操作时通过操作底层数组的新副本实现，读写分离。

### 4. 为什么ConcurrentHashMap和HashTable的key和value不能为空，而HashMap却可以
线程安全的map不允许key/value为null，因为在多线程条件下，对于key/value是否真的存在有歧义


## 八、如何优化垃圾回收机制

### 1. GC衡量指标
- 吞吐量
> GC的吞吐量一般不低于95%
>
>这里的吞吐量是指应用程序所花费的时间和系统总运行时间的比值。系统总运行时间 = 应用程序耗时 +GC 耗时

- 停顿时间
> 指垃圾收集器正在运行时，应用程序的暂停时间

- 垃圾回收频率
> 通常垃圾回收的频率越低越好
>
> 增大堆内空间 -> 降低回收发生的频率 + 增加回收的停顿时间

### 2. GC调优策略
- 降低Minor GC频率

单次 Minor GC 时间是由两部分组成：T1（扫描新生代）和 T2（复制存活对象）。通常在虚拟机中，复制对象的成本要远高于扫描成本。因此，单次 Minor GC 时间更多取决于 GC 后存活对象的数量，而非 Eden 区的大小

> 如果堆中短期对象较多，增加年轻代空间，会减低Minor GC的频率，但不会显著增加单次Minor GC的时间。
> <br>增大了空间 -> 回收时间变长 -> 如果对象生存时长小于回收的间隔时间 -> 对象会在复制前被回收掉 -> 增加了T1时间，减少了T2时间

> 在堆内存中存在较多长期存活的对象，增加年轻代空间，反而会增加Minor GC的时间


- 降低Full GC频率

由于堆内存空间不足或老年代对象太多，会触发 Full GC，频繁的 Full GC 会带来上下文切换，增加系统的性能开销

> 减少大对象的创建

> 增大堆内空间



### 3. JVM内存分配的调优过程



线上的JVM内存溢出事故往往是应用程序创建对象导致的内存回收对象难，属于编程问题。

JVM内存分配不合理最直接的表现就是频繁GC。如果发现频繁的GC，且是正常的对象创建和回收，这个时候就需要考虑调整JVM内存分配了

- 分析GC日志
> 先吧GC日志dump下来：<br>
>-XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Xloggc:/log/heapTest.log <br>
> GC查看工具分析日志

- 调整堆内存空间减少Full GC
> -Xms:堆初始大小    -Xmx：堆最大值

- 调整年轻代减少Minor GC

- 固定 Eden 区的占用比例，来调优 JVM 的内存分配性能
> 通过 -XX:-UseAdaptiveSizePolicy 关闭AdaptiveSizePolicy







## 九、Meeting调优

### 1. UUID 

UUID的本质，一个128位的数值。转成16进制就是32位字符。

#### 1.1 UUID如何生成



### 1. 调优前的准备

了解接口的作用：查询前后两个月，用户参加或预约的会议记录

- 查询近三个月，最多的会议场数--300场

> **select** hex(mr.staff_id),count(*) **as** c **from** meeting_record mr left join meeting m **on** mr.meeting_no = m.meeting_no **where** start_time between '2020-12-01 00:00:00' and '2021-02-01 00:00:00' and mr.staff_id is not null **GROUP** **by** mr.staff_id **order** **by** c **DESC**

- 查询单场会议人数最多的会议记录--

