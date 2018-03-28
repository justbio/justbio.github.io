---
layout: post
category: "read"
title:  "TOMCAT相关的笔记"
---

本文只是记录一下tomcat运维用到过的知识，都是网络上收集来的资料，侵删

#### JVM的内存
![](../assets/739083-20160415164833566-639029606.png)

<!-- more -->

#### 内存分配
1. -XX:PermSize尽量比-XX:MaxPermSize小，-XX:MaxPermSize>= 2 * -XX:PermSize, -XX:PermSize> 64m，一般对于4G内存的机器，-XX:MaxPermSize不会超过256m；  
2. -Xms =  -Xmx（线上Server模式），以防止抖动，大小受操作系统和内存大小限制，如果是32位系统，则一般-Xms设置为1g-2g（假设有4g内存），在64位系统上，没有限制，不过一般为机器最大内存的一半左右；
3. -Xmn，在开发环境下，可以用-XX:NewSize和-XX:MaxNewSize来设置新生代的大小（-XX:NewSize<=-XX:MaxNewSize），在生产环境，建议只设置-Xmn，一般-Xmn的大小是-Xms的1/2左右，不要设置的过大或过小，过大导致老年代变小，频繁Full GC，过小导致minor GC频繁。如果不设置-Xmn，可以采用-XX:NewRatio=2来设置，也是一样的效果；
4. -Xss一般是不需要改的，默认值即可。
5. -XX:SurvivorRatio一般设置8-10左右，推荐设置为10，也即：Survivor区的大小是Eden区的1/10，一般来说，普通的Java程序应用，一次minorGC后，至少98%-99%的对象，都会消亡，所以，survivor区设置为Eden区的1/10左右，能使Survivor区容纳下10-20次的minor GC才满，然后再进入老年代，这个与 -XX:MaxTenuringThreshold的默认值15次也相匹配的。如果XX:SurvivorRatio设置的太小，会导致本来能通过minor回收掉的对象提前进入老年代，产生不必要的full gc；如果XX:SurvivorRatio设置的太大，会导致Eden区相应的被压缩。
6. -XX:MaxTenuringThreshold默认为15，也就是说，经过15次Survivor轮换（即15次minor GC），就进入老年代，如果设置的小的话，则年轻代对象在survivor中存活的时间减小，提前进入年老代，对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即被回收的概率。需要注意的是，设置了 -XX:MaxTenuringThreshold，并不代表着，对象一定在年轻代存活15次才被晋升进入老年代，它只是一个最大值，事实上，存在一个动态计算机制，计算每次晋入老年代的阈值，取阈值和MaxTenuringThreshold中较小的一个为准。
7. -XX:PretenureSizeThreshold一般采用默认值即可。

#### 各种默认值

64位默认为server模式,32位的阀值是2CPU和2GB内存

client    串行    串行  
server    并行回收GC    并发GC

#### GC组合
Young区GC的方式：

1. 串行GC（Serial Copying）  
client模式下的默认GC方式，也可使用-XX:+UseSerialGC指定。

2. 并行回收GC（Parallel Scavenge）  
server模式下的默认GC方式，也可用-XX:+UseParallelGC强制指定。采用PS时，默认情况下JVM会在运行时动态调整Eden:S0:S1的比例，如果不希望自动调整可以使用-XX:-UseAdaptiveSizePolicy参数，内存分配和回收的算法和串行相同，唯一不同仅在于回收时为多线程。

3. 并行GC（ParNew）  
CMS GC时默认采用，也可以采用-XX:+UseParNewGC指定。内存分配、回收和PS相同，不同的仅在于会收拾会配合CMS做些处理。

Old区GC的方式：

1. 串行GC（Serial MSC）  
client模式下的默认GC方式，可通过-XX:+UseSerialGC强制指定。每次进行全部回收，进行Compact，非常耗费时间。

2. 并行GC（Parallel MSC） 
server模式下的默认GC方式，也可用-XX:+UseParallelGC=强制指定。可以在选项后加等号来制定并行的线程数。

3. 并发GC（CMS）线上环境采用的GC方式，也就是Realese环境的方式  
使用CMS是为了减少GC执行时的停顿时间，垃圾回收线程和应用线程同时执行，可以使用-XX:+UseConcMarkSweepGC=指定使用，后边接等号指定并发线程数。CMS每次回收只停顿很短的时间，分别在开始的时候（Initial Marking），和中间（Final Marking）的时候，第二次时间略长。
![](../assets/739083-20160415164834645-1086874067.png)

故障调查用命令

jps  
查看JVM的进程号

jstat -option PID  
查看GC和JVM内存  
Option:gc,gccapacity,gccause, gcnew,gcnewcapacity,gcold,gcoldcapacity,gcpermcapcacity,gcutil

jmap  
jmap -heap:format=b [pid] (JAVA1.5)  
jmap -dump:format=b,file=[filePath] [pid] (JAVA1.6)  

#### 参考资料

【原】GC的默认方式, http://iamzhongyong.iteye.com/blog/1447314

Java系列笔记(4) - JVM监控与调优，http://www.cnblogs.com/zhguang/p/Java-JVM-GC.html

官网，http://docs.oracle.com/javase/jp/7/technotes/tools/share/jstat.html
