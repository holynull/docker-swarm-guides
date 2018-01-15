# 删除service

教程剩下的部分不需要`helloworld`这个service了，所以我们现在可以将它删除掉。

1. 通过ssh登录到manager节点。

2. 运行`docker service rm helloworld`来删除`helloworld`服务（service）：

    ```
    $ docker service rm helloworld

    helloworld
    ```
    
3. 运行`docker service inspect <SERVICE-ID>`来确认service是否被删除。CLI将会返回service无法找到信息。

    ```
    $ docker service inspect helloworld
    []
    Error: no such service: helloworld
    ```
    
4. 移除service后，task的container会在若干秒后被彻底清楚干净。可以登录到相应的节点上，运行`docker ps`命令来查看task是否被清理。

    ```
    $ docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    db1651f50347        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.5.9lkmos2beppihw95vdwxy1j3w
    43bf6e532a92        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.3.a71i8rp6fua79ad43ycocl4t2
    5a0fb65d8fa7        alpine:latest       "ping docker.com"        44 minutes ago      Up 45 seconds                           helloworld.2.2jpgensh7d935qdc857pxulfr
    afb0ba67076f        alpine:latest       "ping docker.com"        44 minutes ago      Up 46 seconds                           helloworld.4.1c47o7tluz7drve4vkm2m5olx
    688172d3bfaa        alpine:latest       "ping docker.com"        45 minutes ago      Up About a minute                       helloworld.1.74nbhb3fhud8jfrhigd7s29we

    $ docker ps
       CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS               

    ```
