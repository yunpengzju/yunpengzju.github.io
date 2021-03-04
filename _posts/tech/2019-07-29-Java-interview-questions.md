---
layout: post
title: Java面试的一些常见问题
description: Java面试FAQ
category: tech
---

最近面试了很多Java开发人员，直观的感受时，短短几年，现在的年轻人比我刚毕业时对于Java基础知识的掌握好了很多。说来惭愧，我拿来问面试者的很多问题，如果是当年刚刚毕业的自己，是定然答不出的。

#### 1，String，StringBuilder，StringBuffer的区别
String是不可变的。任何对String的sub、concat、replace操作，都是重新生成了一个String对象。
对String做追加操作时，实际上是先生成了StringBuilder对象，然后使用StringBuilder的append方法完成追加，最后在通过StringBuilder的toString方法获取到String对象的。这个过程中，如果直接使用String对象进行追加操作，可能会生成大量多余的StringBuilder对象。
StringBuffer则是在StringBuilder的基础上保证了线程安全。

#### 2，Date，DateFormat，Calendar的关系
Date表示特定的瞬间，精确到毫秒
DateFormat用于处理Date与String类型之间的转换问题
Calendar则用于方便地对年月日时分秒这些日历参数进行操作
Date与Calendar对象之间的转换：
```
Calendar testCal = Calendar.getInstance();
Date testDate = new Date();

testCal.setTime(testDate);
Date newDate = testCal.getTime();
```

在Java8中，为了解决Date和Calendar两个类库存在的易用性差的问题，增加了LocalDate和LocalTime两个类来处理和日期、时间相关的信息。

Java8的日期和时间类包含LocalDate、LocalTime、Instant、Duration以及Period，这些类都包含在java.time包中。

1）LocalDate类表示一个具体的日期，但不包含具体时间，也不包含时区信息。
2）LocalTime和LocalDate类似，他们之间的区别在于LocalDate不包含具体时间，而LocalTime包含具体时间
3）LocalDateTime类是LocalDate和LocalTime的结合体
4）Instant用于表示一个时间戳，它与我们常使用的System.currentTimeMillis()有些类似，不过Instant可以精确到纳秒（Nano-Second），System.currentTimeMillis()方法只精确到毫秒（Milli-Second）
5）Duration的内部实现与Instant类似，也是包含两部分：seconds表示秒，nanos表示纳秒。两者的区别是Instant用于表示一个时间戳（或者说是一个时间点），而Duration表示一个时间段，所以Duration类中不包含now()静态方法

#### 3，volatile关键词的作用
为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

为了防止内存的脏读，如果使用synchronized，虽然可以达到目的，但是显得有些重。使用volatile则更为合适，但是要注意，volatile关键词声明的域并不能保证原子性。


#### 4，Java中的Collections.sort是如何实现的
在java8中，Sort方法使用的算法是TimSort。这个排序算法混合使用了插入排序、归并排序、二分搜索等算法。对于数组中有部分元素有序的情况（现实中经常出现这种情况）进行了优化，用更少的Comparison操作完成排序（对于Comparison操作开销大的情况效果更优）。

算法简述：
1）扫描数组，确定连续单调上升和严格单调下降段，将严格下降段反转，得到一个runLen
2）定义最小基本片段长度，对于runLen小于此长度的情况，通过插入排序使runLen达到最小长度
3）依次将run压入栈中，若栈顶run X，run Y，run Z 的长度违反了X>Y+Z 或 Y>Z 则Y run与较小长度的run合并，并再次放入栈中。 依据这个法则，能够尽量使得大小相同的run合并，以提高性能。注意Timsort是稳定排序故只有相邻的run才能归并。

网上有一段视频对此算法的执行过程讲解的比较清楚：
[https://vimeo.com/146478455](https://vimeo.com/146478455)

#### 参考资料：
1，[String、StringBuilder与StringBuffer](https://www.cnblogs.com/dolphin0520/p/3778589.html)

2，[Date与Calendar](https://juejin.im/post/5d09a1f76fb9a07eff008b93)

3，[Java8中的LocalDate和LocalTime](https://lw900925.github.io/java/java8-newtime-api.html)

4，[深入了解volatile关键字](https://juejin.im/post/5ae9b41b518825670b33e6c4)

5，[TimSort与Python SampleSort的对比](https://svn.python.org/projects/python/trunk/Objects/listsort.txt)