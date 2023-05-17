# 1.JVM概述

## Java虚拟机发展

Classic -> Hotspot
IBM J9

虚拟机的作用：解释执行Java字节码。支持Java、Groovy、Scala、Jython等语言

## Java的数据类型分为：基本数据类型和引用数据类型

​	基本数据类型分为数字型和布尔型。
​		数字类型：byte、short、int、long、char、float、double。char被定义为整数型，占2个字节，16位无符号整数
​		布尔类型：boolean，只有两个取值true、false
​		

	引用数据类型：类或接口、泛型类型、数组类型。


​	

## Java虚拟机规范：

​	Java虚拟机是一台执行Java字节码的虚拟计算机，其运行的Java字节码也未必是由Java语言编译而成，例如Goovvy、Scala、Jython。
​	与Java语言不同，Java虚拟机是一个高效的、性能优异的、商用级别的软件运行和开发平台。
​	Java虚拟机规范：
​		定义了虚拟机的内部结构
​		定义了虚拟机执行的字节码类型和功能
​		定义了Class文件的结构
​		定义了类的装载、连接和初始化
​		
原码：符号位加上数字的二进制表示。以int为例，第1位表示符号位（0表正，1表负），其余31位表示该数字的二进制值。
反码：在原码的基础上，符号位不变，其余位取反
补码：负数的补码是反码加1，正数的补码就是原码

既然有了原码了，为什么还需要反码、补码呢？

1. 使用补码可以统一数字0的表示。由于0既非正数又非负数，使用原码表示时符号位难以确定，把0归入正数或负数的到的原码是不同的。而使用补码表示，正数0和负数0都是相同的结果。
2. 使用补码可以简化整数的加减法运算，将减法运算视为加法运算，实现加法和减法的完全统一，实现正数和负数加法的统一

## 术语：

Java Language Specification：Java语言规范
**语法**定义：规定在Java中什么样的语句是合理的
**词法**定义：规定在Java中什么样的单词是合理的

# 2. JVM使用

# 3.JVM原理

# Java跨平台的原理

​	通过Java虚拟机做中介。
​	虚拟机：一款软件，用来执行一系列虚拟计算机指令。虚拟机可分为系统虚拟机和程序虚拟机。
​		系统虚拟机：是对物理计算机的仿真，提供一个可运行完整操作系统的软件平台。例如：Visual Box、VMware。
​		程序虚拟机：专门为执行单个计算机程序而设计。例如：Java虚拟机。在Java虚拟机中执行的指令我们称之为Java字节码指令。编译之后的Java程序就是Java字节码的集合
​		
​		

## JVM GC原理

### JVM有哪些GC？

不同的GC算法中，叫不同的GC名字。

**Minor GC**：

MinorGC 一般指清理 Young space (Eden and Survivor spaces) 的 GC，也叫Young GC，即年轻代的GC。

 触发时机：

在新生代的Eden区域满了之后就会触发，采用标记-复制算法来 回收新生代的垃圾 

Allocation Failure： 分配对象失败，空间不足. 内存分配流程，涉及到了 bump-the-pointer， TLAB，Allocation Prematch 这些机制， 请参考。
 Survivor 区满了，需要拷贝。

不同的 GC 还会有自己个性化的触发机制，例如 G1GC 还有Shenandoah GC 的 TLAB 分配失败剩余空间大于最大浪费空间直接在Eden分配也失败，ZGC 的预热触发等等。



**Major GC**：

MajorGC 一般指清理 Tenured space 的 GC，也叫Old GC，即老年代的GC。

触发时机：

1、发生Young GC之前进行检查，如果“老年代可用的连续内存空间” < “新生代历次Young GC后升入老年代的对象 总和的平均大小”，说明本次Young GC后可能升入老年代的对象大小，可能超过了老年代当前可用内存空间，此时必须先触发一次Old GC给老年代腾出更多的空间，然后再执行Young GC

这也就是 ygc之前，触发old gc

2、执行Young GC之后有一批对象需要放入老年代，此时老年代就是没有足够的内存空间存放这些对象了，此时必 须立即触发一次Old GC

这也就是ygc之后，触发old gc

 一般由 MinorGC 触发，并且回收的空间依然不足，则可能触发 MajorGC。还有一些特殊的机制，例如 G1GC 的Homongous Allocation（大对象分配），在分配超过 RegionSize 一半大小的对象时，会触发 OldGC。 

3、老年代内存使用率超过了92%，也要直接触发Old GC，当然这个比例是可以通过参数调整的。
其实说白了，上述三个条件你概括成一句话，就是老年代空间也不够了，没法放入更多对象了，这个时候务必执行Old GC对老年代进行垃圾回收。

顺便说一句，大家在很多地方看到一个说法，意思是说Old GC执行的时候一般都会带上一次Young GC
可能很多人不理解，其实如果你把咱们这里的几个条件分析清楚了就知道了

一般Old GC很可能就是在Young GC之前触发或者在Young GC之后触发的，所以自然Old GC一般都会跟一次Young GC连带关联在一起了。

另外一个，在很多JVM的实现机制里，其实在上述几种条件达到的时候，他触发的实际上就是Full GC，这个Full GC会 包含Young GC、Old GC和永久代的GC

其实满足上述一些条件的时候，在GC日志中看到的就是Full GC的字样，同时在Full GC中大家会看到同 时对年轻代、老年代、永久代都进行了垃圾回收。

**Full GC**：

FullGC 一般指清理 所有 space 的 GC。触发时机一般是：
 System.gc()被调用并且没有指定关闭显示GC，就是没有指定-XX:+DisableExplicitGC这个JVM flag。
 老年代也满了。
 堆外内存满了，例如metaspace，代码即时编译缓存，直接内存，mmap内存。
 gc 担保失败，请参考：-XX:-HandlePromotionFailure。



### 什么时候触发JVM的GC？

MinorGC 在年轻代空间不足的时候发生。
 MajorGC 指的是老年代的 GC，出现 MajorGC 一般经常伴有 MinorGC。
 FullGC 老年代无法再分配内存；元空间不足；显式调用 System.gc；像 CMS 一类的垃圾回收器，在 MinorGC 出现 promotion failure 时也会发生 FullGC。



# 4. JVM配置







# 1.3.1 环境变量、系统变量和用户变量

环境变量包括系统变量和用户变量
系统变量的设置针对该操作系统下的所有用户起作用；
用户变量的设置只针对当前用户起作用 



## JVM GC

https://zhuanlan.zhihu.com/p/25539690

https://www.cnblogs.com/metoy/p/3703128.html