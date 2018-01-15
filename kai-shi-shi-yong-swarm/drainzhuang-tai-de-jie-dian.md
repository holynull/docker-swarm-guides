# DRAIN状态的节点

之前讲述的内容，都是在所有的节点都是可用的，状态都是`ACTIVE`。因此，manager可以将任务分配到所有节点上，所以到目前位置所有的节点都可以接受manager分配的任务。

有的时候，例如计划系统维护时，我们需要将节点的状态变成`DRAIN`。`DRAIN`状态将阻止节点接收manager分配的任务。同时也意味着，manager将停止该节点上的task，并将task转移到其他`ACTIVE`状态的节点上运行。

1. 通过ssh登录到manager节点。

2. 查看我们有哪些`ACTIVE`状态的节点：

    ```
    $ docker node ls

    ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
    1bcef6utixb0l0ca7gxuivsj0    worker2   Ready   Active
    38ciaotwjuritcdtn9npbnkuz    worker1   Ready   Active
    e216jshn25ckzbvmwlnh5jr3g *  manager1  Ready   Active        Leader
    ```