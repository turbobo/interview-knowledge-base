# JVM 垃圾回收全景图

> 来源：[JavaGuide — JVM 垃圾回收详解](https://javaguide.cn/java/jvm/jvm-garbage-collection.html)
> 配套可视化图：[JVM垃圾回收全景图.html](./JVM垃圾回收全景图.html)（浏览器打开查看）

---

## 一、对象存活判断

### 1. 可达性分析算法（主流方案）

从 **GC Roots** 出发，沿引用链向下搜索，无引用链的对象即为可回收。解决了引用计数法的循环引用缺陷。

**GC Roots 类型：**

| 类型 | 说明 |
|------|------|
| 虚拟机栈中的局部变量 | 方法内正在使用的对象引用 |
| 方法区中的静态属性 | `static` 修饰的字段引用 |
| 方法区中的常量引用 | `final static` 等常量池引用 |
| 同步锁持有的对象 | `synchronized(obj)` 中的 obj |
| Native 方法栈中的引用 | JNI 调用中的对象引用 |
| 基本类型的 Class 对象 | `int.class` 等不会被回收 |

**无用类判定**（三个条件同时满足）：
1. 该类所有实例都已被回收
2. 加载该类的 ClassLoader 已被回收
3. 该类的 `java.lang.Class` 对象不再被引用

### 2. 四种引用类型

| 引用类型 | 回收时机 | 典型用途 | API |
|----------|----------|----------|-----|
| **强引用** | 绝不回收 | 普通对象引用 | `Object o = new Object()` |
| **软引用** | 内存告急时回收 | 缓存 | `SoftReference` |
| **弱引用** | 下次 GC 即回收 | `WeakHashMap` | `WeakReference` |
| **虚引用** | 随时可回收 | 配合队列处理清理工作 | `PhantomReference` |

---

## 二、垃圾回收算法

| 算法 | 原理 | 优点 | 缺点 | 适用区域 |
|------|------|------|------|----------|
| **标记-清除** | 标记存活对象，清除未标记对象 | 实现简单 | 产生大量内存碎片 | 很少单独使用 |
| **复制算法** | 内存分两块，存活对象复制到另一块 | 无碎片，分配高效 | 可用空间减半 | **新生代**（存活率低） |
| **标记-整理** | 标记存活对象，向一端移动，清理边界外 | 无碎片 | 移动对象开销大 | **老年代**（存活率高） |

**分代收集策略**：新生代用复制算法，老年代用标记-整理或标记-清除。

---

## 三、内存分配与晋升机制

### 分配策略

- **优先 Eden**：新对象在 Eden 区创建，空间不足时触发 Minor GC
- **大对象直接入老年代**：避免在新生代和老年代之间大量复制
- **G1 Humongous Object**：超过 Region 一半大小的对象直接分配到大对象 Region

### 晋升机制

```
Eden（新对象）→ Minor GC（存活检查）→ Survivor（年龄+1）→ 老年代（年龄达标/大对象）
```

- **年龄晋升**：对象每经过一次 GC 年龄 +1，达到 `-XX:MaxTenuringThreshold`（默认 15）时晋升
- **动态年龄**：Survivor 区同年龄对象总大小超过 Survivor 空间一半时，≥ 该年龄的对象直接晋升
- **空间分配担保**：Minor GC 前确认老年代剩余空间足以容纳新生代全部对象或历次晋升平均值

---

## 四、GC 类别

| GC 类型 | 回收范围 | 触发条件 | 停顿时间 |
|---------|----------|----------|----------|
| **Minor GC** | 仅新生代 | Eden 区空间不足 | 较短 |
| **Major GC** | 仅老年代 | 老年代空间不足 | 较长 |
| **Full GC** | 整堆 + 方法区 | 老年代/方法区不足、`System.gc()`、CMS failure | 最长 |
| **Mixed GC**（G1） | 新生代 + 部分老年代 | 老年代占用达到阈值 | 可控 |

---

## 五、垃圾收集器演进

### Serial / Serial Old
- **区域**：新生代（Serial，复制）/ 老年代（Serial Old，标记-整理）
- **特点**：单线程，Stop The World
- **适用**：客户端模式，`-XX:+UseSerialGC`

### ParNew
- **区域**：新生代
- **特点**：Serial 的多线程版本，常搭配 CMS
- **适用**：多核环境

### Parallel Scavenge / Old（JDK 8 默认）
- **区域**：新生代（Scavenge，复制）/ 老年代（Old，标记-整理）
- **特点**：关注吞吐量，自适应调节（`-XX:+UseAdaptiveSizePolicy`）
- **启用**：`-XX:+UseParallelGC`

### CMS（JDK 14 移除）
- **区域**：老年代
- **特点**：并发标记清除，追求低停顿
- **四阶段**：初始标记（STW）→ 并发标记 → 重新标记（STW）→ 并发清除
- **缺点**：内存碎片、浮动垃圾、CPU 敏感

### G1（JDK 9+ 默认）
- **区域**：整堆
- **特点**：基于 Region 划分（~2048 个 Region，每个 1-32MB）
- **优势**：可预测停顿（`-XX:MaxGCPauseMillis`）、Mixed GC
- **启用**：`-XX:+UseG1GC`

### ZGC
- **区域**：整堆
- **特点**：毫秒级停顿（<10ms），支持 TB 级堆
- **技术**：着色指针 + 读屏障，并发转移
- **Java 21**：引入分代 ZGC，进一步降低停顿
- **启用**：`-XX:+UseZGC`

### 收集器速查对比

| 收集器 | 区域 | 算法 | 线程 | 停顿 | 状态 |
|--------|------|------|------|------|------|
| Serial | 新生代 | 复制 | 单线程 | STW | 可用 |
| Serial Old | 老年代 | 标记-整理 | 单线程 | STW | 可用 |
| ParNew | 新生代 | 复制 | 多线程 | STW | 可用 |
| Parallel Scavenge | 新生代 | 复制 | 多线程 | STW | JDK8 默认 |
| Parallel Old | 老年代 | 标记-整理 | 多线程 | STW | JDK8 默认 |
| CMS | 老年代 | 标记-清除 | 并发 | 低停顿 | JDK14 移除 |
| **G1** | 整堆 | Region 复制+整理 | 并发 | 可控 | **JDK9+ 默认** |
| **ZGC** | 整堆 | 并发转移 | 并发 | <10ms | **推荐** |

**选型建议**：JDK 8 → Parallel GC · JDK 9+ → G1 · 超低延迟 → ZGC · 客户端 → Serial

---

## 六、核心调优参数

| 参数 | 说明 |
|------|------|
| `-Xms` / `-Xmx` | 堆初始大小 / 最大大小 |
| `-Xmn` | 新生代大小 |
| `-XX:NewRatio=N` | 老年代:新生代比例（默认 2:1） |
| `-XX:MaxTenuringThreshold=N` | 晋升年龄上限（默认 15） |
| `-XX:TargetSurvivorRatio=N` | Survivor 期望使用率（默认 50%） |
| `-XX:+UseSerialGC` | 启用 Serial 收集器 |
| `-XX:+UseParallelGC` | 启用 Parallel 收集器 |
| `-XX:+UseG1GC` | 启用 G1 收集器 |
| `-XX:+UseZGC` | 启用 ZGC 收集器 |
| `-XX:MaxGCPauseMillis=N` | G1 目标最大停顿时间（ms） |
| `-XX:G1HeapRegionSize=Nm` | G1 Region 大小（1-32MB） |
| `-XX:+UseAdaptiveSizePolicy` | 自适应调节堆大小和晋升阈值 |
| `-XX:+PrintGCDetails` | 输出 GC 详细日志 |

---

## 七、面试高频追问

**Q1：Minor GC 和 Full GC 的区别？**
> Minor GC 只回收新生代，触发频繁但停顿短；Full GC 回收整堆+方法区，停顿长，应尽量避免。

**Q2：G1 为什么能实现可预测停顿？**
> G1 将堆划分为多个 Region，每次 GC 只选择收益最大的 Region（Garbage-First），通过 `-XX:MaxGCPauseMillis` 控制停顿目标。

**Q3：CMS 为什么被废弃？**
> ① 内存碎片（标记-清除不整理）；② 浮动垃圾（并发清除阶段新产生的垃圾要下次清理）；③ CPU 敏感（并发阶段占用 CPU）。G1 全面替代。

**Q4：ZGC 如何做到 <10ms 停顿？**
> 着色指针（Colored Pointers）+ 读屏障（Load Barrier），实现并发转移和并发重定位，几乎不需要 STW。

**Q5：什么时候对象会直接进入老年代？**
> ① 大对象（超过 `-XX:PretenureSizeThreshold`）；② Survivor 空间不足时；③ 动态年龄判断达标；④ G1 中超过 Region 一半的 Humongous Object。
