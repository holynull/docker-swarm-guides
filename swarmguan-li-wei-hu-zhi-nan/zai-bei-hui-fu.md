# 灾备与恢复

## 备份

Docker的manager节点将Swarm的状态和日志存储在`/var/lib/docker/swarm/`目录下。在1.13或者更高版本中，这个目录下包含加密Raft日志的密钥。如果没有这些密钥，就没有办法恢复Swarm。

我们可以在任何一个manager节点上，按照下面的过程进行备份。

1. 如果Swarm`auto-lock`设置被打开，我们需要`unlock key`才能进行从备份恢复。如何多的`unlock key`请参看[Lock your swarm to protect its encryption key](https://docs.docker.com/engine/swarm/swarm_manager_locking/)。

2. 在备份之前在manager节点停止Docker，这样依赖在备份期间就不会有数据发生改变。不建议采用Manager在备份过程中一直运行的热备份，因为在恢复时结果是不可预知的。manager节点停止的同时产生的数据不会被备份。
    
    > **注意：**要确保manager节点的选举算法维持不变。在一个manager节点关闭时，如果这时有节点丢失，Swarm变得容易无法完成选举算法。所以需要权衡一下留下运行的manager节点的个数。如果经常有规律的进行备份，可以考虑创建一个5个manager节点的Swarm，这样的话就可以在备份时允许再失去一个manager节点，而不影响service的运行。
    
3. 备份目录`/var/lib/docker/swarm`

4. 重启manager节点

## 从备份恢复

备份了Swarm之后，通过下面的过程将备份的数据恢复到一个新的集群上：

1. 在要恢复Swarm的主机上关闭Docker。

2. 移除目录`/var/lib/docker/swarm`。

3. 将之前备份的内容恢复到目录`/var/lib/docker/swarm`。

    > **注意：新的节点将跟以前节点一样的磁盘加密密钥。此时不能修改这个磁盘加密密钥。**
    在Swarm`auto-lock`打开的情况下，`unlock key`与之前的备份的`unlock-key`是一样的，也会要进行恢复。
    
4. 在恢复Swarm的节点上开启Docker。解锁Swarm如果需要的话。使用下面的命令重新初始化Swarm，这样的话这个节点将不会再去尝试链接之前的Swarm节点。

    ```
    $ docker swarm init --force-new-cluster
    ```
    
5. 检查确认Swarm的状态。也许这一步还要包含应用测试，或者通过玲玲`docker service ls`来查看确定service的状态。

6. 如果使用`auto-lock`，则需要轮换`unlock key`。详情请参看[rotate unlock key](https://docs.docker.com/engine/swarm/swarm_manager_locking/#rotate-the-unlock-key)。

7. 添加manager节点和worker节点，使集群容量达到跟之前一下样。

8. 在新的集群上恢复之前的应用备份。

## 选举计算恢复

Swarm具有容错性，并且能够从临时的节点不可用（机器重启或者崩溃重启）中，或者其他瞬时的错误中恢复。然而，选举算法出问题时，Swarm是不能自动恢复的。worker节点上的task还在继续执行，但是负责管理的任务将不能运行，包括扩展、或者更新service，添加或者移除节点。最好解决方法是将不可用的manager节点恢复回来。如果无法恢复manager节点怎么办？

在一个N个manager节点的Swarm中，必须保证法定人数以上的节点可用。例如，5个manager，最少要3个manager可用才并能够互相通信。如果可用的manager的数量低于法定人数，就不能对Swarm进行任何管理操作。当试图进行管理操作时，会出现如下错误提示：

```
Error response from daemon: rpc error: code = 4 desc = context deadline exceeded
```

最佳的回复方法当然是将不可用的manager节点恢复回来。如果发布恢复manager节点，我们只能通过使用参数`--force-new-cluster`来恢复manager节点。这个操作将降级所有的manager节点，除了执行命令的节点。然后再手动将worker节点升级回manager节点，保持与原来一样个数的manager节点为止。

```
# From the node to recover
docker swarm init --force-new-cluster --advertise-addr node01:2377

```

