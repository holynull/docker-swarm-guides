# 安装

请注意本教程有如下要求：

- 3台在同一个网络中的预装Linux操作系统的宿主机，并且宿主机上安装Docker。
- Docker Engine的版本1.12+
- 需要清楚作为Manager的主机的IP地址
- 主机之间需要能够互相开放端口

## 3个联网的宿主机

本教程要求具备3台联网的Linux操作系统的宿主机，并且安装Docker。这些宿主机可以是物理服务器，也可以是虚拟机或者各种云服务器。读者也可使用Docker Machine来简历主机。

3台宿主机中的一台将作为Manager，我们这里称作manager1。其他两台将作为Worker，分别称作worker1和worker2。

> **注意：** 读者同样可以通过教程的步骤实验创建一个单节点的swarm，这种情况下你只需要一台宿主机。但是多节点命令将不能运行，但是你可以初始化一个swarm、创建service和设定service的scale。

## Docker Engine 1.12 or newer

本教程要求要求每台宿主机上安装Docker Engine 1.12 +。安装完成以后确定Docker Engine Daemon成功启动运行。

## Manager宿主机的IP地址

宿主机必须被分配一个固定的IP地址。Swarm中的节点必须能够通过IP地址访问到Manager节点。

如果使用Docker Machine建立的宿主机，我们可以通过`docker-machine ls`或者`docker-machine -ip <MACHINE-NAME>`来获得IP地址。

本教程中的manager1的IP地址是： 192.168.99.100.

## Open protocols and ports between the hosts

宿主机需要开发如下端口，某些操作系统中这些端口是默认开放的：

- TCP  2377 - 集群管理通信端口
- TCP and UDP 7946 - 节点之间的通信端口
- UDP 4789 - Overlay Network数据传输端口

如果你计划创建加密的Overlay Network，还需要设置IP协议支持50 (ESP) traffic。


