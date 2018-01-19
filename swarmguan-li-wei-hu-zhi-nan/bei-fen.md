# 备份

Docker的manager节点将Swarm的状态和日志存储在`/var/lib/docker/swarm/`目录下。在1.13或者更高版本中，这个目录下包含加密Raft日志的密钥。如果没有这些密钥，就没有办法恢复Swarm。

我们可以在任何一个manager节点上，按照下面的过程进行备份。

1. 如果Swarm`auto-lock`设置被打开，我们需要`unlock key`才能进行从备份恢复。如何多的`unlock key`请参看[Lock your swarm to protect its encryption key](https://docs.docker.com/engine/swarm/swarm_manager_locking/)。

2. 在备份之前在manager节点停止Docker，这样依赖在备份期间就不会有数据发生改变。不建议采用Manager在备份过程中一直运行的热备份，因为在恢复时结果是不可预知的。manager节点停止的同时产生的数据不会被备份。
    
    > **注意：**要确保manager节点的选举算法维持不变。在一个manager节点关闭时，如果这时有节点丢失，Swarm变得容易无法完成选举算法。所以需要权衡一下留下运行的manager节点的个数。如果经常有规律的进行备份，可以考虑创建一个5个manager节点的Swarm，这样的话就可以在备份时允许再失去一个manager节点，而不影响service的运行。
    
3. 备份目录`/var/lib/docker/swarm`

4. 重启manager节点





