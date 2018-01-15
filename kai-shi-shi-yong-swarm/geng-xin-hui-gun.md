# 更新回滚

本章中，我们首先使用Redis 3.0.6镜像部署一个服务，然后使用Redis 3.0.7镜像对service进行升级。最后，再将service的升级回滚到Redis 3.0.6。

1. 通过ssh登录到manager节点。

2. 使用Redis 3.0.6部署一个服务。

    ```
    $ docker service create \
      --replicas 3 \
      --name redis \
      --update-delay 10s \
      redis:3.0.6

    0u6a4s31ybk7yw2wyvtikmu50
    ```
    在部署时，我们需要对升级操作的策略进行设置。`--update-delay`指定每一个task升级的延时时间。例如，我们要求每个task较之前一个task开始升级的时间延时T秒，则把参数设置成Ts，s表示单位秒，m表示单位分钟，h表示单位小时。所以10m30s，表示将延时10分30秒。
    默认情况下一次更新1个task，可以通过参数`--update-parallelism`来设置同时开始更新的最大task个数。
    默认情况下当一个执行更新任务的task的状态成为`RUNNING`时，才会继续开始下一个task的更新，知道所有的task都成为`RUNNING`状态为止。一旦更新操作过程中，有一个task返回`FAILED`状态，则更新会立即停止。我们可以在`docker service create`或者`docker service update`命令中使用`--update-failure-action`参数，来控制更新失败后的操作。
    
3. 检查`redis`服务

    ```
    $ docker service inspect --pretty redis

    ID:             0u6a4s31ybk7yw2wyvtikmu50
    Name:           redis
    Service Mode:   Replicated
    Replicas:      3
    Placement:
     Strategy:	    Spread
    UpdateConfig:
     Parallelism:   1
     Delay:         10s
    ContainerSpec:
     Image:         redis:3.0.6
    Resources:
    Endpoint Mode:  vip
    ```
    
4. 下面开始更新服务。manager根据`UpdateConfig`对节点上的task进行更新。

    ```
    $ docker service update --image redis:3.0.7   redis
    redis
    ```
    