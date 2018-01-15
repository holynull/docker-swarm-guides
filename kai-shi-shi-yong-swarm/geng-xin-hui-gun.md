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
    
    