# 灾备恢复

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
