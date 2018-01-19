# Swarm中的manager节点

Swarm中的manager节点使用[Raft Consensus](https://docs.docker.com/engine/swarm/raft/)算法来管理Swarm的状态。为了管理Swarm，我们需要了解一些Raft的概念。

对于manager节点个数其实是没有限制的。manager节点数量需要从性能和容错性之间来权衡利弊。增加manager节点的数量可以更好的提高容错性。然而，大量的manager节点会降低数据写的性能，因为在Swarm状态更新时，更多的manager节点需要赞同状态更新（参考Raft Consensus算法，或者选举算法 ）。这意味着节点间需要大量的来回反复的网络通信。

Raft算法要求大部分的manager节点（这个数量在算法中要求超过manager节点个数半数以上）同意Swarm的状态更新，才能使状态更新成功，例如节点的增加和移除。节点之间的关系操作也同样受到这样的约束。

## Manager节点的法定人数（quorum）

如果Swarm中的mananger不能在选举算法中产生投票结果，Swarm就不能执行管理任务。如果我们有多个manager节点，那么节点数必须为2个以上。要保证正常运行，就必须保证法定人数（quorum）以及法定人数以上个manager节点可用。而且manager的个数建议采用奇数个，因为n（n为奇数）个manager节点和（n+1）个manager节点在算法上产生的效果是一样的。3个manager节点和4个mananger节点，都只允许最多1个manager节点不可用，5个manager节点和6个manager节点都只允许最多2个节点不可用。

一旦Swarm中的manager节点不能通过选举算法得到结果，worker节点上的Task仍然继续运行。但是，Swarm的节点不能被添加、更新和移除，并且新的task不能启动，已存在的task不能被停止、移动和更新。

## Manager节点配置静态IP地址

当初始化Swarm是，我们指定参数`--advertise-addr`来广播当前节点的IP地址给其他Manager节点。由于manager节点作为一个稳定的基础组件，我们应该给它分配一个固定的静态IP地址，以防止服务器重启造成IP地址改变。

如果Swarm重启，并且所有的manager节点都因为重启而获得一个新的IP地址，其他节点就无法链接到manager节点。Swarm就会出现故障被挂起。

Worker节点可以使用动态IP地址。



