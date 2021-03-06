---
layout: post
title: 重新开始学习数据结构
description: 我们为什么学习数据结构
category: tech
---

还记得大学的时候上数据结构课，那时刚刚接触计算机编程一个学期，只知道数据结构是计算机专业的一门核心课程，但对于这门课提及的各种数据结构学来有什么用，完全没有概念。于是，这门课可以说学的很被动，老师教什么，便也就照着记什么。课程结束了，怕也就忘记了一大半。多年之后进入职场，恍然发现当年的那些专业核心课程原来真的都有实际应用场景，遗憾当年没能体味这些学问的精妙之余，也在不断地问自己：如果让我重学一遍这些核心课程，我该如何学？

初学数据结构时，每一个概念对于我来说都非常的新鲜。彼时学习的重点也便聚焦在了什么是链表、桟、队列、树等等，以及如何用C语言来实现这些数据结构。再加上学到的一些粗浅的算法设计方法，便也觉得掌握了这些数据结构。现在回想起来，这学习过程中显然是漏掉了非常重要的一环，那就是这些数据结构从何而来？我们为什么要学习这些数据结构？

每一种数据结构或算法，都是从计算机专家们遇到的实际问题总结而来的。科学家们先是在实践中遇到了算法的性能或是空间存储限制问题，而后才会不断地改进数据结构与算法使得程序执行的效率达到最优。先有问题，然后针对性地寻找答案，这是一个自然而然的过程。如果脱离问题的背景，只凭空地考虑不同数据结构的异同，实在是不容易的。但偏偏在学生时代，我们面对的问题数据规模都十分有限，可以说数组就可以搞定一切。在这种情况下，学不透数据结构也就不稀奇了。

那现在为何又想起重学数据结构呢？自然是遇到问题了。一次在做一个IPSET和域名列表的增删测试时，偶然发现在增加100W个IP时，耗时只有1s，而增加100W个域名时，耗时居然高达数小时（等了5个小时依然没有结果于是终止了程序）。在我看来这两种数据的添加操作并没有什么本质区别，是什么造成了最终执行时间的巨大差异呢？带着疑问我询问了编写IPSET模块和域名模块的两位同事，结果发现这中间的差异源自于每次添加新元素时的一个查询操作。因为要对数据去重，每次添加新的IP或者域名时，都要先查询该元素是否已经存在，这就涉及到对已有数据的搜索。编写IPSET模块的同事在存储数据时使用了Trie树结构，所以每次查询都是O(n)的时间复杂度（n为IP的长度，也就是4段），也就是说，每一次查询只需要最多4次操作。而域名的存储呢，使用了链表的结构，每次查询都是O(n)的复杂度。于是在数据规模达到100W这个量级的时候，两种实现的差距就显得非常惊人。

类似上述问题的情况在实际工作中每天都会遇到。但实际上，关于这些问题的更优化的解决方案早已被研究的非常透彻，我们完全不需要重复发明轮子。学习数据结构，就是为了让我们在面对这些大规模数据问题时，可以举重若轻。学好数据结构，会让我们在工作中拥有降维打击般的巨大优越感。

废话不多说，让我们从问题出发，重新认识一下熟悉又陌生的数据结构吧。

