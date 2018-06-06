---
title: JVM 性能监控与故障处理工具
date: 2018-03-29 09:47:21
tags:
- JVM
categories:
- Tech
---
JDK 中包含一些命令行工具，这些工具能在处理应用程序性能问题、定位故障时发挥很大的作用。



## jps

JVM Process Status Tool，用于显示系统内所有的 HotSpot 虚拟机进程。

jps 可以列出正在运行的 JVM 进程，显示 JVM 执行主类及进程的本地虚拟机唯一 ID（与操作系统进程 ID 一致）。

常用选项：

* -q：只输出进程 ID，隐藏主类名称
* -m：输出进程启动时传递给主类 main() 函数的参数
* -l：输出主类全名，如进程执行的是 Jar 包，输出 Jar 路径
* -v：输出虚拟机进程启动时 JVM 参数



<!--more-->



## jstat

JVM Statistics Monitoring Tool，用于收集 HotSpot 各方面的运行数据。

可以显示 JVM 进程中的类装载、内存、垃圾收集、JIT 编译等运行数据，格式为 

`jstat option vmid [interval] [count]`

interval 参数为查询间隔，count 参数为查询次数，省略这两个参数则只查询一次。

主要选项：

* -class：监视类装载、卸载数量、总空间及耗时
* -gc：监视 Java 堆状态，包括各个区的容量、已用空间、GC 时间合计等信息
* -gccapacity：与 -gc 类似，更关注堆内各区使用到的最大、最小空间
* -gcutil：与 -gc 类似，更关注已使用空间占比
* -gccause：与 -gcutil 类似，额外输出上一次 GC 原因
* -gcnew：监视新生代 GC 状况
* -gcnewcapacity：与 -gcnew 类似，更关注使用到的最大、最小空间
* -gcold：监视老年代 GC 状况
* -gcoldcapacity：与 -gcold 类似，更关注使用到的最大、最小空间





## jinfo

Configuration Info for Java，用于显示虚拟机配置信息，可以实时查看和调整虚拟机各项参数。

jps -v 命令可以查看 JVM 启动时显示指定的参数列表，各项参数的默认值可以通过 jinfo 的 -flag 选项进行查询，还可以通过 -sysprops 选项打印出进程 System.getProperties() 的内容。



## jmap

Memory Map for Java，用于生成 JVM 内存转储快照（heapdump文件）。命令格式 `jmap [option] vmid` ，主要选项：

* -dump：生成 Java 堆转储快照
* -head：显示 Java 堆详细信息
* -histo：显示堆中对象的统计信息





## jhat

JVM Heap Dump Browser，用于分析 heapdump 文件，建立 HTTP / HTML 服务器，可以在浏览器上查看分析结果。

实际工作中一般不使用jhat 来分析 dump 文件，因为通常不会再服务器上直接分析 dump 文件，耗时且消耗资源，功能也相对简陋。



## jstack

Stack Trace for Java，用于生成虚拟机线程快照，目的是定位线程出现长时间停顿的原因。常见参数：

* -F：当正常输出的请求不被响应时，强制输出线程堆栈
* -l：除堆栈外，显示关于锁的附加信息
* -m：如调用本地方法，可以显示 C / C++ 的堆栈




## JConsole

Java Monitoring and Management Console，基于 JMX 的可视化 Java 监视与管理控制台。



## Visual VM

All-in-One Java Troubleshooting Tool，功能强大的运行监视和故障处理程序。不需要被监视程序基于特殊 Agent 运行，对程序的实际性能影响很小。
