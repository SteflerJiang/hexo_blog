---
title: Paxos算法
date: 2017-03-23 19:27:16
categories: 算法
tags: [Paxos]
---

Paxos算法是莱斯利·兰伯特(Leslie Lamport)于1990年提出的一种基于消息传递且具有高度容错性的一致性算法，是目前公认的解决分布式一致性问题最有效的算法之一。

两篇经典论文
1. [The Part-Time Parliament](http://lamport.azurewebsites.net/pubs/lamport-paxos.pdf)
2. [Paxos Made Simple](http://lamport.azurewebsites.net/pubs/paxos-simple.pdf)

我主要针对《从Paxos到Zookeeper分布式一致性原理与实现
》书里第二章的内容和《Paxos Made Simple》论文的原文，对Paxos算法进行详细的阐述。

<!-- more -->

# The Problem
假设有一组可以提出提案(原文中用value来表示)的进程集合，一致性算法可以保证以下几点：
- 只有唯一的一个提案会被选中
- 如果没有任何提案，那么就什么都不会被选中
- 当一个提案被选中之后，进程应该可以获取被选定的提案信息

对于一致性来说，安全性(safety)需求如下：
- 只有被提出的提案才能被选定
- 只有一个提案被选出来
- 如果某个进程认为某个提案被选定了，那么这个提案必须是真的被选定的那个

在对Paxos算法的讲解过程中，我们不去精确地定义其活性(liveness)需求(注：指的是，能有一个提案被选出来，不然大家都不提出提案，也就没有最终结果了)，从整体上来说，Paxos算法的目标就是要保证最终有一个提案会被选定，当提案被选定之后，进程最终也能获取到被选定的提案。

在该一致性算法中，有三种参与角色，我们用Proposer、Acceptor和Learner来表示，在具体的实现中，一个进程可以充当多个角色，在这里我们并不关心进程如何映射到各种角色。假设不同参与者之间可以通过收发消息来进行通信，那么：
- 每个参与者以任意的速度执行，可能会因为出错而停止，也可能会重启。同事，即使一个提案被选定后，所有的参与者也都有可能失败或重启，因为除非那些失败或者重启的参与者可以记录某些信息，否则将无法确定最终的值。
- 消息在传输过程中可能会出现不可预知的延迟，也可能会重复或丢失，但是消息不会被损坏，即消息内容不会被篡改。

说了这么多，我来解释一下这里提案的意思，或者用通俗的话来讲一下。就是有多个人，大家都会提出一个数字，比如A提出1，B提出2，C提出3，不管怎么样，我们要做的是，在一段时间以后，所有人，必须提出相同的数字。这个就是分布式一致性。

# 提案的选定
要选定一个唯一的提案的最简单的方式莫过于只允许一个Accpetor存在，这样的话，Proposer只能发生提案给该Accpetor，Accpetor会选择它接收到的第一个提案作为被选定的提案。这种解决方式尽管实现起来非常简单，但是却很难让人满意。因为，一旦这个Acceptor出现问题，那么整个系统就无法工作了。

那么，我们应该尝试另外一种方法。用多个Acceptor。一个Proposer把它的提案发给多个Acceptor。每个Acceptor也可以批准(accept)该提案。当有一个足够大的Acceptor集合都批准了这个提案，我们就认为这个提案被选定了。那么，足够大是多大呢。为了保证只有一个提案被选定，我们可以选定一个大的集合包含'任意?'Acceptor集合中的大多数成员。(这句翻译的不好，原文是: To ensure that only a single value is chosen, we can let a large enough set consist of any majority of the agents.)因为任何两个大多数集(majority)的交集至少有一个公共成员，如果每个Acceptor最多只批准一个提案的话，那么这个是可行的。

在不考虑失败和消息丢失的情况下，如果我们希望即使在只有一个提案被提出的情况下，仍然可以选出一个提案，这就暗示了如下的需求：

>P1. 一个Acceptor必须批准它收到的第一个提案。

这个需求就会导致另外一个问题。如果有多个提案同时被不同的Proposer选中，这可能会导致虽然每个Acceptor都批准了它收到的第一个提案，但是没有一个提案是由多数人都批准的。(意思的4个Proposer，4个Acceptor，每个Acceptor都批准对应的Proposer提案)。

还有一种情况是，即使只有2个提案被提出，如果每个提案都被差不多一半的Acceptor批准了，此时即使一个Acceptor出错，都有可能导致无法确定该选哪个提案。

因此，在P1的基础上，再加上一个提案被选定需要由半数以上的Acceptor批准的需求，暗示着一个Acceptor必须能够批准不止一个提案。在这里，我们使用一个全局的编号来唯一标识每一个被Acceptor批准的提案，当一个具有某Value值的提案被半数以上的Acceptor批准后，我们就认为该Value被选定了，此时我们也认为该提案被选定了。需要注意的是，此处讲到的提案已经和Value不是同一个概念了，提案变成了一个由编号和Value组成的组合体，因此我们以"[编号, Value]"来表示一个提案。

根据上面交到的内容，我们虽然允许提出多个提案，但我们必须保证多有被选定的提案都具有相同的Value值。结合提案的编号，该约定可以定义如下：

>P2. 如果编号为M<sub>0</sub>、Value为V<sub>0</sub>的提案被选定了，那么所有比编号M<sub>0</sub>更高的，且被选定的提案，其Value值必须也是V<sub>0</sub>。

因为提案的编号是顺序的，条件P2就保证了只有一个Value值被选定这一个关键安全性属性。同时，一个提案要被选定，其首先必须被至少一个Acceptor批准，因此我们可以通过满足如下条件来满足P2。

>P2a. 如果编号为M<sub>0</sub>、Value为V<sub>0</sub>的提案被选定了，那么所有比编号M<sub>0</sub>更高的，且被选Acceptor批准的提案，其Value值必须也是V<sub>0</sub>。

至此，我们仍然需要P1来保证提案会被选定，但是因为通信是异步的，一个提案可能会在某个Acceptor还未收到任何提案时就被选定了。假设一个新的Proposer醒来了，然后提出了一个编号更高的提案，并且Value和被选定的不同，然后发给一个没有收到过任何提案的Acceptor，按道理Acceptor应该批准它，但是这与P2a矛盾，因此如果要同时满足P1和P2a，需要对P2a进行如下强化：

>P2b. 如果编号为M<sub>0</sub>、Value为V<sub>0</sub>的提案被选定了，那么只会任何Proposer产生的编号更高的提案，其Value值都是V<sub>0</sub>。

因为一个提案必须在被Proposer提出后才能被Acceptor批准，因此P2b包含了P2a，进而包含P2。于是，接下去的重点就是论证P2b成立即可：

>假设某个提案[M<sub>0</sub>, V<sub>0</sub>]已经被选定，证明任何编号M<sub>n</sub> > M<sub>0</sub>的提案，其Value值都是V<sub>0</sub>。

# 数学归纳法证明
我们可以对通过M<sub>n</sub>进行第二数学归纳法来进行证明，也就是说需要证明以下结论:

>假设编号在M<sub>0</sub>到M<sub>n-1</sub>之间的提案，其Value值都是V<sub>0</sub>，证明编号为M<sub>n</sub>的提案的Value值也为V<sub>0</sub>。

因为编号为M<sub>0</sub>的提案已经被选定了，这就意味着肯定存在一个由半数以上的Acceptor组成的集合C，C中的每个Acceptor都批准了该提案。再结合归纳假设，"编号为M<sub>0</sub>的提案被选定"意味着：

>C中的每个Acceptor都批准了一个编号在M<sub>0</sub>到M<sub>n-1</sub>范围内的提案，并且每个编号在M<sub>0</sub>到M<sub>n-1</sub>范围内被Acceptor批准的提案，其Value值都为V<sub>0</sub>。

因为任何包含半数以上Acceptor的集合S都至少包含C中的一个成员，因此我们可以认为如果保持了下面P2c的不变性，那么编号为M<sub>n</sub>的提案的Value也为V<sub>0</sub>。

>P2c. 对于任意的M<sub>n</sub>和V<sub>n</sub>，如果提案[M<sub>n</sub>, V<sub>n</sub>]被提出，那么肯定存在一个由半数以上的Acceptor组成的集合S，满足以下两个条件中的任意一个。
- S中不存在任何批准过编号小于M<sub>n</sub>的提案的Acceptor。
- 选取S中所有Acceptor批准的编号小于M<sub>n</sub>的提案，其中编号最大的那个提案其Value值是V<sub>n</sub>。

至此，只要保证P2c，就能保证P2b。

从上面的内容中，我们可以看到，从P1到P2c的过程其实是对一系列条件的逐步加强，如果需要证明这些条件可以保证一致性，那么就需要进行反向推导：P2c => P2b => P2a => P2，然后通过P2和P1来保证一致性。

我们再来看P2c，实际上P2c规定了每个Proposer如何产生一个提案：对于产生的每个提案[M<sub>n</sub>, V<sub>n</sub>]，需要满足如下条件：

>存在一个由超过半数的Acceptor组成的集合S：
- 要么S中没有Acceptor批准过编号小于M<sub>n</sub>的任何提案。
- 要么S中的所有Acceptor批准的所有编号小于M<sub>n</sub>的提案中，编号最大的那个提案的Value值为V<sub>n</sub>。

当每个Proposer都按照这个规则来产生提案时，就可以保证满足P2b了，接下来我们就使用第二数学归纳法来证明P2c。

首先假设提案[M<sub>0</sub>, V<sub>0</sub>]已经被选定，设比该提案编号大的提案为[M<sub>n</sub>, V<sub>n</sub>]，我们需要证明的就是在P2c的前提下，对于所有的[M<sub>n</sub>, V<sub>n</sub>]，存在V<sub>n</sub> = V<sub>0</sub>。

1. 当M<sub>n</sub> = M<sub>0</sub> + 1时，如果有这样一个编号为M<sub>n</sub>的提案，首先我们知道[M<sub>0</sub>, V<sub>0</sub>]已经被选定了，那么就一定存在一个Acceptor的子集S，且S中的Acceptor已经批准了小于M<sub>n</sub>的提案，于是V<sub>n</sub>只能是多数集S中编号小于M<sub>n</sub>但为编号最大的那个提案的值。而此时因为M<sub>n</sub> = M<sub>0</sub> + 1，因此理论上编号小于M<sub>n</sub>但为最大编号的那个提案肯定是[M<sub>0</sub>, V<sub>0</sub>]，同时由于S和通过[M<sub>0</sub>, V<sub>0</sub>]的Acceptor集合都是多数集，也就是说二者肯定有交集--这样Proposer在确定V<sub>n</sub>取值的时候，就一定会选择V<sub>0</sub>。
2. 根据假设，编号在M<sub>0</sub> + 1 到M<sub>n</sub> - 1区间内所有的提案的Value值为V<sub>0</sub>，需要证明的是编号为M<sub>n</sub>的提案的Value值也为V<sub>0</sub>。根据P2c，首先同样一定存在一个Acceptor的自己S，且S中的Acceptor已经批准了小于M<sub>n</sub>的提案，那么编号为M<sub>n</sub>的提案的Value值只能是这个多数集S中编号小于M<sub>n</sub>但为最大编号的那个提案的值。如果这个最大编号落在M<sub>0</sub> + 1到M<sub>n</sub> - 1区间内，那么Value值肯定是V<sub>0</sub>。如果没有落在这个区间，那么它的编号不可能比M<sub>0</sub>再小了，肯定就是M<sub>0</sub>。因为S也肯定会与批准[M<sub>0</sub>, V<sub>0</sub>]这个提案的Acceptor集合S'有交集，而如果编号是M<sub>0</sub>，那么它的Value值也是V<sub>0</sub>，由此得证。

补充一下数学归纳法的知识，证明分为以下两步：
1. 证明当n = 1时命题成立。
2. 证明如果在n = m时命题成立，那么可以推导出在n = m+1时命题也成立。（m代表任意自然数）

# Proposer生成提案
现在我们来看看，在P2c的基础上如何进行提案的生成。对于一个Proposer来说，获取那些已经被通过的提案远比预测未来可能会被通过的提案来的简单。因此，Proposer在产生一个编号为M<sub>n</sub>的提案时，必须要知道当前某一个将要或已经被半数以上Acceptor批准的编号小于M<sub>n</sub>但为最大编号的提案。并且，Proposer会要求所有的Acceptor都不要再批准任何编号小于M<sub>n</sub>的提案了--这就引出了如下的提案生成算法。

1. Proposer选择一个新的提案编号M<sub>n</sub>，然后向某个Acceptor集合的成员发送请求，要求该集合的Acceptor做出如下回应。
    - 向Proposer承诺，保证不再批准任何编号小于M<sub>n</sub>的提案。
    - 如果Acceptor已经批准过任何提案，那么其就向Proposer反馈当前该Acceptor已经
    批准过的编号小于M<sub>n</sub>但为最大编号的那个提案的值。

    我们将该请求成为编号为M<sub>n</sub>的提案的**Prepare请求**。

2. 如果Proposer收到了来自半数以上的Acceptor的响应结果，那么它就可以产生编号为M<sub>n</sub>、Value值为V<sub>n</sub>的提案，这里的V<sub>n</sub>是所有相应中编号最大的提案的Value值。当然还存在另一种情况，就是半数以上的Acceptor都没有批准过任何提案，即相应中不包含任何的提案，那么此时V<sub>n</sub>值就可以由Proposer任意选择。

在确定提案之后，Proposer就会将该提案再次发送给某个Acceptor集合，并期望获得它们的批准，我们称此请求为**Accept请求**。需要注意的一点是，此时接收Accept请求的Acceptor集合不一定是之前响应Prepare请求的Acceptor集合。

# Acceptor批准提案
在上文中，我们已经讲解了Paxos算法中Proposer的处理逻辑，下面我们来看看Acceptor是如何批准提案的。

根据上面的内容，一个Acceptor可能会收到来自Proposer的两种请求，分别是Prepare请求和Accept请求，对这两类请求做出响应的条件分别如下。

- Prepare请求：Acceptor可以在任何时候响应一个Prepare请求。
- Accept请求：在不违背Accept现有承诺的前提下，可以任意相应Accept请求。

因此，对Acceptor逻辑处理的约束条件，大体可以定义如下。

>P1a：一个Acceptor只要尚未响应过任何编号大于M<sub>n</sub>的Prepare请求，那么它一定就可以接收这个编号为M<sub>n</sub>的提案。

从上面这个约束条件中，我们可以看出，P1包含了P1。同时，值得一提的是，Paxos算法允许Acceptor忽略任何请求而不用担心破坏其算法的安全性。
