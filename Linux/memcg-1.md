# memcg

一花一世界。 -- 《华严经》

少则得，多则惑。 -- 《道德经》

------------------

memcg（内存资源控制器）正在容器领域，终端领域发挥很大的作用。同时，作为Linux社区热点之一，近年来不断有关于它的新特性推出。

这个专项旨在让我成为memcg领域的专家，时限暂时定为半年至一年。就从memcg这个'小领域'开始我的记录与分享之路吧。


* 整体计划

> 1. cgroup 测试用例的熟悉（偏内存方向, 可以尝试riscv）
> 2. cgroup 原理架构 与 核心代码 
> 3. cgroup-memory 子系统 原理架构 与 核心代码
> 4. 近几年的patch演变分析。

这个过程的输出以总结博客为主。


* 可能外延（贯穿）：

> 1. Per memcg lru locking 的原理 与 代码
> 2. /cgroup-v1 的 memory.rst 已out-date，分析并更新
> 3. cgroup-v2.rst 有被翻译成中文的必要性
> 4. 过程中发现的可重构/问题代码 及时 发patch
> 5. Per CPU
> 6. per slub

材料:
> xxx
> 内核源码