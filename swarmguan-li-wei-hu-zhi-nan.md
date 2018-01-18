> **编者提示：**法定人数可以理解为超过总人数的半数以上的一个整数。例如，3个manager的法定人数就是2，其中有一个缺席，同样有可能有选举结果，只要剩下的2个manager投同样的票，投票数就会超过3的半数。4个manager中，法定人数是3，而不是2。与3个manager情况相同，当缺席1个人时还可能有选举结果。但是缺席2个人时，是没有办法得到一个选举结果的，因为选票如何也超不过4的半数。所以，3个manager和4个mananger的情况都只能允许1个manager缺席选举或投弃权票，两种情况在算法要达到的结果是一样的。

# Swarm管理维护指南

当我们运行一个Docker Engine集群时，manage节点是管理Swarm和存储Swarm状态的关键组件。所以为了更好的管理维护，了解manager节点的一些关键特性是很重要的。

## Swarm中的manager节点

Swarm中的manager节点使用[Raft Consensus](https://docs.docker.com/engine/swarm/raft/)算法来管理Swarm的状态。为了管理Swarm，我们需要了解一些Raft的概念。

对于manager节点个数其实是没有限制的。manager节点数量需要从性能和容错性之间来权衡利弊。增加manager节点的数量可以更好的提高容错性。然而，大量的manager节点会降低数据写的性能，因为在Swarm状态更新时，更多的manager节点需要赞同状态更新（参考Raft Consensus算法，或者选举算法 ）。这意味着节点间需要大量的来回反复的网络通信。

Raft算法要求大部分的manager节点（这个数量在算法中称之为法定人数，quorum）同意Swarm的状态更新，才能使状态更新成功，例如节点的正佳和移除。节点之间的关系操作也同样受到这样的约束。

### Manager节点的法定人数（quorum）

如果Swarm中的manager节点的个数，不能满足确定一个法定人数（quorum），那么Swarm将不能执行任何管理任务。