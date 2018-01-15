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