# Java 面试题题库（全难度覆盖）

> 共 120+ 高频面试题，按主题分类，每题标注难度（初级/中级/高级）和简答要点。
> 适合：校招 / 1-3 年 / 3-5 年 Java 后端面试准备。

---

## 一、Java 基础（初级）

### Q1. Java 为什么是值传递不是引用传递？
**难度**：初级  
**关键词**：值传递、对象引用  
**简答要点**：Java 中方法参数传递的是实参的副本（基本类型传值，对象传引用的副本）。对引用副本修改不影响原引用指向，所以是值传递。

### Q2. String、StringBuilder、StringBuffer 的区别？
**难度**：初级  
**关键词**：不可变、线程安全、性能  
**简答要点**：String 不可变；StringBuilder 可变但非线程安全（单线程最快）；StringBuffer 可变且线程安全（性能最慢）。

### Q3. equals() 和 == 的区别？
**难度**：初级  
**关键词**：引用比较、内容比较  
**简答要点**：`==` 比较引用地址；`equals()` 默认比较引用，但 String/Integer 等重写了 equals 用于比较内容。重写 equals 必须重写 hashCode。

### Q4. Java 中的 8 种基本数据类型是什么？
**难度**：初级  
**关键词**：byte/short/int/long/float/double/boolean/char  
**简答要点**：4 种整型（byte 1B / short 2B / int 4B / long 8B）、2 种浮点、1 种布尔、1 种字符。注意自动装箱/拆箱的坑。

### Q5. 接口和抽象类的区别？
**难度**：初级  
**关键词**：多继承、默认方法、设计意图  
**简答要点**：接口强调行为规范（Java 8+ 有默认方法）；抽象类强调代码复用（可有状态）。Java 单继承限制下，接口实现多继承效果。

### Q6. 深拷贝和浅拷贝的区别？
**难度**：初级  
**关键词**：Cloneable、对象引用、嵌套对象  
**简答要点**：浅拷贝复制引用（嵌套对象共享）；深拷贝递归复制所有对象。实现深拷贝可用序列化、手动 clone 或拷贝构造函数。

### Q7. Java 反射是什么？有什么用途？
**难度**：初级  
**关键词**：运行时、Class 对象、动态代理  
**简答要点**：反射允许程序在运行时获取类的信息并操作对象。用途：框架（Spring IOC）、动态代理、测试工具、序列化。

### Q8. 动态代理的两种实现方式？
**难度**：初级  
**关键词**：JDK 代理、CGLIB  
**简答要点**：JDK 动态代理（基于接口）；CGLIB（基于继承，生成子类）。Spring AOP 默认策略：有接口用 JDK，无接口用 CGLIB。

### Q9. Java 异常体系？Checked 和 Unchecked 的区别？
**难度**：初级  
**关键词**：Exception、Error、RuntimeException  
**简答要点**：Checked（编译期必须处理，如 IOException）；Unchecked（运行时异常，如 NullPointerException）。Error 表示系统级错误（OOM/StackOverflow）。

### Q10. try-catch-finally 中 finally 一定会执行吗？
**难度**：初级  
**关键词**：return、System.exit、线程终止  
**简答要点**：一般会执行，除了 System.exit(0)、JVM 崩溃、线程被强制终止。finally 中不要写 return（会覆盖 try 的 return）。

---

## 二、集合框架（初级 → 中级）

### Q11. ArrayList 和 LinkedList 的区别？
**难度**：初级  
**关键词**：数组 vs 链表、随机访问、插入删除  
**简答要点**：ArrayList 底层动态数组（随机访问 O(1)）；LinkedList 双向链表（头尾插入 O(1)，随机访问 O(n)）。实际开发 ArrayList 更常用。

### Q12. HashMap 的底层实现原理？
**难度**：中级  
**关键词**：数组+链表+红黑树、hash、扩容  
**简答要点**：JDK 8 是数组+链表+红黑树（链表长度>8 且数组长度>64 转红黑树）。put 时计算 hash 定位桶，冲突时链表/树存储。扩容：容量翻倍，重新哈希。

### Q13. HashMap 为什么线程不安全？
**难度**：中级  
**关键词**：并发 put、数据覆盖、size 不一致  
**简答要点**：多线程并发 put 时，可能：① hash 相同覆盖数据；② 扩容时死循环（JDK 7）或数据丢失（JDK 8）。解决方案：ConcurrentHashMap。

### Q14. ConcurrentHashMap 的实现原理？
**难度**：高级  
**关键词**：CAS、synchronized、分段锁演进  
**简答要点**：JDK 7 是 Segment 分段锁；JDK 8 改为 CAS + synchronized（锁粒度到桶）。size 用 baseCount + CounterCell 数组统计。

### Q15. HashMap 的扩容机制？
**难度**：中级  
**关键词**：resize、threshold、capacity、rehash  
**简答要点**：当 size > capacity * loadFactor（默认 0.75）时扩容。扩容为原来 2 倍，重新计算每个元素的桶位置（高位不变或高位+旧容量）。

### Q16. TreeMap 和 HashMap 的区别？
**难度**：初级  
**关键词**：红黑树、有序、无序  
**简答要点**：HashMap 无序（O(1)）；TreeMap 基于红黑树，按 key 排序（O(log n)）。需要按 key 排序时用 TreeMap。

### Q17. HashSet 的底层实现？
**难度**：初级  
**关键词**：HashMap、去重  
**简答要点**：HashSet 底层是 HashMap，元素作为 key，value 是固定对象 PRESENT。去重依赖 hashCode 和 equals。

### Q18. 为什么 ArrayList 的默认容量是 10？扩容策略是什么？
**难度**：初级  
**关键词**：初始容量、1.5 倍扩容  
**简答要点**：默认容量 10 是经验值（平衡内存和扩容开销）。扩容：新容量 = 旧容量 * 1.5（位运算 oldCapacity >> 1）。

### Q19. 如何实现线程安全的 List？
**难度**：中级  
**关键词**：Collections.synchronizedList、CopyOnWriteArrayList、Vector  
**简答要点**：Vector（全锁，性能差）；Collections.synchronizedList（包装锁）；CopyOnWriteArrayList（写时复制，读多写少场景）。

### Q20. Iterator 和 ListIterator 的区别？
**难度**：初级  
**关键词**：单向遍历、双向遍历  
**简答要点**：Iterator 只能正向遍历；ListIterator 支持双向遍历、添加、替换元素，仅适用于 List。

---

## 三、并发编程（中级 → 高级）

### Q21. 线程的 6 种状态？
**难度**：初级  
**关键词**：NEW/RUNNABLE/BLOCKED/WAITING/TIMED_WAITING/TERMINATED  
**简答要点**：NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → RUNNABLE → TERMINATED。

### Q22. synchronized 和 ReentrantLock 的区别？
**难度**：中级  
**关键词**：隐式锁 vs 显式锁、公平性、中断  
**简答要点**：synchronized 是隐式锁（JVM 实现）；ReentrantLock 是显式锁（API），支持公平锁、可中断、超时。性能：JDK 6 后 synchronized 优化后接近。

### Q23. volatile 关键字的作用？
**难度**：中级  
**关键词**：可见性、有序性、禁止重排序  
**简答要点**：保证变量对所有线程立即可见；禁止指令重排序（内存屏障）。不保证原子性（如 i++）。

### Q24. 什么是 JMM（Java 内存模型）？
**难度**：高级  
**关键词**：主内存、工作内存、happens-before  
**简答要点**：JMM 定义了多线程下变量的访问规则。所有变量在主内存，线程工作内存是副本。happens-before 规则保证可见性和有序性。

### Q25. 什么是 happens-before 规则？
**难度**：高级  
**关键词**：程序顺序、监视器锁、volatile 变量  
**简答要点**：8 条规则：程序顺序、监视器锁、volatile 变量、线程启动/终止、中断、finalizer、传递性。保证操作 A 对 B 可见。

### Q26. CAS 是什么？有什么问题？
**难度**：中级  
**关键词**：Compare And Swap、ABA、原子性  
**简答要点**：CAS 是比较内存值与预期值，相同则更新。问题：ABA（用 AtomicStampedReference 解决）、自旋开销、只能保证单个变量原子性。

### Q27. AQS 是什么？
**难度**：高级  
**关键词**：AbstractQueuedSynchronizer、state、CLH 队列  
**简答要点**：AQS 是并发包核心框架，维护 state 和 FIFO 队列。子类通过重写 tryAcquire/tryRelease 实现锁逻辑。ReentrantLock、CountDownLatch 都基于 AQS。

### Q28. 线程池的核心参数？
**难度**：中级  
**关键词**：7 大参数、拒绝策略  
**简答要点**：corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue、threadFactory、handler。4 种拒绝策略：AbortPolicy（默认）、CallerRunsPolicy、DiscardPolicy、DiscardOldestPolicy。

### Q29. 为什么不建议用 Executors 创建线程池？
**难度**：中级  
**关键词**：FixedThreadPool、CachedThreadPool、OOM  
**简答要点**：FixedThreadPool 和 SingleThreadPool 的队列是 LinkedBlockingQueue（无界，可能 OOM）；CachedThreadPool 允许创建最大线程数（可能 CPU 过载）。

### Q30. CountDownLatch、CyclicBarrier、Semaphore 的区别？
**难度**：中级  
**关键词**：倒计数、屏障、信号量  
**简答要点**：CountDownLatch（一次使用，等事件）；CyclicBarrier（可重置，线程互等）；Semaphore（控制并发数，如连接池）。

### Q31. ThreadLocal 是什么？有什么问题？
**难度**：中级  
**关键词**：线程局部变量、内存泄漏  
**简答要点**：ThreadLocal 为每个线程保存独立副本。问题：线程池复用时未 remove 会内存泄漏；父子线程传递用 InheritableThreadLocal。

### Q32. 什么是死锁？如何避免？
**难度**：中级  
**关键词**：互斥、占有等待、循环等待  
**简答要点**：四个必要条件：互斥、占有等待、非抢占、循环等待。避免：破坏循环等待（按序加锁）、超时、死锁检测。

### Q33. CompletableFuture 的常用方法？
**难度**：中级  
**关键词**：异步编程、thenApply、thenCompose、allOf  
**简答要点**：thenApply（同步转换）、thenCompose（扁平化）、thenCombine（合并）、exceptionally（异常处理）、allOf（全部完成）。

### Q34. 虚拟线程（Virtual Thread）是什么？
**难度**：高级  
**关键词**：Java 21、协程、轻量级线程  
**简答要点**：虚拟线程是轻量级线程，由 JVM 调度，不绑定 OS 线程。适合 IO 密集型任务，不适合 CPU 密集。创建成本低（KB 级内存）。

---

## 四、JVM（中级 → 高级）

### Q35. JVM 内存区域划分？
**难度**：中级  
**关键词**：堆、栈、方法区、元空间、程序计数器  
**简答要点**：线程私有（程序计数器、栈、本地方法栈）；线程共享（堆、方法区/元空间）。堆最大，栈默认 1MB。

### Q36. 堆和栈的区别？
**难度**：初级  
**关键词**：对象 vs 局部变量、GC、大小  
**简答要点**：堆存对象（GC 管理）；栈存局部变量和方法调用（自动回收）。栈小（1MB）堆大（几百 MB 到 GB）。

### Q37. 什么是方法区？元空间是什么？
**难度**：中级  
**关键词**：永久代、元空间、直接内存  
**简答要点**：方法区存类信息、常量、静态变量。JDK 8 之前叫永久代（堆内），JDK 8 后改为元空间（本地内存，避免 OOM）。

### Q38. JVM 垃圾回收算法有哪些？
**难度**：中级  
**关键词**：标记-清除、复制、标记-整理、分代  
**简答要点**：标记-清除（碎片）；复制（新生代）；标记-整理（老年代）；分代收集（新生代用复制，老年代用标记-整理）。

### Q39. 常见的垃圾回收器有哪些？
**难度**：高级  
**关键词**：Serial、Parallel、CMS、G1、ZGC、Shenandoah  
**简答要点**：Serial（单线程）；Parallel（吞吐量优先）；CMS（低延迟，已废弃）；G1（默认，Region 化）；ZGC（<1ms 停顿）；Shenandoah（并发压缩）。

### Q40. 什么是 GC Roots？
**难度**：中级  
**关键词**：可达性分析、根对象  
**简答要点**：GC Roots 是可达性分析的起点：栈引用、静态变量、常量、JNI 引用。从 Roots 不可达的对象会被回收。

### Q41. 类加载过程？
**难度**：中级  
**关键词**：加载、验证、准备、解析、初始化  
**简答要点**：加载（读字节码）→ 验证（格式检查）→ 准备（分配内存、赋零值）→ 解析（符号引用转直接引用）→ 初始化（执行 static 代码）。

### Q42. 双亲委派模型是什么？为什么要打破？
**难度**：高级  
**关键词**：Bootstrap、Extension、Application、SPI  
**简答要点**：类加载器收到请求先委托父加载器，防止重复加载。打破场景：Tomcat（Web 应用隔离）、SPI（JDBC）、OSGi。

### Q43. 如何排查 JVM 问题？
**难度**：高级  
**关键词**：jps、jstat、jmap、jstack、Arthas  
**简答要点**：jps（进程列表）；jstat（GC 统计）；jmap（堆 dump）；jstack（线程 dump）；Arthas（在线诊断）。线上 CPU 100% 先 jstack 找线程。

### Q44. 什么是 OOM？常见类型？
**难度**：中级  
**关键词**：Java heap space、Metaspace、Direct buffer  
**简答要点**：Java heap space（对象过多）；Metaspace（类过多）；Direct buffer（NIO 直接内存）；Unable to create new native thread（线程过多）。

### Q45. 如何调优 JVM 参数？
**难度**：高级  
**关键词**：-Xms、-Xmx、-XX:NewRatio、-XX:+UseG1GC  
**简答要点**：-Xms/-Xmx 设堆初始/最大；-Xmn 新生代大小；-XX:NewRatio 老年代比例；-XX:+UseG1GC 启用 G1。调优基于 GC 日志和监控数据。

---

## 五、Spring 全家桶（中级 → 高级）

### Q46. Spring IOC 是什么？
**难度**：初级  
**关键词**：控制反转、依赖注入、BeanFactory  
**简答要点**：IOC 把对象创建权交给容器，通过依赖注入管理对象关系。好处：解耦、易测试。核心：BeanFactory 和 ApplicationContext。

### Q47. Spring AOP 是什么？
**难度**：初级  
**关键词**：切面编程、动态代理、横切关注点  
**简答要点**：AOP 把日志、事务等横切关注点抽离，通过动态代理织入。实现：JDK 代理（接口）或 CGLIB（类）。

### Q48. Bean 的生命周期？
**难度**：中级  
**关键词**：实例化、属性注入、初始化、销毁  
**简答要点**：实例化 → 属性注入 → BeanNameAware → BeanFactoryAware → BeanPostProcessor 前置 → InitializingBean → init-method → BeanPostProcessor 后置 → 使用 → 销毁。

### Q49. Spring 如何解决循环依赖？
**难度**：高级  
**关键词**：三级缓存、提前暴露  
**简答要点**：三级缓存：singletonObjects（成品）、earlySingletonObjects（半成品）、singletonFactories（工厂）。通过提前暴露半成品解决 setter 注入的循环依赖（构造器注入不行）。

### Q50. @Transactional 失效的场景？
**难度**：高级  
**关键词**：同类方法调用、异常类型、传播行为  
**简答要点**：① 同类方法调用（绕过代理）；② 异常被 catch 吞掉；③ 异常类型不是 RuntimeException（默认）；④ 方法不是 public；⑤ 数据库引擎不支持（如 MyISAM）。

### Q51. Spring Boot 自动装配原理？
**难度**：中级  
**关键词**：@EnableAutoConfiguration、spring.factories、条件注解  
**简答要点**：@SpringBootApplication 包含 @EnableAutoConfiguration，通过 spring.factories 加载自动配置类。条件注解（@ConditionalOnClass、@ConditionalOnMissingBean）控制是否生效。

### Q52. Spring MVC 的请求处理流程？
**难度**：中级  
**关键词**：DispatcherServlet、HandlerMapping、HandlerAdapter  
**简答要点**：请求 → DispatcherServlet → HandlerMapping（找 Handler）→ HandlerAdapter（执行 Handler）→ ModelAndView → ViewResolver → 响应。

### Q53. Spring 中的设计模式？
**难度**：中级  
**关键词**：工厂、单例、代理、模板方法  
**简答要点**：工厂（BeanFactory）；单例（默认 scope）；代理（AOP）；模板方法（JdbcTemplate）；观察者（ApplicationEvent）；适配器（HandlerAdapter）。

### Q54. @Autowired 和 @Resource 的区别？
**难度**：初级  
**关键词**：byType vs byName  
**简答要点**：@Autowired 默认按类型注入（可加 @Qualifier 指定名称）；@Resource 默认按名称注入（Spring 提供 vs JDK 提供）。

### Q55. Spring 事务的传播行为？
**难度**：高级  
**关键词**：REQUIRED、REQUIRES_NEW、NESTED  
**简答要点**：7 种：REQUIRED（默认）、SUPPORTS、MANDATORY、REQUIRES_NEW、NOT_SUPPORTED、NEVER、NESTED。最常用 REQUIRED 和 REQUIRES_NEW。

### Q56. Spring 事务的隔离级别？
**难度**：中级  
**关键词**：READ_UNCOMMITTED、READ_COMMITTED、REPEATABLE_READ、SERIALIZABLE  
**简答要点**：4 种，默认 READ_COMMITTED（MySQL 默认 REPEATABLE_READ）。隔离级别越高，并发越低，性能越差。

---

## 六、MySQL & 数据库（中级）

### Q57. MySQL 的索引类型？
**难度**：中级  
**关键词**：B+ 树、Hash、聚簇、非聚簇  
**简答要点**：B+ 树（默认，范围查询）；Hash（等值查询）；聚簇索引（主键，叶子存数据）；非聚簇索引（二级索引，叶子存主键）。

### Q58. 什么是聚簇索引和非聚簇索引？
**难度**：中级  
**关键词**：主键索引、二级索引、回表  
**简答要点**：聚簇索引（InnoDB 主键，叶子节点存完整数据）；非聚簇索引（二级索引，叶子存主键，需要回表）。覆盖索引避免回表。

### Q59. 什么情况下索引会失效？
**难度**：中级  
**关键词**：最左前缀、函数、隐式转换、OR  
**简答要点**：① 不满足最左前缀；② 对索引列用函数/运算；③ 隐式类型转换；④ LIKE '%xxx'；⑤ OR 条件中有非索引列。

### Q60. MySQL 事务的 ACID 特性？
**难度**：中级  
**关键词**：原子性、一致性、隔离性、持久性  
**简答要点**：Atomicity（undo log）；Consistency（约束）；Isolation（锁+MVCC）；Durability（redo log）。

### Q61. MySQL 的隔离级别？
**难度**：中级  
**关键词**：脏读、不可重复读、幻读  
**简答要点**：4 种：READ UNCOMMITTED（脏读）；READ COMMITTED（不可重复读）；REPEATABLE READ（幻读，MySQL 默认）；SERIALIZABLE（串行）。InnoDB 用 next-key lock 解决幻读。

### Q62. MVCC 是什么？
**难度**：高级  
**关键词**：多版本并发控制、ReadView、undo log  
**简答要点**：MVCC 通过 undo log 保存历史版本 + ReadView 判断可见性，实现非锁定读。RR 级别 ReadView 在第一次读时生成；RC 每次读都生成。

### Q63. 如何优化慢查询？
**难度**：中级  
**关键词**：explain、索引、SQL 改写  
**简答要点**：① 开启慢查询日志；② 用 explain 分析执行计划；③ 加合适索引；④ 改写 SQL（避免子查询、大表 join）；⑤ 分库分表。

### Q64. 分库分表的方案？
**难度**：高级  
**关键词**：ShardingSphere、垂直拆分、水平拆分  
**简答要点**：垂直拆分（按业务拆库/按字段拆表）；水平拆分（按规则拆到多库/多表）。工具：ShardingSphere、MyCat。问题：分布式事务、跨库 join。

### Q65. MySQL 主从复制原理？
**难度**：高级  
**关键词**：binlog、relay log、GTID  
**简答要点**：主库写 binlog → 从库 IO 线程拉取到 relay log → SQL 线程重放。模式：异步、半同步、全同步。GTID 简化故障切换。

### Q66. 死锁如何排查？
**难度**：高级  
**关键词**：show engine innodb status、死锁日志  
**简答要点**：`show engine innodb status` 看 LATEST DETECTED DEADLOCK。常见原因：不同顺序加锁、长事务。解决：缩短事务、统一加锁顺序、降低隔离级别。

---

## 七、Redis & 缓存（中级）

### Q67. Redis 为什么这么快？
**难度**：中级  
**关键词**：单线程、内存、IO 多路复用  
**简答要点**：单线程避免上下文切换；纯内存操作；IO 多路复用（epoll）；高效数据结构（跳表、压缩列表）。Redis 6.0 引入多线程处理网络 IO。

### Q68. Redis 的数据类型？
**难度**：初级  
**关键词**：String、List、Hash、Set、ZSet  
**简答要点**：5 种基础（String/List/Hash/Set/ZSet）+ 3 种高级（Bitmap/HyperLogLog/Stream）。底层：SDS、ziplist、quicklist、skiplist、intset、hashtable。

### Q69. Redis 的持久化方式？
**难度**：中级  
**关键词**：RDB、AOF、混合持久化  
**简答要点**：RDB（快照，快但可能丢数据）；AOF（追加日志，完整但慢）；混合持久化（Redis 4.0+，AOF 重写时 RDB + 增量 AOF）。

### Q70. Redis 集群方案？
**难度**：高级  
**关键词**：主从、哨兵、Cluster  
**简答要点**：主从（读写分离）；哨兵（自动故障转移）；Cluster（去中心化，16384 槽分片）。Cluster 适合大数据量场景。

### Q71. 缓存穿透、击穿、雪崩？
**难度**：中级  
**关键词**：布隆过滤器、互斥锁、过期时间  
**简答要点**：穿透（查不存在的数据）→ 布隆过滤器/缓存空值；击穿（热点 key 过期）→ 互斥锁/永不过期；雪崩（大量 key 同时过期）→ 随机过期时间/多级缓存。

### Q72. 如何保证缓存和数据库一致性？
**难度**：高级  
**关键词**：Cache Aside、延迟双删、binlog 订阅  
**简答要点**：Cache Aside（先更新 DB 再删缓存）；延迟双删（先删缓存→更新 DB→再删缓存）；订阅 binlog（Canal）异步更新缓存。

### Q73. Redis 大 key 问题？
**难度**：高级  
**关键词**：拆分、压缩、异步删除  
**简答要点**：大 key 影响单线程性能。解决：拆分（Hash 代替大 String）；压缩；UNLINK 异步删除；定期扫描大 key。

### Q74. Redis 的内存淘汰策略？
**难度**：中级  
**关键词**：LRU、LFU、TTL  
**简答要点**：8 种：noeviction（默认，不淘汰）；allkeys-lru（最近最少）；volatile-lru（带 TTL 的 LRU）；allkeys-lfu；volatile-lfu；allkeys-random；volatile-random；volatile-ttl。

---

## 八、分布式 & 微服务（中级 → 高级）

### Q75. CAP 定理是什么？
**难度**：中级  
**关键词**：一致性、可用性、分区容错  
**简答要点**：分布式系统最多满足 2 个：Consistency、Availability、Partition tolerance。P 必须满足，所以选 CP（强一致）或 AP（高可用）。

### Q76. BASE 理论是什么？
**难度**：中级  
**关键词**：基本可用、软状态、最终一致  
**简答要点**：Basically Available（基本可用）；Soft state（软状态）；Eventually consistent（最终一致）。是对 CAP 中 AP 的延伸。

### Q77. 分布式事务的解决方案？
**难度**：高级  
**关键词**：2PC、TCC、Saga、本地消息表  
**简答要点**：2PC（强一致，阻塞）；TCC（Try-Confirm-Cancel，业务侵入）；Saga（长事务，补偿）；本地消息表（最终一致）；Seata AT 模式（无侵入）。

### Q78. 分布式 ID 生成方案？
**难度**：中级  
**关键词**：UUID、数据库自增、雪花算法  
**简答要点**：UUID（无序、长）；数据库自增（性能瓶颈）；雪花算法（64 位，时间+机器+序列）；Redis 自增；Leaf（美团）。

### Q79. 分布式锁的实现方式？
**难度**：高级  
**关键词**：Redis、Zookeeper、数据库  
**简答要点**：Redis（setnx + expire + Lua 脚本）；Zookeeper（临时有序节点）；数据库（悲观锁/乐观锁）。Redisson 提供完整实现（看门狗续期）。

### Q80. 一致性 Hash 是什么？
**难度**：中级  
**关键词**：哈希环、虚拟节点、数据迁移  
**简答要点**：将节点映射到 0~2^32 的环上，数据按顺时针找到最近节点。节点变化只影响相邻节点。虚拟节点解决数据倾斜。

### Q81. 微服务架构的优缺点？
**难度**：中级  
**关键词**：独立部署、服务治理、复杂度  
**简答要点**：优点：独立开发/部署/扩容；技术栈灵活。缺点：分布式复杂性（网络、事务、调试）；运维成本高；服务治理复杂。

### Q82. 服务熔断和降级的区别？
**难度**：中级  
**关键词**：熔断、降级、限流  
**简答要点**：熔断（上游故障，自动切断调用，防止雪崩）；降级（主动降低非核心服务，保核心）；限流（控制 QPS）。Sentinel/Hystrix 实现。

### Q83. RPC 框架的原理？
**难度**：高级  
**关键词**：序列化、网络传输、服务注册  
**简答要点**：客户端调用本地代理 → 序列化 → 网络传输 → 服务端反序列化 → 执行 → 返回。核心：服务注册发现、负载均衡、容错。Dubbo/gRPC/HSF。

### Q84. 消息队列如何保证消息不丢失？
**难度**：高级  
**关键词**：生产者确认、持久化、消费者确认  
**简答要点**：生产者：同步发送 + 重试；Broker：持久化到磁盘 + 副本；消费者：手动 ACK + 重试队列。RocketMQ/Kafka/RabbitMQ 各有机制。

---

## 九、消息队列（中级）

### Q85. 为什么要用消息队列？
**难度**：初级  
**关键词**：异步、解耦、削峰  
**简答要点**：异步（提升响应）；解耦（系统独立演进）；削峰（缓冲突发流量）。

### Q86. Kafka、RocketMQ、RabbitMQ 的区别？
**难度**：中级  
**关键词**：吞吐量、延迟、功能  
**简答要点**：Kafka（高吞吐，日志场景）；RocketMQ（金融级可靠，阿里）；RabbitMQ（功能全，延迟低，Erlang）。选型看场景。

### Q87. 如何保证消息顺序性？
**难度**：高级  
**关键词**：分区、单队列、并发消费  
**简答要点**：Kafka：同 key 发同分区；RocketMQ：同 MessageQueue；RabbitMQ：单队列单消费者。消费端也要保证顺序（单线程或内存队列）。

### Q88. 消息重复消费如何处理？
**难度**：中级  
**关键词**：幂等、去重表、唯一 ID  
**简答要点**：消费端保证幂等：数据库唯一约束；去重表（消息 ID）；Redis 记录已处理 ID；状态机校验。

### Q89. 消息积压如何处理？
**难度**：高级  
**关键词**：扩容消费者、临时队列、降级  
**简答要点**：① 扩容消费者；② 临时把积压消息转到更多分区/队列；③ 跳过非重要消息；④ 分析根因（消费者慢/生产过快）。

### Q90. 如何保证消息可靠投递？
**难度**：高级  
**关键词**：生产者确认、持久化、消费者 ACK  
**简答要点**：生产者：confirm 机制 + 重试；Broker：持久化 + 集群；消费者：手动 ACK + 死信队列处理失败消息。

---

## 十、系统设计（高级）

### Q91. 设计一个秒杀系统？
**难度**：高级  
**关键词**：限流、库存扣减、防超卖  
**简答要点**：前端（倒计时+按钮防抖）→ 网关（限流+黑名单）→ 服务（Redis 预扣库存 → MQ → DB 异步扣减）。关键：库存防超卖（Redis Lua / DB 乐观锁）。

### Q92. 设计一个短链系统？
**难度**：高级  
**关键词**：发号器、哈希、存储  
**简答要点**：发号器（雪花算法/数据库自增）→ Base62 编码 → 存储（KV 或关系型）。考虑：哈希碰撞、302 重定向、缓存热点短链、统计访问。

### Q93. 设计一个 Feed 流系统？
**难度**：高级  
**关键词**：推拉模式、Timeline、Redis ZSet  
**简答要点**：推模式（写扩散，适合大 V）；拉模式（读扩散，适合普通用户）；推拉结合（大 V 推，普通用户拉）。存储：Redis ZSet（score=时间戳）。

### Q94. 设计一个排行榜系统？
**难度**：中级  
**关键词**：Redis ZSet、实时、历史  
**简答要点**：Redis ZSet（score 排序）；ZADD 更新；ZRANGE 查询 Top N；ZREVRANK 查排名。历史数据定期归档到 DB。

### Q95. 设计一个分布式限流系统？
**难度**：高级  
**关键词**：令牌桶、滑动窗口、Redis + Lua  
**简答要点**：算法：令牌桶（允许突发）/ 滑动窗口（精确）。实现：Redis + Lua（分布式计数）；网关层（Nginx/Sentinel）；客户端限流。

---

## 十一、软技能 & 项目表达（全难度）

### Q96. 如何介绍自己的项目？
**难度**：初级  
**关键词**：STAR、量化结果  
**简答要点**：用 STAR：Situation（背景）→ Task（任务）→ Action（行动）→ Result（结果，带数字）。例："我们系统 QPS 从 1000 优化到 5000，RT 降低 60%"。

### Q97. 项目中最有挑战的问题？
**难度**：初级  
**关键词**：问题描述、排查过程、解决方案  
**简答要点**：结构：问题现象 → 排查过程（工具/思路）→ 根因 → 解决方案 → 沉淀（文档/规范）。展示思考过程，不是只说结果。

### Q98. 为什么从上一家公司离职？
**难度**：初级  
**关键词**：正面表达、成长诉求  
**简答要点**：避免抱怨，聚焦成长：希望接触更大规模/更深技术/更有挑战业务。例："想从业务开发转向基础架构，寻求更大技术挑战"。

### Q99. 你有什么问题要问我？
**难度**：初级  
**关键词**：团队、技术、业务  
**简答要点**：问团队规模和分工；技术栈和架构演进；业务未来规划；岗位挑战。避免：薪资福利（放 HR 面）、"没什么问题"。

### Q100. 如何准备一场面试？
**难度**：初级  
**关键词**：JD 分析、项目梳理、模拟面试  
**简答要点**：① 分析 JD 找关键词；② 梳理 2-3 个项目（STAR）；③ 按高频题复习；④ 模拟面试练表达；⑤ 准备反问问题。

---

## 附录：按公司高频题分布

| 公司 | 高频方向 |
|------|----------|
| 阿里 | 并发、JVM、分布式、MySQL、Redis、系统设计 |
| 腾讯 | 算法、C++/Go、网络、高并发、游戏场景 |
| 字节 | 算法（Hard）、系统设计、项目深度、Go/Java |
| 美团 | 算法、Spring、MySQL、Redis、业务理解 |
| 百度 | 算法、C++、搜索、推荐系统 |
| 外企（Google/MS） | 算法（Medium-Hard）、系统设计、行为面 |

---

## 使用建议

1. **按难度分层复习**：初级先过 → 中级重点 → 高级冲刺
2. **不要只背答案**：每题用"三层回答法"（是什么/为什么/怎么用）
3. **绑定项目经验**：把知识点和你的项目关联，面试时讲得出故事
4. **模拟面试**：每周 1-2 次，录音回听，优化表达
5. **间隔复习**：用 Anki 或飞书表格，按遗忘曲线复习

祝面试顺利！
