# Manager节点的故障排查

不能从其他节点上将`raft`目录拷贝到mananger节点上，然后重启节点。对于每一个节点ID数据目录是唯一对应的。加入Swarm时每一个节点ID只能被节点使用一次。节点ID控件是全局唯一的。

重新将manager节点加入到集群中，需要如下操作：

1. 将节点降级为worker节点。`docker node demote <NODE>`

2. 将节点移除。`docker node rm <NODE>`

3. 将节点重新加入回来，并使用新的状态。`docker swarm join`

