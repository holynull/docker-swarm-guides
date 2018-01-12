# 创建Swarm

在完成安装步骤以后，就可以准备创建Swarm。首先要确定Docker Engine 已经在所有的主机上已经运行起来。

1. 打开终端通过ssh进入到manager节点，即进入本教程称之为manager1的宿主机。如果你使用Docker Machine，你可以通过如下命令进入宿主机manager1：

  ```
  $ docker-machine ssh manager1

  ```
2. 执行如下命令创建一个新的Swarm：

  ```
  docker swarm init --advertise-addr <MANAGER-IP>
  ```
  
  > **注意：**如果你使用Docker for Mac或者Docker for Windows实验单节点Swarm，那么命令后面就不要加任何参数。这种情况下不需要指定`--advertise-addr`。

  在本教程重，执行如下命令在manager1主机上创建一个swarm：
  
  ```
  $ docker swarm init --advertise-addr 192.168.99.100
  Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

  To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

  To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
  ```
  
  参数`--advertise-addr`将功能对外公布manager节点的IP地址为`192.168.99.100`。其他节点必须能够通过这个IP地址访问到该节点。

  我们可以看见输出的结果包含了一条在Swarm中添加新节点的命令。节点将依赖参数`--token`，作为manager或者worker加入Swarm。

3. 运行`docker info`命令查看Swarm的当前状态。
  
  ```
  $ docker info

  Containers: 2
  Running: 0
  Paused: 0
  Stopped: 2
    ...snip...
  Swarm: active
    NodeID: dxn1zf6l61qsb1josjja83ngz
    Is Manager: true
    Managers: 1
    Nodes: 1
    ...snip...
  ```
  
4. 运行`docker node ls`命令查看节点的信息。

  ```
  $ docker node ls

  ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER   STATUS
dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
  ```

`*`表示当前链接的节点，即我们执行操作命令的节点。

Docker Engine在swarm模式下，使用宿主机的host name来命名节点。