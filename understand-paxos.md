---
title: "理解 Paxos 算法"
date: 2020-02-21T21:52:03+08:00
draft: false 
---

本文参考资料是 Lamport 的两个论文：
    1. Paxos Made Simple: http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf
    2. Time, Clocks, and the Ordering of Events in a Distributed System: http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf

本文主要是翻译加注释 Paxos Made Simple 的 2.2 节。理解了这一节，其他部分就容易多了。参考论文2没有特别详细地读，粗看一遍以后用他的思路来理解 2.2 节事半功倍。

时钟这个论文总结起来就是说，由于狭义相对论的关系，我们无法确定不相关的两个事件的发生顺序。所以要设计一个分布式系统时，如果需要定义时序，必须用因果关系来定义。也就是说，你无法保证两个没有因果关系的消息，谁发送的更早。但无论在任何情况下，接收一个消息，总是发生在发送这个消息之后。Paxos 算法就是利用了这个时序关系，定义了一个验证链，达到一致状态机的目的。

paxos 算法的目的是，设计一个一致的分布式状态机。比如一个系统，初始值是0，此时系统是一致的。给这个系统发送一系列的命令，算法可以保证命令执行完成后，系统依然是一致的。这就要求每一个状态都是确定的，也就是说，这个系统的每一个进程，在每一轮都要选出且只选出一个相同的动作。下面描述的是如何前进一步，也就是说，假如现在是一致的，如何进入到下一个一致状态。那么如果初始是一致的，并且这个步骤可以保证下一个一致，归纳法可证，系统最终是一致的。

第一段：选择确定一个值最简单的方式是只用一个 acceptor, 把他第一个收到的第一个值作为 chosed value。但这个方式不对的原因是，如果 acceptor 挂了，系统就不能运行了。

第二段：所以可以使用多个 acceptor，每个 acceptor 只可以接受一个值，当一个集合的 acceptor 中的大部分接受了某个值时，这个值就可以当做 chosed value。这个设计是正确的。

第三段：在没有消息失败和丢失的情况下，如果只有一个人发出了一个提议，这个提议应该是被被接受的。所以需要第一个要求：

P1. An acceptor must accept the first proposal that it receives.
一个 acceptor 必须接受他收到的第一个提议。

接下来就是对第二段的设计的具体化了。每一个条件都证明了前面的条件，最后得出计算的步骤。

第四段：此时又有一个问题，可能造成空的结果。比如说 5 台机器，机器1提议值是1，机器2提议值是2，机器5挂了，机器1,3接受值1，机器2,4接受值2，都没有达到大多数。结果为空。

第五段：所以 acceptor 要可以接受多次提议，最后选一个值。每个提议使用一个独立的编号，由实现决定。此时，一个提议被选择了的意思是，这个提议的号码和他提议的值被选择了（chosed value）。依然和上面的条件一样，大多数 acceptor 接受的那个值就是 chosed value。

此时，依然可能会造成活锁，比如每次提议都是第四段所说的那个情况。paxos 没有解决这个问题，实际中也不太可能出现。有很多优化的方法可以把活锁出现的概率降低到很小。下面继续细化如何选择一个值。

第六段：我们可以接受多个提议，只要提议的值是相同的就可以了。比如说，提议5，值8，接受。提议6，值8，接受。这是可以的。这就等同于要求：

P2. If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v .
如果一个值为 v 的提议被选择了。那么比他更高提议号的那些被选择的提议的值也必须是 v。

第七段：因为提议是完全有序的，所以 P2 可以保证只选择一个值：一旦一个值被选定，后面被选定提议的值一定和他相同。

第八段：要想被选择，提议必须至少被一个 acceptor 接受。

P2a: P2 a . If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v .
    如果一个值为 v 的提议被选择，那每个被任何 acceptor 接受的更高提议号的提议的值都是 v。

快看，因果时序来了！

任何情况下，一个值必须被接受之后才有可能被选择，同时，一个提议的被接受总是发生在被选择之前。P2 要求被选择的值必须是 v，因为被选择意味着至少被一个 acceptor 接受，我现在要求任何一个被接受的值都是 v。

第九段：P2a 的要求和 P1 的要求不兼容：比如5台机器，机器4和5一直和其他没有交流，突然机器4发了一个提议号更高的提议，同时值和之前的不同，按 P1 的要求，机器5必须接受这个提议。这就不符合 P2a 了。所以需求变成了：
P2b . If a proposal with value v is chosen, then every higher-numbered proposal issued by any proposer has value v .

同样是因果时序，你不是怕接收到一个不同值的更高提议号的提议么，我现在要求发出的时候这个值必须一样。任何情况下，发出总是在接受之前发生。并且，这里用了 every ... any 这个集合要求，以后都要这用了，不然就约束不了了。现在兼容了，继续具体化。

第十段看起来很长很复杂，但有了前面两次的训练我觉得反而简单了。怎么样达到 P2b 的要求呢。和前面的一样，因果时序。
 
现在的要求是：
一旦一个值被选择，更高的提议的值都必须是这个值。

我们考虑，这个新的更高的提议有哪些要求。逻辑上有两种情况，第一是，原来没有值，那我们有两种选择，继续保持没有值；选一个新的。都符合要求。但继续保持没有值最后导致没结果，不行。所以必须选择一个新值。第二是，原来有值，则必须和原来的值一样。原来的值是大多数 acceptor 选出来的。也就是说，我们要选择那个大多数 acceptor 选的那个值。另外，所谓的原来没有值，也必须是由大多数 acceptor 才能确定。但现在没有，不代表以后没有，所以新提议提出时，要给大多数 acceptor 加一个要求，不能再产生旧的值了，现在没有，以后就不能有，因为我要选新的了。

这样以来，一旦一个值被选择，我们可以保证下一个被选择的值和他一样。那根据归纳法，所有被选择的值都是同一个。

这就得出了最新要求：
P2 c . For any v and n, if a proposal with value v and number n is issued, then there is a set S consisting of a majority of acceptors such that either (a) no acceptor in S has accepted any proposal numbered less than n, or (b) v is the value of the highest-numbered proposal among
all proposals numbered less than n accepted by the acceptors in S .
[水平差，翻译不出来了]

所以发出一个提议时，如果此时还没有值，就要给大部分的 acceptor 加一个约束，不能再选旧提议号提出的值。必须用我现在这个。干脆，不让 acceptor 接受旧的提议了。他就没法选了。

这就是 paxos 算法最核心的逻辑推倒了。证明推倒过程中我发现有两个重要的地方，一是之前论文里的因果时序。另外一个是用逻辑的方式思考，比看描述要容易些。总结：单机系统通常只需要关心逻辑关系，分布式系统还有个时序，根据狭义相对论的结论，时序只能用因果来定义。合适的因果关系同时保证了逻辑和时序。还有就是感叹，为什么他在避免问题的时候，依然能满足要求，而不是出现了更多问题。果然是大牛。
