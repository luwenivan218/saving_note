> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/linbingdong/p/6253479.html)

Paxos 算法在分布式领域具有非常重要的地位。但是 Paxos 算法有两个比较明显的缺点：1. 难以理解 2. 工程实现更难。

网上有很多讲解 Paxos 算法的文章，但是质量参差不齐。看了很多关于 Paxos 的资料后发现，学习 Paxos 最好的资料是论文《Paxos Made Simple》，其次是中、英文版维基百科对 Paxos 的介绍。本文试图带大家一步步揭开 Paxos 神秘的面纱。

Paxos 是什么
---------

> Paxos 算法是基于**消息传递**且具有**高度容错特性**的**一致性算法**，是目前公认的解决**分布式一致性**问题**最有效**的算法之一。

Google Chubby 的作者 Mike Burrows 说过这个世界上**只有一种**一致性算法，那就是 Paxos，其它的算法都是**残次品**。

虽然 Mike Burrows 说得有点夸张，但是至少说明了 Paxos 算法的地位。然而，Paxos 算法也因为晦涩难懂而臭名昭著。本文的目的就是带领大家深入浅出理解 Paxos 算法，不仅理解它的执行流程，还要理解算法的推导过程，作者是怎么一步步想到最终的方案的。只有理解了推导过程，才能深刻掌握该算法的精髓。而且理解推导过程对于我们的思维也是非常有帮助的，可能会给我们带来一些解决问题的思路，对我们有所启发。

问题产生的背景
-------

在常见的分布式系统中，总会发生诸如**机器宕机**或**网络异常**（包括消息的延迟、丢失、重复、乱序，还有网络分区）等情况。Paxos 算法需要解决的问题就是如何在一个可能发生上述异常的分布式系统中，快速且正确地在集群内部对**某个数据的值**达成**一致**，并且保证不论发生以上任何异常，都不会破坏整个系统的一致性。

注：这里**某个数据的值**并不只是狭义上的某个数，它可以是一条日志，也可以是一条命令（command）。。。根据应用场景不同，**某个数据的值**有不同的含义。

![](http://upload-images.jianshu.io/upload_images/1752522-d2136179b456e13e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

相关概念
----

在 Paxos 算法中，有三种角色：

*   **Proposer**
*   **Acceptor**
*   **Learners**

在具体的实现中，一个进程可能**同时充当多种角色**。比如一个进程可能**既是 Proposer 又是 Acceptor 又是 Learner**。

还有一个很重要的概念叫**提案（Proposal）**。最终要达成一致的 value 就在提案里。

**注：**

*   **暂且**认为『**提案 = value**』，即提案只包含 value。在我们接下来的推导过程中会发现如果提案只包含 value，会有问题，于是我们再对提案**重新设计**。
*   **暂且**认为『**Proposer 可以直接提出提案**』。在我们接下来的推导过程中会发现如果 Proposer 直接提出提案会有问题，需要增加一个学习提案的过程。

Proposer 可以提出（propose）提案；Acceptor 可以接受（accept）提案；如果某个提案被选定（chosen），那么该提案里的 value 就被选定了。

回到刚刚说的『对某个数据的值达成一致』，指的是 Proposer、Acceptor、Learner 都认为同一个 value 被选定（chosen）。那么，Proposer、Acceptor、Learner 分别在什么情况下才能认为某个 value 被选定呢？

*   Proposer：只要 Proposer 发的提案被 Acceptor 接受（刚开始先认为只需要一个 Acceptor 接受即可，在推导过程中会发现需要半数以上的 Acceptor 同意才行），Proposer 就认为该提案里的 value 被选定了。
*   Acceptor：只要 Acceptor 接受了某个提案，Acceptor 就任务该提案里的 value 被选定了。
*   Learner：Acceptor 告诉 Learner 哪个 value 被选定，Learner 就认为那个 value 被选定。

![](http://upload-images.jianshu.io/upload_images/1752522-6980ffa6b43c16d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题描述
----

假设有一组可以**提出（propose）value**（value 在提案 Proposal 里）的**进程集合**。一个一致性算法需要保证提出的这么多 value 中，**只有一个** value 被选定（chosen）。如果没有 value 被提出，就不应该有 value 被选定。如果一个 value 被选定，那么所有进程都应该能**学习（learn）**到这个被选定的 value。对于一致性算法，**安全性（safaty）**要求如下：

*   只有被提出的 value 才能被选定。
*   只有一个 value 被选定，并且
*   如果某个进程认为某个 value 被选定了，那么这个 value 必须是真的被选定的那个。

我们不去精确地定义其**活性（liveness）**要求。我们的目标是保证**最终有一个提出的 value 被选定**。当一个 value 被选定后，进程最终也能学习到这个 value。

> Paxos 的目标：保证最终有一个 value 会被选定，当 value 被选定后，进程最终也能获取到被选定的 value。

假设不同角色之间可以通过发送消息来进行通信，那么：

*   每个角色以任意的速度执行，可能因出错而停止，也可能会重启。一个 value 被选定后，所有的角色可能失败然后重启，除非那些失败后重启的角色能记录某些信息，否则等他们重启后无法确定被选定的值。
*   消息在传递过程中可能出现任意时长的延迟，可能会重复，也可能丢失。但是消息不会被损坏，即消息内容不会被篡改（拜占庭将军问题）。

推导过程
----

### 最简单的方案——只有一个 Acceptor

假设只有一个 Acceptor（可以有多个 Proposer），只要 Acceptor 接受它收到的第一个提案，则该提案被选定，该提案里的 value 就是被选定的 value。这样就保证只有一个 value 会被选定。

但是，如果这个唯一的 Acceptor 宕机了，那么整个系统就**无法工作**了！

因此，必须要有**多个 Acceptor**！

![](http://upload-images.jianshu.io/upload_images/1752522-a902b09159405eab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 多个 Acceptor

多个 Acceptor 的情况如下图。那么，如何保证在多个 Proposer 和多个 Acceptor 的情况下选定一个 value 呢？

![](http://upload-images.jianshu.io/upload_images/1752522-a85c9965be9d1671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面开始寻找解决方案。

如果我们希望即使只有一个 Proposer 提出了一个 value，该 value 也最终被选定。

那么，就得到下面的约束：

> P1：一个 Acceptor 必须接受它收到的第一个提案。

但是，这又会引出另一个问题：如果每个 Proposer 分别提出不同的 value，发给不同的 Acceptor。根据 P1，Acceptor 分别接受自己收到的 value，就导致不同的 value 被选定。出现了不一致。如下图：

![](http://upload-images.jianshu.io/upload_images/1752522-a2449c74a784bd87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

刚刚是因为『一个提案只要被一个 Acceptor 接受，则该提案的 value 就被选定了』才导致了出现上面不一致的问题。因此，我们需要加一个规定：

> 规定：一个提案被选定需要被**半数以上**的 Acceptor 接受

这个规定又暗示了：『一个 Acceptor 必须能够接受不止一个提案！』不然可能导致最终没有 value 被选定。比如上图的情况。v1、v2、v3 都没有被选定，因为它们都只被一个 Acceptor 的接受。

最开始讲的『**提案 = value**』已经不能满足需求了，于是重新设计提案，给每个提案加上一个提案编号，表示提案被提出的顺序。令『**提案 = 提案编号 + value**』。

虽然允许多个提案被选定，但必须保证所有被选定的提案都具有相同的 value 值。否则又会出现不一致。

于是有了下面的约束：

> P2：如果某个 value 为 v 的提案被选定了，那么每个编号更高的被选定提案的 value 必须也是 v。

一个提案只有被 Acceptor 接受才可能被选定，因此我们可以把 P2 约束改写成对 Acceptor 接受的提案的约束 P2a。

> P2a：如果某个 value 为 v 的提案被选定了，那么每个编号更高的被 Acceptor 接受的提案的 value 必须也是 v。

只要满足了 P2a，就能满足 P2。

但是，考虑如下的情况：假设总的有 5 个 Acceptor。Proposer2 提出 [M1,V1] 的提案，Acceptor25（半数以上）均接受了该提案，于是对于 Acceptor25 和 Proposer2 来讲，它们都认为 V1 被选定。Acceptor1 刚刚从宕机状态恢复过来（之前 Acceptor1 没有收到过任何提案），此时 Proposer1 向 Acceptor1 发送了 [M2,V2] 的提案（V2≠V1 且 M2>M1），对于 Acceptor1 来讲，这是它收到的第一个提案。根据 P1（一个 Acceptor 必须接受它收到的第一个提案。）,Acceptor1 必须接受该提案！同时 Acceptor1 认为 V2 被选定。这就出现了两个问题：

1.  Acceptor1 认为 V2 被选定，Acceptor2~5 和 Proposer2 认为 V1 被选定。出现了不一致。
2.  V1 被选定了，但是编号更高的被 Acceptor1 接受的提案 [M2,V2] 的 value 为 V2，且 V2≠V1。这就跟 P2a（如果某个 value 为 v 的提案被选定了，那么每个编号更高的被 Acceptor 接受的提案的 value 必须也是 v）矛盾了。

![](http://upload-images.jianshu.io/upload_images/1752522-e517a6fd3d55e2c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们要对 P2a 约束进行强化！

P2a 是对 Acceptor 接受的提案约束，但其实提案是 Proposer 提出来的，所有我们可以对 Proposer 提出的提案进行约束。得到 P2b：

> P2b：如果某个 value 为 v 的提案被选定了，那么之后任何 Proposer 提出的编号更高的提案的 value 必须也是 v。

由 P2b 可以推出 P2a 进而推出 P2。

那么，如何确保在某个 value 为 v 的提案被选定后，Proposer 提出的编号更高的提案的 value 都是 v 呢？

只要满足 P2c 即可：

> P2c：对于任意的 N 和 V，如果提案 [N, V] 被提出，那么存在一个半数以上的 Acceptor 组成的集合 S，满足以下两个条件中的任意一个：

*   S 中每个 Acceptor 都没有接受过编号小于 N 的提案。
*   S 中 Acceptor 接受过的最大编号的提案的 value 为 V。

### Proposer 生成提案

为了满足 P2b，这里有个比较重要的思想：Proposer 生成提案之前，应该先去**『学习』**已经被选定或者可能被选定的 value，然后以该 value 作为自己提出的提案的 value。如果没有 value 被选定，Proposer 才可以自己决定 value 的值。这样才能达成一致。这个学习的阶段是通过一个**『Prepare 请求』**实现的。

于是我们得到了如下的**提案生成算法**：

1.  Proposer 选择一个**新的提案编号 N**，然后向**某个 Acceptor 集合**（半数以上）发送请求，要求该集合中的每个 Acceptor 做出如下响应（response）。  
    (a) 向 Proposer 承诺保证**不再接受**任何编号**小于 N 的提案**。  
    (b) 如果 Acceptor 已经接受过提案，那么就向 Proposer 响应**已经接受过**的编号小于 N 的**最大编号的提案**。
    
    我们将该请求称为**编号为 N** 的 **Prepare 请求**。
    
2.  如果 Proposer 收到了**半数以上**的 Acceptor 的**响应**，那么它就可以生成编号为 N，Value 为 V 的**提案 [N,V]**。这里的 V 是所有的响应中**编号最大的提案的 Value**。如果所有的响应中**都没有提案**，那 么此时 V 就可以由 Proposer **自己选择**。  
    生成提案后，Proposer 将该**提案**发送给**半数以上**的 Acceptor 集合，并期望这些 Acceptor 能接受该提案。我们称该请求为 **Accept 请求**。（注意：此时接受 Accept 请求的 Acceptor 集合**不一定**是之前响应 Prepare 请求的 Acceptor 集合）
    

### Acceptor 接受提案

Acceptor **可以忽略任何请求**（包括 Prepare 请求和 Accept 请求）而不用担心破坏算法的**安全性**。因此，我们这里要讨论的是什么时候 Acceptor 可以响应一个请求。

我们对 Acceptor 接受提案给出如下约束：

> P1a：一个 Acceptor 只要尚**未响应过**任何**编号大于 N** 的 **Prepare 请求**，那么他就可以**接受**这个**编号为 N 的提案**。

如果 Acceptor 收到一个编号为 N 的 Prepare 请求，在此之前它已经响应过编号大于 N 的 Prepare 请求。根据 P1a，该 Acceptor 不可能接受编号为 N 的提案。因此，该 Acceptor 可以忽略编号为 N 的 Prepare 请求。当然，也可以回复一个 error，让 Proposer 尽早知道自己的提案不会被接受。

因此，一个 Acceptor **只需记住**：1. 已接受的编号最大的提案 2. 已响应的请求的最大编号。

![](http://upload-images.jianshu.io/upload_images/1752522-09a81e90de7f722b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Paxos 算法描述

经过上面的推导，我们总结下 Paxos 算法的流程。

Paxos 算法分为**两个阶段**。具体如下：

*   **阶段一：**
    
    (a) Proposer 选择一个**提案编号 N**，然后向**半数以上**的 Acceptor 发送编号为 N 的 **Prepare 请求**。
    
    (b) 如果一个 Acceptor 收到一个编号为 N 的 Prepare 请求，且 N **大于**该 Acceptor 已经**响应过的**所有 **Prepare 请求**的编号，那么它就会将它已经**接受过的编号最大的提案（如果有的话）**作为响应反馈给 Proposer，同时该 Acceptor 承诺**不再接受**任何**编号小于 N 的提案**。
    
*   **阶段二：**
    
    (a) 如果 Proposer 收到**半数以上** Acceptor 对其发出的编号为 N 的 Prepare 请求的**响应**，那么它就会发送一个针对 **[N,V] 提案**的 **Accept 请求**给**半数以上**的 Acceptor。注意：V 就是收到的**响应**中**编号最大的提案的 value**，如果响应中**不包含任何提案**，那么 V 就由 Proposer **自己决定**。
    
    (b) 如果 Acceptor 收到一个针对编号为 N 的提案的 Accept 请求，只要该 Acceptor **没有**对编号**大于 N** 的 **Prepare 请求**做出过**响应**，它就**接受该提案**。
    

![](http://upload-images.jianshu.io/upload_images/1752522-44c5a422f917bfc5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Learner 学习被选定的 value
--------------------

Learner 学习（获取）被选定的 value 有如下三种方案：

![](http://upload-images.jianshu.io/upload_images/1752522-0fab48ed2bdf358a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何保证 Paxos 算法的活性
----------------

![](http://upload-images.jianshu.io/upload_images/1752522-28b18dd606777074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过选取**主 Proposer**，就可以保证 Paxos 算法的活性。至此，我们得到一个**既能保证安全性，又能保证活性**的**分布式一致性算法**——**Paxos 算法**。

参考资料
----

*   论文《Paxos Made Simple》
*   论文《The Part-Time Parliament》
*   英文版维基百科的 Paxos
*   中文版维基百科的 Paxos
*   书籍《从 Paxos 到 ZooKeeper》

![](http://upload-images.jianshu.io/upload_images/1752522-2e4b0e5141927479.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

扫一扫关注公众号 FullStackPlan 获取更多干货~